---
title: "Scaling Threat Modeling with AI: Generating 1000 Threat Models Using Gemini 2.0 and AI Security Analyzer"
date: 2025-01-01T08:59:02+01:00
draft: true
tags: [security, llm, gpt, ai, github, langchain, gemini]
description: "An in-depth look at how I leveraged Gemini 2.0 to create a massive security documentation repository, complete with practical examples and lessons learned."
---

"Can AI help us scale security analysis?" This question led me down a fascinating path of experimenting with Google's Gemini 2.0 to generate threat models at an unprecedented scale. In this post, I'll share how I turned this ambitious idea into reality, complete with code samples, real outputs, and valuable lessons learned along the way.

## The Challenge: Automating Security Analysis at Scale

Security documentation is crucial but often becomes a bottleneck in fast-moving development cycles. With the release of Google's Gemini 2.0 Flash Thinking Experimental model, I saw an opportunity to tackle this challenge head-on, despite some notable limitations:

```
Model Constraints:
✗ 8k output token limit
✗ Markdown formatting issues
✗ Knowledge cutoff (Oct 2023)
```

The model presented an opportunity due to its "thinking" capabilities, utilizing chain-of-thought reasoning to tackle complex problems. And it's free 💸...

## The Quest to Leverage Gemini for Threat Modeling

Building upon my previous work like [Threat Modeling with Fabric Framework]({{< ref "/posts/threat_modelling_with_fabric_framework.md" >}}), I've been a long-time advocate for the STRIDE methodology. I had a finely tuned [prompt](https://github.com/danielmiessler/fabric/blob/main/patterns/create_stride_threat_model/system.md) that performed exceptionally with OpenAI's o1-preview model. Naturally, I was eager to see how it fared with Gemini 2.0.

## Hitting the Initial Roadblocks

To my surprise, Gemini didn't play well with my existing prompts:

- **Inconsistent Markdown Generation**: The model struggled to produce valid and consistent Markdown, which was crucial for documentation.
- **Less Effective Prompts**: My go-to prompts yielded erratic results, lacking the determinism I needed.

It became clear that I couldn't just copy and paste my methods — I had to rethink my approach.

## The Journey to Better Prompts

My initial attempts were... interesting, to say the least. Simple prompts like "Create threat model for X" produced wildly inconsistent results, while complex prompts often led to the AI equivalent of a deer in headlights. 

Here's what I learned about prompt engineering through trial and error:

```
Effectiveness Spectrum:
Too Simple        <---|--------------|---> Too Complex
"Create threat model" |  Sweet Spot  | Multi-page instructions
                             ↑
                    Where magic happens
```

I needed a balanced approach — a "sweet spot" where the prompts were sufficiently detailed to guide the model but not so complex that they overwhelmed it.

## Crafting Effective Prompts: The Game Changer

The breakthrough came when I shifted from single, complex prompts to a sequence of targeted prompts. I developed a new agent, [Github2Agent](https://github.com/xvnpw/ai-security-analyzer/blob/main/ai_security_analyzer/github2_agents.py), to facilitate a multi-turn conversation with Gemini.

## The Multi-Step Prompt Strategy

Here's the refined set of prompts:

```python
GITHUB2_THREAT_MODELING_PROMPTS = [
    "You are cybersecurity expert, working with development team. Your task is to create threat model for application that is using {}. Focus on threats introduced by {} and omit general, common web application threats. Use valid markdown formatting. Don't use markdown tables at all, use markdown lists instead.",
    "Create threat list with: threat, description (describe what the attacker might do and how), impact (describe the impact of the threat), which {} component is affected (describe what component is affected, e.g. module, function, etc.), risk severity (critical, high, medium or low), and mitigation strategies (describe how can developers or users reduce the risk). Use valid markdown formatting. Don't use markdown tables at all, use markdown lists instead.",
    "Update threat list and return only threats that directly involve {}. Return high and critical threats only. Use valid markdown formatting. Don't use markdown tables at all, use markdown lists instead.",
]
```

**Key elements of this strategy:**

- **Sequential Guidance**: Each prompt builds upon the previous response, allowing the model to refine and focus its output incrementally.
- **Dynamic Placeholders**: The `{}` placeholders are dynamically replaced with the specific GitHub repository URL and name, tailoring the prompts to each project. I don't analyze code from the repository but rely on the model's knowledge 🧠 (by the way, AI Security Analyzer is able to analyze code from the repository in `dir` mode).
- **Focused Filtering**: By instructing the model to "Return only high and critical threats," we ensure the output is relevant and actionable.

## An Example Output

Applying this approach to the Python **Flask** framework yielded promising results:

```markdown
Here are the high and critical threats that directly involve the Flask framework:

*   **Threat:** Route Parameter Injection
    *   **Description:** An attacker manipulates route parameters within the URL. Flask's routing mechanism allows for dynamic parameters, and if these are not properly validated, attackers can inject malicious input to access unintended resources or trigger unexpected application behavior. This could involve path traversal or injecting commands.
    *   **Impact:** Unauthorized access to data or functionality, potentially leading to data breaches, privilege escalation, or denial of service.
    *   **Affected Flask Component:** `flask.Flask.route`, `flask.request.view_args`
    *   **Risk Severity:** High
    *   **Mitigation Strategies:**
        *   Implement strict input validation and sanitization for all route parameters.
        *   Avoid directly using route parameters in file system operations or other sensitive actions without thorough checks.
        *   Utilize type converters in route definitions to enforce expected data types.

*   **Threat:** Insecure Session Cookie Configuration
    *   **Description:** Flask uses signed cookies for session management. If the `SECRET_KEY` configured within the `flask.Flask` application is weak or compromised, attackers can forge session cookies and impersonate legitimate users. Additionally, if the `secure` and `httponly` flags are not properly set for the session cookie by Flask's session management, it can be vulnerable to interception or client-side scripting attacks.
    *   **Impact:** Account takeover, unauthorized access to user data and functionalities.
    *   **Affected Flask Component:** `flask.session`, `flask.Flask.secret_key`, Flask's cookie setting mechanism.
    *   **Risk Severity:** Critical
    *   **Mitigation Strategies:**
        *   Use a strong, randomly generated `SECRET_KEY` and store it securely (e.g., environment variables).
        *   Configure session cookie flags (`httponly=True`, `secure=True` in production) appropriately, either through Flask configuration or directly when setting the cookie.

*   **Threat:** Debug Mode Enabled in Production
    *   **Description:** Running a Flask application with `debug=True` configures the `flask.Flask` application to expose an interactive debugger in the browser when an error occurs. Attackers can exploit this debugger to execute arbitrary code on the server, access sensitive information, and potentially gain full control of the application.
    *   **Impact:** Complete server compromise, data breaches, denial of service.
    *   **Affected Flask Component:** `flask.Flask`, `debug` configuration parameter.
    *   **Risk Severity:** Critical
    *   **Mitigation Strategies:**
        *   **Never** run Flask applications with `debug=True` in production environments. Ensure `app.debug = False` or the `FLASK_DEBUG=0` environment variable is set.
        *   Implement proper logging and error reporting mechanisms for production.

*   **Threat:** Blueprint Route Conflicts and Overlapping
    *   **Description:** When using Flask Blueprints to structure an application, developers might unintentionally define routes that overlap or conflict within the `flask.Blueprint` instances or when registering them with the main `flask.Flask` application. An attacker could exploit this by accessing a route intended for a different blueprint, potentially bypassing security checks or accessing unintended functionality.
    *   **Impact:** Unexpected application behavior, potential security bypasses, access to unintended resources or functionalities.
    *   **Affected Flask Component:** `flask.Blueprint`, `flask.Flask.register_blueprint`
    *   **Risk Severity:** Medium (While listed as medium before, in certain scenarios leading to significant bypasses, it can be High. Let's keep it as High for this filtered list focusing on direct Flask involvement).
    *   **Mitigation Strategies:**
        *   Carefully plan and manage route definitions within blueprints.
        *   Use unique prefixes or subdomains for blueprints to avoid naming collisions.
        *   Thoroughly test route configurations to identify and resolve any conflicts. Flask provides tools to inspect the registered routes.

*   **Threat:** Incorrect HTTP Method Handling
    *   **Description:** Developers using the `@app.route()` decorator or `add_url_rule()` on the `flask.Flask` application might not correctly restrict the allowed HTTP methods (GET, POST, PUT, DELETE, etc.) for specific routes. An attacker could leverage this by using an unintended method to perform actions they shouldn't be able to, such as modifying data via a GET request if the route handler doesn't properly validate the method.
    *   **Impact:** Data modification, unauthorized actions, potential security breaches.
    *   **Affected Flask Component:** `flask.Flask.route`, `methods` argument in route definition.
    *   **Risk Severity:** Medium (Similar to Blueprint conflicts, if leading to critical data modification, it can be High. Let's keep it as High for this focused list).
    *   **Mitigation Strategies:**
        *   Explicitly define the allowed HTTP methods for each route using the `methods` argument in the `@app.route()` decorator.
        *   Implement proper handling for each allowed method and reject requests with other methods.

These threats directly involve the core components and functionalities provided by the Flask framework itself. Remember to also consider general web application security best practices.
```

The output was surprisingly detailed and relevant, though not perfect. It highlighted critical security concerns while providing actionable mitigation strategies.

## Expanding to Four Document Types

Encouraged by the success with threat modeling, I extended the approach to generate four different types of security documents:

- 🔒 **Security Design Documentation**: Generating detailed security design review.
- 🎯 **Threat Modeling**: Performing threat modeling analysis.
- 🔍 **Attack Surface Analysis**: Identifying potential entry points and vulnerabilities in the project's attack surface.
- 🌳 **Attack Tree Analysis**: Visualizing potential attack vectors and their hierarchies through attack tree.

The specific prompts for these documents are defined in the [prompts.py](https://github.com/xvnpw/ai-security-analyzer/blob/dabfc57b6e5da9d99b3df5229fd496a224dac862/ai_security_analyzer/prompts.py#L763) file.

## Scaling to 1000: The Infrastructure

To achieve the goal of 1000 threat models, I needed more than just good prompts. I built a pipeline using GitHub Actions that could:

1. Queue and process repositories
2. Generate four types of security documentation using my [AI Security Analyzer](https://github.com/xvnpw/ai-security-analyzer)

## Organizing the Results

To keep things navigable, the repository is structured by programming language, with folders for each major project. Each project contains subfolders with detailed analyses, organized by date and the specific LLM model used.

**The full list of projects analyzed is [available here](https://github.com/xvnpw/sec-docs/blob/main/.data/origin_repos.txt).** This compilation draws from:

- **Generative AI Suggestions**: Enhanced with manual curation to ensure relevance.
- **[GitHub Rankings](https://github.com/EvanLi/Github-Ranking/tree/master)**: Leveraging popularity metrics to select impactful projects.
- **[Set of Critical Open Source Projects](https://github.com/ossf/wg-securing-critical-projects/tree/main/Initiatives/Identifying-Critical-Projects/Version-1.1)**: Identifying critical projects that are important for the security of the software supply chain.

## Reflecting on the Journey: Evaluating the Results

While I haven't yet conducted an exhaustive review of all 1,000 threat models 😅, initial assessments are promising. The methodology demonstrates significant potential in producing valuable security documentation at scale. some interesting patterns emerged:

- **Holistic Insights**: Having all four documents provides different perspectives and insights. Individually, none are perfect, but together they offer a comprehensive view of the project's security posture.
- **Solid Threats**: The threats identified are solid — not mind-blowing, but relevant and actionable.
- **Knowledge Base Limitations**: Relying on the model's knowledge base is a drawback — we don't always know what version of the code was used for the analysis.
- **Focused Priorities**: Concentrating on high and critical threats makes the documents shorter and easier to comprehend.

**Future posts will delve deeper into analysis and refinement.**

## Closing Thoughts

This experiment has been both challenging and thrilling. It highlights the transformative potential of AI models like Gemini 2.0 in automating and scaling critical cybersecurity processes. I would definitely consider using the generated documentation when working on new technology. It serves as a valuable starting point — not replacing human security experts, but making it easier to navigate complexities.

I would love to generate 1,000 threat models using different LLM models, especially the final version of Gemini 2.0 and the new OpenAI o1, o1-pro, and the upcoming o3.

## Want to Try It Yourself?

**I invite you to explore the [sec-docs](https://github.com/xvnpw/sec-docs) repository.** Review the generated documents, scrutinize the analyses, and share your insights. Your feedback is crucial in refining this approach and enhancing the quality of automated security assessments.

---

Thanks for reading! You can contact me and/or follow me on [X](https://x.com/xvnpw) and [LinkedIn](www.linkedin.com/in/marcin-niemiec-304349104).