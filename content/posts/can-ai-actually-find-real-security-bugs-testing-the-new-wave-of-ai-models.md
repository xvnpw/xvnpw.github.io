---
title: "Can AI Actually Find Real Security Bugs? Testing the New Wave of AI Models"
date: 2025-02-14T16:00:00+01:00
draft: false
tags: [security, ai, vulnerabilities, screenshot-to-code, github, llm]
description: "A practical exploration of how well reasoning LLMs identify vulnerabilities in real-world code, comparing results across models and against a traditional SAST tool (Semgrep)."
---

Since the release of GPT-3.5, I've been experimenting with using Large Language Models (LLMs) to find vulnerabilities in source code. Initially, the results were underwhelming. LLMs frequently hallucinated or misidentified issues. However, the advent of "reasoning models" sparked my curiosity. Could these newer models, designed for more complex reasoning tasks, succeed where their predecessors struggled? This post documents my experiment to find out.

{{< figure src="https://github.com/user-attachments/assets/746d9b98-e685-481a-8b8b-16f78f4fc32d" class="image-center" width=400 >}}

## The Challenge: LLMs and Vulnerability Detection

Can LLMs *reliably* find vulnerabilities? It's a complex question. Google's [Project Zero](https://googleprojectzero.blogspot.com/2024/10/from-naptime-to-big-sleep.html) outlined a sophisticated approach involving an LLM agent, a debugger, and potentially other tools, all working together in a loop.

My experiment takes a simpler approach. I'm focusing solely on the reasoning capabilities of the models, *without* any external tools. The question is:  Have reasoning models improved to the point where they can effectively identify vulnerabilities in raw source code – a task that was previously unreliable?

## Introducing `ai-security-analyzer` and a New Prompt

I've enhanced my open-source project, [ai-security-analyzer](https://github.com/xvnpw/ai-security-analyzer), with a new prompt specifically designed for vulnerability discovery. You can use it with the `--agent-prompt-type vulnerabilities` flag:

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

This prompt works with both the `dir` (local directory) and `github` modes. While I'd be cautious about trusting it completely in `github` mode (which uses model knowledge to analyze repositories), the `dir` mode is more promising as it directly uses the project's source code and can even quote relevant lines.

### Deconstructing the Prompt

Let's break down the [`dir`](https://raw.githubusercontent.com/xvnpw/ai-security-analyzer/refs/heads/main/ai_security_analyzer/prompts/default/dir/vulnerabilities-default.yaml) mode prompt (the [`github`](https://raw.githubusercontent.com/xvnpw/ai-security-analyzer/refs/heads/main/ai_security_analyzer/prompts/default/github/vulnerabilities-default.yaml) prompt is conceptually similar):

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

Key design choices in this prompt:

* **Avoiding "Exploit":** I intentionally avoid the word "exploit". OpenAI models are often aligned *not* to generate exploit code, which can lead to output refusals. Instead, I use "security test case", a broader term that still allows for vulnerability reproduction.
* **Excluding Documentation-Related "Vulnerabilities":** Some models generated issues related to missing documentation, which are more accurately classified as *threats* rather than *vulnerabilities*. I've excluded these to avoid excessively long lists of potential issues.
* **DoS Exclusion (for now):** My initial tests showed a high frequency of Denial-of-Service (DoS) vulnerability reports. I've temporarily excluded them. *I'm considering adding an argument in the future to allow users to include/exclude specific vulnerability classes.*
* **Markdown Lists, Not Tables**:  The prompt explicitly instructs the model to use Markdown lists to avoid formatting issues in Gemini 2.0 models.

## The Experiment: Testing Against Real Code

### Methodology

A robust benchmark would require a comprehensive dataset with true/false positives and negatives. Due to time constraints, I'm using a more streamlined approach. I'll run the prompt with different LLMs against a project I'm familiar with: [screenshot-to-code](https://github.com/abi/screenshot-to-code). I'll then manually verify each reported vulnerability by attempting to reproduce it. This will give us a sense of the true positive and false positive rates for each model.

### Running `ai-security-analyzer`

I used the following command (with variations for each model):

```bash
python ai_security_analyzer/app.py \
    dir \
    -t ../screenshot-to-code/ \
    -o examples/dir-vulnerabilities-screenshot-to-code-${safe_agent_model}.md \
    --agent-model $agent_model \
    --agent-temperature ${temperatures[$agent_model]} \
    --agent-prompt-type vulnerabilities \
    --agent-provider $agent_provider
```

For details, see the testing [script](https://github.com/xvnpw/ai-security-analyzer/blob/main/create_examples.sh).

### Model Selection

I tested the following models:

* OpenAI o1
* OpenAI o3-mini-high
* Gemini 2.0 Flash Thinking Experimental
* Gemini 2.0 Pro Experimental - *Not* a dedicated reasoning model, but included for comparison.
* DeepSeek R1

## Results and Analysis

Here's a summary of the results for each model. I've included detailed outputs in the `ai-security-analyzer` repository.

#### OpenAI o1

Detailed output: [vulnerabilities.md](https://github.com/xvnpw/ai-security-analyzer/blob/main/examples/dir-vulnerabilities-screenshot-to-code-o1.md)

| Vulnerability | Valid | Comment |
| --- | --- | --- |
| Unauthenticated and Unlimited Access to Code Generation Endpoints | 🤔 | Correctly identifies the lack of authentication, but this is by design for the project. However, it's a *valid risk* if deployed publicly with a user's API key. |
| Arbitrary Local File Reading Through Evals Endpoints | ✅ | Well-described and *completely valid* vulnerability. |
| Rendering LLM-Generated HTML Without Sanitization (Possible XSS) | ✅ | Well-described and *completely valid* XSS vulnerability. |
| Middle-Man SSRF via Unrestricted Screenshot Service | 🤔 | Correctly identifies a threat to the *ScreenshotOne.com service* used by the project, not a direct vulnerability in the project's code. A valid concern, but perhaps a "nice-to-have" mitigation. |

#### OpenAI o3-mini-high

Detail output: [vulnerabilities.md](https://github.com/xvnpw/ai-security-analyzer/blob/main/examples/dir-vulnerabilities-screenshot-to-code-o3-mini.md)

| Vulnerability | Valid | Comment |
| --- | --- | --- |
| Unauthenticated and Rate‑Limit‑Free WebSocket Code Generation Abuse | 🤔 | Similar to o1, flags the lack of authentication, a design choice but a deployment risk. |
| Arbitrary File Disclosure in Evaluation Endpoints | 🤔 | Similar to o1, but less precise. Misses the crucial detail that it only affects `.html` files and incorrectly describes it as "disclosure" rather than "reading". |
| CORS Misconfiguration Allowing Wildcard with Credentials | ✅ | Correctly identifies a CORS misconfiguration, though the impact is low. |
| Insufficient Input Validation in the Screenshot API (Potential SSRF) | 🤔 | Again, identifies the threat to the external ScreenshotOne.com service, not a direct vulnerability in the project code. |
| Lack of Authentication and Authorization on Sensitive Endpoints | 🤔 | Redundant with the first vulnerability, highlighting the authentication issue. |

#### Gemini 2.0 Flash Thinking Experimental

Detail output: [vulnerabilities.md](https://github.com/xvnpw/ai-security-analyzer/blob/main/examples/dir-vulnerabilities-screenshot-to-code-gemini-2.0-flash-thinking-exp.md)

| Vulnerability | Valid | Comment |
| --- | --- | --- |
| Permissive Cross-Origin Resource Sharing (CORS) Configuration | ✅ | Correctly identifies the CORS misconfiguration (low impact). |
| Local File Inclusion (LFI) in Evals Endpoints | 🤔 | Valid, but misses the critical detail that it's limited to `.html` files, like o3-mini-high. |

#### Gemini 2.0 Pro Experimental

Detail output: [vulnerabilities.md](https://github.com/xvnpw/ai-security-analyzer/blob/main/examples/dir-vulnerabilities-screenshot-to-code-gemini-2.0-pro-exp.md)

This model produced mostly *false positives*. Examples include:

* **Cross-Site Scripting (XSS) - Stored (❌):** Incorrect. It's a self-XSS, as correctly identified by o1.
* **Arbitrary File Write (❌):** Incorrect. Refers to a debug mode that requires explicit enabling.
* **Insecure Direct Object Reference (IDOR)... (❌):** Incorrect and misnamed.
* **Lack of Input Validation for LLM Model Selection (❌):** A potential bug, but not a security vulnerability.
* **Potential Path Traversal in `image_to_data_url` (❌):** Requires the attacker to *already have directory creation privileges* on the server – a highly unlikely precondition for path traversal.

I've omitted the full list for brevity, but the performance was significantly worse than the other models.

#### DeepSeek R1

Detail output: [vulnerabilities.md](https://github.com/xvnpw/ai-security-analyzer/blob/main/examples/dir-vulnerabilities-screenshot-to-code-deepseekdeepseek-r1.md)

| Vulnerability | Valid | Comment |
| --- | --- | --- |
| Server-Side Request Forgery (SSRF) in Screenshot Endpoint | 🤔 | Again, flags the SSRF risk related to the external ScreenshotOne.com service. |
| Insecure CORS Configuration | ✅ | Correctly identifies the CORS misconfiguration. |
| XSS Risk in AI-Generated HTML Output | ✅ | Correctly identifies the XSS vulnerability. |
| Image Processing Vulnerabilities | 🤔 | Valid vulnerability, but the description is poor and includes *hallucinated source code*. |
| API Key Exposure Risk | 🤔 | A valid *risk*, but not a vulnerability in the default configuration (which uses environment variables). Highlights a potential improvement (using a secure storage). |
| Insecure Defaults in Docker Configuration | ❌ | Incorrect. The `docker-compose.yml` is intended for local use, not production. |

### Key Findings

* **Reasoning Models Show Promise:** Reasoning models demonstrably outperform earlier LLMs in vulnerability detection. They are capable of identifying genuine, exploitable vulnerabilities.
* **OpenAI o1 is the Top Performer:** It consistently provided the most accurate and detailed reports, including crucial nuances that other models missed.
* **Model Size Matters:** o3-mini-high, a smaller variant, performed worse than o1, highlighting the importance of model capacity for complex reasoning.
* **Free & Open-Source (DeepSeek R1) Holds Its Own:** Surprisingly, DeepSeek R1 outperformed Gemini 2.0 Pro and identified several valid vulnerabilities, although it did exhibit some hallucination.
* **Gemini 2.0 Pro Struggles:** The non-reasoning model performed poorly, generating numerous false positives.

## AI vs. SAST: A Comparison with Semgrep

I also ran [Semgrep](https://semgrep.dev/), a popular Static Application Security Testing (SAST) tool, on the same project to compare the results.

Semgrep (version 1.93.0) identified three vulnerabilities:

[Detailed Output](https://github.com/xvnpw/ai-security-analyzer/blob/main/examples/dir-vulnerabilities-screenshot-to-code-semgrep.txt)

| Vulnerability | Valid | Comment                                                                                                                                   |
| --- | --- | --- |
| CORS Misconfiguration | ✅ | Correctly identifies the CORS misconfiguration. |
| Missing User in Dockerfile | 🤔 | A potential misconfiguration that could be exploited in specific scenarios. |
| XSS in EJS Template | ❌ | *False positive*. The injected value in the EJS template is [static](https://github.com/abi/screenshot-to-code/blob/a240914b93d35ce17bc2263e2f9924b473e8bfa5/frontend/vite.config.ts#L18). |

### Key Takeaways: AI and SAST

* **Complementary Strengths:** The LLMs and Semgrep identified *different* types of vulnerabilities. This suggests that they can be used *complementarily* for a more comprehensive analysis.
* **AI for Contextual Understanding:** LLMs, particularly o1, demonstrated a better understanding of the project's context (e.g., the intended use of unauthenticated endpoints). SAST tools typically focus on pattern matching.
* **Semgrep's AI-Assisted Triaging:** Semgrep proposes using AI to assist in triaging findings, and their [research](https://semgrep.dev/blog/2025/building-an-appsec-ai-that-security-researchers-agree-with-96-of-the-time/) shows promising results. My experiment suggests that reasoning models can go *beyond* triaging and actively discover novel vulnerabilities.

## The Limits of Context Windows

While large context windows are a feature of modern LLMs, they don't automatically translate to perfect code analysis. My experience aligns with common observations: LLMs can struggle with very large codebases. It's often more effective to break down the code into smaller, manageable chunks and process them iteratively. In `ai-security-analyzer`, I'm currently using a context window of approximately 100k tokens.

## Conclusion: A Promising but Imperfect Tool

This experiment, while not a formal scientific benchmark (#vibes), provides valuable insights. Reasoning models represent a significant step forward in AI-powered vulnerability detection. They are no longer just "hit or miss" but can provide genuinely useful results.

The performance of OpenAI o1 and Gemini 2.0 Flash Thinking Experimental is particularly encouraging. Future work could involve:

* **Custom Input Context:** Providing additional project-specific information to the LLM.
* **Vulnerability Class Filtering:** Allowing users to include or exclude specific types of vulnerabilities.
* **Combining AI with Traditional Techniques:** Integrating AI-based analysis with SAST and other security tools for a more holistic approach.
* **Iterative Refinement:** Using the output of one LLM run as input for a subsequent run, potentially with a different model or prompt, to refine the results.

From a security analysis perspective, I recommend generating multiple document types using `ai-security-analyzer`. LLMs are not precision instruments, and diverse outputs provide a more comprehensive understanding of the project's security posture.

I encourage you to experiment with `ai-security-analyzer` and these models, especially the free and accessible Gemini 2.0 Flash Thinking Experimental.

---

Thanks for reading! You can contact me and/or follow me on [X](https://x.com/xvnpw) and [LinkedIn](www.linkedin.com/in/marcin-niemiec-304349104).