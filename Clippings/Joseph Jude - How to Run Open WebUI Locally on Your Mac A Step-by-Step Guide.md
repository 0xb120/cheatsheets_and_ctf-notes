---
title: "How to Run Open WebUI Locally on Your Mac: A Step-by-Step Guide"
source: "https://www.jjude.com/tech-notes/run-owui-on-mac/"
author:
  - "Joseph Jude"
published: 2025-01-05
created: 2026-01-03
description: "Run Open WebUI on MacBook Air M2 easily. Follow my guide for setup and automation to get ChatGPT-like responses locally."
tags:
  - "clippings/articles"
  - "_inbox"
---
# How to Run Open WebUI Locally on Your Mac: A Step-by-Step Guide

![](https://cdn.jjude.com/openwebui-install.webp)

> [!summary]
> This guide provides a step-by-step process for running Open WebUI locally on a Mac, specifically tested on a MacBook Air M2.
> It outlines the installation of `uvx`, setting up a Python virtual environment, installing Open WebUI, and then running the application.
> The article also includes a bash script to automate the startup and shutdown of Open WebUI, piping logs to a file.
> Finally, it covers options for selecting models, either via the OpenAI API key or by running models locally with Ollama.

How to Run Open WebUI Locally

## Introduction to Open WebUI

1. Install `uvx`
2. Initialize a Python virtual environment
3. Install Open WebUI
4. Run the application

## Step-by-Step Installation Guide

```
brew install uv
```

```
mkdir openwebui && cd openwebui
uv init --python=3.11 .
uv venv
source .venv/bin/activate
```

```
uv pip install open-webui
```

```
open-webui serve
```

Open WebUI is now accessible via: `http://0.0.0.0:8080/`.

## Selecting models to play with

You have two options to select models to work with OpenWebUI. I'm using the OpenAI API key to run chatGPT & GPT4-o models.

### 1. OpenAI API

Go to [OpenAI API Key Settings](https://platform.openai.com/settings/organization/api-keys) and register a new API Key. If you don't have an OpenAI account, you've to register first.

### Locally running models with Ollama

Another option is to [run Ollama locally](https://github.com/ollama/ollama).