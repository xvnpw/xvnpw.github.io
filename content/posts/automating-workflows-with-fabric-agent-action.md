---
title: "Automating Workflows with Fabric Agent Action"
date: 2024-11-22T13:59:02+01:00
draft: false
tags: [security, fabric, llm, gpt, ai, github, langchain]
description: "Introducing the Fabric Agent Action - a GitHub Action that automates complex workflows using AI-powered agents and Fabric Patterns."
---

[Fabric Agent Action](https://github.com/xvnpw/fabric-agent-action) is a GitHub Action that bridges the gap between [fabric](https://github.com/danielmiessler/fabric) patterns and GitHub workflows. Instead of manually executing patterns or building custom integrations, you can automate complex tasks directly in your GitHub workflows using an agent-based approach.

{{< figure src="https://github.com/user-attachments/assets/14110f11-1250-4792-8d1a-4da3fd85197e" class="image-center" width=200 >}}

The action introduces different types of agents, each designed for specific use cases, from simple pattern execution to complex reasoning about GitHub issues and pull requests. Let's explore how these agents work and evaluate their effectiveness.

**Watch demo:**
{{< twitter user="xvnpw" id="1859981296774316495" >}}

## Agents in Action

The action supports multiple agent types, each with its own characteristics:

1. Router Agent - Makes single pattern selection
2. ReAct Agent - Implements reasoning and multiple pattern execution
3. Specialized GitHub Agents - Optimized for issues and pull requests

Let's see how to use the Router Agent, which is the simplest one:

```yaml
- name: Execute Fabric Agent Action
  uses: xvnpw/fabric-agent-action@v1
  with:
    input_file: path/to/input.md
    output_file: path/to/output.md
    agent_type: "router"
  env:
    OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
```

## Security First Approach 🛡️

Before we dive deeper into capabilities, it's crucial to understand security implications. Use workflow built-in security controls to prevent unauthorized usage:

```yaml
if: >
  github.event.comment.user.login == github.event.repository.owner.login &&
  startsWith(github.event.comment.body, '/fabric')
```

This ensures that only repository owners can trigger the action, preventing potential abuse in public repositories.

## Agent Types Comparison

Let's compare different agents:

| Agent Type | Approach | Advantages | Limitations |
| --- | --- | --- | --- |
| Router | Single pattern selection | Simple, predictable | No complex workflows |
| ReAct | Multiple patterns with reasoning | Can chain actions | More complex, potentially less reliable |
| GitHub Specialized | Context-aware execution | Understands GitHub context | Limited to specific scenarios |

{{< figure src="https://github.com/user-attachments/assets/986d3989-0267-41d9-bd44-4f69262bccb3" class="image-center no-border" >}}

In practice, there's often a trade-off between autonomy and reliability. Increasing LLM autonomy can sometimes reduce reliability due to factors like non-determinism or errors in tool selection.

## Real-World Examples

Let's explore a practical example of how Fabric Agent Action can process GitHub issues. Below is a workflow that enables AI-powered responses to issue comments:

```yaml
name: Fabric Pattern Processing using ReAct Issue Agent
on:
  issue_comment:
    types: [created, edited]

jobs:
  process_fabric:
    if: >
      github.event.comment.user.login == github.event.repository.owner.login &&
      startsWith(github.event.comment.body, '/fabric') &&
      !github.event.issue.pull_request
    runs-on: ubuntu-latest
    permissions:
      issues: write
      contents: read

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Prepare Input
        uses: actions/github-script@v7
        id: prepare-input
        with:
          script: |
            const issue = await github.rest.issues.get({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo
            });

            const comment = await github.rest.issues.getComment({
              comment_id: context.payload.comment.id,
              owner: context.repo.owner,
              repo: context.repo.repo
            });

            // Get all comments for this issue to include in the output
            const comments = await github.rest.issues.listComments({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo
            });

            // Extract command from the triggering comment
            const command = comment.data.body;

            let output = `INSTRUCTION:\n${command}\n\n`;

            // Add issue information
            output += `GITHUB ISSUE, NR: ${issue.data.number}, AUTHOR: ${issue.data.user.login}, TITLE: ${issue.data.title}\n`;
            output += `${issue.data.body}\n\n`;

            // Add all comments
            for (const c of comments.data) {
              if (c.id === comment.data.id) {
                break;
              }
              output += `ISSUE COMMENT, ID: ${c.id}, AUTHOR: ${c.user.login}\n`;
              output += `${c.body}\n\n`;
            }

            require('fs').writeFileSync('fabric_input.md', output);

            return output;

      - name: Execute Fabric Patterns
        uses: docker://ghcr.io/xvnpw/fabric-agent-action:v1
        with:
          input_file: "fabric_input.md"
          output_file: "fabric_output.md"
          agent_type: "react_issue"
          fabric_temperature: 0.2
          fabric_patterns_included: "clean_text,create_stride_threat_model,create_design_document,review_design,refine_design_document,create_threat_scenarios,improve_writing,create_quiz,create_summary"
          debug: true
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}

      - name: Post Results
        uses: peter-evans/create-or-update-comment@v4
        with:
          issue-number: ${{ github.event.issue.number }}
          body-path: fabric_output.md
```

This workflow demonstrates several key features:
1. **Security Controls**: Only repository owners can trigger the action using `/fabric` commands
2. **Context Gathering**: Collects issue content, comments, and metadata using GitHub's API
3. **Pattern Execution**: Processes input through selected Fabric patterns using the ReAct Issue agent
4. **Automated Response**: Posts results back to the issue as a new comment

When executed, the action provides AI-generated responses directly in your GitHub issues:

{{< figure src="https://github.com/user-attachments/assets/707399e3-0c8e-4145-b11c-9e73e97fa66a" class="image-center no-border" >}}

## Key Learnings

### Context Matters

Similar to threat modeling, context is crucial for agent effectiveness. The action allows you to provide this context through:
- Input files
- GitHub issue/PR content
- Git diffs
- Comments history

### Integration Options

The action can be run in multiple ways:
- As a GitHub Action
- Using Docker
- From source code

This flexibility allows for different integration scenarios and development workflows.

### Pattern Management

One interesting challenge is managing the number of available patterns. Models like `gpt-4o` have a limit of 128 tools, while Fabric includes 175 patterns. The action provides two ways to handle this:

1. Including specific patterns:
```yaml
fabric_patterns_included: "clean_text,improve_writing"
```

2. Excluding patterns:
```yaml
fabric_patterns_excluded: "create_threat_scenarios"
```

## Looking Forward

I welcome your feedback and contributions. You can reach out to me on [LinkedIn](https://www.linkedin.com/in/marcin-niemiec-304349104/) or [X](https://x.com/xvnpw).

For those interested in extending the action's capabilities:
- Consider contributing new patterns to the [fabric](https://github.com/danielmiessler/fabric) repository first
- For custom patterns, you can fork this action and add them as additional tools
- Share your use cases and experiences to help shape future development

The goal is to make AI-powered automation more accessible and useful in everyday development workflows while maintaining security and reliability.

## Final Thoughts

In a world where AI is becoming increasingly integrated into our development workflows, tools like Fabric Agent Action show us how we can leverage AI's capabilities in practical, secure, and efficient ways. It's not just about automation - it's about smart automation that understands context and delivers value.

[Code](https://github.com/xvnpw/fabric-agent-action) and [examples](https://github.com/xvnpw/fabric-agent-action-examples) are available on GitHub.

---

Thanks for reading! You can contact me and/or follow me on [X](https://x.com/xvnpw).