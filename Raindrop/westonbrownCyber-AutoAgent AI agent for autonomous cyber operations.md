---
raindrop_id: 1403357568
raindrop_highlights:
  6933020f9fae5c8ff0e6c4ce: 9e98ec179256abf30c10faa75ef37dd5
  693302ba6f0bd53a4f415b47: 3c18de523efccf8cb07501c92065fd21
  693302ccf4b83213ea02afe0: ee724c7ad2125069e1739d92975bfa67
  693302e3aa8c4e596d85d24a: 670d4858d639c45ff7457b7e7c3f23af
  69330308aa8c4e596d85e26a: 56b5d5aa5abceeb40816d9327e904f42
  69330314ab8d3e2db3988fac: 0b18ab5397b9863e8c546ef4b5799ae4
title: "westonbrown/Cyber-AutoAgent: AI agent for autonomous cyber operations"

description: |-
  AI agent for autonomous cyber operations

source: https://github.com/westonbrown/Cyber-AutoAgent

created: 1760986984251
type: link
tags: ["_index"]

 
  - "AI" 
  - "LLM" 
  - "SAST" 
  - "DAST"

---
# westonbrown/Cyber-AutoAgent: AI agent for autonomous cyber operations

![](https://github.com/westonbrown/Cyber-AutoAgent/raw/main/docs/cover_art.png)

> [!summary]
> AI agent for autonomous cyber operations





AI agent for autonomous cyber operations
Cyber-AutoAgent started as an experimental side project to explore autonomous offensive security agents and black box pentesting. After achieving 85% on the XBOW valdiation benchmark and building an engaged community, it became clear this work requires dedicated full-time focus to reach production-grade maturity.
Quick Start

The React-based terminal interface is now the default UI, providing interactive configuration, real-time operation monitoring, and guided setup in all deployment modes.

Local Development - Recommended
# Clone and setup
git clone https://github.com/westonbrown/Cyber-AutoAgent.git
cd Cyber-AutoAgent

# Build React terminal interface
cd src/modules/interfaces/react
npm install
npm run build

# Run interactive terminal (guided setup on first launch)
npm start

# Or run directly with parameters
node dist/index.js \
  --target "http://testphp.vulnweb.com" \
  --objective "Security assessment" \
  --auto-run
Complete User Guide - Detailed setup, configuration, operation modules, troubleshooting, and examples
Docker Deployment
Single Container
# Interactive mode with React terminal
docker run -it --rm \
  -e AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID} \
  -e AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY} \
  -e AWS_REGION=${AWS_REGION:-us-east-1} \
  -v $(pwd)/outputs:/app/outputs \
  cyber-autoagent

# Or start directly with parameters
docker run -it --rm \
  -e AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID} \
  -e AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY} \
  -e AWS_REGION=${AWS_REGION:-us-east-1} \
  -v $(pwd)/outputs:/app/outputs \
  cyber-autoagent \
  --target "http://testphp.vulnweb.com" \
  --objective "Identify SQL injection vulnerabilities" \
  --auto-run
Docker Compose (Full Stack with Observability)

Setup: Create .env file in project root with your configuration:

# Copy example and configure
cp .env.example .env
# Edit .env with your provider settings

Example .env for LiteLLM:

CYBER_AGENT_PROVIDER=litellm
CYBER_AGENT_LLM_MODEL=gemini/gemini-2.5-flash
GEMINI_API_KEY=your_api_key_here

Run assessments with the full observability stack:

# Run with React terminal UI and full observability
docker compose -f docker/docker-compose.yml run --rm cyber-autoagent

# With root access for dynamic tool installation
docker compose -f docker/docker-compose.yml run --user root --rm cyber-autoagent

Note: The .env file in the project root is automatically loaded by docker-compose.

The compose stack automatically provides:

Langfuse observability at http://localhost:3000 (login: admin@cyber-autoagent.com / changeme)
Persistent databases (PostgreSQL, ClickHouse, Redis, MinIO)
Network access to challenge containers
React terminal UI as the default interface
Documentation
User Guide - Complete usage, configuration, and examples
Agent Architecture - Strands framework, tools, and metacognitive design