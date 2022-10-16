---
title: "Threat Modeling 101"
date: 2022-10-13T08:59:02+01:00
draft: true
tags: [appsec, threat-modeling]
description: "What is Threat Modeling? There are many definitions, but for me it's a technique to find security problems before implementation happened."
---

What is Threat Modeling? There are many definitions, but for me it's a technique to find security problems **before** implementation happened. And what do we have before implementation, you may ask? Creating a **design**. Design can be as simple as drawing few boxes and arrows among them or complex with multiple diagrams. **Anything is enough** (especially for beginning), it's only a model, it's wrong, but it can be useful. 

{{< figure src="https://user-images.githubusercontent.com/17719543/195526702-922b05ae-f28e-4889-af64-6962c31122d0.png" class="image-center" >}}

This is simple [Data Flow Diagram](https://learn.microsoft.com/en-us/training/modules/tm-create-a-threat-model-using-foundational-data-flow-diagram-elements/). It's not perfect. It's not covering all items. But it can be useful for us. 

## Techniques

Same as with definitions of TM (Threat Modeling), there are multiply techniques of performing it. You can read more about it in Adam Shostack article: [Fast, Cheap + Good: An Unusual Tradeoff Available in Threat Modeling](https://shostack.org/files/papers/Fast-Cheap-and-Good.pdf). I will only take one picture from it:

{{< figure src="https://user-images.githubusercontent.com/17719543/195985325-d1cb494f-5eab-41f6-abf8-69dd834cceeb.png" class="image-center" title="Adam Shostack: Fast, Cheap + Good: An Unusual Tradeoff Available in Threat Modeling" >}}

For me most important observation is that, you need to match technique to skill level of team. Jumping right into most robust will not be easy for both security people and engineering team. Apart of skill level you need also match company culture, team setup, processes and more.

### Rapid Threat Model Prototyping

I could not mention Geoffrey Hill and his [Rapid Threat Model Prototyping](https://github.com/geoffrey-hill-tutamantic/rapid-threat-model-prototyping-docs). In his view, the most important are **data**, and those should be protected at most. He is also focused on **countermeasures** and not threats. I can understand that, if team is not doing anything to stop threats before implementation, fast and easy to perform technique is big plus. There is one caveat in my opinion, this technique is coming with assumed risk appetite of organization. If you want to know more check this [OWASP DevSlop](https://www.youtube.com/watch?v=6eUlRVzcbaU) talk.

### Everyone is doing Threat Modeling

From process perspective I have lately encounter [Redefining Threat Modeling: Security Team Goes on Vacation by Jeevan Singh](https://www.youtube.com/watch?v=kRiXDpq-nd4). Jeevan gave himself goal to train every dev in his company in TM. It was enormous effort, and he is sharing his way.

### STRIDE

[STRIDE](https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats) is threat categorization, invented by Microsoft. It's used to make it easier to talk about threats. 

There are many materials on TM with STRIDE. I personally appreciate those three:

* [Agile Threat Modelling](https://thoughtworksinc.github.io/sensible-security-conversations/) with excellent [workshop guide](https://thoughtworksinc.github.io/sensible-security-conversations/materials/Sensible_Agile_Threat_Modelling_Workshop_Guide.pdf)
* A Guide to Threat Modelling for Developers on Martin Fowler [blog](https://martinfowler.com/articles/agile-threat-modelling.html)
* Adam Shostack [courses](https://shostack.org/training/courses/linkedin) (paid)

#### Spoofing & Tempering

I want to focus only on explaining those two categories of threats to use it later in example:

|       Category       |      Description     | Property violated |
|:-------------------:|:--------------------:|:---|
| 👓 Spoofing | Pretending to be someone or something other than yourself | Authentication |
| ✋ Tampering | 

## Risk Assessment

I will quickly mention one step **before** even doing Threat Modeling. Because not every organization, not every system and not every piece of code need to be covered with TM. Here to plays come Risk Assessment, that will define risk profile and will allow to make prioritization. As introduction to this topic, please check [Anshuman Bhartiya article](https://www.anshumanbhartiya.com/posts/secure-sdlc), especially Rapid Risk Assessment (RRA) section.




