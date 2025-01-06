---
title: "Deep Analysis Mode in AI Security Analyzer"
date: 2025-01-05T14:00:00+01:00
draft: true
tags: [security, ai, gemini, threat modeling, cilium, github]
description: "Discover how the new Deep Analysis Mode in AI Security Analyzer provides in-depth security insights, with practical examples using Google's Gemini 2.0 Flash Thinking Experimental model."
---

First off, **a big thank you** 🙇 to everyone who provided such positive feedback on my previous post, [Scaling Threat Modeling with AI: Generating 1000 Threat Models Using Gemini 2.0 and AI Security Analyzer]({{< ref "posts/scaling-threat-modeling-with-ai.md" >}}). Your insights and suggestions have been incredibly valuable.

Inspired by your comments, I've added a new feature to the AI Security Analyzer: **Deep Analysis Mode**. In this post, I'll walk you through how it works and showcase its capabilities using Google's Gemini 2.0 Flash Thinking Experimental model to perform an in-depth threat modeling on the [Flask](https://github.com/pallets/flask) project. We'll compare outputs between Normal Mode and Deep Analysis Mode to highlight the differences.

## Motivation

Some of you asked for more detailed analyses of projects in the `sec-docs` repository. I wasn't sure if Gemini 2.0 could provide deeper insights, so I decided to try it out. With Google planning to remove free access to Gemini 2.0 Flash soon, I rushed to implement this feature. The final results aren't perfect, but I believe they can be quite useful in many cases.

## Activating Deep Analysis

Enabling deep analysis is straightforward. You simply add the `--deep-analysis` flag when running the tool in **github** mode:

```bash
poetry run python ai_security_analyzer/app.py \
    github \
    -t https://github.com/user/repo \
    -o output.md \
    --agent-prompt-type threat-modeling \
    --deep-analysis
```

## The Multi-Document Approach

What makes deep analysis different from normal mode is its multi-document output strategy. Depending on what you're analyzing, the tool generates different sets of documentation:

```markdown
Output Structure:
📄 Main Document (output.md)
└── Detailed Analysis Files:
    ├── 🎯 threats/*.md            # For threat modeling
    ├── 🔍 attack_surfaces/*.md    # For attack surface analysis
    ├── 🌳 attack_tree_paths/*.md  # For attack tree analysis
    └── 🔒 output-deep-analysis.md # For security design
```

This structured approach means you get both a high-level overview and detailed deep dives into specific aspects of your security analysis.

## A Word of Caution

While Deep Analysis Mode provides richer insights, it's essential to be mindful of a few points:

- **Currently limited to `github` mode**
- **Outputs require careful verification**
- **Potential for AI hallucinations**
- **Cost Implications**: Currently, using Gemini 2.0 Flash is free (as of January 5, 2025), but this might change. Deep Analysis Mode doesn't support the `--dry-run` flag, so you can't get an estimated cost upfront. However, you can refer to the [output-metadata.json](https://github.com/xvnpw/sec-docs/blob/0fb6183ff9d58e719685fd498efeffdea370fe98/go/cilium/cilium/2025-01-05-gemini-2.0-flash-thinking-exp/output-metadata.json) from previous executions to gauge potential costs. For instance, analyzing Flask cost around 180,000 tokens (`"actual_token_usage": "175994"`).

## Comparing Normal vs Deep Analysis

Normal Mode provides an overview with easily digestible documents, serving as a starting point for further analysis or as a checklist for potential issues. Deep Analysis Mode, on the other hand, offers a much more detailed examination. However, depending on the model used, the outputs might:

- Contain deeper, specific analyses (**preferred**)
- Include more general, common threats (**less preferred**)
- Contain verbose text without added value (**less preferred**)

It's important to note that Deep Analysis Mode starts with the same initial steps as Normal Mode. We need a high-level overview of the project before we can delve deeper.

## Prompts

Prompts are very simple. For example, for threat modeling, we use the following prompt:

```
GITHUB2_GET_THREAT_DETAILS_PROMPT = """You are cybersecurity expert, working with development team. Your task is to create deep analysis of particular threat from threat model for application that is using {}.

THREAT:
{}

{}
"""
```

For `{}` I will provide:
1. GitHub repository URL, e.g. https://github.com/pallets/flask
2. Threat title from threat model
3. Threat description from threat model

You can check other prompts for deep analysis in the [prompts.py](https://github.com/xvnpw/ai-security-analyzer/blob/337cd9aa56e9ae66017f04fe3fdbb3c2f855a21e/ai_security_analyzer/prompts.py#L844C1-L850C4) file and see how they are used in [github2tm_agents.py](https://github.com/xvnpw/ai-security-analyzer/blob/337cd9aa56e9ae66017f04fe3fdbb3c2f855a21e/ai_security_analyzer/github2tm_agents.py#L165).

## Deep Analysis of Flask framework

To illustrate the capabilities of Deep Analysis Mode, let's compare the outputs of Normal Mode and Deep Analysis Mode for the Flask framework, focusing specifically on the threat modeling documents.

### Threats Identified in Normal Mode

In the previous post, the Normal Mode analysis of Flask identified the following **five threats** ([github](https://github.com/xvnpw/ai-security-analyzer/blob/main/examples/GITHUB-THREAT-MODEL-FLASK-gemini-2.0-flash-thinking-exp.md)):

1. **Route Parameter Injection**
2. **Insecure Session Cookie Configuration**
3. **Debug Mode Enabled in Production**
4. **Blueprint Route Conflicts and Overlapping**
5. **Incorrect HTTP Method Handling**

### Threats Identified in Deep Analysis Mode

After enabling Deep Analysis Mode, the analysis resulted in **four threats** ([github](https://github.com/xvnpw/sec-docs/blob/main/python/pallets/flask/2025-01-03-gemini-2.0-flash-thinking-exp/threat-modeling.md)):

1. **Server-Side Template Injection (SSTI)**
2. **Insecure Secret Key Management**
3. **Exposure of Debug Mode in Production**
4. **Session Fixation**

🤔 **Note**: The variance in the number of threats is due to the inherent randomness in LLM outputs, influenced by the temperature setting (set at 0.5 in this case).

### Common Threat: Debug Mode Enabled in Production

Both modes identified the threat related to **Debug Mode in Production**. Let's examine how each mode handles this threat.

#### Normal Mode Output

```markdown
*   **Threat:** Debug Mode Enabled in Production
    *   **Description:** Running a Flask application with `debug=True` configures the `flask.Flask` application to expose an interactive debugger in the browser when an error occurs. Attackers can exploit this debugger to execute arbitrary code on the server, access sensitive information, and potentially gain full control of the application.
    *   **Impact:** Complete server compromise, data breaches, denial of service.
    *   **Affected Flask Component:** `flask.Flask`, `debug` configuration parameter.
    *   **Risk Severity:** Critical
    *   **Mitigation Strategies:**
        *   **Never** run Flask applications with `debug=True` in production environments. Ensure `app.debug = False` or the `FLASK_DEBUG=0` environment variable is set.
        *   Implement proper logging and error reporting mechanisms for production.
```

This output provides a concise and actionable summary of the threat, its impact, and mitigation strategies.

#### Deep Analysis Mode Output

In Deep Analysis Mode, the threat is explored in much greater depth. The analysis includes:

1. **Understanding the Threat in Detail**
2. **Attack Vectors and Exploitation Scenarios**
3. **Detailed Impact Analysis**
4. **Prevention Strategies (Elaborated)**
5. **Detection Strategies**
6. **Remediation Steps (If Debug Mode is Found Enabled)**
7. **Communication and Collaboration**
8. **Conclusion**

**Example Excerpt from Deep Analysis Mode:**

```markdown
**1. Understanding the Threat in Detail:**

While the initial description provides a good overview, let's delve deeper into the mechanics and implications of running a Flask application with debug mode enabled in a production environment.

* **The Nature of Flask's Debug Mode:**
  * **Automatic Code Reloading:** Any changes to the application's Python code will automatically restart the server. While useful during development, this can lead to unexpected downtime and instability in production if files are inadvertently modified or if the reloading process encounters errors.
  * **Interactive Debugger Exposure:** The Werkzeug debugger allows for code execution within the browser. If exposed, an attacker can execute arbitrary code on the server.

...

**2. Attack Vectors and Exploitation Scenarios:**

How could an attacker exploit this vulnerability?

* **Direct Access to Error Pages:**
  * **Submitting Invalid Input:** Crafting malicious requests designed to cause exceptions.
  * **Exploiting Existing Vulnerabilities:** Leveraging other vulnerabilities that lead to errors.
  * **Accessing Non-Existent Routes:** Triggering 404 errors that reveal application structure.

...

**3. Detailed Impact Analysis:**

* **Information Disclosure (Critical):**
  * Exposure of source code, configuration details, and local variables.
  * Attackers gain significant insights into the application's security mechanisms.

...

**4. Prevention Strategies (Elaborated):**

* **Explicitly Disable Debug Mode in Production:**
  * **Environment Variables (`FLASK_DEBUG`):** Set `FLASK_DEBUG=0` or `False` in production environments.
  * **Application Configuration (`app.debug`):** Ensure `app.debug = False`.

...

**5. Detection Strategies:**

* **Manual Inspection:** Check environment variables and application configuration.
* **Automated Checks:** Implement scripts or CI/CD pipeline checks to ensure `FLASK_DEBUG` is disabled.

...

**6. Remediation Steps (If Debug Mode is Found Enabled):**

1. **Immediately Disable Debug Mode.**
2. **Investigate for Potential Compromise:** Review logs and monitor alerts.
3. **Assess and Update Security Policies.**

...

**7. Communication and Collaboration:**

* **Clear Communication:** Ensure all team members understand the risks.
* **Training:** Provide security awareness training.

...

**Conclusion:**

Running Flask's debug mode in production poses a severe security risk. Immediate steps should be taken to disable it and secure the application.

```

#### Analysis of the Outputs

While the Deep Analysis Mode provides a more exhaustive examination, including attack vectors, detailed impacts, and remediation steps, it's worth considering:

- **Value Addition**: The extra details can be beneficial for deeper understanding and planning comprehensive security measures.
- **Relevance**: Some sections may contain generalized information that doesn't add significant value to seasoned professionals.
- **Time Investment**: Reviewing longer documents requires more time and may include redundant information.

## Lessons Learned

Key Takeaways:

- Choose between modes based on your specific needs
- Consider the trade-offs between depth and efficiency
- Always verify AI-generated outputs

## `sec-docs` Repository

I generated a new set of documentation for 1,000 projects in my [sec-docs](https://github.com/xvnpw/sec-docs) repository. You can find it by browsing the language-specific folders or by looking at specific projects:

{{< figure src="https://github.com/user-attachments/assets/3c85e401-c11e-4034-bfff-b97306748e63" class="image-center" >}}

## Looking Forward

I plan to continue refining the tool and explore how other models, especially those from OpenAI, perform in Deep Analysis Mode.

## Want to Try It Yourself?

The AI Security Analyzer tool is available on [GitHub](https://github.com/xvnpw/ai-security-analyzer), and I encourage you to experiment with both modes. Your feedback is highly appreciated.

---

Thanks for reading! You can contact me and/or follow me on [X](https://x.com/xvnpw) and [LinkedIn](www.linkedin.com/in/marcin-niemiec-304349104).