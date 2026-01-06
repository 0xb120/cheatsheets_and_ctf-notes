---
raindrop_id: 1473709052
raindrop_highlights:
  6932ebd8cbea73cbcc480ab9: 18b0c24aede26c1bf429f0e22de54bc8
  6932ebfd2496dfc5cf76538d: 9f732260d337f0b9194f86ddb261d264
  6932ec2ad9344396f73c1a5f: 92c741998e453892c2022e545911052b
  6932ec5376ef15ac83471daa: 35aa9dda3b24708c9463af8063bd6937
  6932ec6bcbea73cbcc48335c: 8c8263a630c1438c1790dc41f80e70a8
  6932ec8776ef15ac83472d8a: 014f28f6ec95e491782f40129a0145ed
title: "GitHub - gadievron/raptor: Raptor turns Claude Code into a general-purpose AI offensive/defensive security agent. By using Claude.md and creating rules, sub-agents, and skills, and orchestrating security tool usage, we configure the agent for adversarial thinking, and perform research or attack/defense operations."

description: |-
  Raptor turns Claude Code into a general-purpose AI offensive/defensive security agent. By using Claude.md and creating rules, sub-agents, and skills, we configure the agent for adversarial thinking...

source: https://github.com/gadievron/raptor/

created: 1764871570258
type: link
tags:
  - "_index"

 
  - "AI" 
  - "LLM" 
  - "SAST" 
  - "Tools"

---
# GitHub - gadievron/raptor: Raptor turns Claude Code into a general-purpose AI offensive/defensive security agent. By using Claude.md and creating rules, sub-agents, and skills, and orchestrating security tool usage, we configure the agent for adversarial thinking, and perform research or attack/defense operations.

![](https://repository-images.githubusercontent.com/1078171437/0428b245-c9ef-4eaf-abb0-742725f8720a)

> [!summary]
> Raptor turns Claude Code into a general-purpose AI offensive/defensive security agent. By using Claude.md and creating rules, sub-agents, and skills, we configure the agent for adversarial thinking...





Raptor turns Claude Code into a general-purpose AI offensive/defensive security agent. By using Claude.md and creating rules, sub-agents, and skills, and orchestrating security tool usage, we configure the agent for adversarial thinking, and perform research or attack/defense operations.
What is RAPTOR?

RAPTOR is an autonomous offensive/defensive security research framework, based on Claude Code. It empowers security research with agentic workflows and automation.
RAPTOR autonomously:

Scans your code with Semgrep and CodeQL and tries dataflow validation
Fuzzes your binaries with American Fuzzy Lop (AFL)
Analyses vulnerabilities using advanced LLM reasoning
Exploits by generating proof-of-concepts
Patches with code to fix vulnerabilities
FFmpeg-specific patching for Google's recent disclosure    (https://news.ycombinator.com/item?id=45891016)
Reports everything in structured formats
Quick Start
You have two options, install on your own, or deploy the devcontainer.

**Install**
# 1. Install Claude Code
# Download from: https://claude.ai/download

# 2. Clone and open RAPTOR
git clone https://github.com/gadievron/raptor.git
cd raptor
claude

# 3. Let Claude install dependencies, and check licenses for the various tools
"Install dependencies from requirements.txt"
"Install semgrep"
"Set my ANTHROPIC_API_KEY to [your-key]"

**devcontainer**
# 4. Get the devcontainer
A devcontainer with all prerequisites pre-installed is available. Open in VS Code or any of
its forks with command Dev Container: Open Folder in Container, or build with docker:
docker build -f .devcontainer/Dockerfile -t raptor-devcontainer:latest ..

Runs with --privileged flag for rr.

# 5. Notes
The devcontainer is massive (~6GB), starting with Microsoft Python 3.12 massive devcontainer and
adding static analysis, fuzzing and browser automation tools.

# 6. Getting started with RAPTOR
Just say "hi" to get started
Try /analyze on one of our tests in /tests/data
Available Commands

Main entry point:

/raptor   - RAPTOR security testing assistant (start here for guidance)


Security testing:

/scan     - Static code analysis (Semgrep + CodeQL)
/fuzz     - Binary fuzzing with AFL++
/web      - Web application security testing
/agentic  - Full autonomous workflow (analysis + exploit/patch generation)
/codeql   - CodeQL-only deep analysis with dataflow
/analyze  - LLM analysis only (no exploit/patch generation - 50% faster & cheaper)


Exploit development & patching:

/exploit  - Generate exploit proof-of-concepts (beta)
/patch    - Generate security patches for vulnerabilities (beta)
/crash-analysis - Analyze an FFmpeg crash and generate a validated root-cause analysis


Development & testing:

/create-skill    - Save custom approaches (experimental)
/test-workflows  - Run comprehensive test suite (stub)


Expert personas: (9 total, load on-demand)

Mark Dowd, Charlie Miller/Halvar Flake, Security Researcher, Patch Engineer,
Penetration Tester, Fuzzing Strategist, Binary Exploitation Specialist,
CodeQL Dataflow Analyst, CodeQL Finding Analyst

Usage: "Use [persona name]"


See: docs/CLAUDE_CODE_USAGE.md for detailed examples and workflows
Python CLI (Alternative)

For scripting or CI/CD integration:

python3 raptor.py agentic --repo /path/to/code
python3 raptor.py scan --repo /path/to/code --policy_groups secrets
python3 raptor.py fuzz --binary /path/to/binary --duration 3600

See: docs/PYTHON_CLI.md for complete Python CLI reference