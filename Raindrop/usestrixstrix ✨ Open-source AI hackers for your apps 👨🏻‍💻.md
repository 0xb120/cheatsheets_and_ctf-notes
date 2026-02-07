---
raindrop_id: 1336155869
raindrop_highlights:
  68c447ea04fed0079c7fcd9b: db5bda2e0a837c1a0360edad09ff501a
  68c4480a0dbbf1d411e6b764: fb14539a1434d31c1ead97a1181d463f
  68c44814844ec9dfaf73bb87: 210c00cb4d4dd351d3b6418bef363252
  68c44819ed92012ad76f490e: fcd9e7a119ed758094422052617dd4a4
title: "usestrix/strix: ✨ Open-source AI hackers for your apps 👨🏻‍💻"
description: ✨ Open-source AI hackers for your apps 👨🏻‍💻 . Contribute to usestrix/strix development by creating an account on GitHub.
source: https://github.com/usestrix/strix
created: 1757693875642
type: link
tags:
  - Tools
  - LLM
  - AI
aliases:
  - strix
---
# usestrix/strix: ✨ Open-source AI hackers for your apps 👨🏻‍💻

![](https://opengraph.githubassets.com/be70b622e404d4c2ca95428039aa0694b9cf6721f6ebad1b1a1c75bd35404ecf/usestrix/strix)

> [!summary]
> ✨ Open-source AI hackers for your apps 👨🏻‍💻 . Contribute to usestrix/strix development by creating an account on GitHub.


✨ Open-source AI hackers for your apps 👨🏻‍💻
Documentation: https://docs.strix.ai/
## 🦉 Strix Overview

Strix are autonomous AI agents that act just like real hackers - they run your code dynamically, find vulnerabilities, and validate them through actual exploitation. Built for developers and security teams who need fast, accurate security testing without the overhead of manual pentesting or the false positives of static analysis tools.

## 🚀 Quick Start

```sh
# Install Strix
curl -sSL https://strix.ai/install | bash

# Or via pipx
pipx install strix-agent

# Configure your AI provider
export STRIX_LLM="openai/gpt-5"
export LLM_API_KEY="your-api-key"

# Optional
export LLM_API_BASE="your-api-base-url"  # if using a local model, e.g. Ollama, LMStudio
export PERPLEXITY_API_KEY="your-api-key"  # for search capabilities
export STRIX_REASONING_EFFORT="high"  # control thinking effort (default: high, quick scan: medium)

# Run your first security assessment
strix --target ./app-directory
```
## ✨ Features

### Agentic Security Tools

Strix agents come equipped with a comprehensive security testing toolkit:

- **Full HTTP Proxy** - Full request/response manipulation and analysis
- **Browser Automation** - Multi-tab browser for testing of XSS, CSRF, auth flows
- **Terminal Environments** - Interactive shells for command execution and testing
- **Python Runtime** - Custom exploit development and validation
- **Reconnaissance** - Automated OSINT and attack surface mapping
- **Code Analysis** - Static and dynamic analysis capabilities
- **Knowledge Management** - Structured findings and attack documentation

### Comprehensive Vulnerability Detection

Strix can identify and validate a wide range of security vulnerabilities:

- **Access Control** - IDOR, privilege escalation, auth bypass
- **Injection Attacks** - SQL, NoSQL, command injection
- **Server-Side** - SSRF, XXE, deserialization flaws
- **Client-Side** - XSS, prototype pollution, DOM vulnerabilities
- **Business Logic** - Race conditions, workflow manipulation
- **Authentication** - JWT vulnerabilities, session management
- **Infrastructure** - Misconfigurations, exposed services

### Graph of Agents

Advanced multi-agent orchestration for comprehensive security testing:

- **Distributed Workflows** - Specialized agents for different attacks and assets
- **Scalable Testing** - Parallel execution for fast comprehensive coverage
- **Dynamic Coordination** - Agents collaborate and share discoveries

## Usage Examples

### Basic Usage

```shell
# Scan a local codebase
strix --target ./app-directory

# Security review of a GitHub repository
strix --target https://github.com/org/repo

# Black-box web application assessment
strix --target https://your-app.com
```

### Advanced Testing Scenarios

```shell
# Grey-box authenticated testing
strix --target https://your-app.com --instruction "Perform authenticated testing using credentials: user:pass"

# Multi-target testing (source code + deployed app)
strix -t https://github.com/org/app -t https://your-app.com

# Focused testing with custom instructions
strix --target api.your-app.com --instruction "Focus on business logic flaws and IDOR vulnerabilities"

# Provide detailed instructions through file (e.g., rules of engagement, scope, exclusions)
strix --target api.your-app.com --instruction-file ./instruction.md
```

### Headless Mode

Run Strix programmatically without interactive UI using the `-n/--non-interactive` flag—perfect for servers and automated jobs. The CLI prints real-time vulnerability findings, and the final report before exiting. Exits with non-zero code when vulnerabilities are found.

```shell
strix -n --target https://your-app.com
```
