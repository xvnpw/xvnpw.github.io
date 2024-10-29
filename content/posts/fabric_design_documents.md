---
title: "Create design documents with Fabric"
date: 2024-10-29T16:59:02+01:00
draft: true
tags: [design, security, fabric, llm, gpt]
description: "How I use Fabric patterns to create, review and refine design documents."
---

I stumbled into a problem of creating high quality design documents for my threat modelling research. About one and half years ago, I created [AI Nutrition-Pro](https://github.com/xvnpw/ai-nutrition-pro-design-gpt3.5/blob/main/ARCHITECTURE.md) architecture and have been using it since then. What if it's already in LLMs training data 🤔? Testing threat modelling capabilities could give me false results.

I decided to create few prompts that will help me in daunting task of creating design documents. I coded those as [Fabric](https://github.com/danielmiessler/fabric) patterns so everyone can benefit. If you don't know Fabric - it's great cli tool created by [Daniel Miessler](https://danielmiessler.com).

You can use my prompts not only for fictional projects but also real ones.

🔒 In this article I will especially focus on security part of prompts and AI generated output.

{{< figure src="https://github.com/user-attachments/assets/df07a470-769e-48e6-ac79-3584c9e8bb22" class="image-center" width=500 >}}

## create_design_document

`create_design_document` - as name suggests, can be used to create design document. It takes description of idea, project or system and provides well written, detailed design document.

Usage:

```bash
cat PROJECT.md | fabric -p create_design_document > design.md
```

- **input** - idea, project or system description
- **output** - design document, containing:
  - business risks, 
  - **existing and recommended security controls**, 
  - C4 design diagrams,
  - risk assessment - **critical business processes and data**,
  - AI questions and assumptions

Example of input ([FULL](https://github.com/xvnpw/ai-nutrition-pro-design-gpt3.5/blob/main/PROJECT.md)): 

```markdown
# AI Nutrition-Pro

## Business background

Dietitians use online applications to create meals, diets and calculate calories called meal planners. Different professionals have different ways of creating diets, which gives a personal style to it. LLMs can reproduce this personal style of writing based on samples of already created content. Meal planners can use LLMs to speed up diet creation for dietitians.
...
```

Example of output ([FULL](https://gist.github.com/xvnpw/1b57a2594790b94bfca58e3bf32be7d3#file-design-md)) - generated with `-m openai/o1-preview`:

```markdown
## BUSINESS POSTURE

The business priorities and goals for AI Nutrition-Pro [...]

## SECURITY POSTURE

Existing Security Controls:
- security control: Use of AWS cloud services with built-in security features for data storage and processing.
- security control: Integration with ChatGPT 3.5 will be secured using OpenAI's recommended practices.
- security control: API access will be secured using API keys and OAuth2.

Accepted Risks:
- accepted risk: Dependency on third-party services like OpenAI for LLM functionality, which may introduce latency or service outages.

Recommended Security Controls:
- Implement encryption for data at rest and in transit.
- Establish a comprehensive logging and monitoring system to detect and respond to security incidents.
- Conduct regular security audits and vulnerability assessments.

Security Requirements:
- The system must comply with data protection regulations such as GDPR and HIPAA.
- Role-based access control should be implemented to restrict access to sensitive data.
- Data anonymization techniques should be applied to reduce the risk of exposing PII.

[...]

## RISK ASSESSMENT

- What are critical business processes we are trying to protect?
  The critical business processes include secure data handling, content generation, and maintaining service availability for meal planner integrations.

- What data are we trying to protect, and what is their sensitivity?
  We are trying to protect PII and personal health data of customers, which are highly sensitive and subject to data protection regulations.

[...]
```

If you read carefully PROJECT.md you will notice that it didn't mention API keys or OAuth2, but in SECURITY POSTURE section you can find:

> Existing Security Controls:
>
> - security control: API access will be secured using API keys and OAuth2.

Both "SECURITY POSTURE" and "RISK ASSESSMENT" sections are meaningful, yet not detailed. For some people it can be enough as draft that can be enhanced with manual work. We will compare those sections to review and refined version - both created by next prompts. Stay tuned!


## review_design

This pattern takes several aspects into consideration and review provided design document.

Usage:

```bash
cat design.md | fabric -p review_design > review.md
```

- **input** - design document
- **output** - review, containing:
  - Architecture Clarity and Component Design,
  - External System Integrations, 
  - **Security Architecture**,
  - **Data Management and Storage Security**,
  - Maintainability, Flexibility, and Future Growth,
  - Potential Risks and Areas for Improvement,
  - Document readability,
  - Conclusion.

Example of input: created in previous section by [create_design_document](#create_design_document) pattern.

Example of output ([FULL](https://gist.github.com/xvnpw/1b57a2594790b94bfca58e3bf32be7d3#file-review-md)) - generated with `-m openai/o1-preview`:

```markdown
## 1. Architecture Clarity and Component Design

The architecture diagram provides a clear overview [...]

## 3. Security Architecture

**Current security mechanisms**:

- **Authentication**: Each Meal Planner application uses an individual API key.
- **Authorization**: API Gateway utilizes ACL rules to allow or deny actions.
- **Encryption**: All network traffic between clients and the API Gateway is encrypted using TLS.

**Potential weaknesses**:

- API keys can be compromised; lacking mechanisms for rotation or expiration.
- No mention of security measures for communication between internal components.
- Data at rest encryption is not specified for the databases.

**Improvements**:

- Adopt token-based authentication (e.g., OAuth 2.0) for enhanced security and easier key management.
- Implement mutual TLS or internal API keys for secure communication between internal services.
- Enable encryption at rest for databases to protect sensitive data.
- Establish a security monitoring and incident response plan to detect and handle breaches.

[...]

## 5. Data Management and Storage Security

**Data handling and storage**:

- **API Database**: Stores dietitians' content samples, LLM requests, and responses.
- **Control Plane Database**: Contains control plane data, tenant information, and billing details.

**Security considerations**:

- No explicit mention of data encryption at rest or in backups.
- Potential risk of data leaks if client data is not properly segregated.

**Data flow optimization**:

- All data exchanges occur over TLS, securing data in transit.
- The efficiency of data flow between components is not addressed.

**Improvements**:

- Implement database encryption to secure data at rest.
- Enforce strict access controls and auditing on database operations.
- Use tenant-specific schemas or databases to ensure data segregation and client isolation.
- Regularly update and maintain database indexes for optimized performance.

[...]
```








Before we jump into the results, I first needed to create a new pattern in Fabric. To do that, I used three baseline threat models created for the same input file. This allowed me to compare results with already existing and tested solutions:

| Threat Model | Link | LLM Model |
| --- | --- | --- |
| Baseline threat model created by Fabric using the [create_threat_scenarios](https://github.com/danielmiessler/fabric/blob/main/patterns/create_threat_scenarios/system.md) pattern | [baseline_threat_scenarios.md](https://github.com/xvnpw/fabric-stride-threat-model/blob/main/baseline_threat_scenarios.md) | Claude 3 Opus |
| Baseline threat model created by [STRIDE GPT](https://github.com/mrwadams/stride-gpt) | [baseline_stride_gpt.md](https://github.com/xvnpw/fabric-stride-threat-model/blob/main/baseline_stride_gpt.md) | GPT-4o |
| Baseline threat model from my [ai-threat-modeling-action](https://github.com/xvnpw/ai-threat-modeling-action) | [baseline_threat_modeling_action.md](https://github.com/xvnpw/fabric-stride-threat-model/blob/main/baseline_threat_modeling_action.md) | Claude 3 Opus |

These three solutions are unique in their approach:
- `create_threat_scenarios` is the most generic prompt, and I was very surprised to see concrete threats and mitigations there.
- `STRIDE GPT` gives a different perspective. Before you can run threat generation, you need to answer a few questions. I picked the application type as **cloud**, sensitivity of data as **confidential**, said **yes** for internet-facing, and picked **Basic** auth. The results are very interesting. Threats are not grouped for specific data flows or components; instead, they are listed as scenarios, similar to the `create_threat_scenarios` approach.
- Finally, my previous work - `ai-threat-modeling-action`. I chose a different approach by splitting the design by data flows and components. The AI has more work to do (and it takes several round trips to model). The advantage is having a threat model using the STRIDE per element methodology as I would imagine it to be. The disadvantage is that AI sometimes misunderstands components, elements, or data flows. You cannot find any reference to AWS or cloud in this threat model. In both previous threat models, you can find some threats for AWS ECS or RDS.

Considering that the already existing pattern `create_threat_scenarios` is not based on STRIDE and lists threats without distinction on components and data flows, I decided to use the STRIDE per element approach. This way, Fabric will have two patterns for threat modeling but different.

## Interrogating AI

Interrogation is the best way to describe my prompt. You may already notice it in `ai-threat-modeling-action`, where there are columns like _Explanation_ or _How threat is already mitigated in architecture_. Why do that? There are a few points here:

- LLMs generate the next token based on previously generated ones - by having more details, I hope for better output.
- LLMs consider the semantics of things - by grouping and relating ideas in multi-dimensional space - something like clustering - getting back to the first point, more related tokens should lead to better output.
- It gives a view on how LLM "understands" input - it provides an opportunity to get back and improve design documents.

### Output Format

I organized the output as markdown with the following sections:

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

## Comparison of Results to Baselines

Let's compare in detail how the new pattern differs from baselines:

| Difference | To | Comment |
| --- | --- | --- |
| Lower number of threats | All baselines | This is an obvious difference. All baselines have more threats. **Why?** The prompt is very much focused on "likelihood," "impact," "what's worth defending," no "fantastical concerns" (credits to Daniel Miessler). I can request more threats by explicitly forcing it to "generate at least 10 threats" or "create at least 10 rows in the threats table", but those threats are not high priority (in my opinion) anymore. I can't help but refer this to Isaac Asimov's Robot Series of books. Robots there behaved similarly, and skilled prompting was something like art.
| No cloud threats | `create_threat_scenarios` and STRIDE GPT | I mentioned this before. I think STRIDE per element is a bit old methodology and not mentioned in publications (=training data) in case of cloud.
| Important and actionable threats for startups | All baselines | I got only **5 threats** generated by the new pattern, but those are very important and you need to act if you were a startup. Especially in the case of STRIDE GPT and `ai-threat-modeling-action`, you can get a feeling of mature enterprise and not startup.
| Interrogation | `create_threat_scenarios` and STRIDE GPT | I extended descriptions on why, how, and what from those in `ai-threat-modeling-action`. In my opinion, it's important to get more output. This can help us define input and understand why we got certain threats.

## Threat Models

It's time to check the results of my new pattern. To make things more interesting, I ran it for three different LLM models:

- [Threat model](https://github.com/xvnpw/fabric-stride-threat-model/blob/main/threat_model_claude_3_opus.md) created using Claude 3 Opus
- [Threat model](https://github.com/xvnpw/fabric-stride-threat-model/blob/main/threat_model_gpt_4o.md) created using GPT-4o
- [Threat model](https://github.com/xvnpw/fabric-stride-threat-model/blob/main/threat_model_gemini_1.5_pro.md) created using Gemini-1.5 Pro Latest

In my opinion, Claude 3 Opus is the best and GPT-4o second. There is a noticeable gap between these two and Gemini 1.5 Pro (which is worst).

### Assets Section

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

### Trust Boundaries Section

```markdown
# TRUST BOUNDARIES

The following trust boundaries are identified in the AI Nutrition-Pro system:

1. Between the Meal Planner application and the API Gateway
2. Between the API Gateway and the API Application
3. Between the API Application and the ChatGPT-3.5 LLM service
4. Between the Web Control Plane and the Control Plane Database
5. Between the API Application and the API database
```

Trust boundaries are subjective, but in this case, I would argue that AI did it **wrong**. It's something that might need a better prompt or more explicit input document.

### Data Flows Section

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

These data flows are correct; however, data flows 5 and 6 also cross trust boundaries per the previous section. Something is not correct here.

### Threat Model Section

#### Spoofing of API Gateway

I will show you only a single threat: **spoofing** of API Gateway:

| Column | Value | Comment |
| --- | --- | --- |
| THREAT ID | 0001 | ✅ |
| COMPONENT NAME | API Gateway | ✅ |
| THREAT NAME | Unauthorized access to API endpoints | ✅ |
| STRIDE CATEGORY | Spoofing | ✅ |
| WHY APPLICABLE | Attackers may try to access API endpoints without proper authentication to gain unauthorized access to data or functionality. | Somehow good but also generic |
| HOW MITIGATED | Authentication with API keys for each Meal Planner application is implemented. | ✅ - this is exactly mentioned in input document |
| MITIGATION | Ensure strong API key generation, secure storage, and regular rotation. Implement rate limiting and input validation at the API Gateway level. | ✅ |
| LIKELIHOOD EXPLANATION | Low, as authentication and rate limiting are in place, making it difficult for attackers to gain unauthorized access. | ✅ |
| IMPACT EXPLANATION | High, as unauthorized access could lead to data breaches or misuse of the system. | ✅ |
| RISK SEVERITY | Medium | ✅ - likelihood x impact |

#### Comparison to Baselines

Let's see how spoofing of API Gateway is described in baseline threat models:

| Baseline Name | Threat Name | Mitigations | Comment |
| --- | --- | --- | --- |
| `create_threat_scenarios` from Fabric | Unauthorized access to API Gateway via stolen API keys. | None | We get a focus on stolen API key, which is most likely way to obtain an API key by an attacker. Sadly no mitigation.
| STRIDE GPT | An attacker uses a stolen API key to impersonate a Meal Planner application. | Implement API key rotation and revocation policies. Use mutual TLS for authentication and enforce strict validation of certificates. Implement anomaly detection to identify unusual API usage patterns. | Both threat and mitigations are very good; once again theft of API keys is mentioned.
| ai-threat-modeling-action | Attacker bypasses weak authentication and gains unauthorized access to API Gateway | Ensure strong authentication mechanisms are in place, such as using secure and properly implemented API keys or OAuth tokens. Regularly rotate and revoke API keys. Implement rate limiting and monitoring to detect and prevent brute-force attempts.| Didn't mention stolen API keys as previous ones; similar overall.

It's very good that `create_threat_scenarios` and STRIDE GPT give more meaningful attack scenarios with stolen API keys; it's realistic and can be used to convince stakeholders for investing into mitigations.

### Questions & Assumptions Section

```markdown
1. Q: Are there any additional security measures in place for the API database, such as encryption at rest or strong access controls?
   A: The design document does not explicitly mention encryption at rest or specific access controls for the API database. It is assumed that standard security practices are followed, but additional measures may be necessary to protect sensitive data.
...
```

This section can be a source of inspiration to update input documents in order to make them more clear and explicit.

## Key Learnings

### Context Matters

There is something missing in all solutions to threat modeling that we checked today: **context**. In real life, we don't create a threat model for its own sake but with a purpose in mind. Context can describe what kind of company will implement this design; it could be a startup company just starting out with AI Nutrition-Pro as their only project or a mature company where AI Nutrition-Pro will be one of many features in their platform. Risk appetite and existing platform or company security posture will matter here; also it will be different who will be consuming the threat model; developers or security team?

It's definitely worth checking how LLM can handle this additional context; but this is for another experiment.

### Fabric Makes Integration Easy

If you are console-oriented, Fabric will be perfect for you.

### Co-intelligence

Ethan Mollick coined this term _co-intelligence_ as a way to describe how LLMs help us in thinking processes. I cannot predict the future but have a feeling that LLMs will become tools that help us think and solve problems better; there is no security engineer to replace in companies that don't hire any in the first place; there are only developers who will do their job better using AI.

[Code](https://github.com/xvnpw/fabric-stride-threat-model) used in this experiment is published on GitHub.

---

Thanks for reading! You can contact me and/or follow me on [X](https://x.com/xvnpw).
