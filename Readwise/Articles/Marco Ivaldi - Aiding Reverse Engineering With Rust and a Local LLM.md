---
author: Marco Ivaldi
aliases:
  - Aiding Reverse Engineering With Rust and a Local LLM
tags:
  - readwise/articles
url: https://security.humanativaspa.it/aiding-reverse-engineering-with-rust-and-a-local-llm/?__readwiseLocation=
date: 2025-04-24
summary: A large fraction of the flawsThe post Aiding reverse engineering with Rust and a local LLM appeared first on hn security.
---
# Aiding Reverse Engineering With Rust and a Local LLM

There’s a new entry in my ever-growing *vulnerability divination* [tool suite](https://security.humanativaspa.it/streamlining-vulnerability-research-with-ida-pro-and-rust/), designed to assist with **reverse engineering and** **vulnerability research against binary targets**: [oneiromancer.](https://github.com/0xdea/oneiromancer) [](https://read.readwise.io/read/01jrw3fdzsz9ssjnjqgry1j5xf)
> #tools 

It’s a reverse engineering assistant that uses a [fine-tuned](https://www.atredis.com/blog/2024/6/3/how-to-train-your-large-language-model), **locally running LLM to aid with code analysis**. It can analyze a function or a smaller code snippet, returning a high-level description of what the code does, a recommended name for the function, and variable renaming suggestions based on the results of the analysis. [](https://read.readwise.io/read/01jrw3g1vtn96xvgrwr48kh2q7)

Abither **useful extension** that’s relatively new and therefore wasn’t mentioned in [my previous article](https://security.humanativaspa.it/big-update-to-my-semgrep-c-cpp-ruleset/) that introduced a simple code review workflow: [SARIF Explorer](https://marketplace.visualstudio.com/items?itemName=trailofbits.sarif-explorer) by [Trail of Bits](https://blog.trailofbits.com/2024/03/20/streamline-the-static-analysis-triage-process-with-sarif-explorer/) [^1]. Much better than the [alternatives](https://marketplace.visualstudio.com/items?itemName=MS-SarifVSCode.sarif-viewer), in my opinion. [](https://read.readwise.io/read/01jrw3k8ydted8c3f65yw27az9)
> #tools

[^1]: [SARIF Explorer in VSCode](../../Dev,%20ICT%20&%20Cybersec/Tools/vscode.md#^b54027)

