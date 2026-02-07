---
raindrop_id: 1321645898
raindrop_highlights:
  696d0f84dddaf158dc71b8d6: 34da32b8664b7ff497b5eb46c6ec0eb5
  696d0f8e508a8738f42d1bd6: 31492a6708bf7b22e49bd972cb522de7
  696d0f9bdddaf158dc71bcce: b86409b2a42afa0040dc6842d9a82f70
title: "GitHub - atiilla/whiterabbit-mcp: A lightweight, extensible cybersecurity toolkit that connects AI assistants to security tools through the Model Context Protocol (MCP), enabling AI-assisted security research, scanning, and analysis. [app]"

description: |-
  A lightweight, extensible cybersecurity toolkit that connects AI assistants to security tools through the Model Context Protocol (MCP), enabling AI-assisted security research, scanning, and analysi...

source: https://github.com/atiilla/whiterabbit-mcp

created: 2025-07-28
sync-date: 1769114384380
tags:
  - "_index"

 
  - "AI" 
  - "MCP" 
  - "LLM" 
  - "Tools"

---
# GitHub - atiilla/whiterabbit-mcp: A lightweight, extensible cybersecurity toolkit that connects AI assistants to security tools through the Model Context Protocol (MCP), enabling AI-assisted security research, scanning, and analysis. [app]

![](https://opengraph.githubassets.com/4e8f7c233fc00acfef61922a339766728dab9fcd51f18c7c53f177e6b760deed/atiilla/whiterabbit-mcp)

> [!summary]
> A lightweight, extensible cybersecurity toolkit that connects AI assistants to security tools through the Model Context Protocol (MCP), enabling AI-assisted security research, scanning, and analysi...





A lightweight, extensible cybersecurity toolkit that connects AI assistants to security tools through the Model Context Protocol (MCP), enabling AI-assisted security research, scanning, and analysis.
Installation

Build te Docker image

git clone https://github.com/atiilla/whiterabbit-mcp.git
cd whiterabbit-mcp
docker build -t whiterabbitmcp .
Usage

Edit your claude_desktop_config.json

{
  "mcpServers": {
    "WhiteRabbitMCP": {
      "command": "docker",
      "args": ["run", "--rm", "-i","--name","whiterabbitmcp", "whiterabbitmcp"]
    }
  }
}
Or Copilot in vscode

mkdir -p .vscode
cd .vscode
touch mcp.json
```json
{
    "servers": {
        "WhiteRabbitMCP": {
            "command": "docker",
            "args": [
                "run",
                "--rm",
                "-i",
                "--net=host",
                "--privileged",
                "--name",
                "whiterabbitmcp",
                "whiterabbitmcp"
            ]
        }
    }
}