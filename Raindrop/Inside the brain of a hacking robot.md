---
raindrop_id: 1321645856
raindrop_highlights:
  68c43b48792cd23ca3783147: 8c3285b3e4bb4e869207c37c309702aa
  68c43b9e76da1dfaaf49595f: d3ce76afe943c07797688b066343a596
  68c43ba5844ec9dfaf70965d: 66d245eff004dba851eb98b1e92fb5ef
  68c43bb31959d9105a6d7ac4: ccb6087a830859201d39a98e624fd346
  68c43bf41959d9105a6d8a2e: 7f28b09cc6d7e00688aa655d499e5b0f
  68c43c05add581f8c995cdca: a313598bea386095f7c230e5decb1309
  68c43c2976da1dfaaf497902: 144c34a57c2140c8903f5e9cca872c40
  68c43d1a606ead7ea7f5f7a8: c9ef2a6a672ba112ec8c661592aa3bb3
  68c43d4b0dbbf1d411e4012d: d91c1ff21868d0e1ca0718132f187d09
  68c43d5d8cf791bcf6539cfe: 7e573570c8b3a888c0a700d4b8d63beb
  68c43d80606ead7ea7f61344: 32eedc5b9c70ce550ba7d757f8a888f9
  68c43e0dadd581f8c9964fab: dccbf626189c39c5de73d6a0e62beb4c
  68c43e27606ead7ea7f63ddd: 24de3d2ed01983d39475b1ccb1f690cc
  68c43e3382b9b2b1ba155e52: 2ea0b5db66c9ea6649ee0308bcb479d4
  68c43e380dbbf1d411e43ce5: 67aeaf5f3d2d2324c0ebc079e525cc9c
  68c43e86ed92012ad76ceb7d: dcdfe2c35813c72126e91120776731ba
  68c43e8a04fed0079c7d65bd: 0feef1efca096b246ba219dfaad935a2
  68c43ea1606ead7ea7f65c5d: b914473ee09ef18d28bfadecfa3c47f0
title: "Inside the brain of a hacking robot"

description: |-
  Agent trajectory walkthroughs from interesting examples | AI for Security, AIxCC

source: https://theori.io/blog/exploring-traces-63950

created: 1754681146000
type: article
tags:
  - "_index"

 
  - "AI" 
  - "LLM" 
  - "SAST" 
  - "tech-blog"

---
# Inside the brain of a hacking robot

![](https://source.inblog.dev/featured_image/2025-08-08T19:24:24.222Z-ea1e4de2-7f6f-4324-9adb-62c83b2c3714)

> [!summary]
> Agent trajectory walkthroughs from interesting examples | AI for Security, AIxCC





Introduction

Building LLM Agents to solve complex security tasks unfortunately is still a complicated task with a lot of human effort involved.
To gain insights to the strengths and limitations of LLMs, we can simply read through the agent log trajectories!
Keep in mind that in all of these cases, the only input our CRS had was the code repository (without commit history), and no human intervention to select, triage, or prioritize bugs.
SQLite

The first traces we will examine are from examining SQLite.
During an AIxCC practice round, we were given a version of SQLite with a harness that allowed running arbitrary SQL queries. There were some injected bugs by the organizers that we identified, but what was more interesting for us were the bugs the organizers did NOT inject.
Out-of-bounds Write

This bug is a basic heap buffer-overflow inside of the zipfile extension in SQLite, which is enabled by default.
This is one of the few traces where we include the VulnAnalyzer agent. This agent runs early on in the pipeline to take a low-confidence bug report, verify it, and then enrich it with more detailed information to help downstream processes.
Out-of-bounds Read

Another overflow was identified by our system in the same area of code, but a different bug path. This time the bug involves loading and reading a corrupted zip file. Once again, this is the type of bug that would be quite difficult to fuzz! it involves not only a crafted SQL query, but also a properly formatted and specially crafted zip file which is hex encoded.
FreeRDP

Next, let’s look at FreeRDP.
Synthetic Backdoor

During an AIxCC practice round, we were given a sample of FreeRDP with some bugs inserted. One of them is an obfuscated backdoor.
The good news is that even if this is a bit inscrutable, LLM based bug detection on the code base easily flags this as a potential bug to investigate! After reading this code, our LLM system produces a bug report like:

name: Backdoor
reason: The function contains obfuscated code that allocates an executable memory region, copies data from the network stream into it, and then executes it. This is a backdoor allowing remote code execution on the client by a malicious server.
source: if (ber_read_application_tag(s, 0x42, &length))
Unintended Integer Overflow

While it is nice to see that our system can find and trigger intended, synthetic bugs, the goal is to find real software vulnerabilities. Fortunately, this challenge provided us with a great example. In addition to the synthetic vulnerabilities, our system spotted other bugs in the FreeRDP code.
This bug is a signed integer overflow in the RDP T.124 Generic Conference Control handling when reading client monitor information.
Notably we never surfaced this vulnerability in our fuzzing using libfuzzer
. In fact, the only time we observed any code in the Generic Conference Control handling being exercised was when an LLM crafted inputs to it.
Apache Tomcat
Here we once again include all the POV producers. One fun thing here is a backdoor that we injected which is only triggered if a SHA-256 sum of some data has a certain prefix. This would be very difficult with fuzzing, but with our agents it is solvable!
Conclusion

This was just a quick summary of a few interesting pieces of some of the agent traces from our system. During development, we found reading through these logs to be incredibly useful for adjusting prompts, tools, and work boundaries between different parts of our system.