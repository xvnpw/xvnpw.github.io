---
title: "From . in regex to SSRF - part 3"
date: 2020-07-07T10:14:47+01:00
draft: false
tags: [SSRF, bugbounty]
---

This is last part of my stories about exploiting service with SSRF bug. Part 1 is available [here]({{< ref "/posts/from-dot-in-regex-to-ssrf-part-1.md" >}}), and part 2 [here]({{< ref "/posts/from-dot-in-regex-to-ssrf-part-2.md" >}}).

This part is focused on abusing Node.js and *node-fetch* library. I will try to "talk" with Redis service using CRLF injection in http parser.

For convenience Redis service will be simulated by `nc -vvlp 6379`.

Test environment from my Kali 2020.1b:

* Node.js version 10.19.0
* node-fetch version 2.6.0

## CRLF Injection

Lets start with PayloadsAllTheThings. It contains couple of CRLF Injection payloads. I will loop over them and check result in second console:

{{< figure src="https://user-images.githubusercontent.com/17719543/139583089-ecf56f8e-5c1a-465d-b0ff-01204ef313aa.png" >}}

None success here. All payloads failed 🙁

Next step is to check payloads from two great articles by Orange Tsai: [first from Red Hat 2017](https://www.blackhat.com/docs/us-17/thursday/us-17-Tsai-A-New-Era-Of-SSRF-Exploiting-URL-Parser-In-Trending-Programming-Languages.pdf) and [second from his blog](https://blog.orange.tw/2017/07/how-i-chained-4-vulnerabilities-on.html). It's giving few more options to test:

```
－＊Set-Cookie:injection－＊ (Unicode U+FF0D U+FF0A)
http://0\r\n SET foo 0 60 5\r\n :6379/
https://0\r\nSET foo 0 60 5\r\n:6379/

```

Still no success here. I seams that this version of Node.js is not vulnerable for CRLF attacks.

Let's try harder and dig dipper into node-fetch, maybe something interesting will be in code 😃

## Investigation of node-fetch code

What am I trying to achieve here? I have in mind two types of possible errors:

1. Url parsing
2. Handling url input as object not as *string*

Let's see what I will find.

Debug of Node.js code is quite nice with Visual Studio Code:

{{< figure src="https://user-images.githubusercontent.com/17719543/139583200-2686e457-092e-4385-8a01-f7716babf711.png" >}}

Problem number one is not existing as node-fetch is using standard Node.js `Url.parse` for input. There are not doing much fancy stuff with it.

For second problem I needed to do more investigation.

First of all I will explain why I'm interested in processing *object* instead of *string*. In many dynamic languages you can make valid request like this:

```http://localhost:3000/c?url[href]=localhost&url[method]=POST```

This leads to created object instead of string. Could be quite handy for some scenarios. Especially if developers didn't predict it 😃 See below example of parsing such url in Node.js Express framework.


{{< figure src="https://user-images.githubusercontent.com/17719543/139583272-0e1f3070-7ec9-44a3-8df7-ac9cf858d883.png" >}}

In node-fetch I have found one possible attacking vector:

{{< figure src="https://user-images.githubusercontent.com/17719543/139583293-12f60351-0548-41c5-a92b-45711eac279f.png" >}}

It look like possible to use object instead of string for input parameter. This `input.method` could change method type in some specific conditions. After spending some time in debugger it turn out as **dead end**.

## Summary

I didn't manage to escalate blind SSRF to anything more. I have spent couple of days trying different approaches. Nevertheless after submitting report I was awarded with **400$** and bug was marked as **medium**.

---

Thanks for reading! You can follow me on [Twitter](https://twitter.com/xvnpw).
