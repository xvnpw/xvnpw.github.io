---
title: "Path traversal in authorization context in Emissary"
date: 2021-11-24T19:59:02+01:00
draft: false
tags: [kubernetes, ingress, emissary, path-traversal]
---

{{< figure src="https://user-images.githubusercontent.com/17719543/139592951-0fafc921-437e-4bb7-b0ee-199dd72b36c3.png" class="image-center" >}}

After checking Apache APISIX and Traefik, for path traversal in authZ context, now I will research Emissary ingress.

In Emissary there is feature called [Basic authentication](https://www.getambassador.io/docs/emissary/latest/howtos/basic-auth/), which is very similar to forward authentication discussed in Traefik.

>Emissary-ingress can authenticate incoming requests before routing them to a backing service.

**I can already tell you that Emissary is secure and you cannot bypass using path traversal**. What is even better, it's (and envoy) aware of this kind of security concern. There is full description in documentation. I encourage you to read it:

* [rejecting-client-requests-with-escaped-slashes](https://www.getambassador.io/docs/edge-stack/latest/topics/running/ambassador/#rejecting-client-requests-with-escaped-slashes) - although it's not directly for Emissary. It's describing well envoy concerts for escaped slashes
* [GHSA-xcx5-93pw-jw2w](https://github.com/envoyproxy/envoy/security/advisories/GHSA-xcx5-93pw-jw2w) (CVE-2019–9901) - description of risks associated with normalizing paths in envoy
* [envoy http connection manager options](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/network/http_connection_manager/v3/http_connection_manager.proto) - look for two particular: `normalize_path` and `path_with_escaped_slashes_action`

Emissary is setting `normalize_path` to `true` and `path_with_escaped_slashes_action` to `KEEP_UNCHANGED`. Which is preventing from path traversal bypass. 
In this place I would like to point out difference between Emissary ingress and Edge Stack. The latter is using Filter as [authentication service](https://www.getambassador.io/docs/edge-stack/latest/topics/running/services/auth-service/). I did not cover it in my research.

## Setting the stage

Install Emissary ingress in Kubernetes:

```bash
helm repo add datawire https://app.getambassador.io
helm repo update
kubectl create namespace emissary && helm install emissary-ingress --devel --namespace emissary datawire/emissary-ingress && kubectl -n emissary wait --for condition=available --timeout=90s deploy -lapp.kubernetes.io/instance=emissary-ingress
```

If you need more info check [here](https://www.getambassador.io/docs/emissary/latest/topics/install/helm/).

Check if all emissary pods are running: `kubectl get pods -n emissary`

{{< figure src="https://user-images.githubusercontent.com/17719543/139598827-d5ca6657-597d-4c02-b6ea-2b399d1ad828.png" >}}

Deploy listener:

```yaml
apiVersion: getambassador.io/v3alpha1
kind: Listener
metadata:
  name: emissary-ingress-listener-8080
  namespace: emissary
spec:
  port: 8080
  protocol: HTTP
  securityModel: XFP
  hostBinding:
    namespace:
      from: ALL
```

Deploy auth service definition and Emissary mappings:

```yaml
apiVersion: getambassador.io/v3alpha1
kind: AuthService
metadata:
  name: authentication
spec:
  auth_service: "http://auth-service.default.svc.cluster.local:8080"
  proto: http
  path_prefix: "/verify"
  allowed_request_headers:
    - "X-Api-Key"
---
apiVersion: getambassador.io/v3alpha1
kind: Mapping
metadata:
  name: public-service
spec:
  hostname: "app.test"
  prefix: /public-service/
  service: public-service:8080
---
apiVersion: getambassador.io/v3alpha1
kind: Mapping
metadata:
  name: protected-service
spec:
  hostname: "app.test"
  prefix: /protected-service/
  service: protected-service:8080
```

Code for `auth-service` is changed:

```python
from flask import Flask, Response, request
from http import HTTPStatus
import sys

app = Flask(__name__)

@app.route('/verify/<path:path>')
def verify(path):
    print(request.headers, file=sys.stderr)
    print(path, file=sys.stderr)
    api_key = request.headers.get('X-Api-Key')

    if path and path.startswith("public-service/"):
        return Response(status = HTTPStatus.OK)

    if api_key == "secret-api-key":  
        return Response(status = HTTPStatus.OK)

    return Response(status = HTTPStatus.UNAUTHORIZED)
```

It's not using headers as Traefik. Instead it's passing requested uri as path into `/verify` route.

## Test

Let's check my payloads:

### 1° payload

`curl -v http://app.test/public-service/..%2Fprotected-service/protected`

{{< figure src="https://user-images.githubusercontent.com/17719543/139598911-64682f0c-a199-4125-b928-5070d8e1a658.png" >}}

I have got 404. And logs from `public-service` are giving answer why:

{{< figure src="https://user-images.githubusercontent.com/17719543/139598944-dd4c4f2e-27f5-46e7-b2d0-1e78a640e4e9.png" >}}

There is no route defined in `public-service` that is called `/..%2Fprotected-service/protected` 😃

What was logged by `auth-service`:

{{< figure src="https://user-images.githubusercontent.com/17719543/139598957-8c9c8e9c-d0f9-4bb2-bd35-1ba97270d5b1.png" >}}

Everything is align. Prefix is `public-service`, `%2F` was not decoded (due to `path_with_escaped_slashes_action` envoy option) and `auth-service` got right path for decision.

### 2° payload

With second payload it's a bit different as `normalize_path` envoy option is set to `true`.

`curl  --path-as-is -v http://app.test/public-service/../protected-service/protected`

{{< figure src="https://user-images.githubusercontent.com/17719543/139598981-b426d3ac-c5b1-494c-bb11-be05eca1bfc4.png" >}}

This time it's not 404, but 401. Why ? Path was normalized. Everywhere. Not only in routes decision making but also in authentication service. Let's check logs from `auth-service`:

{{< figure src="https://user-images.githubusercontent.com/17719543/139599021-1efcd692-eb39-4224-9a41-7a62be8dc67d.png" >}}

Yep. Logs are confirming this.

Emissary vs ingress authZ bypass - 1 : 0 😃

## Summary

Emissary is resistant for ingress authZ bypass using path traversal. I'm impressed that both Emissary and envoy are aware of this concern. And whole documentation about it is in place. Also default configuration is secure. I was even looking how to change it to less secure, but didn't find a way 😅

### Versions of components that I was using:

minikube v1.23.2 on Microsoft Windows 10 Pro 10.0.19043 Build 19043; Kubernetesa v1.22.2 on Docker 20.10.8; Emissary 2.0.4

### Other articles from this series

* [CVE-2021-43557: Apache APISIX: Path traversal in request_uri variable]({{< ref "/posts/CVE_2021_43557_Apache_APISIX_Path_traversal_in_request_uri_variable.md" >}})
* [Path traversal in authorization context in Traefik and HAProxy]({{< ref "/posts/path_traversal_in_authorization_context_in_Traefik_and_HAProxy.md" >}})

---

Thanks for reading! You can follow me on [Twitter](https://twitter.com/xvnpw).