---
raindrop_id: 1358478587
raindrop_highlights:
  69679142c9e7cd1df5f78fc1: 85c775c9cfdb5d0c964083545a61c587
  6967915aebcc6c6528bbb68e: f9b28d5455d4528d2b2969c5e540c6e3
  696791edc4166060ada4f521: b0f89f20f0ec5dbaee9e95e78a04bf9c
  6967920198e1a520c92ceb69: 6fb2cc2e0783b482add3fa48a0bd4066
  6967924198e1a520c92cfb52: a7558276b5d7ae24f4654b7b8810bd56
  69679250190a3d4d4c721aa4: f5e721ed2ae02f475ac7d4270f14e313
  6967926cc4166060ada51625: 4195dad24abb274add0d84354686289d
  696792753200cedebbc247b9: c74d4a80f6f3070eb7898ebe5014f3ed
  69679295a59e08399f1d457b: 8fdbaba1c108c88ff68518237b937f05
  6967929debcc6c6528bc06c6: d702691548a336a790506f7446656bee
  696792a902ae78226beaba79: c8dd80a30beb2e0d6fd6b35136ea94bf
  696792b398e1a520c92d17a0: fdd52d8650895b548d087cf471fb80e3
  696792d83200cedebbc26034: 79d1cde504337cc06550f87aa6ae2f20
  696792f6a59e08399f1d5c19: 7470487a559530110981a95368db957a
  696792fad202445116235512: 90293c304f00195e6c345ca4b409641d
  6967930bf0d02295c99f221d: 6550a8293df762a38b0c8afcdf239804
  696793343c9ce0f5444e6f02: 0f80653844a3ba7a11b600a225ba8430
  6967934a98e1a520c92d3b6a: 9835fde6ee0dab89894a67906b6ffd84
  69679357f0d02295c99f33ef: 97645b8dbbf178e267682db9613f2fef
  6967935b3200cedebbc27eef: a757020b99e56de1475b116319cc4eb9
  69679366d44afdbd7c228cab: 694a272c901813c2d7db45a7dbaf1331
  6967938cc9e7cd1df5f82251: 688f95adee9ff6902fbc71c1ff66c6b8
  696793a2d44afdbd7c229bed: 21acd226f0be99667c278f8fe527d9eb
  696793a99e7ab5b88a6963be: 069f5ea38799963e8e4d31762c761d85
  696793c7d44afdbd7c22a4af: b2bcfa42fdc3d39f102c05f36328094f
  696793cb190a3d4d4c7276b0: 793d4e162e5eaaf6503f999ac371559c
  696793d598e1a520c92d5d09: ceb62ed56d7d32d641c748a2d75890d9
  696793f09e7ab5b88a6975b5: 1b93cf1eb68e8eeffc6a30d4249a9024
  69679407d44afdbd7c22b4bb: b35afc48977ec13c91c65b942bc2238b
  696794138226ac55cad817bd: a52641854d45117a31c5b2c1579f4eb0
  69679417ebcc6c6528bc6313: e31f297f591a46fdfc9522936d18c7b0
  6967941d02ae78226beb1110: dbf9ba322f896863fe2244f7e161f521
  6967942202ae78226beb123c: c5bd71e216e1f7151b8083a2fee81800
  6967942d3c9ce0f5444ead77: be6bfec6a58e25a9bf998cc99c98bca6
title: "AI agents building security tests - Labs Detectify"

description: |-
  The Detectify AI Agent Alfred fully automates the creation of security tests for new vulnerabilities, from research to a merge request. In its first six ...

source: https://labs.detectify.com/writeups/ai-agents-building-security-tests/

created: 2025-09-25
sync-date: 1769114383369
tags:
  - "_index"

 
  - "AI" 
  - "LLM" 
  - "tech-blog" 
  - "exploit-dev"

---
# AI agents building security tests - Labs Detectify

![](https://labsadmin.detectify.com/app/uploads/2025/09/Alfred.png)

> [!summary]
> The Detectify AI Agent Alfred fully automates the creation of security tests for new vulnerabilities, from research to a merge request. In its first six ...





The Detectify AI Agent Alfred fully automates the creation of security tests for new vulnerabilities, from research to a merge request.
How did we build this? Which prompts did we use? What did we learn?
we decided to focus on building a system with two core principles:

Source everything out there.
Automate what matters.
Our AI Security Researcher Alfred is a workflow based on a 10-step process, implemented in Go as a chain of agents implemented in OpenAI-mini mode.
Step 1: The Funnel of Sourcing
Alfred continuously sources vulnerabilities from over 200 sources, including CERTs (like CERT-EU and CERT-SE), public vendor advisories (like Acunetix and Rapid7), and news sites and communities (like Reddit and HackerNews).
providing a much wider range of vulnerabilities compared to relying solely on the NVD,
Step 2: Getting All the References
Once a vulnerability is identified, Alfred gets all supporting references. This includes scouring GitHub commits, vendor advisories, and even social media mentions to collect every piece of technical information available.
Step 3: Prioritizing with EPSS
To ensure we’re focusing on the most critical threats, Alfred sorts all vulnerabilities by their Exploit Prediction Scoring System (EPSS) score.
Step 4: Grouping and Structuring Data
Alfred fetches all content from all URLs and has an LLM group the content into categories. The LLM uses critical rules to categorize content as a “poc” if executable exploit code is present, or other descriptive categories like “advisory,” “remediation,” or “analysis”.
Categorize this security content related to %s using your best judgment.

CRITICAL RULES:

- You MUST use "poc" if and ONLY if executable exploit code is present with sufficient detail to reproduce the exploit

- For all other content, choose a descriptive category that best represents the content (e.g., "advisory", "remediation", "analysis", "detection", "discussion", etc.)

- Choose a single category that most accurately describes the primary nature of the content

- Be specific and descriptive with your chosen category

- Create a concise title (5-10 words) that accurately summarizes the document's content and its type (e.g., "WordPress RCE Exploit Code" or "Apache Advisory for CVE-2024-1234")

IMPORTANT: "poc" has a strict definition - it MUST contain actual code or commands that could be executed to exploit the vulnerability.

Your response must be a single JSON object with two properties:

{"category": "category_name", "title": "Your concise document title"}`
Step 5: Note-taking
An LLM will learn the exploit and take notes on how it works
Alfred’s task is to analyze content and extract all technical information necessary to understand and potentially reproduce the vulnerability.
These notes are a precise, exhaustive documentation of the attack vector, prerequisites, and every technical detail needed for reproduction.
Your task is to analyze this content related to vulnerability and extract ALL technical information necessary to understand and potentially reproduce the vulnerability.

IMPORTANT: Base your analysis STRICTLY on the content provided. Do not add information from your own knowledge or assumptions.

Document EXHAUSTIVELY:

- The complete attack vector and exploitation methodology

- ALL technical details about how the vulnerability works

- EVERY prerequisite and environmental requirement

- ALL steps in the exploitation process

- EXACT specifications of any unusual formatting or techniques

- FULL details on target behavior during and after exploitation

- (Prompt cut to fit text) 




For ANY code, commands, or HTTP requests:

- Include them COMPLETELY and EXACTLY as presented

- Preserve ALL syntax, formatting, and structure

- Document ALL parameters, flags, and options

- Note ALL external dependencies or tools required

REMEMBER: These notes will become your ONLY reference for future analysis of this vulnerability. You will never see this content again, so be exhaustive, precise, and avoid omitting ANY technical details.`
Step 6: Triaging for Feasibility
Alfred acts as a security analyst to triage how feasible a vulnerability is for implementation.
It evaluates the previously documented notes and answers a series of true/false questions based only on the technical details provided.
Step 7: Select good candidates for implementation
Alfred selects good candidates for implementation based on a ranking system.
The system adds bias for vulnerabilities with proof-of-concepts, newer CVEs, higher EPSS and CVSS scores, and more relevant sources to prioritize the most relevant vulnerabilities.
Step 8: Develop the Test Module
The next step is development, which happens through rapid iterations until “it works”.
Alfred’s goal is to port its technical notes into a standardized JSON specification for a Detectify test module. A computer will parse this output, so exact adherence to the schema is critical.
A key requirement is to always use concrete, executable payloads—never placeholders.
Step 9: Creating the Merge Request
Once the module is ready, Alfred opens a merge request in GitLab.
This allows our internal team of security researchers to review the generated test and ensure it meets our high-quality standards.
Step 10: Getting it Production Ready
The final step is to fix smaller issues and prepare the test for production