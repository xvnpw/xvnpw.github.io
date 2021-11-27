---
title: "Bug bounty tips for nginx $request_uri path traversal bypass"
date: 2021-11-27T22:59:05+01:00
draft: false
tags: [kubernetes, nginx, kong, "F5", "apisix", path-traversal, bugbounty]
---

{{< figure src="https://user-images.githubusercontent.com/17719543/139592951-0fafc921-437e-4bb7-b0ee-199dd72b36c3.png" class="image-center" >}}

In this article, I will extend topic by bug bounty tips for weaknesses in authentication/authorization implementation in relation to nginx's `$request_uri` variable.

## APIs

This vulnerability is for APIs. Best scenario are **microservice** deployed to Kubernetes and exposed by ingress controller.

## Using paths

API that you are playing with, need to use paths to address services, e.g.:

```
OK!
https://api.example.com/user-service
https://api.example.com/customer-service
...
NOT OK!
https://user.example.com/
https://customer.example.com/
```

First set of URLs is good for exploitation, as you can try sending request with `https://api.example.com/user-service/..%2F/customer-service/endpoint1`

## Using nginx based ingress controller

In this point we have having two condition, using Kubernetes and using nginx based ingress controller, e.g.: kong, Apache APISIX, F5 NGINX. 

Kubernetes is used in many organizations right now. If you see that API consists of multiply services, you can safely bet on Kubernetes as orchestration. 

To verify if specific ingress is in place you can try to get error message, e.g.: `curl --path-as-is https://api.example.com/sdalksjdeiu23432/cutomer-serivice/endpoint1`

{{< figure src="https://user-images.githubusercontent.com/17719543/139599207-a1c661f7-ac5f-421b-a48f-eabc8c2cea81.png" >}}

This `sdalksjdeiu23432` is just not existing service. You can see that there is nginx in response.

## Normalization of ../ and ..%2F

It's good to check what is happening for normalization of paths. Between your machine and ingress could be other servers, e.g: additional proxies or WAF (Web Application Firewall).

```bash
curl --path-as-is https://api.example.com/sdalksjdeiu23432/../cutomer-serivice/endpoint1
curl --path-as-is https://api.example.com/sdalksjdeiu23432/..%2F/cutomer-serivice/endpoint1
curl --path-as-is https://api.example.com/sdalksjdeiu23432/..%252Fcutomer-serivice/endpoint1 # double encoding
```

Comparing results could give you idea about path normalization.

## External authentication service

This is quite hard to investigate. Idea behind external authentication service is about having it centralized. Having broken authentication proof (e.g. JWT) would issue 401/403 on ingress rather than on upstream.

I would follow those steps:

1. Login into application
2. Get some request to backend service
3. Change it in a way that authentication proof is broken. For JWT it would be just to place any character into it.
4. Send changed request and see results

Something that you would like to see is:

{{< figure src="https://user-images.githubusercontent.com/17719543/139599242-26908b17-554a-4737-af2c-6f163bb0560e.png" >}}

If you cannot get any indication whether centralized authentication is in place, you can also assume so and try to exploit.

## Centralized authorization

Authentication service is checking if you are who you are talking to be. But authorization is making decisions about letting you do some action. Having it centralized in some way is necessary for exploitation. If backend services are doing access control on they own, there is **no way** to exploit it with presented bypass.

You can do assumption here, that there is centralized authorization and move on.

## Exploitation

### Public service

Try to find service that is handling requests for anyone. Without any authentication proof. Some kind of public service. If you have one, that's good, if you don't have there is still one thing you can do (described in next paragraph).

OK. We have some `public-service` and also `protected-service` that is only for logged in users (e.g. with valid JWT token). 

Do some tests:

1. Take a valid request to protected service, e.g. `/protected-service/protected?a=1` and change it to `/public-service/..%2Fprotected-service/protected?a=1` but send it **without any token**. 
2. Make token invalid and send request from point 1.
3. Wait and make token expired and send request from point 1.

What responses did you get? In case of luck you are already getting valid response for point 1. If not maybe point 2 or 3 was successful for you. If not try with different public and protected services. If you still get no valid response it means that there is no vulnerability here.

### Privilege escalation

There could be implementation of centralized access control that is checking to which group/role user belong. 

In this situation try following steps:

1. Find endpoint that you cannot access. 
2. Take a valid request with authentication proof (e.g. JWT).
3. Send request to endpoint from point 1, but using path traversal described in previous paragraph.

If you didn't success it means that there is no vulnerability here. Sadly… ☹️

## Summary

I have presented steps that can be inspiration for you. Do not limit yourself and be creative. Happy hunting!

### Other articles from this series

* [CVE-2021-43557: Apache APISIX: Path traversal in request_uri variable]({{< ref "/posts/CVE_2021_43557_Apache_APISIX_Path_traversal_in_request_uri_variable.md" >}})
* [Path traversal in authorization context in Traefik and HAProxy]({{< ref "/posts/path_traversal_in_authorization_context_in_Traefik_and_HAProxy.md" >}})
* [Path traversal in authorization context in Emissary]({{< ref "/posts/path_traversal_in_authorization_context_in_Emissary.md" >}})
* [Path traversal in authorization context in Kong and F5 NGINX]({{< ref "/posts/path_traversal_in_authorization_context_in_Kong_and_F5_NGINX.md" >}})

---

Thanks for reading! You can follow me on [Twitter](https://twitter.com/xvnpw).