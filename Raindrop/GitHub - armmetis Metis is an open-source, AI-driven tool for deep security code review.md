---
raindrop_id: 1450911678
raindrop_highlights:
  69281a5052f782fef597a06a: b84e3ea14f872aff79fcaa3ffe244ce2
  69281a69efb30975227fa928: 1e60a365d96123163bb7e571c32b0326
  69281a71e14f420fc6871218: fc976d03212748037e798c71c8d74fee
  69281a7752f782fef597af52: 3957725411dc28114384b6835086bdef
  69281a7befb30975227faf1b: 9248e67467a08bae516f07fcc2f080e9
  69281a85ef944e55f1c27e6c: 0b0aa26869b112058b3832ebd170014d
  69281a898062310a4046e182: 970200aee5bf4b36821a9acfbf8751ef
  69281a99d0129212084c19b4: 00f441a0ae2141243bfe26cb7c8b8c28
  69281a9c8062310a4046e547: 615941ee523dbe4834f61a23106582e0
  69281aae372c4554dcd46a99: 66973458f4189fbcb4fd028cd5a1e986
  69281ab085c27a2716bf76bc: 0b18c8d5dc22c909f67c40b9b51c023b
  69281ab33895d0ca6702b41e: 9509947ed26f6e12f35f35bba8abc8e5
  69281ab63895d0ca6702b490: 1574fe62fcaf11535cdc317359384c88
  69281ab7372c4554dcd46d6c: 16f292f399151664176dd7ed2464c10a
  69281ad4372c4554dcd47657: e6e4412c04bbe0b7651781fcab9297f2
  69281adaef944e55f1c296b0: 0275a63245652606c05e2418860356eb
  69281ae00b421e2b354875e0: 4d9ee8ae3113cf18360fa5268d06d134
title: "GitHub - arm/metis: Metis is an open-source, AI-driven tool for deep security code review"

description: |-
  Metis is an open-source, AI-driven tool for deep security code review - arm/metis

source: https://github.com/arm/metis

created: 1764102474086
type: link
tags: ["_index"]

 
  - "AI" 
  - "Tools" 
  - "LLM" 
  - "SAST"

---
# GitHub - arm/metis: Metis is an open-source, AI-driven tool for deep security code review

![](https://opengraph.githubassets.com/e37e4c6185bb15a43669fb7ad2ecafd6cb57e35f3d75fcfa5259b4621d0834fd/arm/metis)

> [!summary]
> Metis is an open-source, AI-driven tool for deep security code review - arm/metis





Metis is an open-source, AI-driven tool for deep security code review
Features

Deep Reasoning Unlike linters or traditional static analysis tools, Metis doesn’t rely on hardcoded rules. It uses LLMs capable of semantic understanding and reasoning.

Context-Aware Reviews RAG ensures that the model has access to broader code context and related logic, resulting in more accurate and actionable suggestions.

Plugin-Friendly and Extensible Designed with extensibility in mind: support for additional languages, models, and new prompts is straightforward.

Provider Flexibility Works with OpenAI and other OpenAI-compatible endpoints (vLLM, Ollama, LiteLLM etc.). See the vLLM guide and the Ollama guide for local setup examples.
Installation
After cloning the repository, you can either create a virtual environment or install dependencies system-wide.
uv venv uv pip install .
Set up LLM Provider
export OPENAI_API_KEY="your-key-here"
Index and Run Analysis
uv run metis --codebase-path <path_to_src>
Run metis by also providing the path to the source you want to analyse:
Then, index your codebase using:
index
Finally, run the security analysis across the entire codebase with:
review_code
Metis Configuration (metis.yaml)

Metis configuration can be over-ridden using a YAML configuration file (metis.yaml) in the working directory when running metis. The default configuration is in src/metis/metis.yaml. This file defines all runtime parameters including:

LLM provider: OpenAI model names, embedding models, token limits
Engine behavior: max workers, max token length, similarity top-k
Database connection: In the case of PostgreSQL: host, port, credentials, and schema name
Vector indexing: HNSW parameters for pgvector

This file is required to run Metis and should be customized per deployment.
Prompt Configuration (plugins.yaml)

Metis uses a plugins.yaml file to define language-specific behavior, including LLM prompt templates and document splitting logic. Each language plugin (e.g., C) references this file to load:
Prompt Templates

You can customize a number of prompts like the following prompts:

security_review: Guides the LLM to perform a security audit of code or diffs.
validation_review: Asks the LLM to assess the correctness or quality of a generated review.
security_review_checks: A list of all the security issues the LLM will try to search for.

These prompts provide natural language context for the LLM and can be tailored to your use case (e.g., stricter audits, privacy reviews, compliance).