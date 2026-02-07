---
raindrop_id: 1571890814
raindrop_highlights:
  697f2cd4bccd68c6b58d2bef: ac1aa144113400fa81b43eee14cc8058
  697f2ce3e81488f12056dd1a: 556a63047e71465f9c6a428b4720ad27
  697f2d208cd529f961e25d81: 3d19975b3a73aea104d877b33578b240
  697f2d2f0eedfe6b6296678f: bfffc164c0a4a764265614de2d7adb92
  697f2d344d80a1b75881c15e: 4dbeb9646c78ed91a29ef89964063da1
  697f2d5e61569c6cd28d293d: 8db0e1e585d811b8bc8e7b17a82bf799
  697f2d6434992f7c88f93988: f35bb7ce21e9d47fc61593ff76647055
  697f2d7661569c6cd28d2cd6: 5dbb47d97e036f0c7e17f07874d81b28
  697f2d7934992f7c88f93c8b: 3640ed86cbbf4cf91666d3d1589d8c30
title: "vxcontrol/pentagi: ✨ Fully autonomous AI Agents system capable of performing complex penetration testing tasks"

description: |-
  ✨ Fully autonomous AI Agents system capable of performing complex penetration testing tasks - vxcontrol/pentagi

source: https://github.com/vxcontrol/pentagi

created: 2026-02-01
sync-date: 1770019411545
tags:
  - "_index"

 
  - "AI" 
  - "LLM" 
  - "Tools"

---
# vxcontrol/pentagi: ✨ Fully autonomous AI Agents system capable of performing complex penetration testing tasks

![](https://repository-images.githubusercontent.com/913030762/c8502908-380f-4897-aaba-87cfa16d67b4)

> [!summary]
> ✨ Fully autonomous AI Agents system capable of performing complex penetration testing tasks - vxcontrol/pentagi





Penetration testing Artificial General Intelligence
PentAGI is an innovative tool for automated security testing that leverages cutting-edge artificial intelligence technologies. The project is designed for information security professionals, researchers, and enthusiasts who need a powerful and flexible solution for conducting penetration tests.
Quick Start
Using Installer (Recommended)

PentAGI provides an interactive installer with a terminal-based UI for streamlined configuration and deployment.
# Create installation directory mkdir -p pentagi && cd pentagi # Download installer wget -O installer.zip https://pentagi.com/downloads/linux/amd64/installer-latest.zip # Extract unzip installer.zip # Run interactive installer ./installer
Manual Installation
Fill in the required API keys in .env file.
# Required: At least one of these LLM providers
OPEN_AI_KEY=your_openai_key
ANTHROPIC_API_KEY=your_anthropic_key
GEMINI_API_KEY=your_gemini_key

# Optional: AWS Bedrock provider (enterprise-grade models)
BEDROCK_REGION=us-east-1
BEDROCK_ACCESS_KEY_ID=your_aws_access_key
BEDROCK_SECRET_ACCESS_KEY=your_aws_secret_key

# Optional: Local LLM provider (zero-cost inference)
OLLAMA_SERVER_URL=http://localhost:11434
OLLAMA_SERVER_MODEL=your_model_name

# Optional: Additional search capabilities
DUCKDUCKGO_ENABLED=true
GOOGLE_API_KEY=your_google_key
GOOGLE_CX_KEY=your_google_cx
TAVILY_API_KEY=your_tavily_key
TRAVERSAAL_API_KEY=your_traversaal_key
PERPLEXITY_API_KEY=your_perplexity_key
PERPLEXITY_MODEL=sonar-pro
PERPLEXITY_CONTEXT_SIZE=medium

# Searxng meta search engine (aggregates results from multiple sources)
SEARXNG_URL=http://your-searxng-instance:8080
SEARXNG_CATEGORIES=general
SEARXNG_LANGUAGE=
SEARXNG_SAFESEARCH=0
SEARXNG_TIME_RANGE=

## Graphiti knowledge graph settings
GRAPHITI_ENABLED=true
GRAPHITI_TIMEOUT=30
GRAPHITI_URL=http://graphiti:8000
GRAPHITI_MODEL_NAME=gpt-5-mini

# Neo4j settings (used by Graphiti stack)
NEO4J_USER=neo4j
NEO4J_DATABASE=neo4j
NEO4J_PASSWORD=devpassword
NEO4J_URI=bolt://neo4j:7687

# Assistant configuration
ASSISTANT_USE_AGENTS=false         # Default value for agent usage when creating new assistants
perl -i -pe 's/\s+#.*$//' .env
curl -O https://raw.githubusercontent.com/vxcontrol/pentagi/master/docker-compose.yml docker compose up -d