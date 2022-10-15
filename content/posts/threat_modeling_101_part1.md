---
title: "Threat Modeling 101 part #1"
date: 2022-10-13T08:59:02+01:00
draft: true
tags: [appsec, threat-modeling]
description: ""
---

What is Threat Modeling? There are many definitions, but for me it's a technique to find security problems before implementation happened. And what we have before implementation, you may ask? Creating a **design**. Design can be as simple as drawing few boxes and arrows among them or complex with multiple diagrams. **Anything is enough** (especially for beginning), it's only a model, it's wrong, but it can be useful. 

{{< figure src="https://user-images.githubusercontent.com/17719543/195526702-922b05ae-f28e-4889-af64-6962c31122d0.png" class="image-center" >}}

This is simple [Data Flow Diagram](https://learn.microsoft.com/en-us/training/modules/tm-create-a-threat-model-using-foundational-data-flow-diagram-elements/). It's not perfect. It's not covering all items. But it can be useful for us. 

## Techniques

Same as with definitions of TM (Threat Modeling), there are multiply techniques of performing it. You can read more about it in Adam Shostack article: [Fast, Cheap + Good: An Unusual Tradeoff Available in Threat Modeling](https://shostack.org/files/papers/Fast-Cheap-and-Good.pdf). I will only take one picture from it:

{{< figure src="https://user-images.githubusercontent.com/17719543/195985325-d1cb494f-5eab-41f6-abf8-69dd834cceeb.png" class="image-center" title="Adam Shostack: Fast, Cheap + Good: An Unusual Tradeoff Available in Threat Modeling" >}}

For me most important observation is that, you need to match technique to skill level of team. Jumping right away to most robust will not be easy for both security people and engineering team. Apart of skill level you need also match company culture, team setup and more.

## STRIDE

[STRIDE](https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats) is threat categorization, invented by Microsoft. It's used to make it easier to talk about threats. 

### Spoofing

I want to talk about :eyeglasses: Spoofing, to give you right example how TM can help in finding problems.

|       Category       |      Description     | Property violated |
|:-------------------:|:--------------------:|:---|
| :eyeglasses: Spoofing | Pretending to be someone or something other than yourself | Authentication |













If you want to go deeper into Threat Modeling and STRIDE, I recommend excelent learning materials from Adam Shostack: https://shostack.org/training/courses/linkedin.

