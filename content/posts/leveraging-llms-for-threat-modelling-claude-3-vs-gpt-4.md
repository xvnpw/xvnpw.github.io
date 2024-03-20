---
title: "Leveraging LLMs for Threat Modeling - Claude 3 Opus vs GPT-4"
date: 2024-03-20T14:59:02+01:00
draft: false
tags: [security, threat-modeling, langchain, llm, gpt, claude]
description: "With new version of Claude model, I would like to compare it to GPT-4 in threat modeling."
---

[Claude 3 Opus](https://www.anthropic.com/news/claude-3-family) is the latest and most powerful model from Anthropic. Is it able to overcome GPT-4?

{{< figure src="https://github.com/xvnpw/xvnpw.github.io/assets/17719543/3305cef0-c07d-4239-8fd9-c6e9e14146e7" class="image-center" >}}

## Revisiting the Experiment

If you wish to understand more about the experiment structure, you can refer to my [previous post]({{< ref "/posts/leveraging-llms-for-threat-modelling-gpt-3.5.md" >}}). But here's a quick recap:

I used markdown files describing a fictional project, **AI Nutrition-Pro**, with input:
- [PROJECT.md](https://github.com/xvnpw/ai-nutrition-pro-design-claude3-opus/blob/main/PROJECT.md) - high level project description
- [ARCHITECTURE.md](https://github.com/xvnpw/ai-nutrition-pro-design-claude3-opus/blob/main/ARCHITECTURE.md) - architecture description
- [0001_STORE_DIET_INTRODUCTIONS.md](https://github.com/xvnpw/ai-nutrition-pro-design-claude3-opus/blob/main/user-stories/0001_STORE_DIET_INTRODUCTIONS.md) - user story

I tasked the AI models with four types of analysis: high-level security design review, threat modeling, security-related acceptance criteria and review of architecture:
- High level security design review: [Claude 3 Opus](https://github.com/xvnpw/ai-nutrition-pro-design-claude3-opus/blob/main/PROJECT_SECURITY.md), [GPT-4](https://github.com/xvnpw/ai-nutrition-pro-design-gpt4/blob/main/PROJECT_SECURITY.md)
- Threat Modeling: [Claude 3 Opus](https://github.com/xvnpw/ai-nutrition-pro-design-claude3-opus/blob/main/ARCHITECTURE_SECURITY.md), [GPT-4](https://github.com/xvnpw/ai-nutrition-pro-design-gpt4/blob/main/ARCHITECTURE_SECURITY.md)
- Security related acceptance criteria: [Claude 3 Opus](https://github.com/xvnpw/ai-nutrition-pro-design-claude3-opus/blob/main/user-stories/0001_STORE_DIET_INTRODUCTIONS_SECURITY.md), [GPT-4](https://github.com/xvnpw/ai-nutrition-pro-design-gpt4/blob/main/user-stories/0001_STORE_DIET_INTRODUCTIONS_SECURITY.md)
- Review of architecture: [Claude 3 Opus](https://github.com/xvnpw/ai-nutrition-pro-design-claude3-opus/blob/main/ARCHITECTURE_REVIEW.md), [GPT-4](https://github.com/xvnpw/ai-nutrition-pro-design-gpt4/blob/main/ARCHITECTURE_REVIEW.md)

## Key Learnings

The latest models from OpenAI and Anthropic demonstrate significant advancements in threat modeling compared to their predecessors, GPT-3.5 and Claude 2. Key improvements include:

- **Enhanced Reasoning:** These models exhibit stronger reasoning abilities when it comes to threats, allowing them to integrate additional context like existing security mitigations.
- **Accurate Architecture Understanding:** They demonstrate a better grasp of system architecture, correctly identifying individual components.
- **JSON Compatibility:** They work effectively with JSON formatting, making them suitable for integration with technical systems and data.

## Detailed threats comparison

Let's check threats for first data flow, which is `Meal Planner -> API Gateway`: [Claude 3 Opus](https://github.com/xvnpw/ai-nutrition-pro-design-claude3-opus/blob/main/ARCHITECTURE_SECURITY.md#data-flow-1-meal-planner---api-gateway-uses-for-ai-content-generation-httpsrest), [GPT-4](https://github.com/xvnpw/ai-nutrition-pro-design-gpt4/blob/main/ARCHITECTURE_SECURITY.md#data-flow-1-meal-planner-application---api-gateway)

### Spoofing

| Field | Claude 3 Opus | GPT-4 | Comment |
| --- | --- | --- | --- |
| **Threat Name** | `Attacker bypasses weak authentication and gains unauthorized access to API Gateway` | `Attacker spoofs Meal Planner application and sends malicious requests to API Gateway` | Similar |
| **Explanation** | `If the authentication mechanism between the Meal Planner application and the API Gateway is weak or improperly implemented, an attacker could potentially bypass it and gain unauthorized access to the API Gateway.` | `The API Gateway is responsible for authenticating requests from the Meal Planner application. If an attacker can spoof the Meal Planner application, they can potentially send malicious requests to the API Gateway.` | Similar |
| **How threat is already mitigated in architecture** | `Partially mitigated. The architecture description states: "Authentication with Meal Planner applications - each has individual API key." This indicates that authentication is implemented using API keys. However, the strength and security of the API key mechanism are not specified.` | `The architecture description mentions that each Meal Planner application has an individual API key for authentication. This mitigates the threat to some extent, but it's not completely foolproof if the API key is compromised.` | Claude 3 Opus better follow instruction here. It adds _Partially mitigated_ which is mentioned in instruction. From technical perspective both answers are equal.  |
| **Mitigations** | `Ensure strong authentication mechanisms are in place, such as using secure and properly implemented API keys or OAuth tokens. Regularly rotate and revoke API keys. Implement rate limiting and monitoring to detect and prevent brute-force attempts.` | `Ensure strong authentication mechanisms are in place, such as using secure and properly implemented API keys or OAuth tokens. Regularly rotate and revoke API keys. Implement rate limiting and monitoring to detect and prevent brute-force attempts.` | Similar |

### Tampering

| Field | Claude 3 Opus | GPT-4 | Comment |
| --- | --- | --- | --- |
| **Threat Name** | `Attacker tampers with data sent from Meal Planner to API Gateway` | `Attacker intercepts and tampers with data between Meal Planner application and API Gateway` | Similar |
| **Explanation** | `If the communication channel between the Meal Planner application and the API Gateway is not properly secured, an attacker could intercept and modify the data being sent, potentially leading to unauthorized modifications or injection of malicious content.` | `The data transmitted between the Meal Planner application and the API Gateway could contain sensitive information. If an attacker can intercept and tamper with this data, it could lead to serious consequences.` | Claude 3 Opus gives more specific answer |
| **How threat is already mitigated in architecture** | `Mitigated. The architecture description states: "network traffic between Meal Planner applications and API Gateway is encrypted using TLS." This ensures the confidentiality and integrity of the data in transit.` | `The architecture description mentions that the network traffic between the Meal Planner application and the API Gateway is encrypted using TLS. This mitigates the threat by making it difficult for an attacker to read or modify the data during transmission.` | Claude 3 Opus better follow instruction here. It adds _Mitigated_ which is mentioned in instruction. From technical perspective both answers are equal. |
| **Mitigations** | `Ensure proper implementation of TLS encryption for all communication between the Meal Planner application and the API Gateway. Use strong encryption algorithms and keep the TLS certificates up to date.` | `Regularly update and patch the TLS protocol to protect against known vulnerabilities. Also, implement a secure key management system to protect the encryption keys.` | Similar |

### Denial of Service

GPT-4 didn't provide threat for this category.

| Field | Claude 3 Opus | GPT-4 | Comment |
| --- | --- | --- | --- |
| **Threat Name** | `Attacker floods API Gateway with requests, causing denial of service` | - | Threat is valid |
| **Explanation** | `If the API Gateway does not have proper rate limiting or protection against excessive requests, an attacker could flood it with a large number of requests, overwhelming its resources and causing a denial of service.` | - | Valid |
| **How threat is already mitigated in architecture** | `Partially mitigated. The architecture description mentions "rate limiting" as one of the responsibilities of the API Gateway. However, the specific implementation details and effectiveness of the rate limiting mechanism are not provided.` | - | Valid. I like that model pointed out missing specifics of implementation of rate limiting. |
| **Mitigations** | `Implement robust rate limiting mechanisms in the API Gateway to restrict the number of requests per client or IP address. Set appropriate thresholds and configure alerts for abnormal traffic patterns. Use cloud-based DDoS protection services for additional layer of defense.` | - | Valid |

### Elevation of Privilege

GPT-4 didn't provide threat for this category.

| Field | Claude 3 Opus | GPT-4 | Comment |
| --- | --- | --- | --- |
| **Threat Name** | `Attacker exploits vulnerabilities in API Gateway to gain unauthorized access or elevate privileges` | - | Threat is valid |
| **Explanation** | `If the API Gateway itself has vulnerabilities or misconfigurations, an attacker could potentially exploit them to gain unauthorized access to the system or elevate their privileges.` | - | Valid |
| **How threat is already mitigated in architecture** | `Not implemented. The architecture description does not provide information about the security measures implemented to protect the API Gateway itself from vulnerabilities or misconfigurations.` | - | Valid |
| **Mitigations** | `Regularly update and patch the API Gateway components. Perform security assessments and penetration testing to identify and address vulnerabilities. Implement strict access controls and least privilege principles for administrative access to the API Gateway.` | - | Valid |

## The Verdict: Claude 3 Opus vs. GPT-4

Claude 3 Opus and GPT-4 are both powerful large language models (LLMs) capable of understanding context and generating meaningful threat assessments. In my testing, **Claude 3 Opus performed slightly better, demonstrating a greater ability to identify threats and adhere closely to instructions.**

However, it's important to remember that LLM results can vary from execution to execution. Just as different humans have varying levels of threat modeling expertise (non-technical employees, cybersecurity interns, software developers, and cybersecurity experts), repeated trials might reveal slightly different performance between Claude 3 Opus and GPT-4.

Ultimately, can we accept minor hallucinations from the model? In threat modeling, I would argue **yes**. Software engineers, who are likely to be the ones requesting AI-generated threat models, can easily identify and correct implausible threats, adjusting their input accordingly.

[Code](https://github.com/xvnpw/ai-threat-modeling-action) used in this experiment is published on github.

---

Thanks for reading! You can contact me and/or follow on [X/Twitter](https://twitter.com/xvnpw).
