---
title: "Hunting for buggy authentication/authorization services on github"
date: 2021-11-28T10:58:59+01:00
draft: false
tags: [kubernetes, ingress, nginx, path-traversal]
description: "To successful bypass access control using path traversal in $request_uri, you need to have buggy authentication/authorization service. Buggy in a way it’s not normalizing url/uri that is part of access control decision. Let me find more of those on github that are relying on X-Original-Url."
---

{{< figure src="https://user-images.githubusercontent.com/17719543/139592951-0fafc921-437e-4bb7-b0ee-199dd72b36c3.png" class="image-center" >}}

To successful bypass access control using path traversal in `$request_uri`, you need to have buggy authentication/authorization service. Buggy in a way it's not normalizing url/uri that is part of access control decision. Let me find more of those on github that are relying on `X-Original-Url`. There is high chance that this header is populated from `$request_uri` variable and not protected in any way.

## pomerium

>Pomerium is an identity-aware proxy that enables secure access to internal applications.

In research I was using pomerium in version **0.15.5**.

Let's look into code. Here is how `X-Original-Url` header is used:

{{< figure src="https://user-images.githubusercontent.com/17719543/139598398-f7236d2f-a6a6-4cc3-8d4e-b384a4f702d7.png" >}}

Later this `originalURL` is taking part in decision based on polices. Policy definition can include path, which is key information here:

{{< figure src="https://user-images.githubusercontent.com/17719543/139598428-2a91dfde-897e-47a1-a5ea-b6b73b7f0f4f.png" >}}

As you can see there are two possibilities interesting for exploitation: `Prefix` and `Regex`.

In official docs you can find how to specify such policy:

{{< figure src="https://user-images.githubusercontent.com/17719543/139598445-2bf29761-e9a4-4738-a5e4-927e716baf60.png" >}}

For now I'm not going to exploit it further as it quite complicated to setup environment for it. I cannot be 100% sure for successful exploitation, but I have strong indicators in code that it will occur.

## authelia

>Authelia is an open-source authentication and authorization server providing two-factor authentication and single sign-on (SSO) for your applications via a web portal. It acts as a companion for reverse proxies like nginx, Traefik or HAProxy to let them know whether requests should either be allowed or redirected to Authelia's portal for authentication.

In research I was using authelia in version **4.32.2**.

The official description of the authelia perfectly matching my exploitation scenario. I just need to have policy based on path and no defense on `X-Original-Url` header.

Let's check code first:

{{< figure src="https://user-images.githubusercontent.com/17719543/139598518-85f428db-bafc-4bcb-ae49-9b108f2cabd6.png" >}}

`X-Original-Url` header is take from request without much of validation and placed later as `targetURL`:

{{< figure src="https://user-images.githubusercontent.com/17719543/139598541-334a5da0-1dc5-4aea-8eb2-f5459cf23db5.png" >}}

After that `targetURL` is part of logic to make decision whether request is passed as is or required to be authenticated:

{{< figure src="https://user-images.githubusercontent.com/17719543/139598551-7038031f-6d80-450b-adb8-94439b31b35d.png" >}}

In official documentation there is example of rule using in resource regex:

{{< figure src="https://user-images.githubusercontent.com/17719543/139598563-4f497082-bd4d-4079-bb18-643631c40178.png" >}}

For me, this case is very similar to pomerium. I will also not go deeper for now in exploitation. It's visible for me, that authelia has strong indicator for successful exploitation, but one more time I cannot be 100% sure.

## travisghansen/external-auth-server

[travisghansen/external-auth-server](https://github.com/travisghansen/external-auth-server) has primary function to help Kubernetes users to deal with different authentication schemas. In [documentation](https://github.com/travisghansen/external-auth-server/blob/master/PLUGINS.md#request_js) I could find ideal case for bypass. It's using `request_js` plugin to make decision based on `X-Original-Url` header:

{{< figure src="https://user-images.githubusercontent.com/17719543/139594490-48b32b0d-0631-49a7-b2c6-0de034871102.png" >}}

To be sure, whether any normalization is in place, I have checked code that is making this `parentReqInfo` object:

It's in [utils.js](https://github.com/travisghansen/external-auth-server/blob/10ad9710390f38803de92f67e611a568e8d2c79f/src/utils.js#L119) in function `get_parent_request_info`:

{{< figure src="https://user-images.githubusercontent.com/17719543/139594519-9c3caaa2-e1d9-4e8a-b9cf-c2f59885ba4e.png" >}}

I had some problems to run travisghansen/external-auth-server in Kubernetes. Mostly because it's quite complicated. So to really verify if it's vulnerable, I have copied part of utils.js and tested only it:

```javascript
'use strict';

const express = require('express');
const utils = require('./utils')

// Constants
const PORT = 8080;
const HOST = '0.0.0.0';

// App
const app = express();
app.get('/verify', (req, res) => {
    console.log(req.headers);

    const parentReqInfo = utils.get_parent_request_info(req);

    console.log(parentReqInfo);

    if (parentReqInfo.parsedUri.path.startsWith('/public-service/')) {
        res.statusCode = 200;
        res.send();
    }

    const apiKey = req.headers['X-Api-Key'];
    if (apiKey == "secret-api-key") {
        res.statusCode = 200;
        res.send();
    }

    res.statusCode = 401;
    res.send();
});

app.listen(PORT, HOST);
console.log(`Running on http://${HOST}:${PORT}`);
```

First send request using curl:

```bash
curl -v http://app.test/public-service/..%2Fprotected-service/protected
```

Next check logs of `auth-service`:

```bash
kubectl logs auth-service-node-859ccc54cc-8cnlp -f
{
    'x-request-id': 'afd1f7fbc4c45c2db17cc1f72c5ec834', 
    host: 'auth-service-node.default.svc.cluster.local', 
    'x-original-url': 'http://app.test/public-service/..%2Fprotected-service/protected', 
    'x-original-method': 'GET', 
    'x-real-ip': '172.17.0.1', 
    'x-forwarded-for': '172.17.0.1'
},
{
    uri: 'http://app.test/public-service/..%2Fprotected-service/protected', 
    parseduri: {
        scheme: 'http', 
        userinfo: undefined, 
        host: 'app.test', 
        port: undefined, 
        path: "/public-service/..%2Fprotected-service/protected", 
        query: undefined, 
        fragment: undefined, 
        reference: 'absolute'
    },
    parsedQuery: {}, 
    method: 'GET'
}
```

First `{…}` is from `console.log(req.headers)` and second `{…} `is from `console.log(parentReqInfo)`. Path is not normalized and wrong decision is made.

{{< figure src="https://user-images.githubusercontent.com/17719543/139594597-49abb40e-7afb-40fb-b4b4-211f871479f0.png" >}}

I got protected data. One more time 😅

## Summary

I have found three repositories that are using `X-Original-Url` header and are not protecting against manipulation.

### Other articles from this series

* [CVE-2021-43557: Apache APISIX: Path traversal in request_uri variable]({{< ref "/posts/CVE_2021_43557_Apache_APISIX_Path_traversal_in_request_uri_variable.md" >}})
* [Path traversal in authorization context in Traefik and HAProxy]({{< ref "/posts/path_traversal_in_authorization_context_in_Traefik_and_HAProxy.md" >}})
* [Path traversal in authorization context in Emissary]({{< ref "/posts/path_traversal_in_authorization_context_in_Emissary.md" >}})
* [Path traversal in authorization context in Kong and F5 NGINX]({{< ref "/posts/path_traversal_in_authorization_context_in_Kong_and_F5_NGINX.md" >}})
* [Bug bounty tips for nginx $request_uri path traversal bypass]({{< ref "/posts/bug_bounty_tips_for_nginx_request_uri_path_traversal_bypass.md" >}})

---

Thanks for reading! You can follow me on [Twitter](https://twitter.com/xvnpw).