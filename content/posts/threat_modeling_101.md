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
| ✋ Tampering | Malicious data modification | Integrity |

## Motivation (Risk Assessment)

I will quickly mention one step **before** even doing Threat Modeling. Because not every organization, not every system and not every piece of code need to be covered with TM. Here to plays come Risk Assessment, that will define risk profile and will allow to make prioritization. As introduction to this topic, please check [Anshuman Bhartiya article](https://www.anshumanbhartiya.com/posts/secure-sdlc), especially Rapid Risk Assessment (RRA) section.

### 👤 "Human" based Risk Assessment

We as people are doing Risk Assessment every day. Consider an example:

> If you have expensive car you will more think about alarm, GPS, trackers. You will be more interested in news and reports regarding car theft. But when you have something that is not in thieves radar, you will probably skip all those details.

Such intuitive thinking can save us time and focus, but can also harm us. How to **compare** risk of losing a car, with other risks in our lives? Should you buy more sophisticated alarm or pay for extended medical insurance? There is no universal answers here, everyone need to value those risks for themselves. 

This topic is far greater then TM, but I will only mention [PwC report](https://www.pwc.com/us/en/services/consulting/cybersecurity-risk-regulatory/library/cyber-risk-quantification-management.html), that is showing how right approach to risk management can help organizations.

For TM, most important in my opinion is that engineers and security team should be aware of **risk management**. There is nothing worse than security people standing on their heads to do TM, and everyone else thinking that we don't need that 👿

Proper **SecDevOps** can only start with accepting fact that you have this "🚗 expensive car" that everyone want to take from you (which is not far from true in current cybersecurity landscape).

## Goals

I have already mentioned main goal of TM: "To find security problems before implementation happened". It's good opening sentence for TM training, but it's not enough for having long lasting program. 

Let's have simple data flow diagram, representing web application:

{{< figure src="https://user-images.githubusercontent.com/17719543/196056425-c2bd34e2-aed5-4ba1-9444-ec2df1331860.png" class="image-center" >}}

We have Web Application, so will can enumerate all threats including XSS, security headers, maybe file upload, injections. You see where this is going. We can enumerate all possible threats and number of those is enormous. There is second thing that is outstanding here. Our list of threats is almost the same for every other Web Application.

We have already too many threats to deal with in 1h session with developers, and we even didn't tackle deployment and CI/CD 💩.

To get things sort out let's divide TM by scopes with different goals:

| Scope|🎯 Goal|
|:----:|:--------------------:|
| System Architecture | We look on system from high level. We focus on threats that can affect system as a whole. We can address compliance **security requirements** that would be hard to change later. Typical problems to solve at this point: **authentication**, WAF, DDoS protection, logging, availability. |
| Reference microservice (component) architecture | Our objective is to find threats that are affecting design of microservice (component) and are not enough mitigated at System Architecture level. Security people can help engineers take right choices here, e.g. on tech stack to use. It's a big plus to create a list of countermeasures that are later easy to pick for developers of particular component. |
| Feature (User Story) | This is about working very closely with developers. It can be 1h meeting each Sprint, as part of process. Goal would be to find and address security issues in technical and business field. In terms of Scrum we can create additional acceptance criteria, perform spikes, create tech dept or new security feature stories. For significant threats "abuse stories" can be used. To not get **overloaded with threats** team should use System Architecture and Reference microservice (component) architecture threat models and pick up right mitigations based on those. |
| Deployment | It's easier to focus on infrastructure threats on dedicated deployment view of system. |
| CI/CD | This is yet another special view. It includes both infrastructure and processes. Same as deployment, goal is to address threats on separated view. | 

This is how connection among scopes can look like:

{{< figure src="https://user-images.githubusercontent.com/17719543/196274640-f08775b9-fa03-4ec0-9325-5dd913e7cdef.png" class="image-center no-border" >}}

### Example of connection among different scopes

To show you how powerful can be building User Story TM, based on high level TMs, I will use example:

{{< figure src="https://user-images.githubusercontent.com/17719543/196270205-f41d2c3e-af52-42c2-948a-004d7760eba6.png" class="image-center no-border" >}}

In this User Story developers need to implement new API. I have listed 3 threats:

| Threat | Mitigation |
|:------:|:----------:|
|No AuthN | Authentication is solved in this particular case on API Gateway level and don't need to be solved in component directly |
|Weak credentials | User accounts are created in IdP. There is configured policy regarding user passwords |
|No rate limiting | Rate limiting issue is also solved in API Gateway |

As you can see those 3 threats, which can be easily found in any STRIDE learning materials, are **not essential** for this system. They are already mitigated!

Very important part of TM program would be maintaining high level TMs and making sure that all devs knows them. It should be part of TM training!
