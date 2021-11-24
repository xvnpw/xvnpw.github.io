---
title: "Path traversal in authorization context in Traefik and HAProxy"
date: 2021-11-23T22:38:58+01:00
draft: false
tags: [kubernetes, ingress, traefik, haproxy, path-traversal]
---

{{< figure src="https://user-images.githubusercontent.com/17719543/139592951-0fafc921-437e-4bb7-b0ee-199dd72b36c3.png" class="image-center" >}}

In my previous post about [Apache APISIX]({{< ref "/posts/CVE_2021_43557_Apache_APISIX_Path_traversal_in_request_uri_variable.md" >}}) I have found path traversal in uri-blocker plugin. In this text I will focus on yet another ingress controller which is [Traefik](https://doc.traefik.io/traefik/providers/kubernetes-ingress/). It has feature called forward auth. At the end I will mention HAProxy ingress controller.

From [docs](https://doc.traefik.io/traefik/v2.0/middlewares/forwardauth/):

>The ForwardAuth middleware delegate the authentication to an external service. If the service response code is 2XX, access is granted and the original request is performed. Otherwise, the response from the authentication server is returned.

{{< figure src="https://user-images.githubusercontent.com/17719543/139594882-8a3e37fb-fc9f-4f51-9e06-018e5128802c.png" class="image-center" >}}

## Setting the stage

First what we need to do is to install traefik in kubernetes:

```bash
helm repo add traefik https://helm.traefik.io/traefik
helm repo update
kubectl create namespace traefik
helm upgrade --install traefik \
    --namespace traefik \
    --set dashboard.enabled=true \
    --set rbac.enabled=true \
    --set="additionalArguments={--api.dashboard=true,--log.level=INFO,--providers.kubernetesingress.ingressclass=traefik-internal,--serversTransport.insecureSkipVerify=true}" \
    traefik/traefik --version 10.6.2
```

If you need more info about installation check [here](https://doc.traefik.io/traefik/getting-started/install-traefik/#use-the-helm-chart) and [here](https://blog.zachinachshon.com/traefik-ingress/).

Check if it's up and running: `kubectl get pods -n traefik`

{{< figure src="https://user-images.githubusercontent.com/17719543/139594927-7ec2ec60-1dca-4541-ac5a-62115fcf5186.png" class="image-center" >}}

Deployment yaml for ingress look like this:

```yaml
kind: IngressRoute
apiVersion: traefik.containo.us/v1alpha1
metadata:
  name: services
  namespace: default
spec:
  entryPoints: 
    - web
  routes:
  - match: Host(`app.test`) && PathPrefix(`/public-service`)
    kind: Rule
    services:
    - name: public-service
      port: 8080
    middlewares:
      - name: public-stripprefix
      - name: auth-service
  - match: Host(`app.test`) && PathPrefix(`/protected-service`)
    kind: Rule
    services:
    - name: protected-service
      port: 8080
    middlewares:
      - name: protected-stripprefix
      - name: auth-service
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: protected-stripprefix
spec:
  stripPrefix:
    prefixes:
      - /protected-service
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: public-stripprefix
spec:
  stripPrefix:
    prefixes:
      - /public-service
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: auth-service
spec:
  forwardAuth:
    address: http://auth-service.default.svc.cluster.local:8080/verify
```

It's using middleware that is specifying forwardAuth.

`auth-service` code is here:

```python
from flask import Flask, Response, request
from http import HTTPStatus
import sys

app = Flask(__name__)

@app.route('/verify')
def verify():
    print(request.headers, file=sys.stderr)
    api_key = request.headers.get('X-Api-Key')
    forwarded_prefix = request.headers.get('X-Forwarded-Prefix')

    if forwarded_prefix and forwarded_prefix.startswith("/public-service"):
        return Response(status = HTTPStatus.NO_CONTENT)

    if api_key == "secret-api-key":  
        return Response(status = HTTPStatus.NO_CONTENT)

    return Response(status = HTTPStatus.UNAUTHORIZED)
```

## Test

Traefik is using headers in communication with forwardAuth service. 

### Headers

Those are:

* X-Forwarded-For
* X-Forwarded-Host
* X-Forwarded-Method
* X-Forwarded-Port
* X-Forwarded-Prefix
* X-Forwarded-Proto
* X-Forwarded-Server
* X-Forwarded-Uri
* X-Real-Ip

Quite a bit, and maybe also some potential for bugs.

## Exploitation

We are ready now to send some request and check how traefik is handling malicious payloads.

```bash
curl -v http://app.test/public-service/..%2Fprotected-service/protected
```

{{< figure src="https://user-images.githubusercontent.com/17719543/139594991-5a92f493-bd6f-421c-bda3-d2294c3af76c.png" class="image-center" >}}

This is interesting. I have got 404. Which is completely different then Apache APISIX. It's returned by Python not traefik. 

But which service got this request. Let's check logs of public-service: `kubectl logs public-service-7d56f8589d-59jqg`

{{< figure src="https://user-images.githubusercontent.com/17719543/139595015-cf573ed1-8ac6-42f0-868c-58f9ca81bcd4.png" class="image-center" >}}

Oh! So **traefik is not normalizing requests** before executing them. That is important observation. 

And now logs from `auth-service`:

{{< figure src="https://user-images.githubusercontent.com/17719543/139595039-e002087c-fb99-49fd-a6ba-3a0f8af6abd7.png" class="image-center" >}}

The `X-Forwarded-Prefix` is containing right value: `/public-service` and `X-Forwarded-Uri` is not having `/public-service` at all.

Good job traefik! 👍

I checked second payload with: `curl --path-as-is -v http://app.test/public-service/../protected-service/protected` but no luck.

## HAProxy

I did similar research for HAProxy based [haproxy-ingress](https://haproxy-ingress.github.io/) as it's also having option for external authentication. Results were very similar to those from Traefik. No bypass is possible, as HAProxy is [not normalizing paths by default](https://www.haproxy.com/blog/announcing-haproxy-2-4/).

## Summary

Trying with different ingress-controller was quite a fun 😃 However I was hoping for similar effect that I got with Apache APISIX. 

Traefik is not normalizing request paths before executing them. In my exploitation this is defense preventing from bypassing forwardAuth.

### Versions of components that I was using:

minikube v1.23.2 on Microsoft Windows 10 Pro 10.0.19043 Build 19043; Kubernetesa v1.22.2 on Docker 20.10.8; traefik:2.5.3

### Other articles from this series

* [CVE-2021-43557: Apache APISIX: Path traversal in request_uri variable]({{< ref "/posts/CVE_2021_43557_Apache_APISIX_Path_traversal_in_request_uri_variable.md" >}})
* [Path traversal in authorization context in Emissary]({{< ref "/posts/path_traversal_in_authorization_context_in_Emissary.md" >}})

---

Thanks for reading! You can follow me on [Twitter](https://twitter.com/xvnpw).