---
raindrop_id: 1473683468
raindrop_highlights:
  6932c6b562cda985ad05addf: 227bba7ecf00d78d7376dbcc1efefd9d
  6932c70d967f0ad1bc0c6782: de7aa4b4dcd9a3da32df411daa93b259
  6932c7df1ff4a506fb1897cb: 6d946a056334feaf5bf6d88be6b2e8c1
  6932c819a8ccb40ed4617153: c72f423d4890c89a3c0de0e1d09dac45
  6932c8369e1087a792671fda: abb98e75dbc38699b2bbf1f12820d699
  6932c847a5633c2d809f9ca4: d7fb67007ca191f9a2156a2adb8769b7
  6932c8a237c4fe98c37d91ac: bf01f248ec234b8eff342a64e2ea4413
  6932c92399e41892ea0d0ad9: 53d8f3420d5cc36a5292da1f881bc228
  6932c95fafb65b671a792d01: fe389e96956ecf1cd142d4af43ffeba8
title: "Home-made LLM Recipe - VoidSec"

description: |-
  A practical guide to building a homemade local LLM platform using Ollama, and Open WebUI: hardware choices, benchmarks, costs, and performance

source: https://voidsec.com/home-made-llm-recipe/

created: 1764870341159
type: article
tags:
  - "_index"

 
  - "AI" 
  - "LLM"

---
# Home-made LLM Recipe - VoidSec

![](https://voidsec.com/wp-content/uploads/2025/12/LLM.png)

> [!summary]
> A practical guide to building a homemade local LLM platform using Ollama, and Open WebUI: hardware choices, benchmarks, costs, and performance





Today I would like to sneak a peek into my high-level process of setting up a homemade local LLM platform, which we’re currently using as an internal pilot project; how we did it, what setup and technologies we’ve used, etc…
Hardware

While deciding on which hardware to use for our pet projects, we started reviewing both dedicated hardware for LLMs, GPU racks and any kind of hardware capable enough to run such models.
While GPUs seem to be the standard, dedicated racks are somewhat expensive, and for a pilot project, we didn’t want to justify new purchases. We opted to test a 2022 Mac Studio that was sitting idle in the office.
Dell XPS:

CPU: 13th Gen Intel Core i9-13900H
RAM: 64 GB
GPU: NVIDIA GeForce RTX 4070
OS: Windows 11 Pro x64

Mac Studio 2022:

CPU: M1 Ultra
RAM: 128 GB
GPU: 64-core
OS: macOS 26.1

When testing the hardware, we opted for the 2 following prompts across different models:

“Write me a poem about the moon”
“Write a Python binary tree lookup function with a sample”

Both are vague enough to see, where present, the “thinking” process of the models.
For the local setup of the models, we opted to use Ollama, which can be directly installed from its main website without further complications. Then we spent some time selecting models we were interested in testing, opting for some small enough to be compared with our workstation and some bigger ones, as the Mac Studio’s RAM allowed us to do so.
We selected the following:

1:8b
qwen3:8b
5-coder:3b
deepseek-r1:8b
deepseek-r1:70b
gpt-oss:120b
3:70b
qwen3-coder:30b

Models can be pulled via the command line: ollama pull llama3.1:8b
Based on the collected performance metrics, the Mac Studio 2022 significantly outperformed our workstation across every meaningful measurement: while the model load times are similar, the total duration drops significantly. The prompt evaluation (~40% faster) and generation throughput (~6x faster) clearly favour the Mac Studio for real-time workload and interactive development.
Code Completion

While we initially used Cursor for fast script and prototyping, we’re trying to replace it with Continue, configured to use our local LLMs. Though maybe just for the ease of use and the fact that I’ve gotten used to it, Cursor still feels way better in terms of usability and results.
Technology Stack
Hardware: Mac Studio 2022
Software: Open WebUI, Ollama