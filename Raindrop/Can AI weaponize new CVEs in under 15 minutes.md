---
raindrop_id: 1321009681
raindrop_highlights:
  68d69e56c09d89001785e3fc: bbe2cb4eb9c2e970fce9c2422227ffea
  68d69e5fb3703b68c14721dd: c6401339bf4097f135f3de304ad4051b
  68d69e6999fd7e9b3ec1ab4d: 44d5c8b0a85133061b1d582df16ba8a8
  68d69e923d2379b5b1325cd6: a3c5733827c406df6df6bc29799f88f5
  68d69eb8b3703b68c1472eca: 5129a40ebc519fc75e92108705f4f6de
  68d69eccd312ed5d6fe26385: e6a2e71599fc9b3375d8878bcde01160
  68d69ee1f9a75cbba11686f9: 584dff6f6947106ca011782a44a9cfad
  68d69ef7b3703b68c14737c2: 19fd22cd6531ce2984fea01116f37a89
  68d69effb81b9bf2809cc14c: 2375d51e1b44d6fe2a6f967c430d098d
  68d69f0f91c768ef7e7cc54c: 5df90e5affc80c3028a7b8e187cb7145
  68d69f2591c768ef7e7cc917: 30956f1bd8407386eb10e8d6e40322d7
  68d69f8c5aebcc0fd33b7172: 5263688da23a3172917e8060372211b2
  68d69fa9d323398e61470972: 394666c705d93e91d76dc8c4c41d2e6c
  68d69fbf5430135b7f9c67e4: 0ebcb50d9f2eb6b172b1f32c89eec478
  68d69fc90ebee95a2aefa81e: 9b815980b3d1893c803e963539bdee4c
  68d69fd83b739d334338ccdc: b7d83d311f913e356b8d2c6218d83e86
  68d69feab3703b68c1475fe7: 0cd5e84ce48395c9fe7bcb7660a27b71
  68d6a02561e4202863dc6f47: ed61e5889c5ccaa7696d70087236a7f7
  68d6a082d312ed5d6fe2a641: 9dba20e5ea88218b67a06ba8f30884cc
  68d6a0a2f45d2985257c7dcc: 0bac9b19de70ecf6b318244d779e84ed
title: "Can AI weaponize new CVEs in under 15 minutes?"

description: |-
  If AI can mass-produce exploits, how much time do defenders really have left?

source: https://valmarelox.substack.com/p/can-ai-weaponize-new-cves-in-under

created: 1756708780758
type: article
tags:
  - "_index"

 
  - "AI" 
  - "LLM" 
  - "vuln-research-blog" 
  - "tech-blog" 
  - "exploit-dev"

---
# Can AI weaponize new CVEs in under 15 minutes?

![](https://substackcdn.com/image/fetch/$s_!6BU3!,w_1200,h_600,c_fill,f_jpg,q_auto:good,fl_progressive:steep,g_auto/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F4d0eb0c7-0d7d-4d56-b2d9-9053113bb136_1068x624.png)

> [!summary]
> If AI can mass-produce exploits, how much time do defenders really have left?





Can AI weaponize new CVEs in under 15 minutes?
We built an AI system that automatically generates working exploits for published CVEs in 10-15 minutes for ~$1 each.
You can see the generated exploits here.
The system we have built uses a multi-stage pipeline: (1) analyzes CVE advisories and code patches, (2) creates both vulnerable test applications and exploit code, and (3) validates exploits by testing against vulnerable vs. patched versions to eliminate false positives.
The methodology we chose going into this was:

Data preparation - Use the advisory + repository to understand how to create an exploit. This is a good job for LLMs - advisories are mostly text, and the same is true for code. The advisory will usually contain good hints to guide the LLM

Context enrichment - Prompt the LLM in guided steps to create a rich context about exploitation - how to construct the payload? What is the flow to the vulnerability?

Evaluation Loop - Create an exploit and an example “vulnerable app” to test the exploit until it works.
Stage 0 - The model
SaaS models like OpenAI, Anthropic, or Google APIs usually have guardrails causing the model to refuse to build POCs - either explicitly or by providing generic “fill here” templates. We started with `qwen3:8b` hosted locally on our MacBooks and later moved to `openai-oss:20b` when it was released. This was really useful as it allowed us to experiment for free until reaching a high level of maturity.
A bit later, we found out that given the long step-by-step prompt chain we ended up making, the SaaS models stopped refusing to help :) Claude-sonnet-4.0 was the best model for generating the PoCs
Stage 0.5 - The Agent
We started with directly interfacing with the LLM APIs, but later refactored to pydantic-ai (which is amazing BTW)
Stage 1 - CVE → Technical analysis
We also created a short pipeline that clones the repository, extracts the patch using the given vulnerable and patched versions (with some LLM magic to solve for edge cases).

Now we can feed the advisory and the patch to the LLM and ask it to analyze it in steps to guide itself to create a plan on how to execute it
We broke down the task into several prompts on purpose - to allow us to debug each prompt quality separately and make the play easier.
After guiding the agent through a thorough enough analysis, we can use a summarized report of the research as context for the next agents.
Stage 2 - Test plan
We want to create a working POC for open-source packages. Anyone who has coded with AI knows that the chances of getting exactly what you expect on the first try are essentially zero.
So to generate good exploits, we must create a test environment - a vulnerable app and an exploit, and test them against each other.
To execute the vulnerability and POC safely, we chose to use dagger to create sandboxes. Dagger is a cool project that lets you very easily spin up containers programmatically (amongst other nice features).
Summary
Results

All the results are available in this GitHub repository and Google Drive. All the zip files in the Google Drive are timestamped using opentimestamps. The entire execution for one CVE takes around 10-15 minutes.

At the time of publication, there are 10 working exploits. You can explore the generated exploits and join the waitlist on our site.
The Future?

There are many directions to take this. This research proves that “7~ days to fix critical vulnerabilities” policies will soon be a thing of the past. Defenders will need to be drastically faster - the response time will shrink from weeks to minutes (Stay tuned :)).