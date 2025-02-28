---
title: "Insecure VSCode Extensions"
date: 2025-02-27T08:59:02+01:00
draft: true
tags: [security, vscode, extensions]
description: "A list of insecure VSCode extensions and how to check if your extensions are insecure."
---

Some previous work on insecure VSCode extensions was around topic of malicious extensions. Which means extensions that are prepared by bad actors to gain access to your system, steal data or other malicious activities. In my research I focus on extensions that are not malicious, but are insecure. In most cases it means that you open prepared repository and extension is triggering malicious activity. Those extensions are used by millions of people.

Previous work on this topic:
- https://medium.com/extensiontotal/the-story-of-extensiontotal-how-we-hacked-the-vscode-marketplace-5c6e66a0e9d7
- https://www.aquasec.com/blog/can-you-trust-your-vscode-extensions/

## RCE on project open

| VSCode Extension | vulnerability | PoC | AI vulnerability description |
| --- | --- | --- | --- |
| `amir9480/vscode-laravel-extra-intellisense` ([GitHub](https://github.com/amir9480/vscode-laravel-extra-intellisense)) | RCE on project open via script in `boostrap` or `autoload` | [xvnpw/vscode-laravel-extra-intellisense-example-app](https://github.com/xvnpw/vscode-laravel-extra-intellisense-example-app) | [Untrusted Laravel Bootstrap and Autoload Execution](https://github.com/xvnpw/sec-docs/blob/main/typescript/amir9480/vscode-laravel-extra-intellisense/2025-02-28-anthropic-claude-3.7-sonnet-thinking/vulnerabilities-workflow-1.md#untrusted-laravel-bootstrap-and-autoload-execution) |
| `denoland/vscode_deno` ([GitHub](https://github.com/denoland/vscode_deno)) | RCE on project open via manipulated `deno.path` | [xvnpw/vscode_deno-poc](https://github.com/xvnpw/vscode_deno-poc) | [Remote Code Execution via deno.path Setting](https://github.com/xvnpw/sec-docs/blob/main/typescript/denoland/vscode_deno/2025-02-28-anthropic-claude-3.7-sonnet-thinking/vulnerabilities-workflow-1.md#1-remote-code-execution-via-denopath-setting) |
| `dotnet/vscode-csharp` ([GitHub](https://github.com/dotnet/vscode-csharp)) | RCE on project open via unvalidated .NET runtime path from `settings.json` when using OmniSharp | [xvnpw/vscode-csharp-poc](https://github.com/xvnpw/vscode-csharp-poc) | [Unvalidated .NET runtime path from setting](https://github.com/xvnpw/sec-docs/blob/main/typescript/dotnet/vscode-csharp/2025-02-28-gemini-2.0-flash-thinking-exp/vulnerabilities-workflow-1.md#vulnerability-1-unvalidated-net-runtime-path-from-settings) |

## RCE on extension action

| VSCode Extension | vulnerability | PoC | AI vulnerability description |
| --- | --- | --- | --- |
| `amir9480/vscode-laravel-extra-intellisense` ([GitHub](https://github.com/amir9480/vscode-laravel-extra-intellisense)) | RCE on Eloquent model open | [xvnpw/vscode-laravel-extra-intellisense-example-app](https://github.com/xvnpw/vscode-laravel-extra-intellisense-example-app) | [Arbitrary Code Execution via Automatic Inclusion in Eloquent Provider](https://github.com/xvnpw/sec-docs/blob/main/typescript/amir9480/vscode-laravel-extra-intellisense/2025-02-26-gemini-2.0-flash-thinking-exp/vulnerabilities-workflow-1.md#vulnerability-name-arbitrary-code-execution-via-automatic-inclusion-in-eloquent-provider) |
| `estruyf/vscode-front-matter` ([GitHub](https://github.com/estruyf/vscode-front-matter)) | RCE on executing custom script |  | [Remote Code Execution via Custom Scripts](https://github.com/xvnpw/sec-docs/blob/main/typescript/estruyf/vscode-front-matter/2025-02-28-gemini-2.0-flash-thinking-exp/vulnerabilities-workflow-1.md#remote-code-execution-via-custom-scripts) |

## Other vulnerabilities

| VSCode Extension | vulnerability | PoC | AI vulnerability description |
| --- | --- | --- | --- |
| `DonJayamanne/gitHistoryVSCode` ([GitHub](https://github.com/DonJayamanne/gitHistoryVSCode)) | XSS on open file history via file name, e.g. `echo 1 > "1';(function(){alert(1)})();var x='a.txt"` <br/> no impact - blocked by VS Code: `VM44:7 Ignored call to 'alert()'. The document is sandboxed, and the 'allow-modals' keyword is not set.` | [xvnpw/gitHistoryVSCode-XSS-example](https://github.com/xvnpw/gitHistoryVSCode-XSS-example) | [Webview Unsanitized FileName Injection (Cross‑Site Scripting)](https://github.com/xvnpw/sec-docs/blob/main/typescript/DonJayamanne/gitHistoryVSCode/2025-02-26-gemini-2.0-flash-thinking-exp/vulnerabilities-workflow-1.md#2-webview-unsanitized-filename-injection-crosssite-scripting) |

Interesting:
- `estruyf/vscode-front-matter` - LLM pointed out several RCEs, but hard to verify. Possibly code was too complex to be handled by LLM.

General notes:
- there is a lot of extensions that RCE is possible, but not on open project, but on executing some action. For example you can set in `settings.json`: `"deno.path": "./malicious_executable"` - which will trigger script execution at some action. 

Funny moments:
- [vscode-git-graph](https://github.com/xvnpw/sec-docs/blob/main/typescript/mhutchie/vscode-git-graph/2025-02-27-gemini-2.0-flash-thinking-exp/vulnerabilities-workflow-1.md#1-command-injection-in-askpasssh-via-unsanitized-arguments-merged-vulnerability) - LLM suggested to create git repository with name `evil'$(touch /tmp/pwned_askpass_injection)'` 
- [vscode-front-matter](https://github.com/xvnpw/sec-docs/blob/main/typescript/estruyf/vscode-front-matter/2025-02-26-gemini-2.0-flash-thinking-exp/vulnerabilities-workflow-1.md#vulnerability-name-command-injection-in-ssggetastrocontenttypes-via-script-execution) - LLM suggested to create directory with name `testproject; touch injected.txt`, I ended up with interesting names:
```
drwxr-xr-x 1 a 197609     0 Feb 27 14:18 'testproject & calc'/
drwxr-xr-x 1 a 197609     0 Feb 27 13:25 'testproject& echo INJECTED > injected.txt'/
```
Where LLM failed:
- Gemini 2.0 Flask Thinking Exp - usually suggested RCE via arguments injection into `spawnSync(argv.exec_path, args, ...)` or similar functions. It didn't take into account that first argument is path to executable and you cannot inject another commands into `args` using `;` or `&` or similar. Example: [deoptexplorer-vscode](https://github.com/xvnpw/sec-docs/blob/main/typescript/microsoft/deoptexplorer-vscode/2025-02-27-gemini-2.0-flash-thinking-exp/vulnerabilities-workflow-1.md#1-argument-injection-in-dexnode-command-line-tool)
- Sonnet 3.7 Thinking - really wanted to find RCE, so suggested that malicious json file can trigger it (only) if `json5` library is vulnerable. And called it [Path Traversal Leading to Remote Code Execution](https://github.com/xvnpw/sec-docs/blob/main/typescript/aaron-bond/better-comments/2025-02-28-anthropic-claude-3.7-sonnet-thinking/vulnerabilities-workflow-1.md#path-traversal-leading-to-remote-code-execution)
- Many models suggested adding `__proto__` to `settings.json` to trigger Prototype Pollution. I tried it few times, but it didn't work.

## Prompts

I started with fairly simple and generic prompt:

```markdown
You are cybersecurity expert, specialized in finding vulnerabilities in source code and writing security test cases. Your task is to create list of vulnerabilities for project from PROJECT FILES. Focus on vulnerabilities introduced by project from PROJECT FILES.

Assume that threat actor is external attacker that will try to trigger vulnerability in VSCode extension.

...

Include only vulnerabilities that:
- are valid and not already mitigated.
- has vulnerability rank at least: high
- 

...
```

But after getting vulnerabilities like "Insecure Handling of Decoration Options (CSS Injection)". I realized that I need to add more details to the prompt:

```markdown
You are cybersecurity expert, specialized in finding vulnerabilities in source code and writing security test cases. Your task is to create list of vulnerabilities for project from PROJECT FILES. Focus on vulnerabilities introduced by project from PROJECT FILES.

Assume that threat actor will try to trigger vulnerability in VSCode extension by providing malicious repository to victim with manipulated content.

...

Include only vulnerabilities that:
- are valid and not already mitigated.
- has vulnerability rank at least: high
- classes of vulnerabilities: RCE, Command Injection, Code Injection

...
```

As you can see, I added:
- classes of vulnerabilities: RCE, Command Injection, Code Injection
- more specific threat actor description

Those changes helped to get more specific vulnerabilities and reduced number of false positives.

## ai-security-analyzer

I also used [ai-security-analyzer](https://github.com/microsoft/ai-security-analyzer) to get list of vulnerabilities. It's a tool that uses LLM to analyze code and find vulnerabilities.

```bash
python ai_security_analyzer/app.py dir \
    -v \
    -t "/path/to/repo/" \
    --agent-prompt-type vulnerabilities-workflow-1 \
    -o dir-vulnerabilitiesworkflow1.md \
    --agent-provider openrouter \
    --agent-model anthropic/claude-3.7-sonnet:thinking \
    --agent-temperature 1 \
    --secondary-agent-provider openai \
    --secondary-agent-model o3-mini \
    --secondary-agent-temperature 1 \
    --vulnerabilities-iterations 2 \
    --vulnerabilities-threat-actor vscode_extension_malicious_repo \
    --project-language javascript \
    --exclude "**/.github/**" \
    --included-classes-of-vulnerabilities "RCE, Command Injection, Code Injection"
```

### Mitigations

Some of RCE vulnerabilities can be mitigated by using `capabilities` field in `package.json`. For example in `vscode-intelephense` extension: [package.json](https://github.com/bmewburn/vscode-intelephense/blob/8546446e2ebab310181434df91748aba2a5419ac/package.json#L60C6-L60C18)

### Hallucinations

- Sonnet 3.7 Thinking -Inventing vulnerabilities only to return something - [editorconfig-vscode](https://github.com/xvnpw/sec-docs/blob/main/typescript/editorconfig/editorconfig-vscode/2025-02-28-anthropic-claude-3.7-sonnet-thinking/vulnerabilities-workflow-1.md) - vulnerability in final output is not in any [intermediate files](https://github.com/xvnpw/sec-docs/tree/main/typescript/editorconfig/editorconfig-vscode/2025-02-28-anthropic-claude-3.7-sonnet-thinking/vulnerabilities-workflow)
- Asking to create new malicious VSCode extension - 


### more
- https://github.com/xvnpw/sec-docs/blob/main/typescript/godotengine/godot-vscode-plugin/2025-02-28-anthropic-claude-3.7-sonnet-thinking/vulnerabilities-workflow-1.md