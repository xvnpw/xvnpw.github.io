---
title: "From . in regex to SSRF - part 1"
date: 2021-01-05T10:14:47+01:00
draft: false
---

![obraz](https://user-images.githubusercontent.com/17719543/139576912-865d0f16-6dc3-4af9-8a39-5e77d7b236c3.png)

In test of one application I have encountered bug in regex that leaded to Server Side Request Forgery (SSRF). Way of finding it was huge fun and excitement. It was also my first bug on production system ever.

During a recon I have found service called image-converter. It was definitely interesting, but not straight forward to exploit. I had no example of usage it and on simple GET request I was just getting:

![obraz](https://user-images.githubusercontent.com/17719543/139576933-06cadc0d-6489-4ac1-8e58-292d5fb1baf8.png)

That was first major problem for me. I was trying with some simple query parameters like:
* `?url=`
* `?width=`
* `?name=`

and so on but without luck. Then I tried with https://github.com/s0md3v/Arjun which is tool for automated parameter discovery. This also failed. I was pretty sure that there is something out there, but I couldn't force it to work.

Then I started digging in what is this error message that I see all the time: `"Cannot read property 'groups' of null"`. This leads me to stackoverflow question about JavaScript and regex error. After that I was wondering: "How the hell they have implemented this?". After hour of trying and failure, I got it:

```https://api.example.org/image-converter/width=100/http://google.com```

![obraz](https://user-images.githubusercontent.com/17719543/139577018-2487fddb-ef0a-449b-a893-7ad929e4aa0b.png)

In my almost 10 years IT career, I didn't see service implementation like that ;)

My positive energy went down, as I realized that there is domain whitelisting implemented. I have picked main domain `www.example.org` and in fact it was working:

```https://api.example.org/image-converter/width=100/https://www.example.org```

I got response:

![obraz](https://user-images.githubusercontent.com/17719543/139577079-083fd93f-3679-4487-9f91-4d7f8fec6be8.png)

In this moment I was sure about SSRF, but still had whitelisting to bypass.

My first approach was to take SSRF from [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery) and test it. I don't want to copy all that here. There is dozen of payloads. Sadly not of it worked. I got very interested in [Orange: A New Era SSRF](https://www.blackhat.com/docs/us-17/thursday/us-17-Tsai-A-New-Era-Of-SSRF-Exploiting-URL-Parser-In-Trending-Programming-Languages.pdf), but that was also death end.

I was pretty puzzled. Having high hope on some nice bug, but it looked like this service was secured. Good thing was that I have learned a lot, especially from Orange paper.

Next day with fresh head I took different way. During recon I have noted two other domains connected with main one: `www.example.net` and `www.example.com`. It turn out that those domains where also whitelisted. Having a background in programming I knew that developers have a tendency to write "nice code", so maybe they used regex to check domain suffix? And guess what? They did! For request:

```https://api.example.org/image-converter/width=100/https://www-example.org```

I got response:

![obraz](https://user-images.githubusercontent.com/17719543/139577185-1b2711fb-6a19-435e-8bd3-015deb803884.png)

Hurray!

What exactly regex they used ? I think something like this regex101:

![obraz](https://user-images.githubusercontent.com/17719543/139577198-3c7995e4-d9ef-4c83-ad57-2608c17305ed.png)

And what they should use is: `www\.example\.(com|net|org)`

Next I have registered `www-example.com` domain and started playing with escalation this. More about it in part 2.

Thanks for reading! You can follow me on [Twitter](https://twitter.com/xvnpw).
