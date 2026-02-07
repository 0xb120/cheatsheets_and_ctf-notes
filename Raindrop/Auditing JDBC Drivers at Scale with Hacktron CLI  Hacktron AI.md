---
raindrop_id: 1517687327
raindrop_highlights:
  696fb746091b98e47448614f: c9d45fb9d30ad264107e0d88d28b1bd8
  696fb75a8f8b26fc1985cb6e: 1a01310e4367b9672f04f4ea8feacde6
  696fb764e1845b97368b1f25: 6d56baeeca61db5f743264b6aec7269b
  696fb76f3d3022fcf395c14b: 2c2a8ba6211e014cc4fffa570d5320f6
  696fb796885ec3fbb411d3d1: 539f89b9f5641e4c93c012a3763abbb8
  696fb79de1845b97368b2e89: 38a3b22ec90476028c74615f8b7eb6f0
  696fb7a74dd9c5ea99c8a918: a03c438399edd46e388cee32746dc418
  696fb7c5884143c84f94c2d5: 59a98168d1af4bf1afa4c0da4c856cb2
  696fb7d64dd9c5ea99c8b54f: 10e83463547fc019e0fa1e2b1e2f7819
  696fb7e8e8539ba3656aa858: 5f2d25daafc880ae735ac441709112bf
  696fb7fed30c4d7ee5572164: 484127b53d216a589abf1c007f6fadd0
  696fb819884143c84f94d916: d28d66faaa5c2c93ebcdcc8f4fe78e6f
  696fb828d30c4d7ee5572c86: 877a9e065f72d43baeeb3303a6377c27
  696fb884d833f7448bdc3bc0: ae677adb920ff45f87bbe9286ea9d52a
  696fb8cdfd52e1356c726577: a1e94b7180fe32b8b078fe9fbe3f8d61
  696fb958885ec3fbb4124e45: 8dedaf8cee0e3942c37486298472bdb2
  696fb96cdf1bb9a65a781593: 38941efd667a84d11c3cbf3bc54c5d6c
  696fb98594442b15d1943af7: 3c5c275edb6043863ea2dca954abd3df
  696fb99794442b15d1943f51: 95d7f31dc557367a794ffe8f7e2caf6b
title: "Auditing JDBC Drivers at Scale with Hacktron CLI | Hacktron AI"

description: |-
  How we used Hacktron CLI to audit JDBC drivers at scale, mapping dangerous sinks to user input and turning file primitives into real-world RCEs and bug bounties.

source: https://www.hacktron.ai/blog/jdbc-audit-at-scale

created: 2025-12-30
sync-date: 1769114383308
tags:
  - "_index"

 
  - "tech-blog" 
  - "AI" 
  - "LLM"

---
# Auditing JDBC Drivers at Scale with Hacktron CLI | Hacktron AI

![](https://www.hacktron.ai/_astro/jdbc_thumbnail.DXYe466s_CLb3K.webp)

> [!summary]
> How we used Hacktron CLI to audit JDBC drivers at scale, mapping dangerous sinks to user input and turning file primitives into real-world RCEs and bug bounties.





Personally, I love reading code and hunting for vulnerabilities, but over the past couple of months I’ve been focused on understanding how LLM-assisted auditing will shape the future of pentesting, source code reviews.
A few months back, a friend of mine,
was participating in a bug bounty event, he mentioned that his target scope relied heavily on JDBC drivers and asked if I wanted to collaborate
this was the perfect chance to treat Hacktron CLI as my co-pilot
So I got the list of all the JDBC drivers in use, and decompiled their sources.
(only 2 days remaining).
We built Hacktron CLI to accelerate vulnerability research or code assisted pentests at scale.
I put together a “JDBC driver pack” for our CLI. A pack is basically a curated list of agents that run on a given repo.
This pack was tailored specifically for JDBC drivers and included agents for each of the vulnerability classes we cared about.
hacktron agent pack JDBC_driver
Hacktron begins by enumerating all file-related sinks in the driver, mapping which ones depended on dynamic inputs.
Once Hacktron finished gathering interesting sinks. It started going though each of these sinks to check if the input can be traced back to user-controllable input.
this was done in mere 15 minutes.
In the Databricks JDBC driver, one pattern stood out the presence of a connection-string property called StagingAllowedLocalPaths, intended to restrict local file staging operations to approved directories. It reasoned that this beats the purpose entirely because the allowlist itself is supplied by the user in the connection string. As soon as the driver trusts this list, the feature becomes an entry point for arbitrary file reads and writes on the client system.
At this point, I had no real familiarity with Databricks or how its JDBC driver worked, and the “PUT query” logic felt like gibberish. So I simply asked Hacktron how a proof-of-concept would look if someone wanted to exploit this behavior. That’s when it pointed me to Databricks’ Volume storage feature (https://docs.databricks.com/aws/en/sql/language-manual/sql-ref-volumes).
Similarly, an arb file read was found in the Exasol driver. However, this has a limitation if the file content has certain characters it causes an exception.
A few more other drivers were found to be vulnerable to full-response SSRF and RCEs. For example the Teradata Driver was found to be vulnerable to command injection.
Result

Across different vendor drivers, Hacktron surfaced multiple vulnerabilities which netted total of $85,000 in bug bounties.
Looking back, this whole journey felt like a glimpse into how vulnerability research is evolving. I still love cracking open code by hand, but having Hacktron alongside me changed the pace entirely.