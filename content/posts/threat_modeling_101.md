---
title: "Threat Modeling 101"
date: 2022-10-13T08:59:02+01:00
draft: true
tags: [appsec, threat-modeling]
description: "What is Threat Modeling? First of all, it’s just thinking about threats. We all do it, every day 😃 How someone could break into my house? But wait a second. How do you know that you need to protected your house in the first place? Maybe you don’t have house, maybe you don’t have money right now to buy protections."
---

What is Threat Modeling? First of all, it's just thinking about threats. We all do it, every day 😃 "How someone could break into my house?" But wait a second. How do you know that you need to protected your house in the first place? Maybe you don't have house, maybe you don't have money right now to buy protections. Or maybe your family thinks you are a bit paranoid? 😕 Just before doing Threat Modeling you need to start doing Risk Management, to assess what is your risk appetite and profile. More on that later. 

Ok, so what is Threat Modeling in *cyber security*? There are many definitions, but for me it's a technique to find security problems **before** implementation happened. And what do we have before implementation, you may ask? Creating a **design**. Design can be as simple as drawing few boxes and arrows among them or complex with multiple diagrams. **Anything is enough** (especially for beginning), it's only a model, it's wrong, but it can be useful. 

{{< figure src="https://user-images.githubusercontent.com/17719543/195526702-922b05ae-f28e-4889-af64-6962c31122d0.png" class="image-center" >}}

This is simple [Data Flow Diagram](https://learn.microsoft.com/en-us/training/modules/tm-create-a-threat-model-using-foundational-data-flow-diagram-elements/). It's not perfect. It's not covering all items. But it can be useful for us. 

In this article, I will talk about three things regarding Threat Modeling: **techniques**, **motivation** and **goals**.

## Techniques

Same as with definitions of TM (Threat Modeling), there are multiply techniques of performing it. You can read more about it in Adam Shostack article: [Fast, Cheap + Good: An Unusual Tradeoff Available in Threat Modeling](https://shostack.org/files/papers/Fast-Cheap-and-Good.pdf). I will only take one picture from it:

{{< figure src="https://user-images.githubusercontent.com/17719543/195985325-d1cb494f-5eab-41f6-abf8-69dd834cceeb.png" class="image-center" title="Adam Shostack: Fast, Cheap + Good: An Unusual Tradeoff Available in Threat Modeling" >}}

For me most important observation is that, you need to match technique to skill level of team. Jumping right into most robust will not be easy for both security people and engineering team. Apart of skill level you also need match company culture, team setup, processes and more.

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

## Motivation (Risk Assessment)

I will quickly mention one step **before** even doing Threat Modeling. Because not every organization, not every system and not every piece of code need to be covered with TM. Here to plays come Risk Assessment, that will define risk profile and will allow to make prioritization. As introduction to this topic, please check [Anshuman Bhartiya article](https://www.anshumanbhartiya.com/posts/secure-sdlc), especially Rapid Risk Assessment (RRA) section.

### 👤 "Human" based Risk Assessment

We as people are doing Risk Assessment every day. Consider an example:

> If you have expensive car you will more think about alarm, GPS, trackers. You will be more interested in news and reports regarding car theft. But when you have something that is not in thieves radar, you will probably skip all those details.

Such intuitive thinking can save us time and focus, but can also harm us. How to **compare** risk of losing a car, with other risks in our lives? Should you buy more sophisticated alarm or pay for extended medical insurance? There is no universal answers here, everyone need to value those risks for themselves. 

This topic is far greater then TM, but I will only mention [PwC report](https://www.pwc.com/us/en/services/consulting/cybersecurity-risk-regulatory/library/cyber-risk-quantification-management.html), that is showing how right approach to risk management can help organizations.

For TM, most important in my opinion is that engineers and security people should be aware of **risk management**. There is nothing worse than security people standing on their heads to do TM, and everyone else thinking that we don't need that 👿

Proper **SecDevOps** can only start with accepting fact that you have this "🚗 expensive car" that everyone want to take from you (which is not far from true in current cybersecurity landscape).

## Goals

I have already mention main goal of TM: "To find security problems before implementation happened". It's good opening sentence for TM training, but it's not enough for having long lasting program. 

Let's have simple data flow diagram, representing web application:

{{< figure src="https://user-images.githubusercontent.com/17719543/196056425-c2bd34e2-aed5-4ba1-9444-ec2df1331860.png" class="image-center" >}}

We have Web Application, so will can enumerate all threats including XSS, security headers, maybe file upload, injections code and command. We have SQL database so probably SQL injections. You see where this is going. We can enumerate all possible threats and number of those is enormous. There is second thing that is outstanding here. Our list of threats is almost the same for every other Web Application and what is more interesting it's the same as any **DAST tool checklist**. 

We have already too many threats to deal with in 1h session with developers, and we even didn't tackle deployment and CI/CD 💩.

To get things sort out let's divide TM by scopes with different goals:

| Scope|Description|Goal|Typical countermeasures|
|:----:|:--------------------:|:---|:----|
| System Architecture | We are looking here on whole system from high level | Find threats that can affect system as a whole, e.g. DoS/DDoS, various injections | DDoS protection, rate limiting, WAF, central authentication on API gateway
| Reference microservice architecture | Engineers probably will try to make as many similar technology choices as possible to simplify development of microservices | Find threats that are affecting particular design of microservices and are not enough mitigated at System Architecture level | Hardened libraries (e.g. database access, XML processing, sending http requests), SAST rules (semgrep is good for that)
| Feature (Use Story) | Here we have this 1h meeting with developers each Sprint | Add additional acceptance criteria, perform spikes, create tech dept or new security feature implementation. For significant threats "abuse stories" can be created | 
| Deployment | It can be included in system architecture, but it's worth to create deployment view of resource | Find threats for infrastructure | IaC scanning tools (e.g. tfsec), benchmarking tools 
| CI/CD | Explicit view on how development process works and how artifacts are landing in the end in production | CI/CD environments are very important for organization to work and deliver value. There are also target for attacker as there need to be easy to work with and have high privileges | IaC scanning tools if possible, 




