---
raindrop_id: 1321645912
raindrop_highlights:
  695ea65516d39ad8c0859a02: 47646eb5d0f8a60e2682c7055e5f0e5d
  695ea6f2587e1384d5edc818: c88bdadf4c05ebcde986c92a550114e6
  695ea70e587e1384d5edcd52: e297c345c336353682aed65c9b7d55ff
title: "How to build secure and scalable remote MCP servers"

description: |-
  More context can mean more attack surfaces for your projects. Be prepared for what lies ahead with this guide.

source: https://github.blog/ai-and-ml/generative-ai/how-to-build-secure-and-scalable-remote-mcp-servers

created: 2025-07-25
sync-date: 1767878124254
tags:
  - "_index"

 
  - "AI" 
  - "MCP"

---
# How to build secure and scalable remote MCP servers

![](https://github.blog/wp-content/uploads/2025/07/wallpaper-generic-blue.png?fit=1920%2C1080)

> [!summary]
> More context can mean more attack surfaces for your projects. Be prepared for what lies ahead with this guide.





To help prevent common pitfalls, the MCP specification now includes security guidelines and best practices that address common attack vectors, like confused deputy problems, token passthrough vulnerabilities, and session hijacking.
Want to dive deeper? Check out the MCP authorization specification and recommended security best practices for complete technical details.
Understanding the MCP authorization

The MCP specification uses OAuth 2.1 for secure authorization. This allows MCP, at the protocol level, to take advantage of many modern security capabilities, including:

Authorization server discovery: MCP servers implement OAuth 2.0 Protected Resource Metadata (PRM) (RFC 9728) to advertise the authorization servers that they support. When a client attempts to access a protected MCP server, the server will respond with a HTTP 401 Unauthorized and include a WWW-Authenticate header pointing to the metadata endpoint.
Dynamic client registration: This is automatic client registration using OAuth 2.0 Dynamic Client Registration Protocol (RFC 7591). This removes the need for manual client setup when AI agents connect to MCP servers dynamically.
Resource indicators: The specification also mandates RFC 8707 Resource Indicators, ensuring that tokens are bound to specific MCP servers. This prevents token reuse attacks and helps maintain clear security boundaries.

Even with the latest changes to authorization specs, like the clean split between the responsibilities of the authorization server and the resource server, developers don’t need to worry about implementing security infrastructure from scratch. (Because the requirement to follow the OAuth2.1 conventions didn’t change.) So developers can just use off-the-shelf authorization servers and identity providers. 

Because MCP requires implementers to snap to OAuth 2.1 as the default approach to authorization, this also means that developers can use existing OAuth libraries to build the authorization capabilities into their MCP servers without anything super-custom. This is a massive time and effort saver.