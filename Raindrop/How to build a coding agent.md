---
raindrop_id: 1321645835
raindrop_highlights:
  693305496f0bd53a4f42448e: a860db2d94c8c0b10907fa162666ae29
  693305595c018a7aa7813871: 4e4c709c5aeda661c89d5f6618437929
  693305939fb2a2c04db076af: 95336c7d3b7278cd1b71b81efea425e4
  693305b1588df34a1cebd25e: 2bfd175b594d565f2c118e96ca7e9a3a
  693305d38ef390c9e7f8604f: b41941115f3bbd74b6edfdb8b3d3477a
  693305f9e1dbc641a5e3c797: 3150164ed6eb37b5618d29a1e2cf4751
  69330603d4e31aca347f0bb5: 46aaabc0e0b6126fb6d1e6e66edb96ce
  693306186f0bd53a4f428901: 96aaa9b75c3cc079d0f43f040bcebf92
  6933062757ba772dcf1bdf52: 41b82ccd06aa4fb116ac6b65bf539f4a
  6933063ecad37bb13d912657: 99c184cfbc5b00ed9fe7ec97ab950f10
  69330670d4e31aca347f2cfb: 04fcd139c15189e54e55255c1b250c2b
title: "How to build a coding agent"

description: |-
  Hello! If you are seeing this you are either early or currently attending my talk at DataEngBytes. Learning how to build an agent is one of the best things you can do for your personal development. Cursor, Windsurf, Claude Code and Ampcode.com are 300 lines of code running in a while true loop.
  
  
  
  
  
  
  Learn how to build your own agent

source: https://ghuntley.com/agent

created: 1756005674000
type: article
tags:
  - "_index"

 
  - "AI" 
  - "HackerNews" 
  - "LLM" 
  - "tech-blog"

---
# How to build a coding agent

![](https://ghuntley.com/content/images/size/w1200/2025/07/how-to-build-an-agent.001.jpg)

> [!summary]
> Hello! If you are seeing this you are either early or currently attending my talk at DataEngBytes. Learning how to build an agent is one of the best things you can do for your personal development. Cursor, Windsurf, Claude Code and Ampcode.com are 300 lines of code running in a while true loop.






Learn how to build your own agent





Not all LLMs are agentic.
The same way that you have different types of cars, like you've got a 40 series if you want to go off-road, and then you've also got people movers, which exist for transporting people.

The same principle applies to LLMs, and I've been able to map their behaviours into a quadrant.

A model is either high safety, low safety, an oracle, or agentic. It's never both or all.
If I were to ask you to do some security research, which model would you use?

That'd be Grok. That's a low safety model.

If you want something that's "ethics-aligned", it's Anthropic or OpenAI. So that's high safety. Similarly, you have oracles. Oracles are on the polar opposite of agentic. Oracles are suitable for summarisation tasks or require a high level of thinking.
Meanwhile, you have providers like Anthropic, and their Claude Sonnet is a digital squirrel (see below).
Sonnet is a robotic squirrel that just wants to do tool calls. It doesn't spend too much time thinking; it biases towards action, which is what makes it agentic. Sonnet focuses on incrementally obtaining success instead of pondering for minutes per turn before taking action.
It seems like every day, a new model is introduced to the market, and they're all competing with one another. But truth be told, they have their specialisations and have carved out their niches.

The problem is that, unless you're working with these models at an intimate level, you may not have this level of awareness of the specialisations of the models, which results in consumers just comparing the models on two basic primitives:

The size of the context window
The cost

It's kind of like looking at a car, whether it has two doors or three doors, whilst ignoring the fact that some vehicles are designed for off-roading, while others are designed for passenger transport.
To build an agent, the first step is to choose a highly agentic model. That is currently Claude Sonnet, or Kimi K2.
Now, you might be wondering, "What if you want a higher level of reasoning and checking of work that the incremental squirrel does?". Ah, that's simple. You can wire other LLMs in as tools into an existing agentic LLM.
We call it the Oracle. The Oracle is just GPT wired in as a tool that Claude Sonnet can function call for guidance, to check work progress, and to conduct research/planning.
The next important thing to learn is that you should only use the context window for one activity. When you're using Cursor or any one of these tools, it's essential to clear the context window after each activity
Context windows are very, very small. It's best to think of them as a Commodore 64, and as such, you should be treating it as a computer with a limited amount of memory. The more you allocate, the worse your outcome and performance will be.