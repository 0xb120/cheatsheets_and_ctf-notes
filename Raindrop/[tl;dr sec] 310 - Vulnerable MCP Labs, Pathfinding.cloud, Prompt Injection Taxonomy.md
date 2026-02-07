---
raindrop_id: 1531958521
raindrop_highlights:
  69625f97d13e7a2a5f359e48: a817469be0972109170a390d49f26964
title: "[tl;dr sec] #310 - Vulnerable MCP Labs, Pathfinding.cloud, Prompt Injection Taxonomy"

description: |-
  null

source: https://tldrsec.com/p/tldr-sec-310

created: 2026-01-08
sync-date: 1769114384113
tags:
  - "_index"

 
  - "tech-newsletters" 
  - "MCP" 
  - "labs" 
  - "AI"

---
# [tl;dr sec] #310 - Vulnerable MCP Labs, Pathfinding.cloud, Prompt Injection Taxonomy

![](https://beehiiv-images-production.s3.amazonaws.com/uploads/publication/thumbnail/080a561f-2435-4477-a549-ab9f115e047c/landscape_Screenshot_2024-11-21_at_10.48.21_AM.png)

> [!summary]
> null





appsecco/vulnerable-mcp-servers-lab By Appsecco’s Riyaz Walikar: A collection of 9 intentionally vulnerable MCP servers designed to help you learn how to penetration test AI agent infrastructure. The labs include servers containing path traversal with code execution via unsafe path joining and unsandboxed Python execution, indirect prompt injection through documents with embedded hidden instructions (both local stdio and remote HTTP+SSE variants), eval-based RCE in a "quote of the day" tool, instruction injection via fabricated tool outputs, supply-chain attacks through typosquatting, secrets/PII exposure in utility tools, and more.