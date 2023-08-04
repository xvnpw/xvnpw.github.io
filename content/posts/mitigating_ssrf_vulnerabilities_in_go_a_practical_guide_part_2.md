---
title: "Mitigating SSRF vulnerabilities in Go. A practical guide. Part 2"
date: 2023-08-04T07:59:02+01:00
draft: false
tags: [appsec, go, ssrf]
description: "In this final part of mitigation guide we will explore doyensec/safeurl library for Go."
---

In this final part of mitigation guide we will explore [doyensec/safeurl](https://github.com/doyensec/safeurl) library for Go.

## Setting the stage

Reminder about our setup:

{{< figure src="https://github.com/xvnpw/xvnpw.github.io/assets/17719543/28c17d0d-ff77-4674-bc27-3aba259bb8e5" class="image-center" >}}

## Explot 3

We have following Public API code and exploit that works:

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

```bash
$ curl -s \
    http://publicapi/debug\?url\=
        http://imageapi/redirect\?target\=
            http://backendapi/internal
This is internal sensitive endpoint
```

## Safeurl

What is it?

> A Server Side Request Forgery (SSRF) protection library. Made with 🖤 by Doyensec LLC.

Features:
- Protect against DNS rebinding
- Validation and issuing request done by one library

### Fix with safeurl

```go
config := safeurl.GetConfigBuilder().
	SetAllowedHosts("imageapi").
	Build()
router.GET("/debug", func(context *gin.Context) {
	urlFromUser := context.Query("url")
	client := safeurl.Client(config)
	resp, err := client.Get(urlFromUser)
```

- in lines 1-3 we defined configuration that will allow only connect to `imageapi`
- in line 4 we defined `/debug` endpoint
- in line 5 we got input from user
- in line 6 we created http client based on configuration
- in line 7 we issue request proxing it via safeurl

### Try with exploit 3

Let's take our existing explot and try it out with new code featured with safeurl:

```bash
$ curl -s \
    http://publicapi/debug\?url\=
      http://imageapi/redirect\?target\=
	      http://backendapi/internal
Get "http://imageapi/redirect?target=http://backendapi/
internal": dial tcp 10.96.45.24:80: ip: 10.96.45.24 not 
found in allowlist
```

Great! We successfully mitigated SSRF+Open Redirect chain 😃

#### Exploit 3 step by step

Let's dive deeper into this chain of vulnerabilities:

```bash
# publicapi
-> /debug request: user-agent=curl/7.86.0
400 | GET "/debug?url=http://imageapi/redirect?target=
  http://backendapi/internal"
ip: 10.96.45.24 not found in allowlist
 
# imageapi
-> /redirect request: user-agent=Go-http-client/1.1
301 | GET "/redirect?target=http://backendapi/internal"
```

- first Public API is called on `/debug` endpoint
- it will validate hostname, which is `imageapi` - OK!
- than it will call `imageapi` on `/redirect` endpoint
- Image API will return `301` redirect to `backendapi` location
- **Public API will not follow redirect**, because safeurl validated this redirect and it's not in allowlist (imageapi)

## Fix with redirect turn off

We don't need to use safeurl to mitigate those flaws. We can also turn off redirects in `http` client:

```go
http.DefaultClient.CheckRedirect =
	func(req *http.Request, via []*http.Request) error {
		return http.ErrUseLastResponse
	}
```

### Try with exploit 3

Let's try it with turned off redirects:

```bash
$ curl -s \
    http://publicapi/debug\?url\=
      http://imageapi/redirect\?target\=
	      http://backendapi/internal
<a href="http://backendapi/internal">Moved Permanently</a>.
```

Bit different response, but similar result. Which is better? Safeurl can handle redirects that targets allowlist, but it's additional library to your project. You need to decide.

## Safeurl in details

Let's dive deep into safeurl to see how it was implemented:

```go
func buildHttpClient(wc *WrappedClient) *http.Client {
  client := &http.Client{
    ...
    CheckRedirect: wc.config.CheckRedirect,
    Transport: &http.Transport{
      TLSClientConfig: wc.tlsConfig,
      DialContext: (&net.Dialer{
        Resolver: wc.resolver,
        Control:  buildRunFunc(wc),
      }).DialContext,
```

- we can see that redirects are controled with config function like this: `wc.config.CheckRedirect`
- most important is `DialContext` which has `Resolver: wc.resolver` and `Control:  buildRunFunc(wc)`
  - this way we can inspect and control every network call of `http` client
 
Remarkable how easy it's in `go` to wire into low level network call for `http`. Well done!

## Other controls for SSRF mitigation

What else we can do to be protected? SSRF mitigation is not only about code but also about architecture:

- all outbound network traffic should go through egress proxy
- internal Kubernetes network should be segmented using Network Policies
- microservice endpoints should require authentication

## Real life SSRF attacks

With this vulnerability attacker can:

- steal cloud metadata credentials (169.254.169.254)
- steal sensitive tokens (attached by service to http call)
- call internal network sensitive resources
- escalate to RCE via 3rd party apps, e.g. redis, Confluence

Typical places where SSRF can occur:

- webhooks
- file imports from urls
- PDF generators

## Summary

- if possible not consume full URLs from users (😅 webhooks)
- apply application level protection - doyensec/safeurl or custom implementation
- apply zero trust architecture protection - network segmentation and authentication

## Code

You can try yourself with code that is ready to run on your local: https://github.com/xvnpw/ssrf-in-go

If you have any comments or feedback, you are welcome to write to me on [Twitter](https://twitter.com/xvnpw).
