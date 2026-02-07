---
raindrop_id: 1571182525
raindrop_highlights:
  697e23947303d25eccd20643: 0f2590eaa7a1aad13c82c273894dc2c6
  697e23c37303d25eccd21100: 5ac3612c08264108b96635b2a284bcf3
  697e23e161569c6cd261fd3b: 3c5d42381f0e8383e4322f771e8dd2dc
  697e23e79945bba15393a314: a2b1a53926233721fe2995be64bee88a
  697e23ea61569c6cd261ff1a: 649b180912f273688823b91eddd01b4d
  697e24053605a0be364e1348: a6cb4b0d2b467551dac58aee23d2dafa
  697e24194d80a1b758569d29: 8d04c783920049e3dd468a2bf5960527
  697e242524748104f10a0ae0: d7cb447498aedf4cabaffb758613b086
  697e242d7303d25eccd22613: 2c82dc33a3e56ca09720797dacf52779
  697e24651d8a92fd81f33e1a: 66cf30e6375e5db43fb404c4f5406748
  697e24704b15709a91de1198: cbcad9c55f16b98d4975d96a7aaedf0d
  697e247fe81488f1202bbfe4: 65548565983a6783081a99939a83ffa1
  697e249ec2a0a6c8202c5619: dd91d0618dd7d8cc6ee434e7ade8e968
  697e24b361569c6cd2622641: 8dbc7579239cf4f384409e1fc58735d0
  697e24be0eedfe6b626b538c: 7c6c266a07d7548a050c462d963403e6
  697e24c4bccd68c6b5623560: dd708175bd2ab6e9a7a8828e161d2f64
  697e24ce35cd89fe75da4ad2: a27b3638a37b777a8c22032e6fe061a0
  697e24d49945bba15393d1c1: 734551caa378ec6b5e97608a7ec94e46
  697e24dd71bd17c33eb1dc7e: 039a27b694a5dee2f63b96b66b3efde8
title: "AGENTS.md outperforms skills in our agent evals - Vercel"

description: |-
  A compressed 8KB docs index in AGENTS.md achieved 100% on Next.js 16 API evals. Skills maxed at 79%. Here's what we learned and how to set it up.

source: https://vercel.com/blog/agents-md-outperforms-skills-in-our-agent-evals

created: 2026-01-31
sync-date: 1770019410950
tags:
  - "_index"

---
# AGENTS.md outperforms skills in our agent evals - Vercel

![](https://assets.vercel.com/image/upload/contentful/image/e5382hct74si/NrCaFKRo4wJnZm4hhbuJ0/752791d02cda6e1f367dbfefed3ed963/agentsmd_outperforms_skills_in_our_agent_evals_og_card.png)

> [!summary]
> A compressed 8KB docs index in AGENTS.md achieved 100% on Next.js 16 API evals. Skills maxed at 79%. Here's what we learned and how to set it up.





Two approaches for teaching agents framework knowledge
Before diving into results, a quick explanation of the two approaches we tested:

Skills are an open standard for packaging domain knowledge that coding agents can use. A skill bundles prompts, tools, and documentation that an agent can invoke on demand. The idea is that the agent recognizes when it needs framework-specific help, invokes the skill, and gets access to relevant docs.

AGENTS.md is a markdown file in your project root that provides persistent context to coding agents. Whatever you put in AGENTS.md is available to the agent on every turn, without the agent needing to decide to load it. Claude Code uses CLAUDE.md for the same purpose.
We started by betting on skills
Skills seemed like the right abstraction. You package your framework docs into a skill, the agent invokes it when working on Next.js tasks, and you get correct code.
There's even a growing directory of reusable skills at skills.sh.
In 56% of eval cases, the skill was never invoked. The agent had access to the documentation but didn't use it.
This isn't unique to our setup. Agents not reliably using available tools is a known limitation of current models.
We tried adding explicit instructions to AGENTS.md telling the agent to use the skill.

1
This improved the trigger rate to 95%+ and boosted the pass rate to 79%.
The hunch that paid off
What if we removed the decision entirely? Instead of hoping agents would invoke a skill, we could embed a docs index directly in AGENTS.md.
Not the full documentation, just an index that tells the agent where to find specific doc files that match your project's Next.js version.
We added a key instruction to the injected content.

1IMPORTANT: Prefer retrieval-led reasoning over pre-training-led reasoning 
2for any Next.js tasks.
This wasn't what we expected. The "dumb" approach (a static markdown file) outperformed the more sophisticated skill-based retrieval, even when we fine-tuned the skill triggers.
Addressing the context bloat concern
Embedding docs in AGENTS.md risks bloating the context window. We addressed this with compression.
The initial docs injection was around 40KB. We compressed it down to 8KB (an 80% reduction) while maintaining the 100% pass rate.
The compressed format uses a pipe-delimited structure that packs the docs index into minimal space:
[Next.js Docs Index]|root: ./.next-docs
2
|IMPORTANT: Prefer retrieval-led reasoning over pre-training-led reasoning
3
|01-app/01-getting-started:{01-installation.mdx,02-project-structure.mdx,...}
4
|01-app/02-building-your-application/01-routing:{01-defining-routes.mdx,...}