---
title: "How good reasoning models are in finding vulnerabilities?"
date: 2025-02-16T09:00:00+01:00
draft: true
tags: [security, ai, threat modeling, flask, github, llm]
description: "I will try find out how good reasoning models are in finding vulnerabilities in source code. Will go through prompt, models and real world examples."
---

Since GPT-3.5, I was testing LLMs on finding vulnerabilities in source code. It was not particularly good. LLMs tend to hallucinate or just flag something wrongly. I was curious how reasoning models are in finding vulnerabilities.

{{< figure src="https://github.com/user-attachments/assets/746d9b98-e685-481a-8b8b-16f78f4fc32d" class="image-center" width=400 >}}

## Theoretical Background

Is it even possible for LLMs to find vulnerabilities in source code? It's but not so easy. Google [Project Zero](https://googleprojectzero.blogspot.com/2024/10/from-naptime-to-big-sleep.html) proposed a necessary ingredients for finding vulnerabilities in source code. Those are:

1. LLM agent
2. Debugger tool that can be used by agent
3. Possibly other tools - but article was not explicit about that
4. Workflow that put together agent and tools in the loop

I'm far from using this approach. My idea is to test how good reasoning models are in finding vulnerabilities in source code. Without any tools. Does something that was not possible before, is possible now with new models?

## New prompt in ai-security-analyzer

In [ai-security-analyzer](https://github.com/xvnpw/ai-security-analyzer) you can use `--agent-prompt-type vulnerabilites` to generate vulnerabilities, like this:

```bash
python ai_security_analyzer/app.py \
    dir \
    -t ../path/to/project \
    -o vulnerabilities.md \
    --agent-prompt-type vulnerabilities \
    --agent-provider google \
    --agent-model gemini-2.0-flash-thinking-exp \
    --agent-temperature 0.7
```

This new prompt is working with `dir` and `github` modes. In `github` mode, I would be very far from trusting it. In `dir` mode, it's using project source code directly and can even quote lines with suspected vulnerabilities.

### The prompt

Let's break down the prompt for `dir` mode (for `github` mode, it's a similar):

```python
# set persona and goal
"""You are cybersecurity expert, specialized in finding vulnerabilities in source code and writing security test cases. Your task is to create list of vulnerabilities for application from PROJECT FILES. Focus on vulnerabilities introduced by application from PROJECT FILES. Assume that threat actor is external attacker that will try to trigger vulnerability in publicly available instance of application."""

# instruct model how to create vulnerability list
"""Create vulnerability list with: vulnerability name, description (describe in details step by step how someone can trigger vulnerability), impact (describe the impact of the vulnerability), vulnerability rank (low,medium,high or critical), currently implemented mitigations (describe if this vulnerability is mitigated in the project and where), missing mitigations (describe what mitigations are missing in the project), preconditions (describe any preconditions that are needed to trigger this vulnerability), source code analysis (go step by step through code and describe how vulnerability can be triggered; if needed use visualization; be detail and descriptive), security test case (describe step by step test for the vulnerability to prove it's valid; assume that threat actor will be external attacker with access to publicly available instance of application)."""

# instruct model how to select vulnerabilities
"""Include only vulnerabilities that are valid and not already mitigated. This should be a complete list of vulnerabilities that addresses the real-world risk to the system in question, as opposed to any fantastical concerns that the input might have included. Exclude vulnerabilities that are only missing documentation to mitigate and deny of service class of vulnerabilities. Focus on high and critical vulnerabilities first."""

# instruction about input - PROJECT FILES
"""I will give you PROJECT FILES and CURRENT VULNERABILITIES. When the CURRENT VULNERABILITIES is not empty, it indicates that a draft of this document was created in previous interactions using earlier batches of PROJECT FILES. In this case, integrate new findings from current PROJECT FILES into the existing CURRENT VULNERABILITIES. Ensure consistency and avoid duplication. If the CURRENT VULNERABILITIES is empty, proceed to create a new vulnerabilities based on the current PROJECT FILES. The PROJECT FILES will contain typical files found in a GitHub repository, such as configuration files, scripts, README files, production code, testing code, and more. Use valid markdown formatting. Don't use markdown tables at all, use markdown lists instead."""
```

No magic here. Very straight forward prompt. Few notes:

- I don't use word 'exploit' in the prompt. I noticed that OpenAI models are align not to create exploits. They may refused to create output in such case. I replaced it with 'security test case' which is more general, but in the end allows to reproduce the vulnerability.
- I instructed model to exclude vulnerabilities that are only missing documentation - some models were returning vulnerabilities that might occur in some environments, but those are more **threats** than **vulnerabilities**. I excluded those to avoid long lists of potential vulnerabilities.
- I also instructed model to exclude DoS vulnerabilities. From my tests, it was returning DoS vulnerabilities very often. _I'm not sure if this is good decision, but maybe in future I will add argument to include/exclude particular classes of vulnerabilities._

## Testing

### How to test for vulnerabilities?

Every good benchmark for finding vulnerabilities in source code would include true/false positives/negatives. I'm not able to spent so much time on it, so I will use very simple approach. Will run my prompt with different models on single project that I know: [screenshot-to-code](https://github.com/abi/screenshot-to-code). Then I will follow steps to reproduce vulnerabilities and see if those are valid or not. In the end, we will see how much true and false positives we got per model.

### Models

I will use those models:

- OpenAI o1
- OpenAI o3-mini-high
- Gemini 2.0 Flash Thinking Experimental
- Gemini 2.0 Pro - not reasoning model, but I will use it for comparison
- DeepSeek R1

### Results

#### OpenAI o1

Detail output: [vulnerabilities.md](https://github.com/xvnpw/ai-security-analyzer/blob/main/examples/dir-vulnerabilities-screenshot-to-code-o1.md)

| Vulnerability | Valid | Comment |
| --- | --- | --- |
| Unauthenticated and Unlimited Access to Code Generation Endpoints | 🤔 | Screenshot to Code project was created as unauthenticated application. In that sense it's not a vulnerability. But in case someone will put it in public with own API key, it will be a vulnerability. |
| Arbitrary Local File Reading Through Evals Endpoints | ✅ | Well described and valid vulnerability. |
| Rendering LLM-Generated HTML Without Sanitization (Possible XSS) | ✅ | Well described and valid vulnerability. |
| Middle-Man SSRF via Unrestricted Screenshot Service | 🤔 | In true it's not vulnerability for Screenshot to Code project, but for ScreenshotOne.com service. It's valid threat and possibly should be mitigated, but rather as nice to have than a must-have. |

#### OpenAI o3-mini-high

Detail output: [vulnerabilities.md](https://github.com/xvnpw/ai-security-analyzer/blob/main/examples/dir-vulnerabilities-screenshot-to-code-o3-mini.md)

| Vulnerability | Valid | Comment |
| --- | --- | --- |
| Unauthenticated and Rate‑Limit‑Free WebSocket Code Generation Abuse | 🤔 | (Same as for o1) Screenshot to Code project was created as unauthenticated application. In that sense it's not a vulnerability. But in case someone will put it in public with own API key, it will be a vulnerability. |
| Arbitrary File Disclosure in Evaluation Endpoints | ✅ | o1 also returned this vulnerability, but with different description. o3-mini-high didn't point out that it's only affecting `.html` files. |
| CORS Misconfiguration Allowing Wildcard with Credentials | ✅ | CORS misconfiguration is valid, but impact is low. |
| Insufficient Input Validation in the Screenshot API (Potential SSRF) | 🤔 | (Same as for o1) In true it's not vulnerability for Screenshot to Code project, but for ScreenshotOne.com service. It's valid threat and possibly should be mitigated, but rather as nice to have than a must-have. |
| Lack of Authentication and Authorization on Sensitive Endpoints | 🤔 | As described in WebSocket Code Generation Abuse vulnerability, it's only vulnerability in case of public instance. |

#### Gemini 2.0 Flash Thinking Experimental

Detail output: [vulnerabilities.md](https://github.com/xvnpw/ai-security-analyzer/blob/main/examples/dir-vulnerabilities-screenshot-to-code-gemini-2.0-flash-thinking-exp.md)

| Vulnerability | Valid | Comment |
| --- | --- | --- |
| Permissive Cross-Origin Resource Sharing (CORS) Configuration | ✅ | CORS misconfiguration is valid, but impact is low. |
| Local File Inclusion (LFI) in Evals Endpoints | 🤔 | It's valid vulnerability, but only limited to `.html` files. Gemini didn't point out that it's only affecting `.html` files. |

#### Gemini 2.0 Pro Experimental

Detail output: [vulnerabilities.md](https://github.com/xvnpw/ai-security-analyzer/blob/main/examples/dir-vulnerabilities-screenshot-to-code-gemini-2.0-pro-exp.md)

| Vulnerability | Valid | Comment |
| --- | --- | --- |
| Cross-Site Scripting (XSS) - Stored | ❌ | It's not valid vulnerability. There is self XSS as described by o1 model |
| Arbitrary File Write | ❌ | It's not valid vulnerability. It's just debug mode that need to be explicitly enabled. |
| Insecure Direct Object Reference (IDOR) in run_evals.py | ❌ | It's not valid vulnerability. Description is not matching name of vulnerability. |
| Lack of Input Validation for LLM Model Selection | ❌ | It might be bug, but it's not vulnerability. There is no impact on security. |
| Potential Path Traversal in image_to_data_url | ❌ | One of preconditions is that user need to create directory on server. I guess bar is already high like for path traversal 😅 |
...

And there are could more false positives. I will not list them all here.

#### DeepSeek R1

Detail output: [vulnerabilities.md](https://github.com/xvnpw/ai-security-analyzer/blob/main/examples/dir-vulnerabilities-screenshot-to-code-deepseekdeepseek-r1.md)

| Vulnerability | Valid | Comment |
| --- | --- | --- |
| Server-Side Request Forgery (SSRF) in Screenshot Endpoint | 🤔 | (Same as before) In true it's not vulnerability for Screenshot to Code project, but for ScreenshotOne.com service. It's valid threat and possibly should be mitigated, but rather as nice to have than a must-have. |
| Insecure CORS Configuration | ✅ | CORS misconfiguration is valid, but impact is low. |
| XSS Risk in AI-Generated HTML Output | ✅ | It's valid vulnerability. |
| Image Processing Vulnerabilities | 🤔 | It's valid vulnerability, but description is not good. It even contains hallucinations of source code! |
| API Key Exposure Risk | 🤔 | It's valid risk, but not vulnerability. By default application is storing API key in environment variable, which is not bad. But can be improved with secure storage. |
| Insecure Defaults in Docker Configuration | ❌ | It's not valid vulnerability. docker-compose.yml is for local usage not production. |

### Key takeaways

- Reasoning models are good in finding vulnerabilities in source code. Definitely better than previous models.
- OpenAI o1 model is better then o3-mini-high. For example, pointed out `.html` files in Arbitrary File Disclosure in Evaluation Endpoints vulnerability.
- Surprisingly free and open DeepSeek R1 model is better than Gemini 2.0 Pro.

## Context windows

Latest models are able to take a lot input and put it in context. But it's not working that good in code analysis. If you have been using LLMs as coding assistant, you probably know what I'm talking about. I would recommend not to set context windows too high. It's better to split code into smaller chunks and loop through them.

## Summary

My experiment is kind of "vibe testing". I know it. It's not scientific. But it shows a way for future experiments for me. Before reasoning models I wasn't motivated to use LLMs for finding vulnerabilities. It was always hit or miss. Now it has much more sense to use them. Results returned by OpenAI o1 and Gemini 2.0 Flash Thinking Experimental are interesting. I would like to future add custom input context with additional information about project. As well as classes of vulnerabilities that should be excluded (or included). 

From security analysis perspective, I recommend to generate different types of documents available in ai-security-analyzer. LLMs are not precise tools, and having more different documents will help to get better understanding of project. In the end, LLM should be helper and not a replacement for human.

---

Thanks for reading! You can contact me and/or follow me on [X](https://x.com/xvnpw) and [LinkedIn](www.linkedin.com/in/marcin-niemiec-304349104).