---
title: "praetorian-inc/julius: Simple LLM service identification - translate IP:Port to Ollama, vLLM, LiteLLM, or 30+ other AI services in seconds"
source: https://github.com/praetorian-inc/julius
author:
  - mikepiotrowski2
published:
created: 2026-02-28
description: Simple LLM service identification - translate IP:Port to Ollama, vLLM, LiteLLM, or 30+ other AI services in seconds - praetorian-inc/julius
tags:
  - clippings/articles
  - _inbox
  - tools
  - AI
  - LLM
---
# praetorian-inc/julius: Simple LLM service identification - translate IP:Port to Ollama, vLLM, LiteLLM, or 30+ other AI services in seconds

![](https://opengraph.githubassets.com/8400aeded28a83824f469423b8dd44a2fba6402d93b10563bfd343132cec7a24/praetorian-inc/julius)

> [!summary]+
> > This page describes Julius, an open-source LLM service fingerprinting tool by Praetorian.
> 
> Julius identifies the server infrastructure behind AI services (e.g., Ollama, vLLM, LiteLLM, Hugging Face TGI, etc.) on network endpoints, as opposed to identifying the LLM model itself. It supports over 30 services across self-hosted, gateway/proxy, RAG/orchestration, and cloud-managed categories.
> 
> Key features include: fast, concurrent scanning; model discovery; specificity scoring; multiple input/output formats (table, JSON, JSONL); extensibility via YAML probes; and offline operation.
> 
> The tool works by sending targeted HTTP probes and matching response signatures against predefined rules (status codes, body content, headers).
> 
> The document also provides quick start instructions (installation, basic usage), detailed usage examples (single/multiple targets, output formats, model discovery, advanced options), an architectural overview, guidelines for adding custom probes, and an FAQ section covering common questions and troubleshooting.
> 
> It emphasizes that Julius is for authorized security testing only and does not exploit vulnerabilities or perform destructive actions.

Simple LLM service identification - translate IP:Port to Ollama, vLLM, LiteLLM, or 30+ other AI services in seconds

# Julius: LLM Service Fingerprinting Tool

Identify Ollama, vLLM, LiteLLM, and 30+ AI services running on any endpoint in seconds.

**Julius** is an LLM service fingerprinting tool for security professionals. It detects which AI server software is running on network endpoints during penetration tests, attack surface discovery, and security assessments.

Unlike model fingerprinting tools that identify which LLM generated text, Julius identifies the **server infrastructure**: Is that endpoint running Ollama? vLLM? LiteLLM? A Hugging Face deployment? Julius answers in seconds.

### Installation

```sh
go install github.com/praetorian-inc/julius/cmd/julius@latest
```

### Basic Usage

```sh
julius probe https://target.example.com
```

## Usage

### Single Target

```sh
julius probe https://target.example.com
julius probe https://target.example.com:11434
julius probe 192.168.1.100:8080
```

### Multiple Targets

```sh
# Command line arguments
julius probe https://target1.example.com https://target2.example.com

# From file (one target per line)
julius probe -f targets.txt

# From stdin (pipe from other tools)
cat targets.txt | julius probe -
echo "https://target.example.com" | julius probe -
```

### Output Formats

```sh
# Table format (default) - human-readable
julius probe https://target.example.com

# JSON format - structured output
julius probe -o json https://target.example.com

# JSONL format - one JSON object per line, ideal for piping
julius probe -o jsonl https://target.example.com | jq '.service'
```
