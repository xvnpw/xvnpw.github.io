---
title: "Threat Modeling 101"
date: 2022-10-13T08:59:02+01:00
draft: true
tags: [appsec, threat-modeling]
description: "What is Threat Modeling? First of all it’s, just thinking about threats. We all do it, every day 😃 How someone could break into my house? But wait a second. How do you know that you need to protected your house in the first place? Maybe you don’t have house, maybe you don’t have money right now to buy protections."
---

What is Threat Modeling? First of all it's, just thinking about threats. We all do it, every day 😃 How someone could break into my house? But wait a second. How do you know that you need to protected your house in the first place? Maybe you don't have house, maybe you don't have money right now to buy protections. Or maybe your family thinks you are a bit paranoid? 😕 Just before doing Threat Modeling you need to start doing Risk Management, to assess what is your risk appetite and profile. More on that later. 

Ok, so what is Threat Modeling in *cyber security*? There are many definitions, but for me it's a technique to find security problems **before** implementation happened. And what do we have before implementation, you may ask? Creating a **design**. Design can be as simple as drawing few boxes and arrows among them or complex with multiple diagrams. **Anything is enough** (especially for beginning), it's only a model, it's wrong, but it can be useful. 

{{< figure src="https://user-images.githubusercontent.com/17719543/195526702-922b05ae-f28e-4889-af64-6962c31122d0.png" class="image-center" >}}

This is simple [Data Flow Diagram](https://learn.microsoft.com/en-us/training/modules/tm-create-a-threat-model-using-foundational-data-flow-diagram-elements/). It's not perfect. It's not covering all items. But it can be useful for us. 

## Techniques

Same as with definitions of TM (Threat Modeling), there are multiply techniques of performing it. You can read more about it in Adam Shostack article: [Fast, Cheap + Good: An Unusual Tradeoff Available in Threat Modeling](https://shostack.org/files/papers/Fast-Cheap-and-Good.pdf). I will only take one picture from it:

{{< figure src="https://user-images.githubusercontent.com/17719543/195985325-d1cb494f-5eab-41f6-abf8-69dd834cceeb.png" class="image-center" title="Adam Shostack: Fast, Cheap + Good: An Unusual Tradeoff Available in Threat Modeling" >}}

For me most important observation is that, you need to match technique to skill level of team. Jumping right into most robust will not be easy for both security people and engineering team. Apart of skill level you need also match company culture, team setup, processes and more.

My advice is to start with something simple, to setup right foundation and build upon that. On the other hand, engineers don't want to spend time on meaningless activities, so be precise on goals of your TM program.

### Rapid Threat Model Prototyping

I could not mention Geoffrey Hill and his [Rapid Threat Model Prototyping](https://github.com/geoffrey-hill-tutamantic/rapid-threat-model-prototyping-docs). In his view, the most important are **data**, and those should be protected at most. He is also focused on **countermeasures** and not threats. I can understand that, if team is not doing anything to stop threats before implementation, fast and easy to perform technique is big plus. There is one caveat in my opinion, this technique is coming with assumed risk appetite of organization. If you want to know more check this [OWASP DevSlop](https://www.youtube.com/watch?v=6eUlRVzcbaU) talk.

### Every developer is doing Threat Modeling

From process perspective, I have lately encounter [Redefining Threat Modeling: Security Team Goes on Vacation by Jeevan Singh](https://www.youtube.com/watch?v=kRiXDpq-nd4). Jeevan gave himself goal to train every dev in his company in TM. It was enormous effort, and he is sharing his way.

### STRIDE

[STRIDE](https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats) is threat categorization, invented by Microsoft. It's used to make it easier to talk about threats. 

There are many materials on TM with STRIDE. I personally appreciate those three:

* [Agile Threat Modelling](https://thoughtworksinc.github.io/sensible-security-conversations/) with excellent [workshop guide](https://thoughtworksinc.github.io/sensible-security-conversations/materials/Sensible_Agile_Threat_Modelling_Workshop_Guide.pdf)
* A Guide to Threat Modelling for Developers on Martin Fowler [blog](https://martinfowler.com/articles/agile-threat-modelling.html) - I have found it useful because developers has tendency to trust content coming from Martin blog 😆
* Adam Shostack [courses](https://shostack.org/training/courses/linkedin) (paid)

#### Spoofing & Tampering

I want to focus only on explaining those two categories of threats to use it later in example:

|       Category       |      Description     | Property violated 
|:-------------------:|:--------------------:|:---|
| 👓 Spoofing | Pretending to be someone or something other than yourself | Authentication 
| ✋ Tampering | | Integrity |

## Risk Assessment

I will quickly mention one step **before** even doing Threat Modeling. Because not every organization, not every system and not every piece of code need to be covered with TM. Here to plays come Risk Assessment, that will define risk profile and will allow to make prioritization. As introduction to this topic, please check [Anshuman Bhartiya article](https://www.anshumanbhartiya.com/posts/secure-sdlc), especially Rapid Risk Assessment (RRA) section.

### 👤 "Human" based Risk Assessment

We as people are doing Risk Assessment every day. Consider an example:

> If you have expensive car you will more think about alarm, GPS, trackers. You will be more interested in news and reports regarding car theft. But when you have something that is not in thieves radar, you will probably skip all that details.

Such intuitive thinking can save us time and focus, but can also harm us. 

This topic is far greater then TM, but I will only mention [PwC report](https://www.pwc.com/us/en/services/consulting/cybersecurity-risk-regulatory/library/cyber-risk-quantification-management.html), that is showing how right approach to risk management can help organizations.

For TM, most important in my opinion is that engineers and security people are aware of **risk biases**. There is nothing worse than security people standing on their heads to do TM, and everyone else thinking that we don't need that 👿

Proper **SecDevOps** can only start with accepting fact that you have this "🚗 expensive car" that everyone want to take from you (which is not far from true in current cybersecurity landscape).

#### Every 👤 Human is doing Threat Modeling

How to secure house? Should I take this route or that route? Those questions are popping in our heads every day. We are constantly thinking about something that can threaten our lives or assets. But can you **easily** prioritize how to spend your money on countermeasures? Do you need guards with dogs or only cheap wifi camera. I'm finding this prioritization thinking very hard in my private life. 

This is just another reason to start with Risk Assessment / Management before any TM.


