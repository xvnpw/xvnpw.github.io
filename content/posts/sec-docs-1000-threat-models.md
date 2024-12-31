---
title: "Scaling Threat Modeling with AI: Generating 1000 Threat Models Using Gemini 2.0 and AI Security Analyzer"
date: 2025-01-01T08:59:02+01:00
draft: true
tags: [security, llm, gpt, ai, github, langchain, gemini]
description: "An in-depth look at how I leveraged Gemini 2.0 to create a massive security documentation repository, complete with practical examples and lessons learned."
---

With the creation of the [AI Security Analyzer]({{< ref "/posts/ai-security-analyzer-all-in-one-tool-preview.md" >}}), I embarked on an ambitious project: generating 1000 threat models using Google's Gemini 2.0 Flash Thinking Experimental model. This post details the technical journey, challenges faced, and solutions crafted to make this possible.

## Exploring Gemini 2.0 Flash Thinking Experimental

At the end of 2024, Google released Gemini 2.0 Flash Thinking Experimental for free. Despite its limitations:

- **8k output token limit**
- **Issues with generating correct Markdown**
- **Knowledge cutoff at the end of October 2023**

The model presented an opportunity due to its "thinking" capabilities, utilizing chain-of-thought reasoning to tackle complex problems.

I was particularly interested in evaluating its potential in automating threat modeling processes.

## Leveraging Gemini for Threat Modeling

Building on my previous work, such as [Threat Modelling with Fabric Framework]({{< ref "/posts/threat_modelling_with_fabric_framework.md" >}}), I have been an advocate for the STRIDE methodology in threat modeling. I had developed a specific [prompt](https://github.com/danielmiessler/fabric/blob/main/patterns/create_stride_threat_model/system.md) tailored for STRIDE, which performed well with OpenAI's o1-preview model.

However, initial experiments with Gemini 2.0 revealed challenges:

- The model struggled with generating consistent and accurate Markdown.
- My existing prompts were less effective, with the model often producing non-deterministic results.

It became evident that to harness Gemini's capabilities, I needed to adapt my approach.

### Finding the Sweet Spot in Prompt Engineering

I observed that simple prompts like "Create threat model for X" yielded inconsistent results with Gemini. However, overly complex, multi-step prompts seemed to overload the model, leading to collapses where it would produce irrelevant outputs.

I needed a balanced approach—a "sweet spot" where the prompts were sufficiently detailed to guide the model but not so complex that they overwhelmed it.

```
Not repeatable results      |    Sweet spot    |   Generic threat model
                            |    ⊂(◉‿◉)つ   |
"Create threat model for X" |                  |   <complex, multi-step prompt>            
```

## Designing Effective Prompts

To achieve more consistent and useful outputs, I experimented with prompt structures and interactions. This led to the creation of a new agent, [Github2Agent](https://github.com/xvnpw/ai-security-analyzer/blob/main/ai_security_analyzer/github2_agents.py), designed to engage in a multi-turn conversation with the Gemini model.

### The Multi-Step Prompt Approach

Instead of a single, complex prompt, I devised a series of prompts to guide the model through the threat modeling process incrementally. The prompts, defined in [prompts.py](https://github.com/xvnpw/ai-security-analyzer/blob/main/ai_security_analyzer/prompts.py#L757-L761), are as follows:

```python
GITHUB2_THREAT_MODELING_PROMPTS = [
    "You are cybersecurity expert, working with development team. Your task is to create threat model for application that is using {}. Focus on threats introduced by {} and omit general, common web application threats. Use valid markdown formatting. Don't use markdown tables at all, use markdown lists instead.",
    "Create threat list with: threat, description (describe what the attacker might do and how), impact (describe the impact of the threat), which {} component is affected (describe what component is affected, e.g. module, function, etc.), risk severity (critical, high, medium or low), and mitigation strategies (describe how can developers or users reduce the risk). Use valid markdown formatting. Don't use markdown tables at all, use markdown lists instead.",
    "Update threat list and return only threats that directly involve {}. Return high and critical threats only. Use valid markdown formatting. Don't use markdown tables at all, use markdown lists instead.",
]
```

Key aspects of the prompt design:

- **Sequential Prompts**: The prompts are used sequentially, each building upon the previous response. This breaks down the task into manageable steps for the model.
- **Placeholder Usage**: `{}` placeholders are used to insert the specific GitHub repository URL and name dynamically.
- **Filtering for Severity**: The instruction to "Return only high and critical threats" helps focus the model on the most significant issues, improving the relevancy of the output.

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

While not perfect, the output provided valuable insights and demonstrated the potential of this method.

## Expanding to Four Document Types

Encouraged by the success with threat modeling, I extended the approach to generate four different types of security documents:

- 🔒 **Security Design Documentation**: Generating detailed security design documents.
- 🎯 **Threat Modeling**: Performing comprehensive threat modeling analyses.
- 🔍 **Attack Surface Analysis**: Identifying potential entry points and vulnerabilities in the project's attack surface.
- 🌳 **Attack Tree Analysis**: Visualizing potential attack vectors and their hierarchies through attack trees.

The specific prompts for these documents are defined in the [prompts.py](https://github.com/xvnpw/ai-security-analyzer/blob/dabfc57b6e5da9d99b3df5229fd496a224dac862/ai_security_analyzer/prompts.py#L763) file.

## Generating 1000 Threat Models

With the framework in place, I aimed to scale up the process. By leveraging GitHub Actions, I implemented a queue system to manage the generation of threat models across numerous projects. The results are available in the [sec-docs](https://github.com/xvnpw/sec-docs) repository.

### Repository Organization

The `sec-docs` repository is structured by programming language, with folders for each major open-source project. Each project contains subfolders with detailed analyses performed on specific dates using particular LLM models.

The list of projects analyzed is available [here](https://github.com/xvnpw/sec-docs/blob/main/.data/origin_repos.txt). This list was compiled from several sources, including:

- Generative AI suggestions (with some manual corrections).
- The [GitHub Rankings](https://github.com/EvanLi/Github-Ranking/tree/master), providing a list of the most popular projects.

## Evaluating the Results

At this stage, I have not conducted a comprehensive evaluation of all the generated threat models. Initial reviews suggest that the methodology can produce valuable security documentation, but there is more work to be done to assess the accuracy and usefulness of the outputs.

I plan to share more detailed analyses and findings in future posts.

## Conclusion and Call for Feedback

This experiment demonstrates the potential of using AI models like Gemini 2.0 to automate and scale threat modeling efforts. While there are challenges and limitations, particularly regarding prompt design and model constraints, the results are promising.

I invite you to explore the [sec-docs](https://github.com/xvnpw/sec-docs) repository, review the generated documents, and share your thoughts. Your feedback will be invaluable in refining the approach and improving the quality of automated security analyses.

---

Thanks for reading! You can contact me and/or follow me on [X](https://x.com/xvnpw).