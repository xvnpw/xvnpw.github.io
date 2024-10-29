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

Considering current level of LLM models, it's clear that complete automation is not possible if we have some chunk of text on input and some chunk of text on output. That's why I put information about 🪄 recommended automation for prompts and patterns.

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
- 🪄 recommended automation - careful review of output with input
- ✨ recommended models - Anthropic Claude 3.5 Sonnet, OpenAI o1-preview


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

👁 hallucination: if you read carefully PROJECT.md you will notice that it didn't mention API keys or OAuth2, but in SECURITY POSTURE section you can find:

> Existing Security Controls:
>
> - security control: API access will be secured using API keys and OAuth2.

Model decided to fill existing security controls with something that was not in input. It's expected behavior of current LLM models.

Apart of hallucination, both "SECURITY POSTURE" and "RISK ASSESSMENT" sections are meaningful, yet not detailed. For some people it can be enough as draft that can be enhanced with manual work. We will compare those sections to review and refined version - both created by next prompts. Stay tuned!


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
- 🪄 recommended automation - careful review of output with input
- ✨ recommended models - Anthropic Claude 3.5 Sonnet, OpenAI o1-preview

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

👁 hallucination: it's not mention that API Gateway do some sort if authorization of access control. In contrary, it's mention that access control is performed by backend.

> 3. Security Architecture
>
> **Current security mechanisms**:
>
> **Authorization**: API Gateway utilizes ACL rules to allow or deny actions.

One more time we got hallucinations, but as previously - it's expected. Apart from that, current LLMs are putting high emphasis on data encryption at rest. From AppSec/CloudSec perspective it's easy to turn it on and usually not being so much considered during design review. Still valid and interesting review.


---

[Code](https://github.com/xvnpw/fabric-stride-threat-model) used in this experiment is published on GitHub.

---

Thanks for reading! You can contact me and/or follow me on [X](https://x.com/xvnpw).
