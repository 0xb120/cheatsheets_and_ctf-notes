---
raindrop_id: 1566811244
raindrop_highlights:
  697e20814d80a1b75855dc32: 88b99578e6b377d2052d96b97b0c66b6
  697e208d4b15709a91dd437c: 9711123eaca0ba192f06654c93000caf
  697e2094cda9c53e0f0020bf: 90ae1675c5d615e6c29c9bb5a58e3786
  697e20a42b0cce59d1524a62: 91e8315910a8179a24000e18934c03d8
  697e20b6f70144491a18d71e: 7b44245341d9c0d920498fa57270819d
  697e20db35cd89fe75d97875: 8787e3d0b5aae1e54f79dd93359055cc
title: "six2dez/burp-ai-agent: Burp Suite extension that adds built-in MCP tooling, AI-assisted analysis, privacy controls, passive and active scanning and more"

description: |-
  Burp Suite extension that adds built-in MCP tooling, AI-assisted analysis, privacy controls, passive and active scanning and more - six2dez/burp-ai-agent

source: https://github.com/six2dez/burp-ai-agent

created: 2026-01-29
sync-date: 1769936780978
tags:
  - "_index"

---
# six2dez/burp-ai-agent: Burp Suite extension that adds built-in MCP tooling, AI-assisted analysis, privacy controls, passive and active scanning and more

![](https://opengraph.githubassets.com/e58b13cd9a39a16c4b3aa5d7788dcc117cdff8d1488d5317fc8bfe3ff33eb6a9/six2dez/burp-ai-agent)

> [!summary]
> Burp Suite extension that adds built-in MCP tooling, AI-assisted analysis, privacy controls, passive and active scanning and more - six2dez/burp-ai-agent





Burp Suite extension that adds built-in MCP tooling, AI-assisted analysis, privacy controls, passive and active scanning and more

burp-ai-agent.six2dez.com/
Burp AI Agent is an extension for Burp Suite that integrates AI into your security workflow. Use local models or cloud providers, connect external AI agents via MCP, and let passive/active scanners find vulnerabilities while you focus on manual testing.
7 AI Backends — Ollama, LM Studio, Generic OpenAI-compatible, Gemini CLI, Claude CLI, Codex CLI, OpenCode CLI.
53+ MCP Tools — Let Claude Desktop (or any MCP client) drive Burp autonomously.
62 Vulnerability Classes — Passive and Active AI scanners across injection, auth, crypto, and more.
3 Privacy Modes — STRICT / BALANCED / OFF. Redact sensitive data before it leaves Burp.
Audit Logging — JSONL with SHA-256 integrity hashing for compliance.
Download the latest JAR from Releases, or build from source (Java 21):

git clone https://github.com/six2dez/burp-ai-agent.git
cd burp-ai-agent
JAVA_HOME=/path/to/jdk-21 ./gradlew clean shadowJar
# Output: build/libs/Burp-AI-Agent-<version>.jar
3. Agent Profiles

The extension auto-installs the bundled profiles into ~/.burp-ai-agent/AGENTS/ on first run. Drop additional *.md files in that directory to add custom profiles.
Documentation

Full documentation is available at burp-ai-agent.six2dez.com.