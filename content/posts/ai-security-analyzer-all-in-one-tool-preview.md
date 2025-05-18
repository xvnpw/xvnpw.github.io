---
title: "AI Security Analyzer - All-in-One Tool Preview"
date: 2024-12-19T08:59:02+01:00
draft: false
tags: [security, llm, gpt, ai, github, langchain]
description: "Preview of the AI Security Analyzer - a new tool that leverages AI to automatically generate comprehensive security design documentation for your projects."
---

[AI Security Analyzer](https://github.com/xvnpw/ai-security-analyzer) is my latest project - a powerful tool that leverages AI to automatically generate comprehensive security documentation for your projects. Whether you're dealing with security design, threat modeling, attack surface analysis, or more, this tool aims to simplify and enhance your security documentation process.

{{< figure src="https://github.com/user-attachments/assets/86ebd729-24ef-48cf-b704-3f42a8e34162" class="image-center" width=200 >}}

**Watch the demo:**
{{< figure src="https://github.com/user-attachments/assets/a9de6ce7-9702-4fae-97a4-424d03a683eb" class="image-center"  >}}

## Threats vs. Vulnerabilities

Before we dive in, I want to clarify an important distinction: **threats** vs. **vulnerabilities**. My tool focuses on identifying threats, which are **potential risks** that could be exploited by an attacker. Vulnerabilities, on the other hand, are specific weaknesses in a system that can be exploited. If you're looking for tools to identify vulnerabilities, I recommend checking out [Google Project Zero](https://googleprojectzero.blogspot.com/2024/10/from-naptime-to-big-sleep.html).

## Background and Motivation

This isn't my first venture into using AI for generating security documentation. I've previously developed the [AI Threat Modeling Action](https://github.com/xvnpw/ai-threat-modeling-action) and contributed to the [Threat Modeling Fabric Pattern](https://github.com/danielmiessler/fabric/blob/main/patterns/create_stride_threat_model/system.md).

While both projects have been valuable, they came with limitations in flexibility and ease of use. That's why I decided to create the AI Security Analyzer. By leveraging [LangGraph](https://academy.langchain.com/courses/intro-to-langgraph), I can create more complex workflows and generate comprehensive, tailored security documents.

## 5 Different Types of Documents

Application can generate 5 different types of documents:

- 🔒 Security Design Documentation - Generate security design documentation for your project.
- 🎯 Threat Modeling - Perform threat modeling for your project using STRIDE method for application, deployment and build.
- 🔍 Attack Surface Analysis - Analyze the attack surface of your project to identify potential entry points.
- ⚠️ Threat Scenarios - Perform threat scenarios analysis for your project using [Daniel Miessler's](https://danielmiessler.com/) [prompt](https://github.com/danielmiessler/fabric/blob/f5f50cc4c94a539ee56bc533e9b1194eb9aa424d/patterns/create_threat_scenarios/system.md)
- 🌳 Attack Tree Analysis - Create attack trees to visualize potential attack vectors and their hierarchies.

You can find the specific prompts for each document type in the [prompts.py](https://github.com/xvnpw/ai-security-analyzer/blob/main/ai_security_analyzer/prompts.py#L92) file.

## 3 Different Ways to Generate Documents

- **`dir` Mode**: Analyze a local directory by sending all (or filtered) files to the LLM. Ideal for existing projects where you want a comprehensive analysis.
- **`github` Mode**: Analyze a GitHub repository using the LLM's knowledge base. This is effective for public repositories and models trained on GitHub data.
- **`file` Mode**: Analyze a single file. This mode is similar to my previous projects and is useful for focused analysis.

## Practical Use Cases

1. **Early Stage Security Review**: Quick security assessment for new projects or codebases.
2. **Documentation Generation**: Automate creation of security design docs for compliance requirements.
3. **Security Knowledge Base**: Generate baseline security documentation that can be refined by security experts.
4. **Continuous Security Assessment**: Integrate into CI/CD pipeline for ongoing security documentation updates.

## How to Use the AI Security Analyzer

Before getting started, make sure you've checked the [prerequisites and installation instructions](https://github.com/xvnpw/ai-security-analyzer#prerequisites). Now, let's walk through an example using `dir` mode to analyze another one of my projects, [fabric-agent-action](https://github.com/xvnpw/fabric-agent-action).

### Example Command

```bash
git clone https://github.com/xvnpw/fabric-agent-action # Clone the fabric-agent-action repository
cd ai-security-analyzer # Navigate to the ai-security-analyzer directory
poetry run python ai_security_analyzer/app.py dir -v -t /path/to/fabric-agent-action/ --dry-run --exclude "**/prompts/**" -o FABRIC-AGENT-ACTION-o1-preview.md --agent-model o1-preview --agent-temperature 1 --agent-prompt-type sec-design
```

#### Breaking Down the Command:

- **`dir`**: Specifies that we want to analyze a directory.
- **`-v`**: Enables verbose mode for detailed logging.
- **`-t /path/to/fabric-agent-action/`**: Sets the target directory for analysis.
- **`--dry-run`**: Performs a dry run to show what would happen without making API calls.
- **`--exclude "**/prompts/**"`**: Excludes the `prompts` directory using a glob pattern.
- **`-o FABRIC-AGENT-ACTION-o1-preview.md`**: Specifies the output file.
- **`--agent-model o1-preview`**: Sets the LLM model to `o1-preview`.
- **`--agent-temperature 1`**: Sets the temperature for the LLM. For `o1-preview`, only a temperature of 1 is accepted.
- **`--agent-prompt-type sec-design`**: Specifies that we want to generate a security design document.

### Understanding the Output

In dry run mode, you'll see output similar to this:

```bash
2024-12-19 11:11:44,844 - __main__ - INFO - Starting AI Security Analyzer
2024-12-19 11:11:45,990 - ai_security_analyzer.full_dir_scan - INFO - Loading files
2024-12-19 11:11:51,205 - ai_security_analyzer.full_dir_scan - INFO - Loaded 27 documents
2024-12-19 11:11:51,209 - ai_security_analyzer.full_dir_scan - INFO - Sorting and filtering documents
2024-12-19 11:11:51,209 - ai_security_analyzer.full_dir_scan - INFO - Documents after sorting and filtering: 27
2024-12-19 11:11:51,212 - ai_security_analyzer.full_dir_scan - INFO - Splitting documents
2024-12-19 11:11:51,233 - ai_security_analyzer.full_dir_scan - INFO - Splitted documents into smaller chunks: 25
=========== dry-run ===========
All documents token count: 33325
List of chunked files to analyze:
..\fabric-agent-action\README.md
..\fabric-agent-action\docs\updating-fabric-patterns.md
...

2024-12-19 11:11:51,267 - __main__ - INFO - AI Security Analyzer completed successfully
```

This output tells you:

- The total number of documents and tokens to be processed.
- The list of files that will be analyzed.
- No API calls are made during a dry run, which helps you estimate the potential cost.

## Prompts and Models Behavior

A few observations about using prompts with LLMs:

- Prompts that have detailed instructions worked better than prompts that were more open-ended, e.g. "create STRIDE threat model" will sometimes make magic happen, but will not give repeatable results.
- Models are bad at generating mermaid diagrams, so I've implemented fixer - it will parse mermaid using javascript library and send error to LLM to fix it. Sadly this in not working 100% of the time. Models cannot get that "(" inside "[  ]" are not allowed 🤷
- Attack surface prompt still needs some work, but it does one thing extremely well - it provides best threat ranking from other prompts.
- Security design prompt is sometimes giving marvels in risk assessment. It can very well describe relation between business and security risks.

## Highlights and Examples

Here are some standout outputs generated by the AI Security Analyzer:

### 1. Security Design Documentation

For [fabric-agent-action](https://github.com/xvnpw/fabric-agent-action):

```markdown
Recommended security controls (high priority to implement if not already mentioned):

Validate the output from LLM before posting any publicly visible content.

...

## RISK ASSESSMENT

1. Critical business processes we are trying to protect:
- Automation around GitHub workflows that post or merge code, open pull requests, or alter repository content.

2. Data we are trying to protect and their sensitivity:
- GitHub repository content (proprietary code).
- LLM API tokens stored in GitHub Secrets to prevent uncontrolled usage or cost infiltration.
- AI-generated outputs that could contain privileged or sensitive data if improperly handled by the LLM.
```

Model really nailed it with risk assessment. It's not just some random text, it's actually risk assessment that can be used to make decisions.

### 2. Threat Modeling

For [screenshot-to-code](https://github.com/abi/screenshot-to-code):

```markdown
- THREAT ID - 0001
- COMPONENT NAME - Backend Application
- THREAT NAME - Code Injection via Malicious Image Data
- STRIDE CATEGORY - Tampering
- WHY APPLICABLE - User-provided images are processed by the backend and sent to AI APIs. Malicious images could exploit vulnerabilities in image processing libraries or lead to code injection.
- HOW MITIGATED - No mention of sanitization or input validation in the project files.
- MITIGATION - Implement strict input validation and sanitization of user-provided images. Use secure libraries for image processing. Restrict allowed file types and enforce file size limits.
- LIKELIHOOD EXPLANATION - Medium likelihood as attackers may attempt to exploit image parsing vulnerabilities by uploading crafted images.
- IMPACT EXPLANATION - High impact as successful exploitation could allow execution of arbitrary code on the backend server, compromising the entire system and potentially exposing sensitive data and API keys.
- RISK SEVERITY - High
```

This is a high-quality threat analysis that aligns with what one might produce during manual threat modeling. LIKELIHOOD and IMPACT EXPLANATION are my usual way of pushing models to give better results on RISK SEVERITY. Not sure if it's working, but it's worth a try.

### 3. Attack Tree Analysis

For [Flask](https://github.com/pallets/flask):

```markdown
## 7. Analyze and Prioritize Attack Paths

### High-Risk Paths

**2. Exploit Template Rendering Vulnerabilities**

- **Justification**: Template rendering vulnerabilities, specifically Server-Side Template Injection (SSTI), can allow attackers to execute arbitrary code on the server. Given that developers might inadvertently use `render_template_string` with user input or disable autoescaping, this poses a critical risk.

**5. Exploit Insecure Configurations in Flask Applications**

- **Justification**: Misconfigurations, such as leaving DEBUG mode enabled or missing security headers, are common and can have severe consequences. Attackers can easily exploit these to gain sensitive information or bypass security protections.

**7. Exploit Cross-Site Scripting (XSS) Vulnerabilities**

- **Justification**: XSS vulnerabilities are prevalent in web applications. If developers improperly handle user input or disable autoescaping, attackers can inject malicious scripts, leading to data theft or session hijacking.

**8. Exploit Cross-Site Request Forgery (CSRF) Vulnerabilities**

- **Justification**: Without proper CSRF protection, attackers can trick authenticated users into performing unwanted actions. Given that CSRF tokens are not automatically implemented in Flask, developers may overlook this protection.

**10. Exploit Misconfiguration of Security Headers**

- **Justification**: Missing security headers like CSP and HSTS can expose applications to XSS attacks and downgrade attacks. Since Flask does not set these headers by default, developers need to proactively implement them.

### Critical Nodes

- **5.2 DEBUG Mode Enabled in Production**: Ensuring DEBUG mode is disabled in production environments is crucial to prevent information disclosure.
- **7.1 Developer Renders User Input Without Proper Escaping**: Properly escaping user input prevents XSS attacks.
- **8.1 Missing CSRF Protection in Forms**: Implementing CSRF tokens in forms is essential to prevent CSRF attacks.
- **10.1 Missing Content Security Policy (CSP) Header**: Setting a CSP header mitigates XSS risks.
```

While some points may seem generic, the model effectively identified critical misconfigurations that are common pitfalls in Flask applications.

## Conclusion

I'm thrilled with how the AI Security Analyzer has turned out. It addresses many of the challenges I've encountered since working with GPT-3.5 and offers a robust solution for generating security documentation.

Some might wonder how and why someone would use this tool. I believe it's an excellent starting point for understanding the security posture of your project. It helps navigate through complexities and provides a foundation that you can build upon.

Let me know what you think about it! I'm waiting for your feedback. Feel free to reach out!

[Code](https://github.com/xvnpw/ai-security-analyzer) and [examples](https://github.com/xvnpw/ai-security-analyzer/tree/main/examples) are available on GitHub.

---

Thanks for reading! You can contact me and/or follow me on [X](https://x.com/xvnpw) and [LinkedIn](https://linkedin.com/in/marcin-niemiec-304349104).