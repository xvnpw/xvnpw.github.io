---
title: "Path traversal in authorization context in Kong and F5 NGINX"
date: 2021-11-25T20:49:02+01:00
draft: false
tags: [kubernetes, ingress, kong, path-traversal]
---

{{< figure src="https://user-images.githubusercontent.com/17719543/139592951-0fafc921-437e-4bb7-b0ee-199dd72b36c3.png" class="image-center" >}}

In this part I will research another ingress controller based on **nginx**: 🦍 [kong](https://konghq.com/solutions/kubernetes-ingress/). At the end of article I will mention in short [F5 NGINX Ingress Controller](https://www.nginx.com/products/nginx-ingress-controller).

In kong there is no explicit feature called external authentication, but developers gave possibility to create it using plugins.

Here are some links describing this process:
* [Custom Authentication and Authorization Framework with Kong](https://konghq.com/blog/custom-authentication-and-authorization-framework-with-kong/)
* [aunkenlabs/kong-external-auth](https://github.com/aunkenlabs/kong-external-auth) - repository with PoC of external-auth. It's old and cannot be run as it with kong 2.6, which is latest at time of writing.

During analysis I have found two possible exploitation paths:
* using [basic-auth](https://docs.konghq.com/hub/kong-inc/basic-auth/) and [acl plugins](https://docs.konghq.com/hub/kong-inc/acl/) - general idea is to create acl for route to protect service. I have tested it but, was not able to exploit 🙁
* using custom plugin to implement external authentication

## Custom Plugin

I based my plugin on `aunkenlabs/kong-external-auth`, but made it compatible with kong 2.6 and align with my `auth-service`:

```lua
local BasePlugin = require "kong.plugins.base_plugin"
local http = require "resty.http"

local ExternalAuthHandler = BasePlugin:extend()

function ExternalAuthHandler:new()
  ExternalAuthHandler.super.new(self, "external-auth")
end

function ExternalAuthHandler:access(conf)
  ExternalAuthHandler.super.access(self)

  local client = http.new()
  client:set_timeouts(conf.connect_timeout, send_timeout, read_timeout)

  local res, err = client:request_uri(conf.url, {
    method = "GET",
    ssl_verify = false,
    headers = {
      ["X-Original-Uri"] = ngx.var.request_uri,
      ["X-Forwarded-Path"] = kong.request.get_path(),
      ["X-Forwarded-Method"] = kong.request.get_method(),
      ["X-Forwarded-Query"] = kong.request.get_raw_query(),
      ["X-Api-Key"] = kong.request.get_headers()["X-Api-Key"]
    }
  })

  if not res then
    return kong.response.exit(500)
  end

  if res.status ~= 200 then
    return kong.response.exit(401)
  end
end

ExternalAuthHandler.PRIORITY = 900

return ExternalAuthHandler
```

To load this plugin in Kubernetes deployment, read this guide: [setting up custom plugins](https://docs.konghq.com/kubernetes-ingress-controller/2.0.x/guides/setting-up-custom-plugins/). I loaded it as `ConfigMap` and added reference to `values.yaml`.

## Test

I'm using kong in version **2.6.0** and kong ingress in **2.0.5**.

This what is most important is value of headers that are coming into `auth-service`. 

First payload: `curl --path-as-is -v http://app.test/public-service/../protected-service/protected`:

{{< figure src="https://user-images.githubusercontent.com/17719543/140642782-6f764efb-af14-433c-9a91-e90624adeacd.png" >}}

and logs from `auth-service`:

{{< figure src="https://user-images.githubusercontent.com/17719543/140642764-128d25d7-a5fd-40c2-8982-cc0f4ae77d86.png" >}}

As you can see from image values of both headers are manipulated:
```
X-Original-Uri: /public-service/../protected-service/protected
X-Forwarded-Path: /public-service/../protected-service/protected
```

Why both headers are important? So first header is taken directly from `ngx.var.request_uri` but second one is taken using kong api: `kong.request.get_path()`. Result is a bit shocking for me as I was expecting to see normalized path in case of call to kong api. This is due fact that in kong [source code](https://github.com/Kong/kong/blob/f27c5868fc48bec1cc9e740bd1d1cf65793c473d/kong/tools/uri.lua#L60) there is path normalization implemented.

In case of second payload: `curl -v http://app.test/public-service/..%2Fprotected-service/protected`. There is **no success**:

{{< figure src="https://user-images.githubusercontent.com/17719543/140642947-3e2302d0-5668-48f8-b318-c503f937fab6.png" >}}

It's interesting that kong is not url decoding `%2F`. What is even more interesting it's decoding `%2E` to `.` 🤔

The `404` is coming from `public-service`:

{{< figure src="https://user-images.githubusercontent.com/17719543/140642967-98e2d374-9b82-4a69-95ed-3074ed39e042.png" >}}

## F5 NGINX Ingress Controller

There is no dedicated feature for external authentication, but using annotations you can add it like this ([read more](https://github.com/nginxinc/kubernetes-ingress/issues/873)):

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.org/location-snippets: |
      auth_request /auth;
    nginx.org/server-snippets: |
      location = /auth {
        return 200;
      }
  name: cafe-ingress
  namespace: default
```

If you can add external auth, you can also add `$request_uri` as some header, which will effectively allow exploitation using path traversal.

## Summary

After failed on exploitation with ingresses based on Traefik and envoy, I had mixed feelings about kong. This time I was successful 😀. From defenders perspective it's good that `acl` plugin is not vulnerable. In external auth case, developers need to build custom plugin and authentication service without path normalization. 

Kong developers are very explicit about `get_path()` function. In documentation there is info that it's not normalized in anyway. This is clear indicator for creators of custom plugins: https://docs.konghq.com/gateway/2.6.x/pdk/kong.request/#kongrequestget_path. I have also check code of various kong plugins and they are secure. **So the only valid case for exploitation is custom plugin using not normalized variables**.

Here my code if you want to try yourself: https://github.com/xvnpw/k8s-ingress-auth-bypass-kong

**F5 NGINX Ingress Controller** is also suffering from `request_uri` being not normalized.

### Other articles from this series

* [CVE-2021-43557: Apache APISIX: Path traversal in request_uri variable]({{< ref "/posts/CVE_2021_43557_Apache_APISIX_Path_traversal_in_request_uri_variable.md" >}})
* [Path traversal in authorization context in Traefik and HAProxy]({{< ref "/posts/path_traversal_in_authorization_context_in_Traefik_and_HAProxy.md" >}})
* [Path traversal in authorization context in Emissary]({{< ref "/posts/path_traversal_in_authorization_context_in_Emissary.md" >}})

---

Thanks for reading! You can follow me on [Twitter](https://twitter.com/xvnpw).