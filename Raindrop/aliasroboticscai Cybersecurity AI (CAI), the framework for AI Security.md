---
raindrop_id: 1335759332
raindrop_highlights:
  68c40f2cadd581f8c98a97af: 5a05d94b8a579beb1372b6dd27e3a15a
  68c40f3c71d9ff417eff7bde: a8562a30cc85a4be16a0ffd1fef10b80
  68c41090639d3fd215d41aa7: ae6217739a16608eeae2853c9a17d848
  68c410c4f00bf652476ce8a8: 42c148153c8cf84d2d3049d98557efc3
  68c4111c844ec9dfaf65db8d: 68fd41e4760f1c8645d00a62320f6b97
  68c4117ac2f5e25ce8c4a503: 7aa96c2ab2b47638f629d7e9540ee2d1
  68c4118582b9b2b1ba0a380a: 50c07ae272f6b525020fb12874a28c43
  68c411a522695b0c1097390d: f79ef2ed46c5301770a60ed2f93f0cba
  68c411db792cd23ca36dd0ab: c6495bfe7667de7fbc9d0439d4682873
  68c412f8844ec9dfaf664e06: 189315420873c7a4b254369b9d05b675
  68c413141274f05ffc583977: 76b9e6e4333225815c4cee3264473530
  68c4131d71d9ff417e005e50: efa89b6da5ecb06f966ea66715a8f860
  68c413268cf791bcf649143b: c3d6b2171c40d96d9b50b07d0986bd48
  68c413331959d9105a6362f7: 09b48aa763e7cc4bace11cef2f25f3fd
  68c4134904fed0079c7290a0: ade1fe43176248b40088990e9479a7d5
  68c4136c71d9ff417e00714f: 187a6580d5854217c7e7ff80d842546d
  68c4137e8cf791bcf6492a8b: f2b0c477a92bd564041c6d627b37e301
  68c41397c2f5e25ce8c5246f: 7a37bdc4bafa2ba07712d6aec1900da4
  68c413b08e1b546c9bd3840d: 808615eefe66584bb554ccc1c41d14b2
  68c413bb68e3a85fa49293b4: 04866482134ffc8bdc720834ee1203b5
  68c413cd68e3a85fa4929862: 6d11d2ff6806d15ce625b2bbf3401ff2
  68c413df844ec9dfaf668afc: 980876665ec589d7670b731545e525f2
  68c413ee82b9b2b1ba0accdc: 4426cb2d3c549f5389afd2272cbb7e60
  68c413f568e3a85fa492a1cf: 9fb06ed6b8c136467931804133cea1a0
  68c4141c04fed0079c72c62d: 927a0bab6ddd28cf1a74cf50189b8ef3
  68c414248cf791bcf6495257: 996b185d53549ccf0a62c52669e80935
  68c41448aa63aac8a5a2a5e9: fc40fbbdb10b96037fb5a9441d32bce7
title: "aliasrobotics/cai: Cybersecurity AI (CAI), the framework for AI Security"

description: |-
  Cybersecurity AI (CAI), the framework for AI Security - aliasrobotics/cai

source: https://github.com/aliasrobotics/cai

created: 1757679357343
type: link
tags: ["_index"]

 
  - "AI" 
  - "LLM" 
  - "Tools"

---
# aliasrobotics/cai: Cybersecurity AI (CAI), the framework for AI Security

![](https://opengraph.githubassets.com/c71f62171c65e7f91d94fdfdba322cf6101253ba6d3fd1f5e698fd674bb0b87e/aliasrobotics/cai)

> [!summary]
> Cybersecurity AI (CAI), the framework for AI Security - aliasrobotics/cai





Cybersecurity AI (CAI)
Cybersecurity AI (CAI) is a lightweight, open-source framework that empowers security professionals to build and deploy AI-powered offensive and defensive automation.
Whether you're a security researcher, ethical hacker, IT professional, or organization looking to enhance your security posture, CAI provides the building blocks to create specialized AI agents that can assist with mitigation, vulnerability discovery, exploitation, and security assessment.
> [!info]
> Read the technical report: CAI: An Open, Bug Bounty-Ready Cybersecurity AI

> [!info]
> More case studies and PoCs are available at https://aliasrobotics.com/case-studies-robot-cybersecurity.php

🔩 Install
pip install cai-framework

Always create a new virtual environment to ensure proper dependency installation when updating CAI.
Ubuntu 24.04
sudo apt-get update && \
    sudo apt-get install -y git python3-pip python3.12-venv

# Create the virtual environment
python3.12 -m venv cai_env

# Install the package from the local directory
source cai_env/bin/activate && pip install cai-framework

# Generate a .env file and set up with defaults
echo -e 'OPENAI_API_KEY="sk-1234"\nANTHROPIC_API_KEY=""\nOLLAMA=""\nPROMPT_TOOLKIT_NO_CPR=1\nCAI_STREAM=false' > .env

# Launch CAI
cai  # first launch it can take up to 30 seconds
📐 Architecture:

CAI focuses on making cybersecurity agent coordination and execution lightweight, highly controllable, and useful for humans. To do so it builds upon 8 pillars: Agents, Tools, Handoffs, Patterns, Turns, Tracing, Guardrails and HITL.
If you want to dive deeper into the code, check the following files as a start point for using CAI:

init.py
cli.py - entrypoint for command line interface
util.py - utility functions
agents - Agent implementations
internal - CAI internal functions (endpoints, metrics, logging, etc.)
prompts - Agent Prompt Database
repl - CLI aesthetics and commands
sdk - CAI command sdk
tools - agent tools
🔹 Agent
An Agent in an intelligent system that interacts with some environment. More technically, within CAI we embrace a robotics-centric definition wherein an agent is anything that can be viewed as a system perceiving its environment through sensors, reasoning about its goals and and acting accordingly upon that environment through actuators (adapted from Russel & Norvig, AI: A Modern Approach).
🔹 Tools
Tools let cybersecurity agents take actions by providing interfaces to execute system commands, run security scans, analyze vulnerabilities, and interact with target systems and APIs - they are the core capabilities that enable CAI agents to perform security tasks effectively
🔹 Handoffs

Handoffs allow an Agent to delegate tasks to another agent, which is crucial in cybersecurity operations where specialized expertise is needed for different phases of an engagement.
🔹 Patterns

An agentic Pattern is a structured design paradigm in artificial intelligence systems where autonomous or semi-autonomous agents operate within a defined interaction framework (the pattern) to achieve a goal.
For more information and examples of common agentic patterns, see the examples folder.
🔹 Turns and Interactions

During the agentic flow (conversation), we distinguish between interactions and turns.

Interactions are sequential exchanges between one or multiple agents.
Turns: A turn represents a cycle of one ore more interactions which finishes when the Agent (or Pattern) executing returns None, judging there're no further actions to undertake.
🔹 Tracing

CAI implements AI observability by adopting the OpenTelemetry standard and to do so, it leverages Phoenix which provides comprehensive tracing capabilities through OpenTelemetry-based instrumentation, allowing you to monitor and analyze your security operations in real-time. This integration enables detailed visibility into agent interactions
🔹 Guardrails

Guardrails provide a critical security layer for CAI agents, protecting against prompt injection attacks and preventing execution of dangerous commands. These guardrails run in parallel to agents, validating both input and output to ensure safe operation.
🔹 Human-In-The-Loop (HITL)
CAI delivers a framework for building Cybersecurity AIs with a strong emphasis on semi-autonomous operation, as the reality is that fully-autonomous cybersecurity systems remain premature and face significant challenges when tackling complex tasks.
Accordingly, the Human-In-The-Loop (HITL) module is a core design principle of CAI, acknowledging that human intervention and teleoperation are essential components of responsible security testing. Through the cli.py interface, users can seamlessly interact with agents at any point during execution by simply pressing Ctrl+C.
🚀 Quickstart

To start CAI after installing it, just type cai in the CLI
MCP

CAI supports the Model Context Protocol (MCP) for integrating external tools and services with AI agents.
SSE (Server-Sent Events) - For web-based servers that push updates over HTTP connections:
CAI>/mcp load http://localhost:9876/sse burp
STDIO (Standard Input/Output) - For local inter-process communication:
CAI>/mcp load stdio myserver python mcp_server.py
Once connected, you can add the MCP tools to any agent:

CAI>/mcp add burp redteam_agent