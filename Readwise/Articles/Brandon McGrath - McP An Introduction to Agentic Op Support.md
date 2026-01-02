---
author: Brandon McGrath
aliases:
  - "McP: An Introduction to Agentic Op Support"
tags:
  - RW_inbox
  - readwise/articles
url: https://trustedsec.com/blog/mcp-an-introduction-to-agentic-op-support?__readwiseLocation=
date: 2025-04-24
---
# McP: An Introduction to Agentic Op Support

![rw-book-cover](https://trusted-sec.transforms.svdcdn.com/production/images/Blog-Covers/MCPIntroAgenticOPSupport_SEO.jpg?w=1200&h=630&q=82&auto=format&fit=crop&dm=1743097540&s=9888df0649ce8a311d50e97abd63d380)

## Highlights


Agents and Large Language Models (LLMs) offer a powerful combination for driving automation. In this post, we’ll explore how to implement a straightforward agent that leverages the capabilities of LLMs alongside common tools to autonomously achieve a goal, showcasing the synergy between these technologies. [](https://read.readwise.io/read/01js1dcg99mnyxt3p3s2w0a8tj)



1.2.1     What is an Agent?
 An agent is an AI-driven system that interacts with its environment to achieve a specific goal. It perceives, reasons, and acts autonomously, often using external tools to complete tasks step by step. [](https://read.readwise.io/read/01js1dd3e27vk790efpnh0p117)



![](https://trusted-sec.transforms.svdcdn.com/production/images/Blog-assets/AgentMCP_McGrath/Fig1_McGrath.png?w=319&h=105&auto=compress%2Cformat&fit=crop&dm=1743096397&s=e8c0d750cf6ba87676ae7a1e95542daa)
 Figure 1 - Simple Reflex Agent [](https://read.readwise.io/read/01js1ddtkcbn45qvtmezc3em21)



An agent is like a problem-solving model [](https://read.readwise.io/read/01js1dezx3b7s3dxazec93yw1j)



that continuously observes, selects the best tool from its "toolbox," and executes actions in a loop to complete tasks. [](https://read.readwise.io/read/01js1df569r5ztnkz9g5ckz6et)



At its core, an agent consists of two (2) components, a “brain” and a “body”. [](https://read.readwise.io/read/01js1dfh8agm8p7t5gwbzc0eag)



The brain (AI model) is responsible for reasoning, planning, and decision-making. So, models with strong reasoning such as [3.7 Sonnet](https://www.anthropic.com/news/claude-3-7-sonnet) and [Grok 3](https://x.ai/grok) are preferable. [](https://read.readwise.io/read/01js1dg0jdd9k9aww27ewbzsd2)



The body (tools and capabilities)is the set of actions and tools the agent can use to interact with its environment. Depending on the purpose, e.g., code, 3.7 Sonnet is currently doing well. [](https://read.readwise.io/read/01js1dh6yr1tgzh4r7nzqyspfa)



By combining these elements, agents operate with autonomy and adaptability, making them valuable for tasks ranging from automation and security to research and decision support. [](https://read.readwise.io/read/01js1dhpbe5s0x5qapm2c2t0ef)



1.2.3     What is an Agent Tool?
 An agent tool is a software utility that an agent uses to perform specific tasks, such as gathering intelligence, analyzing data, or executing security operations. These tools help agents interact with their environment and automate complex workflows. Tools can be anything from functions wrapping requests and API calls, access to OS-level tools with subprocess, or even a simple Python function that adds two (2) numbers together. [](https://read.readwise.io/read/01js1dkccnhvh8ztc5pta0rgmk)



1.3      Agentic Frameworks
 Several modern frameworks support the development of multi-agent systems, and they each have their own pros and cons. [](https://read.readwise.io/read/01js1dkt2f9n4ca0x6nzzmbs30)



for this blog, we will be looking at [Model Context Protocol (MCP)](https://github.com/modelcontextprotocol/python-sdk). Conveniently, at the time of writing this, [Adam Chester](https://x.com/_xpn_) provided a proof of concept using the exact setup I plan to talk about in this post. [](https://read.readwise.io/read/01js1dn6f2mcq0q1dt2gynh7fn)



See here:
 • [https://x.com/_xpn_/status/1902848551463399504?t=KQ_Riy6RJ-d7ZphSl9UFWQ](https://x.com/_xpn_/status/1902848551463399504?t=KQ_Riy6RJ-d7ZphSl9UFWQ)
 • [https://github.com/xpn/mythic_mcp](https://github.com/xpn/mythic_mcp)
 • [https://www.youtube.com/watch?v=ZooTlwajQT4](https://www.youtube.com/watch?v=ZooTlwajQT4) [](https://read.readwise.io/read/01js1dnx8s9gfqpw1n6d1zp07m)



With that said, we are going to look at building a simple agent that uses various tools such as ***ldapsearch***, ***smbclient***, ***nslookup***, and ***ping*** to discover domain controllers on a network. [](https://read.readwise.io/read/01js1dpat1ypa94nerc56zxvsc)



Another project worth investigating is [nerve](https://github.com/evilsocket/nerve) by [evilsocket](https://www.evilsocket.net/). [](https://read.readwise.io/read/01js1dpj3wz0jvm097bqp8mf2b)



1.4      The Agent: Building
 The ***mcp*** library makes this incredibly easy to define. However, it can get a little complicated architecturally, so it is recommended to review the [documentation](https://modelcontextprotocol.io/introduction). MCP follows a client-server model where a single host can connect to multiple servers. [](https://read.readwise.io/read/01js1dqj8xdt5tcnjn57j5hmyn)



To begin, [FastMCP](https://github.com/modelcontextprotocol/python-sdk/tree/main/src/mcp/server/fastmcp) is defined, giving us access to the decorators, which allow us to create the server's structure. [](https://read.readwise.io/read/01js1dr8janzrd20m10xkqjjxz)



We have two (2) main decorators to consider:
 1. ***mcp.prompt()***
 2. ***mcp.tool()*** [](https://read.readwise.io/read/01js1drq3mnxnrxfd9579vj2s1)



Prompts enable servers to define reusable prompt templates and workflows that clients can easily surface to users and LLMs. They provide a powerful way to standardize and share common LLM interactions. [](https://read.readwise.io/read/01js1ds92p2gy2jm477mdqdh96)



Tools are a powerful primitive in the MCP that enable servers to expose executable functionality to clients. Through tools, LLMs can interact with external systems, perform computations, and take actions in the real world. [](https://read.readwise.io/read/01js1dsj7yrzcavsnyefp7qdje)



here is one (1) of the prompts:
 @mcp.prompt()
 def setup_prompt(subnet: str, objective: str) -> str:
 """
 setup network mapping by subnet and objective
 :param subnet: subnet to map
 :type subnet: str
 :param objective: the objective of the mapping
 :type objective: str
 :return:
 :rtype: str
 """
 return f"""
 You are a senior penetration tester specialising in internal networks. Your primary objective is to map the network by scanning the subnet {subnet} and identify key infrastructure for Active Directory assets.
 You should think, carefully, step by step, in order to complete the objective with reasoning divided in Thought, Action, and Observation. 
 Firstly, reflect on the objective, then if required, execute any tools you have access to on the target subnet {subnet} and report your findings.
 Your objective is to: {objective}
 """ [](https://read.readwise.io/read/01js1dt9ws62pe98nnhvbvdvnr)



Tools are just as easy:
 @mcp.tool()
 async def run_ldap_dns_query(host: str) -> str:
 """
 perform LDAP DNS query to identify domain controller.
 :param host: host to query
 :type host: str
 :return: LDAP DNS query results
 """
 command = f"dig -t SRV _ldap._tcp.dc._msdcs.{host}"
 return execute_os_command(command) [](https://read.readwise.io/read/01js1dv2t3day3tmf4nftyns6e)



1.5      The Agent: Execution
 Using the Claude Desktop, MCP servers can be easily configured. To debug an MCP server, [modelcontextprotocol/inspector](https://github.com/modelcontextprotocol/inspector) can be used to provide access to components of the server. Running the following command lets us access the server at [localhost:5173](http://localhost:5173):
 npx @modelcontextprotocol/inspector uv run main.py [](https://read.readwise.io/read/01js1dwj8qrcq6js2vdr3xekmj)



From within the debug UI, we have access to prompts, tools, etc.
 ![](https://trusted-sec.transforms.svdcdn.com/production/images/Blog-assets/AgentMCP_McGrath/Fig5_McGrath.png?w=320&h=154&auto=compress%2Cformat&fit=crop&dm=1743096401&s=07f953b5e3fa8d935fd1306dd6c78650)
 Figure 5 - MCP Inspector: Tools [](https://read.readwise.io/read/01js1dwx6vkawqeacyw5pneceg)



That’s a simple implementation of a singular agent. This can be expanded to do all sorts of things such as password spraying, Kerberoasting, anything. [](https://read.readwise.io/read/01js1dz6sb19japxy6efs7sxq5)

