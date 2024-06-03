---
title: "Threat Modelling with Fabric Framework"
date: 2024-06-03T16:59:02+01:00
draft: true
tags: [security, threat-modeling, fabric, llm, gpt, claude]
description: ""
---

[Fabric](https://github.com/danielmiessler/fabric) is a framework that puts AI at your fingertips. Instead of diving into chat interfaces (e.g., ChatGPT) or writing custom programs that consume APIs, you can create prompts as markdown text and receive output in markdown. Fabric also maintains a database of prompts called [patterns](https://github.com/danielmiessler/fabric/tree/main/patterns).

{{< figure src="https://github.com/xvnpw/xvnpw.github.io/assets/17719543/4fe91d36-3736-4cbf-9835-edfa3943116e" class="image-center" width=300 >}}

With new pattern: [create_stride_threat_model](https://github.com/danielmiessler/fabric/blob/main/patterns/create_stride_threat_model/system.md) you will be able easily create threat models. Let's dive deeper how to use this new pattern, and how good are results.

## New pattern in action

From my [previous post]({{< ref "/posts/leveraging-llms-for-threat-modelling-claude-3-vs-gpt-4.md" >}}) you can learn about my experiment on using Large Language Models for threat modelling. In this article, we will use architecture document of the fictional project called "AI Nutrition-Pro" as input:

```
# Get Fabric installed - https://github.com/danielmiessler/fabric
$ wget https://raw.githubusercontent.com/xvnpw/fabric-stride-threat-model/main/INPUT.md
$ cat INPUT.md | fabric --pattern create_stride_threat_model -m claude-3-opus-20240229
```

In this case, I use `claude-3-opus-20240229` model, in time of writing this, Claude 3 Opus represents best model for threat modelling in my opinion.

## Beginning of the story - creating new pattern

Before we jump into results, I needed first to create new pattern in fabric. In order to do that, I used 3 baseline threat models created for the same input file. This allowed me to compare results with already existing and tested solutions:

| Threat Model | Link | LLM Model |
| --- | --- | --- |
| Baseline threat model created by Fabric using the [create_threat_scenarios](https://github.com/danielmiessler/fabric/blob/main/patterns/create_threat_scenarios/system.md) pattern | [baseline_threat_scenarios.md](https://github.com/xvnpw/fabric-stride-threat-model/blob/main/baseline_threat_scenarios.md) | Claude 3 Opus |
| Baseline threat model created by [STRIDE GPT](https://github.com/mrwadams/stride-gpt) | [baseline_stride_gpt.md](https://github.com/xvnpw/fabric-stride-threat-model/blob/main/baseline_stride_gpt.md) | GPT-4o |
| Baseline threat model from my [ai-threat-modeling-action](https://github.com/xvnpw/ai-threat-modeling-action) | [baseline_threat_modeling_action.md](https://github.com/xvnpw/fabric-stride-threat-model/blob/main/baseline_threat_modeling_action.md) | Claude 3 Opus |

Those 3 solutions are unique in their approach:
- `create_threat_scenarios` is most generic prompt, and I was very surprised to see concrete threats and mitigations there. 
- `STRIDE GPT` gives a different perspective. Before you can run threat generation, you need to fill few questions. I picked application type as **cloud**, sensitivity of data as **confidential**, said **yes** for internet facing and pick **Basic** auth. Results are very interesting. Threats are not grouped for specific data flows or components. Instead they are listed as scenarios. Something similar to `create_threat_scenarios` approach. 
- finally my previous work - `ai-threat-modeling-action`. I choose different approach, I split design by data flows and components. AI has more work to do (and it takes several round trips to model). Advantage of this is to have threat model using STRIDE per element methodology as I would imagine it to be. Disadvantage is that AI currently sometimes misunderstand component, element or data flow. You cannot find any reference to AWS or cloud in case of this threat model. In both previous threat models, you can find some threats for AWS ECS or RDS.

Considering that already existing pattern `create_threat_scenarios` is not based on STRIDE and is listing threats without distinction on components and data flows, I decided to use STRIDE per element approach. This way, fabric will have two patterns on threat modelling, but different.

## Interrogating AI

Interrogation - it's best way to describe my prompt. You may already notice it in `ai-threat-modeling-action`, there are columns: _Explanation_ or _How threat is already mitigated in architecture_. Why to do that? There are few points here:

- LLMs are generating next token based on previous already generated - by having more details, I hope for having better output
- LLMs consider semantic of things - by grouping and relating ideas in multi-dimensional space - something like clustering - getting back to first point, more related tokens, hope for better output
- It gives a view on how LLM "understands" input - it gives possibility to get back and improve design documents

### Output format

I organized output as markdown with following sections:

```markdown
# ASSETS

Section to determine what data or assets need protection

# TRUST BOUNDARIES

Section to identify and list all trust boundaries. 
Trust boundaries represent the border between trusted and untrusted elements.

# DATA FLOWS

Section to identify and list all data flows between components. 
Data flow is interaction between two components. Mark data flows crossing trust boundaries.

# THREAT MODEL

Section with table of STRIDE per element threats. 
Prioritize threats by likelihood and potential impact.

Table with columns:
- THREAT ID 
- COMPONENT NAME
- THREAT NAME
- STRIDE CATEGORY
- WHY APPLICABLE - why this threat is important for component in context of input.
- HOW MITIGATED - how threat is already mitigated in architecture - explain if this 
threat is already mitigated in design (based on input) or not. Give reference to input.
- MITIGATION - provide mitigation that can be applied for this threat. It should be 
detailed and related to input.
- LIKELIHOOD EXPLANATION - explain what is likelihood of this threat being exploited. 
- IMPACT EXPLANATION - explain impact of this threat being exploited. 
- RISK SEVERITY - risk severity of threat being exploited. Based it on LIKELIHOOD and IMPACT. 
Give value, e.g.: low, medium, high, critical.

# QUESTIONS & ASSUMPTIONS

Section to list questions that you have and the default assumptions regarding THREAT MODEL.
```

## Comparison of result to baselines

Let's compare in details how new pattern differs from baselines:

| Different | To | Comment |
| --- | --- | --- |
| Lower number of threats | All baselines | This is obvious difference. All baselines have more threats. **Why?** Prompt is very much focused on "likelihood", "impact", "what's worth defending", no "fantastical concerns" (btw. credits comes to Daniel Miessler). I can request to generate more threats by explicitly forcing it to "generate at least 10 threats" or "create at least 10 rows in threats table", but those threats are no high priority (for my opinion) anymore. I cannot stop myself from referring this to Isaac Asimov Robot Series of books. Robots there behaved in similar way, and skilled prompting was something like art.
| No cloud threats | `create_threat_scenarios` and STRIDE GPT | I mentioned this before. I think, STRIDE per element is a bit old methodology and not mentioned in publications (=training data) in case of cloud. |
| Important and actionable threats for startup | All baselines | I got only **5 threats** generated by new pattern, but those are very important and you need to act if you were startup. Especially in case of STRIDE GPT and `ai-threat-modeling-action` you can get a feeling of mature enterprise and not startup. 
| Interrogation | `create_threat_scenarios` and STRIDE GPT | I extended description on why, how, and what, from those in `ai-threat-modeling-action`. In my opinion it's important to get more output. This can help us define input and understand why we got certain threats.

## Threat Models

It's time to check results of my new pattern. To get things more interesting, I run it for 3 different LLM models:

- [Threat model](https://github.com/xvnpw/fabric-stride-threat-model/blob/main/threat_model_claude_3_opus.md) created using Claude 3 Opus
- [Threat model](https://github.com/xvnpw/fabric-stride-threat-model/blob/main/threat_model_gpt_4o.md) created using GPT-4o
- [Threat model](https://github.com/xvnpw/fabric-stride-threat-model/blob/main/threat_model_gemini_1.5_pro.md) created using Gemini-1.5 Pro Latest

In my opinion, Claude 3 Opus is best and GPT-4o second. There is noticed gap between those two and Gemini 1.5 Pro (which is worst).

### Assets section

```markdown
# ASSETS

The following data and assets need protection in the AI Nutrition-Pro system:

1. Dietitians' content samples stored in the API database
2. Request and response data related to LLM interactions stored in the API database
3. Control plane data (tenant information, billing data, configuration) stored in the Control Plane Database
4. API keys used for authentication of Meal Planner applications
5. Access credentials for the ChatGPT-3.5 LLM service
```

It's correct.

### Trust boundaries section

```markdown
# TRUST BOUNDARIES

The following trust boundaries are identified in the AI Nutrition-Pro system:

1. Between the Meal Planner application and the API Gateway
2. Between the API Gateway and the API Application
3. Between the API Application and the ChatGPT-3.5 LLM service
4. Between the Web Control Plane and the Control Plane Database
5. Between the API Application and the API database
```

Trust boundaries are of course subjective, but in this case, I would argue that AI did it **wrong**. It's something that we might need better prompt or be more explicit in input document.

### Data flows section

```markdown
# DATA FLOWS

The following data flows are identified in the AI Nutrition-Pro system:

1. Meal Planner application uploads dietitians' content samples to AI Nutrition-Pro via API Gateway (crosses trust boundary)
2. Meal Planner application fetches AI-generated results from AI Nutrition-Pro via API Gateway (crosses trust boundary)
3. API Gateway forwards requests to API Application for processing (crosses trust boundary)
4. API Application sends requests to ChatGPT-3.5 LLM service for content generation (crosses trust boundary)
5. Web Control Plane reads/writes data to Control Plane Database
6. API Application reads/writes data to API database
```

Those data flows are correct. However data flow 5 and 6 is also crossing trust boundary, per previous section. Something is not correct here.

### Threat model section

#### Spoofing of API Gateway

I will show you only single threat: **spoofing** of API Gateway:

| Column | Value | Comment |
| --- | --- | --- |
| THREAT ID | 0001 | ✅ |
| COMPONENT NAME | API Gateway | ✅ |
| THREAT NAME | Unauthorized access to API endpoints | ✅ |
| STRIDE CATEGORY | Spoofing | ✅ |
| WHY APPLICABLE | Attackers may try to access API endpoints without proper authentication to gain unauthorized access to data or functionality. | Somehow good, but also generic. |
| HOW MITIGATED | Authentication with API keys for each Meal Planner application is implemented. | ✅ - this is exactly mentioned in input document |
| MITIGATION | Ensure strong API key generation, secure storage, and regular rotation. Implement rate limiting and input validation at the API Gateway level. | ✅ |
| LIKELIHOOD EXPLANATION | Low, as authentication and rate limiting are in place, making it difficult for attackers to gain unauthorized access. | ✅ |
| IMPACT EXPLANATION | High, as unauthorized access could lead to data breaches or misuse of the system. | ✅ |
| RISK SEVERITY | Medium | ✅ - likelihood x impact

#### Comparison to baselines

Let's see how spoofing of API Gateway is described in baseline threat models:

| Baseline name | Threat name | Mitigations | Comment |
| --- | --- | --- | --- |
| `create_threat_scenarios` from Fabric | Unauthorized access to API Gateway via stolen API keys. | None | We get a focus on stolen API key, which is most likely way to obtain API key by attacker. Sadly no mitigation. |
| STRIDE GPT | An attacker uses a stolen API key to impersonate a Meal Planner application. | Implement API key rotation and revocation policies. Use mutual TLS for authentication and enforce strict validation of certificates. Implement anomaly detection to identify unusual API usage patterns. | Both threat and mitigations are very good. One more time, theft of API keys is mentioned.  |
| ai-threat-modeling-action | Attacker bypasses weak authentication and gains unauthorized access to API Gateway | Ensure strong authentication mechanisms are in place, such as using secure and properly implemented API keys or OAuth tokens. Regularly rotate and revoke API keys. Implement rate limiting and monitoring to detect and prevent brute-force attempts. | Didn't mention of stolen API keys as previous ones. In general similar to what I got from pattern. |

It's very good that `create_threat_scenarios` and STRIDE GPT give more meaningful attack scenario with stolen API key. It's realistic and can be used to convince stakeholders for investing into mitigations. 

### Questions & Assumptions section

```markdown
1. Q: Are there any additional security measures in place for the API database, such as encryption at rest or strong access controls?
   A: The design document does not explicitly mention encryption at rest or specific access controls for the API database. It is assumed that standard security practices are followed, but additional measures may be necessary to protect sensitive data.
...
```

This section can be source of inspiration to update input document in order to make it more clear and explicit.

## Key learnings

### Context matters

There is something missing in all solutions to threat modelling that we checked today. It's **context**. In real life, we don't create threat model for sake of creating it, but we do it on purpose. Context can describe by what kind of company we gonna implement this design. It can be in startup company, that just started and AI Nutrition-Pro is their only project. Or it can be mature company that AI Nutrition-Pro will be one of many features in their platform. Risk appetite and existing platform or company security posture will matter here. Also it will be different who will be consuming threat model. Developers or security team? 

It's definitely worth checking how LLM can handle this additional context. But this is for another experiment 😀

### Fabric makes integration easy

If you are console oriented, fabric will be perfect for you.

### Co-intelligence

Ethan Mollick, created this term _co-intelligence_ as a way to describe that LLMs help us in thinking process. I cannot predict future, but have a feeling that LLMs will become a tool that will help us thinking and solving problems. There is no security engineer to replace in company that don't hire any in first place. There are only developers that will do their job better using AI.

[Code](https://github.com/xvnpw/fabric-stride-threat-model) used in this experiment is published on github.

---

Thanks for reading! You can contact me and/or follow on [X](https://x.com/xvnpw).