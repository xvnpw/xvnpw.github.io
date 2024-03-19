---
title: "Leveraging LLMs for Threat Modeling - Claude 3 Opus vs GPT-4"
date: 2024-03-19T022:59:02+01:00
draft: true
tags: [security, threat-modeling, langchain, llm, gpt, claude]
description: "With new version of Claude model, I would like to compare it to GPT-4 in threat modeling"
---

[Claude 3 Opus](https://www.anthropic.com/news/claude-3-family) is latest and most powerful model from Anthropic. Is it able to overcome GPT-4?

{{< figure src="https://github.com/xvnpw/xvnpw.github.io/assets/17719543/3305cef0-c07d-4239-8fd9-c6e9e14146e7" class="image-center" >}}

## Revisiting the Experiment

If you wish to understand more about the experiment structure, you can refer to my [previous post]({{< ref "/posts/leveraging-llms-for-threat-modelling-gpt-3.5.md" >}}). But here's a quick recap:

I used markdown files describing a fictional project, **AI Nutrition-Pro**, as input data:
- [PROJECT.md](https://github.com/xvnpw/ai-nutrition-pro-design-claude3-opus/blob/main/PROJECT.md) - high level project description
- [ARCHITECTURE.md](https://github.com/xvnpw/ai-nutrition-pro-design-claude3-opus/blob/main/ARCHITECTURE.md) - architecture description
- [0001_STORE_DIET_INTRODUCTIONS.md](https://github.com/xvnpw/ai-nutrition-pro-design-claude3-opus/blob/main/user-stories/0001_STORE_DIET_INTRODUCTIONS.md) - user story

I tasked the AI models with four types of analysis: high-level security design review, threat modeling, security-related acceptance criteria and review of architecture:
- High level security design review: [Claude 3 Opus](https://github.com/xvnpw/ai-nutrition-pro-design-claude3-opus/blob/main/PROJECT_SECURITY.md), [GPT-4](https://github.com/xvnpw/ai-nutrition-pro-design-gpt4/blob/main/PROJECT_SECURITY.md)
- Threat Modeling: [Claude 3 Opus](https://github.com/xvnpw/ai-nutrition-pro-design-claude3-opus/blob/main/ARCHITECTURE_SECURITY.md), [GPT-4](https://github.com/xvnpw/ai-nutrition-pro-design-gpt4/blob/main/ARCHITECTURE_SECURITY.md)
- Security related acceptance criteria: [Claude 3 Opus](https://github.com/xvnpw/ai-nutrition-pro-design-claude3-opus/blob/main/user-stories/0001_STORE_DIET_INTRODUCTIONS_SECURITY.md), [GPT-4](https://github.com/xvnpw/ai-nutrition-pro-design-gpt4/blob/main/user-stories/0001_STORE_DIET_INTRODUCTIONS_SECURITY.md)
- Review of architecture: [Claude 3 Opus](https://github.com/xvnpw/ai-nutrition-pro-design-claude3-opus/blob/main/ARCHITECTURE_REVIEW.md), [GPT-4](https://github.com/xvnpw/ai-nutrition-pro-design-gpt4/blob/main/ARCHITECTURE_REVIEW.md)

## Key Learnings

### Making Assumptions

Threat modeling is both an art and a science, largely due to the role of assumptions. These assumptions guide the process and filter potential threats. In my research, I didn't set any assumptions to see how the AI models would handle it. They reacted much like a novice, offering a variety of different potential attacks. This can be managed by listing assumptions directly. 

You can overcome that by listing assumptions directly:
```
Instruction:
...
- I will provide you Assumptions that you need to consider in listing threats

Assumptions:
- SQL Injection threat is already mitigated using SAST tool
...
```

### Seeking Explanations

Asking the AI models to explain why a threat applies to the architecture can yield good results, making the list of threats less random.

```
Instruction:
- Explanation whether or not this threat is already mitigated in architecture
...
```

There's a noticeable difference in the understanding of explanations between Claude 3 Opus and GPT-4:

**Claude 3 Opus**:
- **Threat name:** Attacker spoofs client identity and gains unauthorized access to API Gateway
- **Explanation:** This threat is applicable as API Gateway is the entry point for client requests and authentication is required to ensure the client's identity.
- **Mitigations:** Implement strong authentication mechanisms such as API keys, OAuth, or JWT tokens to verify the client's identity.

**GPT-4**:
- **Threat name:** Attacker bypasses API Gateway and directly accesses API Application
- **Explanation:** This threat is applicable if the API Application is not properly secured and can be accessed without going through the API Gateway.
- **Mitigations:** Ensure that the API Application is only accessible through the API Gateway and cannot be directly accessed.

It could be taken into higher level with step by step thinking. But didn't test that yet:
> When I give you something to do, you will convert that to a step by step plan and tell me what the step by step plan is.

### Understanding Components

The term "component" is often overloaded and can mean different things, which poses a problem for AI models, especially Claude 3 Opus and Claude 2. After several tries, I found a way to clarify the meaning of "component" for them:

```
Instruction:
- List data flows that starts from internal or ends in internal architecture container that are important for security of system
- Data flow should contain two items: A -> B (A and B are architecture containers)
- Architecture container is for example: service, microservice, database, queue, person, gateway, lambda, application
- Architecture containers are included in C4 model in Container diagram
```

It's not perfect, because it expects architecture to be C4 model. This limits flexibility of prompt. 

BTW. GPT-4 can understand what component is 😊 

### Moving to JSON

Initially, I asked the AI models to format their output as markdown. However, I switched to JSON for added flexibility and ease of creating my own markdown. Both GPT models handled this well, though Claude 2 sometimes produced malformed documents. Solution was to use [OutputFixingParser](https://python.langchain.com/docs/modules/model_io/output_parsers/output_fixing_parser) from `langchain`. If it detects malformed json it ask LLM to fix it 😏

```
fixing_parser = OutputFixingParser.from_llm(parser=parser, llm=llm)
gen_threats = fixing_parser.parse(ret)
```

## The Verdict: Claude 3 Opus vs Claude 2 vs GPT-4

Claude 3 Opus and Claude 2 produced similar results. They're useful if your development team lacks extensive security experience, providing a reasonable number of important considerations. However, GPT-4 proved to be the superior model. It's less sensitive to changes in prompts and can be used to build robust threat modeling automation with the right assumptions.

🔥 Check results yourself: [Claude 3 Opus](https://github.com/xvnpw/ai-nutrition-pro-design-claude3-opus/blob/main/ARCHITECTURE_SECURITY.md), [Claude 2](https://github.com/xvnpw/ai-nutrition-pro-design-claude2/blob/main/ARCHITECTURE_SECURITY.md), [GPT-4](https://github.com/xvnpw/ai-nutrition-pro-design-gpt4/blob/main/ARCHITECTURE_SECURITY.md) and let me know what you think about experiment!

[Code](https://github.com/xvnpw/ai-threat-modeling-action) used in this experiment is published on github.

---

Thanks for reading! You can contact me and/or follow on [X/Twitter](https://twitter.com/xvnpw).
