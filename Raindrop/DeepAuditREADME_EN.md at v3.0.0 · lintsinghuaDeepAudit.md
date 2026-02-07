---
raindrop_id: 1550596038
raindrop_highlights:
  696d0c98dafedffded119e94: 6ba7164f2b04b9c1d9b145acafcb1a6e
  696d0cef7ea25d9430886040: e94f3ccd236b00028f611d648e0d6a81
  696d0d18fdfd5d4f7718dc9b: 44bd9fd9910e8d755295012aa0a56d84
title: "DeepAudit/README_EN.md at v3.0.0 · lintsinghua/DeepAudit"

description: |-
  DeepAudit：人人拥有的 AI 黑客战队，让漏洞挖掘触手可及。国内首个开源的代码漏洞挖掘多智能体系统。小白一键部署运行，自主协作审计 + 自动化沙箱 PoC 验证。支持 Ollama 私有部署 ，一键生成报告。支持中转站。​让安全不再昂贵，让审计不再复杂。 - lintsinghua/DeepAudit

source: https://github.com/lintsinghua/DeepAudit/blob/v3.0.0/README_EN.md

created: 2026-01-18
sync-date: 1769114384391
tags:
  - "_index"

 
  - "LLM" 
  - "Tools" 
  - "AI" 
  - "SAST"

---
# DeepAudit/README_EN.md at v3.0.0 · lintsinghua/DeepAudit

![](https://opengraph.githubassets.com/ce75427801caae92ee4326cb5b4736d53da8acf3a1753d8a0eac465a8ba89e68/lintsinghua/DeepAudit)

> [!summary]
> DeepAudit：人人拥有的 AI 黑客战队，让漏洞挖掘触手可及。国内首个开源的代码漏洞挖掘多智能体系统。小白一键部署运行，自主协作审计 + 自动化沙箱 PoC 验证。支持 Ollama 私有部署 ，一键生成报告。支持中转站。​让安全不再昂贵，让审计不再复杂。 - lintsinghua/DeepAudit





DeepAudit is a next-generation code security audit platform based on Multi-Agent collaborative architecture. It's not just a static scanning tool, but simulates the thinking patterns of security experts through autonomous collaboration of multiple agents (Orchestrator, Recon, Analysis, Verification), achieving deep code understanding, vulnerability discovery, and automated sandbox PoC verification.
Quick Start
Option 1: One-Line Deployment (Recommended)

Using pre-built Docker images, no need to clone code, start with one command:

curl -fsSL https://raw.githubusercontent.com/lintsinghua/DeepAudit/v3.0.0/docker-compose.prod.yml | docker compose -f - up -d
For detailed documentation, see Agent Audit Guide