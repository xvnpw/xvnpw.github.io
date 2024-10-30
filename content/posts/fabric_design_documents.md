---
title: "Create design documents with Fabric"
date: 2024-10-30T13:59:02+01:00
draft: true
tags: [design, security, fabric, llm, gpt]
description: "How I use Fabric patterns to create, review and refine design documents."
---

I encountered a challenge in creating high-quality design documents for my threat modeling research. About a year and a half ago, I created [AI Nutrition-Pro](https://github.com/xvnpw/ai-nutrition-pro-design-gpt3.5/blob/main/ARCHITECTURE.md) architecture and have been using it since then. What if it's already in LLMs' training data? Testing threat modeling capabilities could give me false results.

I developed several prompts to assist with the challenging task of creating design documents. I implemented these as [Fabric](https://github.com/danielmiessler/fabric) patterns for everyone's benefit. If you're unfamiliar with Fabric - it's an excellent CLI tool created by [Daniel Miessler](https://danielmiessler.com).

You can use my prompts for both fictional and real projects.

🔒 In this article, I will focus particularly on the security aspects of prompts and AI-generated output.

Given the current capabilities of LLM models, complete automation isn't feasible when dealing with text input and output. That's why I've included information about 🪄 recommended automation for prompts and patterns.

{{< figure src="https://github.com/user-attachments/assets/df07a470-769e-48e6-ac79-3584c9e8bb22" class="image-center" width=500 >}}

## create_design_document

`create_design_document` ([source](https://github.com/danielmiessler/fabric/blob/5373345a3cc77c7dacee78ea2139e303a3389166/patterns/create_design_document/system.md)) - as name suggests, can be used to create design document. It takes description of idea, project or system and provides well written, detailed design document.

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
[...]
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

## RISK ASSESSMENT

- What are critical business processes we are trying to protect?
  The critical business processes include secure data handling, content generation, and maintaining service availability for meal planner integrations.

- What data are we trying to protect, and what is their sensitivity?
  We are trying to protect PII and personal health data of customers, which are highly sensitive and subject to data protection regulations.

[...]
```

👁 hallucination: if you read carefully, PROJECT.md you will notice that it didn't mention API keys or OAuth2, but in the SECURITY POSTURE section you can find:

> Existing Security Controls:  
>  
> - security control: API access will be secured using API keys and OAuth2.

The model decided to fill existing security controls with something that was not in the input.

Apart from hallucination, both the "SECURITY POSTURE" and "RISK ASSESSMENT" sections are meaningful, yet not detailed. For some people, it can be enough as a draft that can be enhanced with manual work. We will compare those sections to review and refine versions - both created by the next prompts. Stay tuned!

## review_design

`review_design` ([source](https://github.com/danielmiessler/fabric/blob/5373345a3cc77c7dacee78ea2139e303a3389166/patterns/review_design/system.md)) pattern takes several aspects into consideration and review provided design document.

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

## 5. Data Management and Storage Security

**Security considerations**:

- No explicit mention of data encryption at rest or in backups.
- Potential risk of data leaks if client data is not properly segregated.

**Improvements**:

- Implement database encryption to secure data at rest.
- Enforce strict access controls and auditing on database operations.
- Use tenant-specific schemas or databases to ensure data segregation and client isolation.
- Regularly update and maintain database indexes for optimized performance.

[...]
```

👁 hallucination: it's not mentioned that the API Gateway does some sort of authorization or access control. In contrast, it's mentioned that access control is performed by the backend.

> 3. Security Architecture  
>  
> **Current security mechanisms**:  
>  
> **Authorization**: API Gateway utilizes ACL rules to allow or deny actions.

Once more, we encounter hallucinations. Apart from that, current LLMs put a high emphasis on data encryption at rest. From an AppSec/CloudSec perspective, it's easy to turn it on and usually not considered much during design review. Nevertheless, we can find interesting points to make the design better.

## `refine_design_document`

`refine_design_document` ([source](https://github.com/danielmiessler/fabric/blob/5373345a3cc77c7dacee78ea2139e303a3389166/patterns/refine_design_document/system.md)) - if we already have a design document and its review, we can try another prompt that will take those and refine the design document so it aligns with the review.

Usage:

```bash
echo "DESIGN DOCUMENT" > design-document.md
cat PROJECT.md | fabric -p create_design_document >> design-document.md

echo "DESIGN REVIEW" > design-review.md
cat design-document.md | fabric -p review_design >> design-review.md

cat design-document.md design-review.md | fabric -p refine_design_document >> design-document-refined.md
```

- **input** - design document, design review
- **output** - refined design document
- 🪄 recommended automation - careful review of output with input
- ✨ recommended models - Anthropic Claude 3.5 Sonnet, OpenAI o1-preview

Example of input: created in previous sections by [create_design_document](#create_design_document) and [review_design](#review_design) patterns.

Example of output ([FULL](https://gist.github.com/xvnpw/1b57a2594790b94bfca58e3bf32be7d3#file-design-refined-md)) - generated with `-m openai/o1-preview`:

```markdown
## SECURITY POSTURE

### Existing Security Controls

- **Use of AWS Cloud Services**: Leveraging AWS cloud services with built-in security features for data storage and processing.
- **Secure Integration with ChatGPT 3.5**: Integration with ChatGPT 3.5 secured using OpenAI's recommended practices and API key management.
- **API Access Security**: API access secured using **OAuth 2.0** for enhanced security and easier key management.

### Accepted Risks

- **Dependency on Third-party Services**: Dependency on third-party services like OpenAI for LLM functionality, which may introduce latency or service outages.

### Recommended Security Controls

- **Encryption**: Implement encryption for data at rest and in transit, including internal communications between components.
- **Logging and Monitoring**: Establish a comprehensive logging and monitoring system to detect and respond to security incidents.
- **Regular Security Audits**: Conduct regular security assessments, vulnerability scans, and penetration testing.
- **Data Anonymization**: Apply data anonymization techniques to reduce the risk of exposing PII.

### Security Requirements

- **Regulatory Compliance**: The system must comply with data protection regulations such as GDPR and HIPAA.
- **Role-Based Access Control (RBAC)**: Implement RBAC to restrict access to sensitive data based on user roles.
- **Tenant Data Isolation**: Enforce strict data segregation to ensure client data remains isolated and secure.

## RISK ASSESSMENT

- **Critical Business Processes**: Protecting sensitive data (PII, health information), ensuring reliable content generation, and maintaining service availability for clients.
- **Data Sensitivity**: High sensitivity due to handling of PII and personal health information, requiring strict compliance with GDPR, HIPAA, and other regulations.

[...]

```

Comparing original version with refined, we can notice that review was applied in few places:

| Original | Review | Refined |
| --- | --- | ---|
| security control: API access will be secured using API keys and OAuth2. | Adopt token-based authentication (e.g., OAuth 2.0) for enhanced security and easier key management. | **API Access Security**: API access secured using **OAuth 2.0** for enhanced security and easier key management. | 
| - | Use tenant-specific schemas or databases to ensure data segregation and client isolation. | **Tenant Data Isolation**: Enforce strict data segregation to ensure client data remains isolated and secure. |

There are also minor improvements in wording and formatting. 

## Summary

Giving LLM a bigger chunk of text as input and requesting some sort of "thinking" leads to hallucinations. At the current state of models, we cannot get rid of it. Maybe in the future, those will be gone.

Does it mean that LLMs are useless? Certainly not. Can we replace a job function with my prompts? Not for sure. Can we replace the task of creating a design document? Also not. So what can we do? We can automate some sub-tasks. And that is a perfect application for LLM. We don't need to suffer the rigor of a blank page; we can get a draft with some sections needing a rewrite, and some being okay.

{{< figure src="https://github.com/user-attachments/assets/0615074e-1b92-4b87-b4e2-e1580f760430" class="image-center" >}}

---

[Code](https://gist.github.com/xvnpw/1b57a2594790b94bfca58e3bf32be7d3) used in this experiment is published on GitHub.

---

Thanks for reading! You can contact me and/or follow me on [X](https://x.com/xvnpw).
