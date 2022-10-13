---
title: "Threat Modeling 101 part #1"
date: 2022-10-13T08:59:02+01:00
draft: true
tags: [appsec, threat-modeling]
description: ""
---

What is Threat Modeling? There are many definitions, but for me it's a technique to find security problems before implementation happend. What is before implementation you may ask? Creating a **design**. Sometimes it's very simple as drawoing few boxes and arrows among them. **It's enought**, it's only a model, it's wrong, but it can be usefull. 

{{< figure src="https://user-images.githubusercontent.com/17719543/195526702-922b05ae-f28e-4889-af64-6962c31122d0.png" class="image-center" >}}

This is simple [Data Flow Diagram](https://learn.microsoft.com/en-us/training/modules/tm-create-a-threat-model-using-foundational-data-flow-diagram-elements/). It's not perfect. It's not covering all information. But it can be usefull. 

## STRIDE

[STRIDE](https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats) is threat catorization, invented by Microsoft. It's used to make it easier to talk about threats.

### Spoofing

In this article I would like to focus on Spoofing category.

|       Category       |      Description     | Propery violated |
|:-------------------:|:--------------------:|:---|
|      :eyeglasses: Spoofing      |     Pretending to be someone or something other than yourself    | Authentication |













If you want to go deeper into Threat Modeling and STRIDE, I recommend excelent learning materials from Adam Shostack: https://shostack.org/training/courses/linkedin.

