---
raindrop_id: 1557439116
raindrop_highlights:
  697f2af10eedfe6b62960f0a: 5b5c61ed6fb1b58a273ecc15da39f06d
  697f2b0371bd17c33edc6bef: 8183d8f17de0a654110b93bdcdc52ef7
  697f2b2bcda9c53e0f2bd3b8: 1e28368c8cdfcbe60dee33c4ca6f8443
  697f2b41e81488f120569be6: aa2e8c64c4345fab02e65da7aa7fb54a
  697f2b444b15709a9108d6ee: 7d4f85a79027bebe4222ed2d6307165d
  697f2b5435cd89fe7504f66b: 1da06ca2b5f5dd3eb94f186f1461034f
  697f2b57c2a0a6c82056fe51: dc77ca58e13af73741e92c1d5a74ba58
  697f2b74c21eb5352be54197: 80f90e7a10f4b78729a1eb54d29b0487
  697f2b7824748104f134f008: 6ddb253ca1d4e3d88223e78b4661ce02
  697f2b7f8cd529f961e21d69: c87c5b65488bacf5d448ab221ab23326
  697f2ba42b0cce59d17e0020: 074c6ff97570a050ac4cd279f010f506
  697f2bddc2a0a6c8205712a1: 12211be38f03854b61031e172f006434
  697f2bf3f70144491a448dd0: 944936b771687e0eceeb64e1b1629586
  697f2bf8cda9c53e0f2bf519: 65da5e09021bbd7f079bc87ac12a73bb
  697f2bfd34992f7c88f902b6: e831fb441e1ede17b8790910f520f87d
  697f2c004d80a1b758819265: 1377d48021a80c739bee14a6cf5ba0c0
  697f2c0af70144491a4491c4: 29ad71cc6564510ec4d1ceab8d7e963b
  697f2c2124748104f13509c2: 4de941339885343656285423fbc21ad2
  697f2c29c21eb5352be55cf6: 5738a0c65af7af557a529b44bc1b4d26
  697f2c3034992f7c88f909dc: 591286cb15d9853c557e195c55611630
  697f2c3934992f7c88f90b81: 3728902410900cadd534dedb44262cc0
title: "XBOW - Why LLMs Hallucinate Vulnerabilities"

description: |-
  LLMs are powerful at spotting patterns and proposing possible vulnerabilities, but confidence is not actual proof. In this post, I explain why raw LLM output can’t be trusted as a finding, and why validation must exist outside the model to separate real vulnerabilities from convincing noise.

source: https://xbow.com/blog/why-llms-hallucinate-vulnerabilities

created: 2026-01-23
sync-date: 1770019410940
tags:
  - "_index"

 
  - "LLM" 
  - "AI"

---
# XBOW - Why LLMs Hallucinate Vulnerabilities

![](https://cdn.prod.website-files.com/686c11d5bee0151a3f8021d6/696914e30d31edadbeffa91d_llmsblog.png)

> [!summary]
> LLMs are powerful at spotting patterns and proposing possible vulnerabilities, but confidence is not actual proof. In this post, I explain why raw LLM output can’t be trusted as a finding, and why validation must exist outside the model to separate real vulnerabilities from convincing noise.





Large Language Models (LLMs) are increasingly used in security tools to analyze applications, suggest exploits, and identify potential vulnerabilities. In many cases, they surface useful leads faster than traditional approaches.

But teams trying to operationalize LLM-driven security quickly hit a familiar problem: The model sounds confident, but the vulnerability isn’t real.
This isn’t a flaw in a specific vendor’s model, nor something solved with better prompts or more context. It’s a fundamental mismatch between how LLMs reason and how vulnerabilities actually exist in the real world.
In AI, hallucination refers to generating information that isn’t grounded in reality. In security, it shows up in subtler (and more expensive) ways. For example:

Inferring SQL injection from a response pattern that only resembles prior exploits
Suggesting an endpoint is exploitable because it matches a known vulnerability class
Asserting impact without ever demonstrating it
These outputs aren’t random.
The model is doing exactly what it’s designed to do: generate the most plausible explanation based on prior examples.
The problem is simple: plausibility is not proof. In security, a vulnerability only exists if it can be exercised against a real system.
Raw LLM output can’t be trusted
LLMs don’t observe reality.
They reason abstractly about what should happen, what usually happens, what has happened elsewhere, and interpret what might have happened here. But they don’t confirm what did happen.
This is why treating raw LLM output as a finding creates noise.
This limitation isn’t fixable by making the model smarter. Even a perfect reasoning engine still lacks direct access to ground truth
Hypotheses are useful but conclusions are dangerous
Used correctly, LLMs are extremely valuable in security workflows. They’re good at:

Generating creative exploit hypotheses
Recognizing subtle vulnerability patterns
Connecting behavior to historical flaws
Exploring unusual inputs and edge cases ‍

This mirrors how human pentesters work. They start with suspicion, not proof. The difference is what happens next.
A hypothesis is a starting point.
A conclusion requires evidence.
Validation must exist outside the model
Because LLMs can’t observe reality directly, validation can’t live inside the model.
When an AI-driven system suspects a vulnerability, that suspicion should trigger a second phase that attempts to prove or disprove the idea using deterministic checks, such as:

Measuring timing differences to confirm blind injection
Verifying whether a server makes an outbound request
Checking for access to data that should be unreachable
Observing real browser behavior for client-side impact
The takeaway for security teams
LLMs are powerful amplifiers for exploration. They help systems think more like attackers and cover more ground than manual testing ever could. But they can’t be the final authority on whether a vulnerability exists.
Security teams don’t need AI that sounds certain. They need systems that are skeptical by default and assume the model might be wrong and demand proof before surfacing results.