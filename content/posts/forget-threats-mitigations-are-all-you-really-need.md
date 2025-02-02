---
title: "Forget Threats, Mitigations are All You REALLY Need"
date: 2025-02-02T09:00:00+01:00
draft: false
tags: [security, ai, threat modeling, flask, github, llm]
description: "A practical perspective shift for security professionals: Learn why focusing on concrete mitigations rather than abstract threats leads to better developer engagement and more secure software. Featuring hands-on examples using AI-powered security analysis tools and real-world project implementations."
---

Ever feel like you're speaking a different language when you talk to developers about security? You're buzzing with threat models, attack vectors, and vulnerabilities, while they're focused on features, deadlines, and, well, *making things work*. I get it. I've been there. And recently, I had a bit of an "aha!" moment that completely shifted my perspective on security discussions.

It happened during a review of security analysis with my colleagues at Form3. We were diving deep into the [terraform-provider-chronicle](https://github.com/form3tech-oss/terraform-provider-chronicle) project (you can see analysis [here](https://github.com/xvnpw/ai-security-analyzer/tree/main/examples/form3tech-oss/README.md), using my [ai-security-analyzer](https://github.com/xvnpw/ai-security-analyzer) tool). As we talked through AI generated threats, it struck me: to make AI tools useful for developers, we need to focus on mitigations, not threats. Developers are less interested in abstract threats and far more engaged when you talk about **mitigations** – concrete steps to *fix* things.

This wasn't a completely new idea, but this time it really clicked. I'd been using AI to generate threat models, asking Large Language Models (LLMs) to identify threats and *then* suggest mitigations. But a question popped into my head: What if I flipped the script? What if I asked the LLM for **mitigations directly**, and *then* explored the threats those mitigations address?

Guess what—it works!

{{< figure src="https://github.com/user-attachments/assets/76ad96e6-ed5f-4d40-8565-00712f46beeb" class="image-center" width=400 >}}

## How to Use This Approach

In [ai-security-analyzer](https://github.com/xvnpw/ai-security-analyzer) you can use `--agent-prompt-type mitigations` to generate mitigations, like this:

```bash
python ai_security_analyzer/app.py \
    dir \
    -t ../path/to/project \
    -o mitigations.md \
    --agent-prompt-type mitigations \
    --agent-provider google \
    --agent-model gemini-2.0-flash-thinking-exp \
    --agent-temperature 0
```

### Choosing the Right Model

It's best to use any reasoning models, like Gemini 2.0 Flash Thinking Experimental, or OpenAI's `o-family`. You can use DeepSeek R1, but it's not as good as those mentioned before.

## Practical Example

Let's see this in action! I put it to the test on the [screenshot-to-code](https://github.com/abi/screenshot-to-code) project. The results were impressive. Here's a snippet focusing on Input Validation and Sanitization – notice how it breaks down *exactly* what needs to be done and *why* it matters:

```markdown
- Mitigation Strategy: Input Validation and Sanitization
  - Description:
    1. **File Path Validation:** In `evals.py`, thoroughly validate user-provided folder paths to prevent path traversal vulnerabilities. Ensure paths are within expected directories and sanitize input to remove malicious characters.
    2. **Limit Input File Sizes:** Implement limits on the size of uploaded screenshots and video files in `generate_code.py` and `video/utils.py` to prevent excessively large files that could cause resource exhaustion or DoS.
    3. **File Type Validation:** Validate the file types of uploaded images and videos to ensure they are expected formats (e.g., PNG, JPG, MOV, MP4) in `generate_code.py` and `video/utils.py`.
    4. **Image/Video Processing Limits:** Implement safeguards in image/video processing functions (`image_processing/utils.py`, `video/utils.py`) to prevent processing of maliciously crafted files that could exploit vulnerabilities in image/video libraries (e.g., Pillow, moviepy). Consider using secure processing libraries and keeping them updated.
    5. **URL Validation:** In `screenshot.py`, validate the input URL to `capture_screenshot` to prevent unexpected behavior or SSRF vulnerabilities. Use a URL parsing library to ensure the URL is well-formed and potentially restrict allowed schemes (e.g., `http`, `https`).
    6. **Content Security Policy (CSP):** Implement a Content Security Policy in the frontend to mitigate potential XSS risks if the generated code is directly rendered in the application.
  - Threats Mitigated:
    - Path Traversal (Medium Severity): Prevents attackers from accessing files outside of the intended directories in `evals.py`.
    - Denial of Service (DoS) (Medium Severity): Prevents resource exhaustion from processing excessively large files or maliciously crafted files.
    - Code Injection (Indirect) (Low to Medium Severity): Reduces the risk of vulnerabilities in generated code by limiting input types and sizes, and through CSP (though indirect).
    - Server-Side Request Forgery (SSRF) (Low Severity): Reduces the risk of unintended external requests via `screenshot.py`.
  - Impact:
    - Path Traversal: Medium reduction in risk. Prevents unauthorized file access.
    - Denial of Service (DoS): Low to Medium reduction in risk. Makes it harder to cause resource exhaustion through malicious inputs.
    - Code Injection (Indirect): Low reduction in risk. Provides a layer of defense against potential vulnerabilities in generated code.
    - Server-Side Request Forgery (SSRF): Low reduction in risk. Limits the scope of potential SSRF issues.
  - Currently Implemented:
    - Not explicitly implemented. File path validation, file type validation and size limits are not evident in the provided files. `image_processing/utils.py` and `video/utils.py` exist for image/video processing, but security aspects are not detailed. URL validation in `screenshot.py` is missing.
  - Missing Implementation:
    - File path validation in `evals.py`.
    - Input file size limits in `generate_code.py` and `video/utils.py`.
    - File type validation in `generate_code.py` and `video/utils.py`.
    - Security review of image/video processing logic and libraries in `image_processing/utils.py` and `video/utils.py`.
    - URL validation in `screenshot.py`.
    - Content Security Policy (CSP) implementation in frontend.
```

I was genuinely impressed with the detail and actionable insights from Gemini 2.0 Flash Thinking Experimental. It's a significant step up in terms of practical security guidance. You can explore the complete output [here](https://github.com/xvnpw/ai-security-analyzer/blob/main/examples/dir-mitigations-screenshot-to-code-gemini-2.0-flash-thinking-exp.md) and see the full range of mitigation strategies it identified.

## Crafting Effective Prompt

You might be surprised at how straightforward the prompt is. Inspired by my experience from [Scaling Threat Modeling with AI: Generating 1000 Threat Models Using Gemini 2.0 and AI Security Analyzer]({{< ref "posts/scaling-threat-modeling-with-ai.md" >}}), I've learned that reasoning models thrive on clear, open-ended questions. Here's the prompt I used:

```text
You are cybersecurity expert, working with development team that is building application described in PROJECT FILES. Your task is to create mitigation strategies for application from PROJECT FILES. Focus on mitigation strategies for threats introduced by application from PROJECT FILES and omit general, common mitigation strategies. Use valid markdown formatting. Don't use markdown tables at all, use markdown lists instead. Create mitigation strategies list with: mitigation strategy, description (describe in details step by step how can developers or users reduce the risk), list of threats mitigated (describe what threats are mitigated and what is their severity), impact (describe the impact of the mitigation strategy - how much risk is reduced for each threat), currently implemented (describe if this mitigation strategy is currently implemented in the project and where), missing implementation (describe where this mitigation strategy is missing in the project). I will give you PROJECT FILES and CURRENT MITIGATION STRATEGIES. When the CURRENT MITIGATION STRATEGIES is not empty, it indicates that a draft of this document was created in previous interactions using earlier batches of PROJECT FILES. In this case, integrate new findings from current PROJECT FILES into the existing CURRENT MITIGATION STRATEGIES. Ensure consistency and avoid duplication. If the CURRENT MITIGATION STRATEGIES is empty, proceed to create a new mitigation strategies based on the current PROJECT FILES. The PROJECT FILES will contain typical files found in a GitHub repository, such as configuration files, scripts, README files, production code, testing code, and more.
```

## sec-docs Update

I've updated the [sec-docs](https://github.com/xvnpw/sec-docs) repository with new mitigations for 1,000 projects. You can browse through language-specific folders or look for specific projects:

{{< figure src="https://github.com/user-attachments/assets/35179ffd-c852-4398-bc2c-a0ee8c90de01" class="image-center" >}}

I will soon update sec-docs with the latest version of Gemini 2.0 Flash Thinking Experimental model, which extended its knowledge cutoff date to August 2024.

## Summary

This journey has shifted my perspective significantly. As someone who previously focused heavily on threats, vulnerabilities, and hacks, I initially found it challenging to accept that threat models might not be the primary focus. However, after implementing threat modeling programs across various organizations, my viewpoint has evolved. I've realized something crucial: **perfect threat models don't ship secure software**.

While the security industry offers extensive resources for threat modeling - manifesto, capabilities, maturity models, and numerous expert presentations - we often hit roadblocks when trying to integrate these into development workflows. It's not about blaming developers; it's about recognizing that **change is hard**, especially when it feels abstract or disconnected from immediate tasks.

For those building application security programs, I offer two key suggestions:

1. Focus on mitigations. It becomes easier to accept imperfectly formatted threat models when you know developers are actively preventing security issues.
2. Read "Application Security Program Handbook" by Derek Fisher for practical program management advice.

Ultimately, security isn't about finding all the threats; it's about building robust defenses. 

---

Thanks for reading! You can contact me and/or follow me on [X](https://x.com/xvnpw) and [LinkedIn](www.linkedin.com/in/marcin-niemiec-304349104).