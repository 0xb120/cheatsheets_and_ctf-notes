---
raindrop_id: 1502269534
raindrop_highlights:
  697f31f61d8a92fd811f2510: 3a306694e76ef1100ddda1ccdd06b124
  697f331fcda9c53e0f2d0fc2: aee933cc0aed80380280afe01e6e2ffc
  697f33268cd529f961e34a8b: d3e8710ad2466ebf7e1afeb2ff1aebd1
  697f334235cd89fe7506343d: 778e15e45e2ad6b936210099510405b5
  697f33562b0cce59d17f3217: f4189a4a981257b308eab14266245f0f
  697f3369bccd68c6b58e2b0a: 68a66ba4fc1e6aa08fa88af9a073e2c1
  697f339dbccd68c6b58e32c0: 07a904b1e77b98db38ab9851a77667b8
  697f33b50eedfe6b62976947: b0d4fe16143c3aa1e2891dc7135c6e08
  697f3446bccd68c6b58e4a0d: 61b11bc2d48b3f5dfd464cc6c7711e46
  697f345624748104f136521f: 78d6f51eb739b456bb7c27c0698d42d2
  697f34634d80a1b75882d57c: 39e27f4b9bf1b46da6efeb45acb403bd
  697f346bcda9c53e0f2d4123: 12a3f24f56772ef844d35b29feb68e83
  697f347cf70144491a45dd14: a17df73d5c5045b85d40450a77df76af
  697f34cdf70144491a45e87b: 3e2af4cc392a4cf83b7affe7c7bd0ef1
  697f34e09945bba153c0061e: a0df52adada5280229136836e961b9b4
  697f34e7f70144491a45eca1: 83af645e59d56e5df5e1fd3c073f5535
  697f34ed4b15709a910a538b: 6f4e1ec45a2de6bf3c21f5ee6dff10b0
  697f34f6c2a0a6c820587c14: 1cbde79a92cceb6c0c963b5c74dee3c8
  697f34fc9945bba153c00b12: 1205cddae03b799fe60bbb945ba9cd00
  697f351224748104f1366ef1: 27f0c9e133637bfa0c936dd72d11d7e2
  697f356d71bd17c33ede0916: f55dea1e5e43a0e016529d344426bb14
  697f35779945bba153c01b99: 2b0c6b7633cfbec0a09f1fb3857b66bb
  697f358c4b15709a910a6c7c: cf73b81320d59874284ebf70be32c957
title: "[Revised] You Don’t Need to Spend $100/mo on Claude Code: Your Guide to Local Coding Models"

description: |-
  What you need to know about local model tooling and the steps for setting one up yourself

source: https://www.aiforswes.com/p/you-dont-need-to-spend-100mo-on-claude

created: 2025-12-22
sync-date: 1770019410932
tags:
  - "_index"

 
  - "AI" 
  - "LLM" 
  - "tech-blog"

---
# [Revised] You Don’t Need to Spend $100/mo on Claude Code: Your Guide to Local Coding Models

![](https://substackcdn.com/image/fetch/$s_!ycw4!,f_auto,q_auto:best,fl_progressive:steep/https%3A%2F%2Fsocietysbackend.substack.com%2Fapi%2Fv1%2Fpost_preview%2F182132050%2Ftwitter.jpg%3Fversion%3D4)

> [!summary]
> What you need to know about local model tooling and the steps for setting one up yourself





What you need to know about local model tooling and the steps for setting one up yourself
Understanding memory

To get going with local models you must understand the memory needed to run them on your machine. Obviously, if you have more memory you’ll be able to run better models, but understanding the nuances of that memory management will help you pick out the right model for your use case.
Local AI has two parts that eat up your memory: The model itself and the model’s context window.
The actual model has billions of parameters and all those parameters need to fit into your memory at once. Excellent local coding models start at around 30 billion (30B, for short) parameters in size. By default, these models use 16 bits to represent parameters. At 16 bits with 30B parameters, a model will take 60 GB of space in RAM (16 bits = 2 bytes per parameter, 30 billion parameters = 60 billion bytes which equals about 60 GB).
The second (and potentially larger) memory consuming part of local AI is the model’s context window. This is the model inputs and outputs that are stored so the model can reference them in future requests. This gives the model memory.
When coding with AI, we prefer this window to be as large as it can because we need to fit our codebase (or pieces of it) within our context window. This means we target a context window of 64,000 tokens or larger. All of these tokens will also be stored in RAM.
Luckily, there are tricks to better manage memory. First, there are architectural changes that can be made to make model inference more efficient so it requires less memory.
The model we set up at the end of this article uses Hybrid Attention which enables a much smaller KV cache enabling us to fit our model and context window in less memory. I won’t get into more detail in this article, but you can read more about that model and how it works here.

The second trick is quantizing the values you’re working with. Quantization means converting a continuous set of values into a smaller amount of distinct values.
Getting set up
For my tool, I ended up going with Qwen Code. It was pretty plug-and-play as it’s a fork of Gemini CLI. It supports the OpenAI compatibility standard so I can easily sub in different models and affords me all of the niceties built into Gemini CLI that I’m familiar with using.
I also know it’ll be supported because both the Qwen team and Google DeepMind are behind the tool. The tool is also open source so anyone can support it as needed.
For models, I focused on GPT-OSS and Qwen3 models since they were around the size I was looking for and had great reviews for coding.
I decided to serve my local models on MLX, but if you’re using a non-Mac device give Ollama a shot.
Install MLX or download Ollama
Increase the VRAM limitation on your MacBook.
Run pip install -U mlx-lm to install MLX for serving community models.
Serve the model as an OpenAI compatible API using python -m mlx_lm.server --model mlx-community/Qwen3-Next-80B-A3B-Instruct-8bit
Download Qwen Code. You might need to install Node Package Manager for this. I recommend using Node Version Manager (nvm) for managing your npm version.
Set up your tool to access an OpenAI compatible API by entering the following settings:

Base URL: http://localhost:8080/v1 (should be the default MLX serves your model at)

API Key: mlx

Model Name: mlx-community/Qwen3-Next-80B-A3B-Instruct-8bit (or whichever model you chose).
One tip I have for using local coding models: Focus on managing your context. This is a great skill even with cloud-based models. People tend to YOLO their chats and fill their context window, but I’ve found greater performance by ensuring that just what my model needs is sitting in my context window.
One wrench thrown into my experiment is how much free quota Google hands out with their different AI coding tools.
Initially, I considered my local coding setup to be a great pair to Google’s free tier. It definitely performs better than Gemini 2.5 Flash and makes a great companion to Gemini 3 Pro.
However, this is foiled a bit now that Gemini 3 Flash was just announced a few days ago. It shows benchmark numbers much more capable than Gemini 2.5 Flash (and even 2.5 Pro!) and I’ve been very impressed with its performance.