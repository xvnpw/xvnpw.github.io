---
title: "Leveraging LLMs for Threat Modeling - GPT-3.5 vs Claude 2 vs GPT-4"
date: 2023-09-02T18:59:02+01:00
draft: true
tags: [security, threat-modeling, langchain, llm, gpt, claude]
description: "I tested most important LLMs for ability to perform threat modeling. Let's see results and find out which performed best"
---

I tested the most important LLMs (GPT-3.5, Claude 2, and GPT-4) for ability to perform **threat modeling**. Let's see results and find out which performed best.

{{< figure src="https://github.com/xvnpw/xvnpw.github.io/assets/17719543/bf036c66-1d66-4468-8dcd-426d6e0f40f6" >}}

This quote from [Ethan Mollick](https://twitter.com/emollick), who is professor at The Wharton School, resonates very much with my experience on LLMs. Answers from GPT look like someone tried to satisfy question in the easiest way possible 😏 You can overcome that by asking for explanation or step-by-step thinking.

## Experiment structure recap

If you want to learn more about structure of experiment check my [previous post]({{< ref "/posts/leveraging-llms-for-threat-modelling-gpt-3.5.md" >}}). Below is just short recap.

I prepared input data of markdown files that describe made up project called **AI Nutrition-Pro**:
- [PROJECT.md](https://github.com/xvnpw/ai-nutrition-pro-design-gpt3.5/blob/main/PROJECT.md) - high level project description
- [ARCHITECTURE.md](https://github.com/xvnpw/ai-nutrition-pro-design-gpt3.5/blob/main/ARCHITECTURE.md) - architecture description
- [0001_STORE_DIET_INTRODUCTIONS.md](https://github.com/xvnpw/ai-nutrition-pro-design-gpt3.5/blob/main/user-stories/0001_STORE_DIET_INTRODUCTIONS.md) - user story

I defined 3 types of analysis to perform by LLMs:
- High level security design review (example: [GPT-3.5](https://github.com/xvnpw/ai-nutrition-pro-design-gpt3.5/blob/main/PROJECT_SECURITY.md))
- Threat Modelling (example: [GPT-3.5](https://github.com/xvnpw/ai-nutrition-pro-design-gpt3.5/blob/main/ARCHITECTURE_SECURITY.md))
- Security related acceptance criteria (example: [GPT-3.5](https://github.com/xvnpw/ai-nutrition-pro-design-gpt3.5/blob/main/user-stories/0001_STORE_DIET_INTRODUCTIONS_SECURITY.md))

## Interesting bits

### Assumptions

Threat modeling is art and science. One reason for that is **assumptions** that are not well specified. Those assumptions filter possible threats and drive whole process. In my research, I didn't set any assumptions in prompts to see how LLMs can handle that. I can tell you that LLM behaves the same as someone new to threat modeling, and starts giving many different injection attacks (starting with SQL Injections...). 

You can overcome that by listing assumptions directly:
```
Instruction:
...
- I will provide you Assumptions that you need to consider in listing threats

Assumptions:
- SQL Injection threat is already mitigated using SAST tool
...
```

### Explanation

I learned that asking LLMs to explain why threat is applicable to architecture can give good results. It made list of threats less random.

```
Instruction:
- Explanation whether or not this threat is already mitigated in architecture
...
```

There is also a big difference between GPT-3.5 and GPT-4 in understanding explanation.

**GPT-3.5**:
- **Threat name:** Attacker spoofs client identity and gains unauthorized access to API Gateway
- **Explanation:** This threat is applicable as API Gateway is the entry point for client requests and authentication is required to ensure the client's identity.
- **Mitigations:** Implement strong authentication mechanisms such as API keys, OAuth, or JWT tokens to verify the client's identity.

**GPT-4**:
- **Threat name:** Attacker bypasses API Gateway and directly accesses API Application
- **Explanation:** This threat is applicable if the API Application is not properly secured and can be accessed without going through the API Gateway.
- **Mitigations:** Ensure that the API Application is only accessible through the API Gateway and cannot be directly accessed.

It could be taken into higher level with step by step thinking. But didn't tested that yet:
> When I give you something to do, you will convert that to a step by step plan and tell me what the step by step plan is.

### Components

What is component? This *word* is so overloaded that can mean almost anything. And LLMs have big problem in understanding it. Especially GPT-3.5 and Claude 2.

Why component? A technique that I choose for threat modeling is "STRIDE per component" (or you can call it STRIDE per element).

After few tries I ended up with:
```
Instruction:
- List data flows that starts from internal or ends in internal architecture container that are important for security of system
- Data flow should contain two items: A -> B (A and B are architecture containers)
- Architecture container is for example: service, microservice, database, queue, person, gateway, lambda, application
- Architecture containers are included in C4 model in Container diagram
```

It's not perfect, because it expects architecture to be C4 model. This limits flexibility of prompt. 

BTW. GPT-4 can understand what component is 😊 

## GPT-3.5 vs Claude 2 vs GPT-4

- GPT-3.5 and Claude 2 gave comparable results. They are good and can be used if you have development team that is not so much experienced in security. Results will give you reasonable amount of most important things to consider. 
- GPT-4 gave best results. It's less fragile to changes in prompt. With assumptions, it can give you very good results.

Check results yourself: [GPT-3.5](https://github.com/xvnpw/ai-nutrition-pro-design-gpt3.5/blob/main/ARCHITECTURE_SECURITY.md), [Claude 2](https://github.com/xvnpw/ai-nutrition-pro-design-claude2/blob/main/ARCHITECTURE_SECURITY.md), [GPT-4](https://github.com/xvnpw/ai-nutrition-pro-design-gpt4/blob/main/ARCHITECTURE_SECURITY.md)

---

Thanks for reading! You can contact me and/or follow on [X/Twitter](https://twitter.com/xvnpw).
