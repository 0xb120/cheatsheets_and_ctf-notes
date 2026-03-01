---
title: "praetorian-inc/augustus: LLM security testing framework for detecting prompt injection, jailbreaks, and adversarial attacks — 190+ probes, 28 providers, single Go binary"
source: https://github.com/praetorian-inc/augustus
author:
  - nsportsman
  - EvanLeleux
published:
created: 2026-02-22
description: LLM security testing framework for detecting prompt injection, jailbreaks, and adversarial attacks — 190+ probes, 28 providers, single Go binary - praetorian-inc/augustus
tags:
  - clippings/articles
  - _inbox
  - tools
  - AI
  - LLM
aliases:
  - augustus
---
# praetorian-inc/augustus: LLM security testing framework for detecting prompt injection, jailbreaks, and adversarial attacks — 190+ probes, 28 providers, single Go binary

> [!summary]+
> > The page describes Augustus, a Go-based LLM vulnerability scanner developed by Praetorian for security professionals.
> It's designed to test large language models against 210+ adversarial attacks, including prompt injection, jailbreaks, encoding exploits, and data extraction.
> Key features include support for 28 LLM providers (with 43 generator variants), 90+ detectors, 7 buff transformations, and flexible output formats (table, JSON, JSONL, HTML).
> Augustus is built for production security testing, offering concurrent scanning, rate limiting, retry logic, and timeout handling in a single portable Go binary.
> It provides a comparison with other tools like `garak` and `promptfoo`, highlighting its performance and distribution advantages.
> Usage details cover installation, basic commands for single or multiple probes, buff transformations for evasion techniques, and testing custom REST endpoints.
> The architecture employs a pipeline for probe selection, buff transformation, generator calls, detector analysis, and result recording, featuring a plugin-style registration system.
> Configuration can be done via YAML files, environment variables, or CLI flags, including proxy support.
> It also includes an FAQ covering local model testing, adding custom probes, and troubleshooting common errors like rate limits and timeouts.
> The project welcomes contributions and emphasizes its use for authorized security testing only, with a mention of a DevPod environment for benchmarking.

LLM security testing framework for detecting prompt injection, jailbreaks, and adversarial attacks — 190+ probes, 28 providers, single Go binary

**Augustus** is a Go-based LLM vulnerability scanner for security professionals. It tests large language models against a wide range of adversarial attacks, integrates with 28 LLM providers, and produces actionable vulnerability reports.

### Attack Categories

- **Jailbreak attacks**: DAN, DAN 11.0, AIM, AntiGPT, Grandma, ArtPrompts
- **Prompt injection**: Encoding (Base64, ROT13, Morse), Tag smuggling, FlipAttack, Prefix/Suffix injection
- **Adversarial examples**: GCG, PAIR, AutoDAN, TAP (Tree of Attack Prompts), TreeSearch, DRA
- **Data extraction**: API key leakage, Package hallucination, PII extraction, LeakReplay
- **Context manipulation**: RAG poisoning, Context overflow, Multimodal attacks, Continuation, Divergence
- **Format exploits**: Markdown injection, YAML/JSON parsing attacks, ANSI escape, Web injection (XSS)
- **Evasion techniques**: Obfuscation, Character substitution, Translation-based attacks, Phrasing, ObscurePrompt
- **Safety benchmarks**: DoNotAnswer, RealToxicityPrompts, Snowball, LMRC
- **Agent attacks**: Multi-agent manipulation, Browsing exploits
- **Security testing**: Guardrail bypass, AV/spam scanning, Exploitation (SQLi, code exec), BadChars

## Quick Start

### Installation
```sh
go install github.com/praetorian-inc/augustus/cmd/augustus@latest
```

Or build from source:
```sh
git clone https://github.com/praetorian-inc/augustus.git
cd augustus
make build
```

### Basic Usage
```sh
export OPENAI\_API\_KEY="your-api-key"
augustus scan openai.OpenAI \
  --probe dan.Dan \
  --detector dan.DanDetector \
  --verbose
```
  