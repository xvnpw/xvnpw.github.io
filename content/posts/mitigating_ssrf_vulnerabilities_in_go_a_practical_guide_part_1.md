---
title: "Mitigating SSRF vulnerabilities in Go. A practical guide. Part 1"
date: 2023-07-29T18:59:02+01:00
draft: false
tags: [appsec, go, ssrf]
description: "Server-Side Request Forgery (SSRF) vulnerabilities have been around for a long time, and they still pose a significant threat to web applications, so much so this kind of vulnerability has been included in OWASP TOP 10. This time I will explain how to mitigate SSRF vulnerability in Go applications."
---

Server-Side Request Forgery (SSRF) vulnerabilities have been around for a long time, and they still pose a significant threat to web applications, so much so this kind of vulnerability has been included in OWASP TOP 10. This type of attack allows an attacker to send unauthorized requests from a vulnerable application, which can lead to data leakage, server-side request smuggling, and even full-scale remote code execution.

## Basic example of SSRF in Go

Let's see basic code that introduce this vulnerability:

```go
router.GET("/debug", func(context *gin.Context) {
    urlFromUser := context.Query("url")
    // no validation yloo
    resp, err := http.Get(urlFromUser)
```

It's very simple:
- in line 1 we defined `/debug` endpoint
- in line 2 we got input from user
- in line 4 we issue request totally forgetting about any security concerns 😇

Before going deeper let's check [OWASP TOP 10](https://owasp.org/www-project-top-ten/).

## OWASP Top 10

If you don't know what OWASP Top 10 is I recommend you to visit their page. 

SSRF is number 10 on the 2021 version of the list. It was voted by people together with "Security Logging and Monitoring Failures".

> SSRF flaws occur whenever a web application is fetching a remote resource without validating the user-supplied URL.

## Setting the stage

{{< figure src="https://github.com/xvnpw/xvnpw.github.io/assets/17719543/bc85bf4c-bb95-48e4-a22d-dcaf52473664" class="image-center" >}}

To have more meaningful example, I will use Kubernetes as my deployment platform. This way I will be able to easily model new services and connections among them.

- attacker will be placed outside of the Kubernetes cluster
- from outside, we can only access Public API. We cannot access Backend API, because it's not exposed

**Attacker objective** is to access Backend API using SSRF flaw in Public API. Our objective is to mitigate that 😈

## Exploit 1

Let's check first exploit:

```bash
$ curl -s \
    http://publicapi/debug\?url\=
        http://backendapi/internal
This is internal sensitive endpoint
```

- we sent `GET` request to `publicapi` `/debug` endpoint
- we set `url` parameter to `http://backendapi/internal`
- and we got response `This is internal sensitive endpoint`

Each time we will see this response `This is internal sensitive endpoint` it means that exploit is **successful**. 

To remind how service code look liked:

```go
router.GET("/debug", func(context *gin.Context) {
    urlFromUser := context.Query("url")
    // no validation yloo
    resp, err := http.Get(urlFromUser)
```

Without any restrictions, we let attacker to access Backend API 😟

## Mitigation

### Code checker 

Code checker or SAST (Static application security testing) tool, can help us find this problem early and not introduce it at all.

Let's use [gosec](https://github.com/securego/gosec):

```bash
$ gosec publicapi/
G107 (CWE-88): Potential HTTP request made with variable url 
(Confidence: MEDIUM, Severity: MEDIUM)
    83: urlFromUser := context.Query("url")
    > 84: resp, err := http.Get(urlFromUser)
```

We got exactly line number and brief explanation.

Other tool that I can recommend for `go` apps is [semgrep](https://semgrep.dev/docs/getting-started/).

### Fix with negative validation

We can try to fix this SSRF by validating user input. Validation can be done in many ways. I will try first something that might not be very wise, just to prove a point:

```go
func validateTargetUrl(input string) bool {
    u, err := url.ParseRequestURI(input)
    if err != nil {
        return false
    }
    if (u.Scheme == "http" || u.Scheme == "https") &&
            (u.Hostname() != "backendapi") {
        return true
    }
    return false
}
```

What this function does is:
- parse `input` to url
- check if `Scheme` is `http` or `https` and
- make sure that we are not trying to connect to `backendapi`

Remember, we don't want to let the attacker connect to the `backendapi`!

### Exploit 2

Let's see Public API code one more time. But now, slightly modified:

```go
router.GET("/debug", func(context *gin.Context) {
    urlFromUser := context.Query("url")
    // validation because world is full of mean people :(
    if !validateTargetUrl(urlFromUser) {
        context.String(http.StatusBadRequest, "Bad url")
    return
    }
    resp, err := http.Get(urlFromUser)
```

and the exploit:

```bash
$ curl -s \
    http://publicapi/debug\?url\=
        http://10.96.155.247/internal
This is internal sensitive endpoint
```

Heh, yes, we have successful exploit.

Why usage of raw IP address was possible? Public API and Backend API are **sharing same network** (Kubernetes!). There is nothing in between.

Using IP addresses in numerous formats is interesting technique for bypassing SSRF filters. 

{{< figure src="https://github.com/xvnpw/xvnpw.github.io/assets/17719543/56b159db-94e1-4478-90e7-3f1883daea1b" class="image-center" >}}

As you see from above picture there are many ways to represent `127.0.0.1`, but there is one missing. Very interesting one.

You can create dns A record for localhost, e.g. `xvnpw.localtest.me`. Try it yourself!

**Important learning**: negative validation is doomed to failure 💀

### Fix with positive validation

We know what was bad last time, let's do better:

```go
func validateTargetUrl(input string) bool {
    u, err := url.ParseRequestURI(input)
    if err != nil {
        return false
    }
    if (u.Scheme == "http" || u.Scheme == "https") &&
            (u.Hostname() == "imageapi") &&
            (u.Port() == "" || u.Port() == "80" || u.Port() == "443")
        return true
    }
    return false
}
```

What this function does is:
- parse `input` to url
- check if `Scheme` is `http` or `https` and
- check if `Port` is empty, 80 or 443
- **check if** `Hostname` is `imageapi`

Pretty robust. But what is this Image API? Let's check our new setup:

{{< figure src="https://github.com/xvnpw/xvnpw.github.io/assets/17719543/28c17d0d-ff77-4674-bc27-3aba259bb8e5" class="image-center" >}}

This time we added Image API to the picture. And in contrast to Backend API, it can be called from Public API `/debug` endpoint.

### Exploit 3

Public API code stays the same:

```go
router.GET("/debug", func(context *gin.Context) {
    urlFromUser := context.Query("url")
    // validation because world is full of mean people :(
    if !validateTargetUrl(urlFromUser) {
        context.String(http.StatusBadRequest, "Bad url")
    return
    }
    resp, err := http.Get(urlFromUser)
```

and new exploit:

```bash
$ curl -s \
    http://publicapi/debug\?url\=
        http://imageapi/redirect\?target\=
            http://backendapi/internal
This is internal sensitive endpoint
```

It might seems that it's a bit unfair to abuse Image API, but this is reality in many bug bounty programs. You can find yet another service with vulnerability that you can chain together.

Do you see what kind of flaw is in Image API? It's [Open Redirect](https://portswigger.net/kb/issues/00500100_open-redirection-reflected). What it does? It returns `301` redirect http code and `Location` taken directly from input parameter (`target=`). It `low` vulnerability but can be used to escalate SSRF.

#### Exploit 3 step by step

Let's dive deeper into this chain of vulnerabilities:

```bash
# publicapi
-> /debug request: user-agent=curl/7.86.0
200 | GET "/debug?url=http://imageapi/redirect?target=
http://backendapi/internal"

# imageapi
-> /redirect request: user-agent=Go-http-client/1.1
301 | GET "/redirect?target=http://backendapi/internal"

# backendapi
-> /internal request: user-agent=Go-http-client/1.1
200 | GET "/internal"
```

- first Public API is called on `/debug` endpoint
- it will validate hostname, which is `imageapi` - OK!
- than it will call `imageapi` on `/redirect` endpoint
- Image API will return `301` redirect to `backendapi` location
- **Public API will follow redirect and call Backend API**

One more view on this:

{{< figure src="https://github.com/xvnpw/xvnpw.github.io/assets/17719543/b06b7a27-2704-40e1-bb7c-c519b34c6e40" class="image-center" >}}

This is very interesting. Go `http` default client will follow any redirects. You will get same behavior in other languages.

Our positive validation code is best what we could done. Why we didn't mitigate this scenario 😟? It's really simple: `http` client knows nothing about validation. 

{{< figure src="https://github.com/xvnpw/xvnpw.github.io/assets/17719543/6217f5b9-59c0-4ce0-a827-ba1325b8a43f" class="image-center" >}}

## Summary

So far attacker **won** this battle, we were not able to protect Backed API service. Positive validation is not enough in case of redirect. We need to take it one step further. I will show you how to do it in part 2 😃

If you have any comments or feedback, you are welcome to write to me on [Twitter](https://twitter.com/xvnpw).
