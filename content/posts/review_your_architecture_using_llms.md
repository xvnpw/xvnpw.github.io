---
title: "Reviewing Your Architecture Using LLMs"
date: 2023-10-25T08:59:02+01:00
draft: false
tags: [security, threat-modeling, langchain, llm, gpt, architecture]
description: "The quality of input data is crucial for LLMs to perform effectively. Learn how you can use these LLMs to improve your architectural descriptions. Explore the new feature in my ai-threat-modeling-action GitHub action."
---

Recently, I've discussed [utilizing LLMs for threat modeling]({{< ref "/posts/leveraging-llms-for-threat-modelling-gpt-3.5-vs-claude2-vs-gpt-4.md" >}}). Building on that, I've incorporated a new feature in my [ai-threat-modeling-action](https://github.com/xvnpw/ai-threat-modeling-action) on GitHub. It now allows you to review your project's architectural description using LLMs. Let's delve deeper 🤿!

## Revisiting the Experiment

For a detailed understanding of the experiment's structure, you can revisit my [previous post]({{< ref "/posts/leveraging-llms-for-threat-modelling-gpt-3.5.md" >}}). However, here's a brief summary:

I utilized markdown files, which described a fictional project named **AI Nutrition-Pro**, as input:
- [ARCHITECTURE.md](https://github.com/xvnpw/ai-nutrition-pro-design-gpt4/blob/main/ARCHITECTURE.md) - providing the architectural overview.

The AI model was then assigned to threat modeling, resulting in this [output](https://github.com/xvnpw/ai-nutrition-pro-design-gpt4/blob/main/ARCHITECTURE_SECURITY.md).

## The Importance of Documentation

{{< figure src="https://github.com/xvnpw/xvnpw.github.io/assets/17719543/43a36068-a828-4e68-8b68-9fe48fead75f" class="image-center" >}}

If a meme exists, it likely holds a kernel of truth 😏 The challenge we face is that LLMs occasionally "hallucinate," meaning they fabricate information. And they often do so with high confidence. It's somewhat akin to a student attempting to bluff their way through a question they're unsure about.

Reducing these hallucinations is tricky. While some models might be more susceptible than others, the probability of hallucination rises if LLMs receive insufficient input. This is the issue we aim to tackle.

For [threat modeling](https://github.com/xvnpw/ai-nutrition-pro-design-gpt4/blob/main/ARCHITECTURE_SECURITY.md), the primary input is the [project's architectural documentation](https://github.com/xvnpw/ai-nutrition-pro-design-gpt4/blob/main/ARCHITECTURE.md). So, how do we ensure its quality?

## Reviewing Architecture Using LLM

I've integrated the review feature into [ai-threat-modeling-action](https://github.com/xvnpw/ai-threat-modeling-action). The prompt is straightforward:

```
Instruction:
You are solution architect. I will provide you Architecture Description and you will:
- Review the provided Architecture Description.
- Identify areas that require improvement or further clarification.
- Point out mistakes, things to improve
- Give advices on how to make it more readable and robust
- Pose questions or identify default assumptions, if any.

Architecture Description:
"{text}"
```

An example output could look like:

```
# (AI Generated) Architecture Review

The architecture description is well-detailed and provides a good understanding of the system. However, there are a few areas that could be improved or clarified:

1. **Security**: The document does not mention any security measures. It's important to include details about how the system will be secured, such as encryption of data at rest and in transit, secure access controls, and secure application development practices.

2. **Scalability and Performance**: The document does not provide information on how the system will handle increased load or how it will perform under stress. It would be beneficial to include details about load balancing, auto-scaling, and performance monitoring.

3. **Disaster Recovery and High Availability**: The document does not mention any strategies for disaster recovery or high availability. It would be helpful to include details about backup and restore procedures, failover strategies, and how the system will ensure data integrity.

...
```

`{text}` is a placeholder for the architecture description from the file.

For a step-by-step guide on setting up the review as a continuous process in GitHub, visit [here](https://github.com/xvnpw/ai-nutrition-pro-design-gpt4/blob/main/.github/workflows/architecture-ai-review.yml).

### Command Line

I've recently refactored the source code of my action and extracted its Python scripts. This led to a new repository [xvnpw/ai-threat-modeling](https://github.com/xvnpw/ai-threat-modeling), which is command-line accessible.

```bash
$ python ai-tm.py architecture --review --inputs <path_to_project>/ARCHITECTURE.md --output ARCHITECTURE_REVIEW.md --verbose
INFO:root:review of architecture started...
INFO:root:finished waiting on llm response
INFO:root:response written to file
```

### ChatGPT

For an applied example, you can view an [example review](https://chat.openai.com/share/70df15bd-551d-47f2-9c55-731f4aae4ed1) in ChatGPT.

## Conclusion

Quality input data is vital for the optimal performance of LLMs. Why not utilize the same LLMs to refine your data? Experiment with my review action, scripts, or prompts and share your experience!

---

Thank you for reading! Connect with me or follow me on [X/Twitter](https://twitter.com/xvnpw).