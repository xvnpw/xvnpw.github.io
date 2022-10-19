---
title: "Threat Modeling 101"
date: 2022-10-19T18:59:02+01:00
draft: false
tags: [appsec, threat-modeling]
description: "What is Threat Modeling? First of all, it’s just thinking about threats. We all do it, every day 😃 How someone could break into my house? But wait a second. How do you know that you need to protect your house in the first place? Maybe you don’t have a house, or maybe you don’t have money right now to buy deterrents. Or maybe your family thinks you are a bit paranoid? 😕"
---

What is Threat Modeling? First of all, it's just thinking about threats. We all do it, every day 😃 "How someone could break into my house?" But wait a second. How do you know that you need to protect your house in the first place? Maybe you don't have a house, or maybe you don't have money right now to buy deterrents. Or maybe your family thinks you are a bit paranoid? 😕 Just before doing Threat Modeling you need to start doing Risk Management, to assess what is your risk appetite and profile. More on that later. 

Ok, so what is Threat Modeling in *cyber security*? There are many definitions, with the most accurate from [Threat Modeling Manifesto](https://www.threatmodelingmanifesto.org/):

> Threat modeling is analyzing representations of a system to highlight concerns about security and privacy characteristics.

But at the same time, it is extremely general and abstract. For me, and I mostly work with developers, it's a technique to find security problems **before** implementation happened. And what do we have before implementation, you may ask? Creating a **design**. Design can be as simple as drawing a few boxes and arrows among them or complex with multiple diagrams. **Anything is enough** (especially for the beginning), it's only a model, it's wrong, but it can be useful.

{{< figure src="https://user-images.githubusercontent.com/17719543/195526702-922b05ae-f28e-4889-af64-6962c31122d0.png" class="image-center" >}}

This is a simple [Data Flow Diagram](https://learn.microsoft.com/en-us/training/modules/tm-create-a-threat-model-using-foundational-data-flow-diagram-elements/). It's not perfect. It's not covering all items. But it can be useful for us. 

Getting back to Threat Modeling Manifesto for a bit. I like its *Values* and *Principles*, and I will talk more about those later. For now, I want to focus on: **techniques**, **motivation**, and **goals**.

## Techniques

You may already know that, there are multiple techniques of performing Threat Modeling. Adam Shostack's article: [Fast, Cheap + Good: An Unusual Tradeoff Available in Threat Modeling](https://shostack.org/files/papers/Fast-Cheap-and-Good.pdf) is providing decent overview of those. I will only take one picture from it:

{{< figure src="https://user-images.githubusercontent.com/17719543/195985325-d1cb494f-5eab-41f6-abf8-69dd834cceeb.png" class="image-center" title="Adam Shostack: Fast, Cheap + Good: An Unusual Tradeoff Available in Threat Modeling" >}}

For me, the most important observation is that you need to match technique to skill level of team. Jumping right into the most robust will not be easy for both security people and engineering team. Apart from skill level you also need to match company culture, team setup, processes, and more.

❕My advice is to start with something simple, to set up right foundation and build upon that. On the other hand, engineers don't want to spend time on meaningless activities. Be precise about the goals of your TM program.

### Rapid Threat Model Prototyping

I could not mention Geoffrey Hill and his [Rapid Threat Model Prototyping](https://github.com/geoffrey-hill-tutamantic/rapid-threat-model-prototyping-docs). In his view, the most important is **data**, and those should be protected at most. He is also focused on **countermeasures** and not threats. I can understand that, if team is not doing anything to stop threats before implementation, fast and easy to perform technique is big plus. There is one caveat in my opinion, this technique is coming with assumed risk assessment. If you want to know more check this [OWASP DevSlop](https://www.youtube.com/watch?v=6eUlRVzcbaU) talk.

### Every developer is doing Threat Modeling

From a process perspective, I have lately encountered [Redefining Threat Modeling: Security Team Goes on Vacation by Jeevan Singh](https://www.youtube.com/watch?v=kRiXDpq-nd4). Jeevan gave himself goal to train every dev in his company in TM. It was an enormous effort, and he is sharing his way.

### STRIDE

[STRIDE](https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats) is threat categorization, invented by Microsoft. It's used to make it easier to talk about threats. 

There are many materials on TM with STRIDE. I appreciate those three:

* [Agile Threat Modelling](https://thoughtworksinc.github.io/sensible-security-conversations/) with excellent [workshop guide](https://thoughtworksinc.github.io/sensible-security-conversations/materials/Sensible_Agile_Threat_Modelling_Workshop_Guide.pdf)
* A Guide to Threat Modelling for Developers on Martin Fowler's [blog](https://martinfowler.com/articles/agile-threat-modelling.html) - I have found it useful because developers tend to trust content coming from Martin's blog 😆
* Adam Shostack [courses](https://shostack.org/training/courses/linkedin) (paid)

## Motivation (Risk Management)

I will quickly mention one step **before** even doing Threat Modeling. Because not every organization, not every system, and not every piece of code needs to be covered with TM. Here to plays comes Risk Management, which will define risk profile and will allow making prioritization. As introduction to this topic, please check [Anshuman Bhartiya article](https://www.anshumanbhartiya.com/posts/secure-sdlc), especially Rapid Risk Assessment (RRA) section.

If your Org is not having strong Risk Management process, probably your Senior Engineers and Managers are having some kind of informal process or understanding. Try to get it from them. Good idea is to ask them "what if..." questions with doom scenarios and see their responses. At some point, they will confess what really is scary for them and what is not. This is sometimes hard truth for security engineers.

### 👤 Everyday Risk Assessment

We are doing Risk Assessments every day. Consider an example:

> If you have expensive car you will more think about alarm, GPS, and trackers. You will be more interested in news and reports regarding car theft. But when you have something that is not on thieves' radar, you will probably skip all those details.

Such intuitive thinking can save us time and focus, but can also harm us. How to **compare** risk of losing a car, with other risks in our lives? Should you buy more sophisticated alarm or pay for extended medical insurance? There are no universal answers here, everyone needs to value those risks for themselves. 

❕For TM, the most important in my opinion is that engineers and security team should be aware of **risk management**. There is nothing worse than security people standing on their heads to do TM, and everyone else thinking that we don't need that 👿

❕Proper **SecDevOps** can only start with accepting fact that you have this "🚗 expensive car" that everyone wants to take from you (which is not far from true in current cybersecurity landscape).

## Goals

I have already mentioned main goal of TM: "To find security problems before implementation happened". It's good opening sentence for TM training, but it's not enough for having long-lasting program. 

Below we have simple data flow diagram, representing web application:

{{< figure src="https://user-images.githubusercontent.com/17719543/196056425-c2bd34e2-aed5-4ba1-9444-ec2df1331860.png" class="image-center" >}}

Let's enumerate all threats for Web Application: XSS, security headers, maybe file upload, injections, etc. You see where this is going. We can enumerate all possible threats and number of those is enormous. There is second thing that is outstanding here. Our list of threats is almost the same for every other Web Application.

We have already too many threats to deal with in 1h session with developers, and we even didn't tackle deployment and CI/CD 💩.

To get things sorted out let's divide TM by scopes with different goals:

| Scope|🎯 Goal|
|:----:|:--------------------:|
| System Architecture | We look at system from a high level. We focus on threats that can affect system as a whole. We can address compliance **security requirements**. Typical problems to solve at this point: **authentication**, WAF, DDoS protection, logging and availability. |
| Reference component architecture (e.g. microservice) | Our objective is to find threats that are affecting reference design of component and are not enough mitigated at System Architecture level. Security people can help engineers take right choices here, e.g. on tech stack to use. It's a big plus to create a list of countermeasures that are later easy to pick for developers of particular components. |
| Component architecture (e.g. microservice) | In previous point, there was assumption on existence of some kind of reference architecture (formal or informal). In this, we can cover threats that are for custom components. It's best to engage early in design process and talk with team frequently. Amount of time spend on TM will be probably related to time that team need to spend on overall design. |
| Feature (User Story) | This is about working very closely with developers. It can be 1h meeting each Sprint, as part of process. Goal would be to find and address security issues in technical and business fields. In terms of Scrum, we can create additional acceptance criteria, perform spikes, create tech dept or new security feature stories. For significant threats "abuse stories" can be used. To not get **overloaded with threats** team should use System Architecture and Reference component architecture TMs and pick up right mitigations based on those. |
| Deployment | It's easier to focus on infrastructure threats on dedicated deployment view of system. |
| CI/CD | This is yet another special view. It includes both infrastructure and processes. Same to deployment, goal is to address threats on separate view. | 

It all can be represented in one picture:

{{< figure src="https://user-images.githubusercontent.com/17719543/196274640-f08775b9-fa03-4ec0-9325-5dd913e7cdef.png" class="image-center no-border" >}}

❕It's worth refining high level TMs continuously with key stakeholders, to be up to date with design and changing threats landscape. 

### Example of connection among different scopes

To show you how powerful can be building User Story TM, based on high level TMs, I will use example:

{{< figure src="https://user-images.githubusercontent.com/17719543/196270205-f41d2c3e-af52-42c2-948a-004d7760eba6.png" class="image-center no-border" >}}

In this User Story developers need to implement new API. I have listed 3 threats:

| Threat | Mitigation |
|:------:|:----------:|
|No AuthN | Authentication is solved in this particular case on API Gateway level and doesn't need to be solved in component directly. But component needs to verify if authentication was performed (check for user context). |
|Weak credentials | User accounts are created in IdP. There is configured policy regarding user passwords. |
|No rate limiting | Rate limiting issue is also solved in API Gateway. Developers can override default by creating PR to platform repository. |

Those 3 threats, which can be easily found in any STRIDE learning materials, are **not essential** for this system. They are already mitigated! Only for "No AuthN" developers need to code something.

❕Very important part of TM program would be maintaining high level TMs and making sure that all devs know them. It should be part of TM training!

## Summary

Number of ideas, techniques, and opinions on Threat Modeling is huge and can easily get you overloaded 😶. This is how I remembered about Threat Modeling Manifesto and its *Values* and *Principles*. I think it's a gem 💎, and I will only quote part of it:

> Values:
> * (...)
> * Doing threat modeling over talking about it.
> * Continuous refinement over a single delivery.
>
> Principles:
> * The best use of threat modeling is to improve the security and privacy of a system through early and frequent analysis.
> * (...)

❕Apart from that, I would underline two ideas:

* Risk Assessment - to filter out only those items that need Threat Modeling.
* Scopes - for each scope we can have different techniques. High level scopes can be more detailed and analyzed by security team and then used by developers as input for their TM sessions. 

During the writing of this article, I was wondering if it's really needed. There are so many good resources on Threat Modeling out there. Hope I have highlighted important aspects of it and connected some dots. If you have any comments or feedback, you are welcome to write to me on [Twitter](https://twitter.com/xvnpw).