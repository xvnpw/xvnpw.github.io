---
title: "Leveraging LLMs for Threat Modelling - GPT-3.5"
date: 2023-08-16T18:59:02+01:00
draft: true
tags: [security, threat-modelling, langchain, llm, gpt]
description: "In this article, I delve into the AI Nutrition-Pro experiment, a research project exploring the potential of LLMs in enhancing security practices during the design phase of DevSecOps: threat modelling and security review"
---

In this article, I delve into the [AI Nutrition-Pro experiment](https://github.com/xvnpw/ai-nutrition-pro-design-gpt3.5), a research project exploring the potential of LLMs in enhancing security practices during the design phase of DevSecOps: **threat modelling** and **security review**.

## DevSecOps: A Brief Overview

DevSecOps merges the principles of development, security, and operations to create a culture of shared responsibility for software security. The three main goals of DevSecOps are:

- **Shift Left Security:** Identifying and addressing security vulnerabilities as early as possible in the software development lifecycle.
- **Developer-Centric:** Integrating security practices seamlessly into the developer's ecosystem, including Integrated Development Environments (IDEs), code hosting platforms, and pull requests.
- **Fast Feedback and Guidance:** Providing developers with rapid feedback on security issues and guidance on secure coding practices.

While security tools like [semgrep](https://semgrep.dev/blog/2023/using-ai-to-write-secure-code-with-semgrep) can already use LLMs in the coding phase, the AI Nutrition-Pro experiment seeks to explore the benefits of LLMs during the design phase, particularly in security design reviews and threat modeling.

## Structure of Experiment

I created **fake** input data mimicking real project in github repository and used github action [xvnpw/ai-threat-modeling-action](https://github.com/xvnpw/ai-threat-modeling-action) to automatically generate output content and commit it directly into repository or create pull request. Action can also comment on issues.

### Input Data 

| Name | File | Description | Security artefact to generate | 
| --- | --- | --- | --- | 
| Project description | [PROJECT.md](https://github.com/xvnpw/ai-nutrition-pro-design-gpt3.5/blob/main/PROJECT.md) | High level description of the project with business explanation and listed core features | High level security design review | 
| Architecture | [ARCHITECTURE.md](https://github.com/xvnpw/ai-nutrition-pro-design-gpt3.5/blob/main/ARCHITECTURE.md) | Architecture of the solution | Threat Modelling |
| User story | [0001_STORE_DIET_INTRODUCTIONS.md](https://github.com/xvnpw/ai-nutrition-pro-design-gpt3.5/blob/main/user-stories/0001_STORE_DIET_INTRODUCTIONS.md) | Technical and user stories to implement | Security related acceptance criteria | 

### Results

Let's check each input data and corresponding results. I will also discuss prompt techniques.

#### Project description

Below part of [PROJECT.md](https://github.com/xvnpw/ai-nutrition-pro-design-gpt3.5/blob/main/PROJECT.md):

> # AI Nutrition-Pro
> ## Business background
> Dietitians use online applications to create meals, diets and calculate calories called meal planners. Different professionals have different ways of creating diets, which gives a personal style to it. LLMs can reproduce this personal style of writing based on samples of already created content. Meal planners can use LLMs to speed up diet creation for dietitians.
> ## Project Overview
> AI Nutrition-Pro will be backend API application that will have the possibility to integrate with any meal planner application for dietitians. It will reproduce the personal style of a nutrition specialist based on samples.
>
> Dietitians will not use the application directly but from their meal planner applications. There will be no user interface exposed to Dietitians. Integration will be using meal plan applications backend.
>
> Direct clients of AI Nutrition-Pro will be applications like DietMaster Pro, Nutritionist Pro, or others. Those clients will send to AI Nutrition-Pro samples of content and AI Nutrition-Pro will generate requested type of content based on that. AI Nutrition-Pro will use LLM to generate requested content.
> ...

**Prompt** is quite simple. First I provided extensive instruction, next example of output format and at the end PROJECT.md content:
```
Instruction:
- You are an security architect.
- Your task is to analyze project description and create high level security and privacy requirements
- Project description will be in markdown format
- Format output as markdown
- Response with at least 10 high level security and privacy requirements formatted as markdown and nothing else
- I will provide you example of requirement

Example of requirement:
### 1. Authentication and Authorization
- **Requirement**: Implement strong authentication mechanisms for all users, applications, and APIs accessing AI Nutrition-Pro.
- **Description**: Utilize secure authentication protocols such as OAuth 2.0 or JWT to authenticate and authorize tenants, dietitians, and other users. Different levels of access should be granted based on roles and responsibilities.

Project description:
"{text}"
```

PROJECT.md file is placed into prompt in `{text}` placeholder.

GPT output is saved into [PROJECT_SECURITY.md](https://github.com/xvnpw/ai-nutrition-pro-design-gpt3.5/blob/main/PROJECT_SECURITY.md):

> # (AI Generated) High Level Security and Privacy Requirements
> ### 1. Authentication and Authorization
> - **Requirement**: Implement secure authentication mechanisms for all users, applications, and APIs accessing AI Nutrition-Pro.
> - **Description**: Utilize strong authentication protocols such as OAuth 2.0 or JWT to authenticate and authorize tenants, dietitians, and other users. Different levels of access should be granted based on roles and responsibilities.
> ### 2. Data Encryption
> - **Requirement**: Encrypt all sensitive data at rest and in transit within AI Nutrition-Pro.
> - **Description**: Utilize industry-standard encryption algorithms and protocols to protect sensitive data, including personal health information (PHI), both when it is stored in databases and when it is transmitted over networks.
> ### 3. Secure Storage and Processing of PII
> - **Requirement**: Implement robust security measures to protect the storage and processing of personally identifiable information (PII) within AI Nutrition-Pro.
> - **Description**: Utilize secure cloud-based services and follow best practices for data storage and processing to ensure the confidentiality, integrity, and availability of PII. Implement access controls, data segregation, and regular security audits.
> ...

**Comment on results:**

These requirements demonstrate a **solid grasp** of security best practices and compliance standards. Their are good for high level description. It could benefit from providing more specific implementation guidance or references to established security frameworks.

#### Architecture

[Architecture](https://github.com/xvnpw/ai-nutrition-pro-design-gpt3.5/blob/main/ARCHITECTURE.md) was described using [C4 Model](https://c4model.com/), with diagrams coded in `mermaid`. Additional each diagram was described in points in case GPT cannot understand it.

> # Architecture
> This document outlines the architecture of the AI Nutrition-Pro application, including system context, containers, and deployment views. The architecture is depicted using C4 diagrams for enhanced clarity.
> ## System Context diagram
>
> ```mermaid
> CC4ontext
>  title System Context diagram for AI Nutrition-Pro
>  Enterprise_Boundary(b0, "AI Nutrition-Pro boundary") {
>    System(anps1, "AI Nutrition-Pro API Service")
>  }
>  Enterprise_Boundary(b1, "OpenAI") {
>    System_Ext(ChatGPT, "ChatGPT", "LLM")
>  }
>  Enterprise_Boundary(b2, "Meal Planner A") {
>    Person_Ext(n1, "Dietitian A")
>    System_Ext(apa, "Meal Planner A System")
>  }
>  Rel(anps1, ChatGPT, "Uses for LLM featured content creation", "REST")
>  Rel(n1, apa, "Uses for diet creation")
>  Rel(apa, anps1, "Uses for AI content generation", "REST")
> ```
>
> - AI Nutrition-Pro is API backend application that uses OpenAI ChatGPT as LLM solution. It connects to OpenAI server using REST.
> - Meal Planner A is one of many Meal Planner applications that can be integrated with AI Nutrition-Pro. It connects to AI Nutrition-Pro using REST.
> - Dietitian A uses Meal Planner A for making diets. For example, dietitian can generate new diet introductions based on existing ones. LLMs can create this content following personal style of dietitian.
> ...

This prompt is more complex. Simple instruction to performing threat model didn't return meaningful results. After playing with prompt, I got good results using 2 stages:

- first I ask to list data flows for architecture
- and than for each data flow I ask for threat modelling using STRIDE per component technique

First prompt:
```
Instruction:
- You are an security architect
- List data flows for all components that are internal and important for security of system
- You should not include any persons in data flows
- You should answer only in list and nothing more. Each data flow should be in separated line
- Architecture description will be in markdown format

Example:
Data flow 1: Client -> API Gateway 
Data flow 2: API Gateway -> API Application
Data flow 3: API Application -> API Database

Architecture description:
"{text}"
```

`{text}` is placeholder for ARCHITECTURE.md file.

Second prompt (executed for each data flow):

```
Instruction:
- You are an security architect
- I will provide you Architecture description
- Perform threat modelling using STRIDE per component technique for data flow
- I will provide you data flow in structure: Data flow 1: Component A -> Component B
- You should answer only in table and nothing more
- Architecture description will be in markdown format
- Format output as markdown

Output of threat modelling should be in table as in example:
### Data flow 1: Component A -> Component B
| Threat Id | Component name | Threat Name | STRIDE category | Mitigations | Risk severity |
| --- | --- | --- | --- | --- | --- |
| 1 | Component A | Attacker is able to spoof client using leaked API key | Spoofing | Invalidation of API keys. Usage of request signing technique | Critical |

Architecture description:
"{text}"

Data flow:
"{dataflow}"
```

`{text}` is one more time placeholder for ARCHITECTURE.md and `{dataflow}` is placeholder for dataflow returned in previous step.

GPT output is saved into [ARCHITECTURE_SECURITY.md](https://github.com/xvnpw/ai-nutrition-pro-design-gpt3.5/blob/main/ARCHITECTURE_SECURITY.md):

> # (AI Generated) Architecture Threat Model
> ### Data flow 1: Client -> API Gateway
> | Threat Id | Component name | Threat Name | STRIDE category | Mitigations | Risk severity |
> | --- | --- | --- | --- | --- | --- |
> | 1 | Client | Attacker intercepts and modifies requests/responses | Tampering | Use HTTPS for secure communication. Implement message integrity checks. | High |
> | 2 | API Gateway | Attacker bypasses authentication and gains unauthorized access | Spoofing | Implement strong authentication mechanisms. Use secure protocols for communication. | High |
> | 3 | API Gateway | Attacker overwhelms the API Gateway with excessive requests | Denial of Service | Implement rate limiting and throttling mechanisms. Use load balancers to distribute traffic. | Medium |
> | 4 | API Gateway | Attacker injects malicious code or scripts in requests | Injection | Implement input validation and sanitization techniques. Use secure coding practices. | High |
> | 5 | API Gateway | Attacker gains unauthorized access to sensitive data | Elevation of Privilege | Implement access control mechanisms. Encrypt sensitive data at rest and in transit. | High |
>
> ...

**Comment on results:**

It was much harder to get meaningful results for threat modelling. For some runs with temperature > 0, I got brilliant results, but most of them were just average. Their are still relevant to the scope, but very general. While the document presents a comprehensive threat model, some areas could benefit from additional elaboration.

#### User story

Description of [user story](https://github.com/xvnpw/ai-nutrition-pro-design-gpt3.5/blob/main/user-stories/0001_STORE_DIET_INTRODUCTIONS.md) is most detailed. Containing new API structure, container diagram from C4 model and listed tasks:

> # Store diet introductions
> As Meal App, I want to store samples of diet introductions of dietitians, so that those can be later used to generate new diet introductions using ChatGPT..
> ## Diagram
> ```mermaid
> CC4ontainer
>    title Container diagram for User Story: Store diet introductions
>    ...
>```
>
> ## New API
> New API to implement:
> 
> ```
> POST /api/v1/storeContent
> {
>   "type": "introduction",
>   "dietitian-uuid": "3beddddb-d8f2-41a3-8b6e-38bf2a39a56c",
>   "client-uuid": "47dba491-8a34-4bca-934b-b32532de975b",
>   "content": [
>     "Hi Mark. I created this diet for you. Hope you will love it :)",
>     "Hi Joanna! Hope you are well. This 3 days diet will help you get started :)"
>  ]
> }
>```
> ...

Prompt also is the most complex. Same as for architecture threat model, I couldn't benefit from simple prompt. It was returning acceptance criteria for elements that are out of scope, e.g. client. I suspect that reason of that is misunderstanding word "component" by GPT. Each time I ask for "components in scope of user story" it returned rubbish. I changed that into question for "architecture containers, services or applications included in architecture". This worked a way better than before. Still for some runs I saw client, but it was very rare. 

As mentioned above prompt has two stages:
- in first I asked to list components (using "architecture containers, services or applications included in architecture")
- in second I asked to list security related acceptance criteria for every component - in contrast to architecture I don't ask for each component individually but all at once. 

As this prompt is the most complex, please review it directly in [repository](https://github.com/xvnpw/ai-threat-modeling-action/blob/main/user_story.py).

GPT output is saved into [0001_STORE_DIET_INTRODUCTIONS_SECURITY.md](https://github.com/xvnpw/ai-nutrition-pro-design-gpt3.5/blob/main/user-stories/0001_STORE_DIET_INTRODUCTIONS_SECURITY.md):

> # (AI Generated) Security Related Acceptance Criteria
> Based on the provided user story, architecture description, and architecture threat model, the following are the security-related acceptance criteria for the specified architecture containers, services, or applications:
>
> **API Gateway:**
> - **AC1:** The API Gateway must enforce authentication mechanisms to prevent unauthorized access.
> - **AC2:** The API Gateway must implement rate limiting and throttling mechanisms to mitigate the impact of excessive requests.
> - **AC3:** The API Gateway must perform input validation and sanitization to prevent injection attacks.
> - **AC4:** The API Gateway must use HTTPS for secure communication to prevent interception and tampering.
>
> ...

**Comment on results:**

Same as for architecture it was hard to get good results. For some of runs I got brilliant output with reference to API path and parameters. But for most of runs I got very general and average results. 

## Summary

GPT-3.5 has some potential for performing threat modeling and security reviews, especially for teams without security engineers. It gives general and high level guidance but lacks detailed descriptions. Prompt need to be tune to match documents structure.

I encourage you to try use [xvnpw/ai-threat-modeling-action](https://github.com/xvnpw/ai-threat-modeling-action) for your documentation, and share results will me!

In next part I will review GPT-4.

---

Thanks for reading! You can follow me on [X/Twitter](https://twitter.com/xvnpw).