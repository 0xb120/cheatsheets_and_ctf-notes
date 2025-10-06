---
raindrop_id: 1328430683
raindrop_highlights:
  68c29cd00ad8333dcbe40e7a: 454d2f7cc0d9e3fcc06a7cd762004bab
  68c29ce34ea391c352ce71a8: dfdbb805ac59d32d38683399833eb0e3
  68c29cee97d4cd30f4d6d847: 5000bf24d57130988955730f1df3c192
  68c29cf4603c309f60e7cc04: 7380e13be27181a7e5c068c92aeb0dfa
  68c29d029c6f15ca4ebe019a: bea59dd89e3a521d58d039be62460180
  68c29d0ca1ea3e96183188d8: 17f5431f3e27e9fc62eb549c234e7e6d
  68c29d129bd200cef7255219: 2e313dd38b5f75330913761457f19da8
  68c29d1db7859f7c6f680ee3: 39e7873ecf6d7bb70c8dd997a4088518
  68c29d276de7bebc57f6fc4e: 273b5900d8b4999bf987053d949c6af8
  68c29d2d6552a70713d69459: 01956d736152cd7db774c1f2640e63cc
  68c29d334297bb0468041e21: af2a21be4dc688fd557b3b15a484e151
  68c29d3597d4cd30f4d6e96b: 60eba967a38e26bdb73a29b236302b9d
  68c29d3ab7859f7c6f6814ab: 2bfbfbec2c73cd7f995128bd94a036e6
  68c29d42fb16e9c4dc1307f9: 62a2ccae7b1aaa1745aaa1c5d353cb9d
  68c29d59b7859f7c6f681abd: ba8636f42bd2f17323bd01690f4eac05
  68c29d6a9c6f15ca4ebe172c: adf5db69ed7c92de17b0ce7e63e38156
  68c29d6f6552a70713d6a197: 2b8a7a71215285a09be4cd2646f5cbf6
  68c29d7a0ad8333dcbe4342a: 99f44bd4e76af562403075354a59496c
  68c29d83a1ddd74ed7a3cfef: 38ccc1095e5789aa3850c5e04d8f5899
  68c29da968835c526496f61d: abc3c481ac2f7afd8e68fa569068ec82
  68c29daf6552a70713d6afdd: f79ca46b109b73f43f57f4a11112116b
  68c29db968835c526496f9d6: 8879cdc9df388ad932f7ab561a7d49f9
  68c29dc04ea391c352cea251: 5c9262259406b6fe2da5fafb0a8c9f5e
  68c29dc74ea391c352cea43f: 3bb5fcf4a29ed37442a55c3f608796fa
  68c29dcbfb16e9c4dc132748: 58fcf7e1746530d635fa34b431040d5a
  68c29dd5a1c5b828edfdc7b4: a6088fabd8bc19de5bc252563271c777
  68c29de87f46d1fd8d3fcd42: 4d32600cec95a2a647b4bc083da1c382
title: "Hunting for Security Bugs in Code with AI Agents: A Full Walkthrough | by Bernhard Mueller | Sep, 2025 | Medium"

description: |-
  Hunting for Security Bugs in Code with AI Agents: A Full Walkthrough In my previous article, I introduced Hound, an open-source code auditing tool that models the cognitive and organizational …

source: https://muellerberndt.medium.com/hunting-for-security-bugs-in-code-with-ai-agents-a-full-walkthrough-a0dc24e1adf0

created: Mon Sep 08 2025 08:58:41 GMT+0200
type: link
tags:
  - "_index"

 
  - "AI" 
  - "LLM" 
  - "Tools" 
  - "vuln-research-blog" 
  - "SAST"

---
# Hunting for Security Bugs in Code with AI Agents: A Full Walkthrough | by Bernhard Mueller | Sep, 2025 | Medium

![](https://miro.medium.com/1*cIFjQ4ZugF3_Lfgu9PVr9A.png)

> [!summary]
>Hunting for Security Bugs in Code with AI Agents: A Full Walkthrough In my previous article, I introduced Hound, an open-source code auditing tool that models the cognitive and organizational …





In this follow-up, we’ll walk through a complete end-to-end audit using Hound in practice.

Hound is a code security analysis agent that uses autonomously generated, adaptive knowledge graphs to model the components and relationships of the target system. These graphs (like mental models) let LLMs reason about different aspects at multiple levels of abstraction while “zooming in” on the precise code snippets needed at any point during the reasoning process.

Installation and Configuration

To follow along, you’ll need API keys for at least one LLM API.

Install Hound: If you haven’t already, grab Hound’s code. You can clone the repo or install via pip:
&gt;
&gt;git clone https://github.com/muellerberndt/hound.git
&gt;cd hound
&gt;pip install -r requirements.txt
&gt;
&gt;API Keys: Export your keys as environment variables (never hard‑code them or store them in files committed to source control):
&gt;
&gt;export OPENAI_API_KEY=&quot;sk-…your OpenAI key…&quot; 
&gt;export GOOGLE_API_KEY=&quot;…your Google (Gemini) key…&quot;
&gt;
&gt;Copy the example config and open it for editing:
&gt;
&gt;cd hound
&gt;cp config.yaml.example config.yaml 
&gt;# Now open hound/config.yaml in your editor

There are five main roles:
&gt;
&gt;Graph builder — builds the knowledge graphs from code. Needs the largest possible context window to ingest and structure the code. Gemini (~1M input tokens) is ideal here. GPT‑4.1 also produces good results but is very expensive.
&gt;Scout (junior agent) — explores code and annotates graphs. The scout relies on the strategist for advice. Can run on a smaller, cheaper model. GPT-5-nano or GPT-5-mini works well here.
&gt;Strategist (senior agent) — plans the audit, generates vulnerability hypotheses and guides the scout. Requires the deepest reasoning ability, so use your strongest model (GPT‑5, Opus 4.1, or whatever else ie best at the moment).
&gt;Finalizer (QA) — reviews and verifies hypotheses. Required a strong reasoning model (GPT‑5 or Opus 4.1).
&gt;Reporter — (optional) generates the final report. Can use a mid‑tier model for summarization and report generation. I normally use GPT-4o.

Step 1: Building Graph Models

In this phase, Hound builds relational models of the target scope. They are called “aspect graphs”.
&gt;
&gt;Let’s start with the limitations: Hound does not perform well on very large codebases.

For large systems, it’s best to isolate subsystems and audit them independently.

Use a high-context model like Gemini for this step, since it must ingest and structure a significant volume of code.

Personally, I use a whitelist builder to collect the most relevant files within a given LOC budget.

To begin, download the Rustic server repo and generate a whitelist (likely all Rust files):
&gt;
&gt;./whitelist_builder.py --input sources/rustic_server &#92;
&gt;  --output whitelists/rustic_server.txt &#92;
&gt;  --print-summary --verbose --limit-loc 50000 --enable-ll

Now build the graphs in auto mode. By default, this will model 5 aspects, which should take a few minutes.
&gt;
&gt;./hound.py project create rustic_server
&gt;./hound.py graph build rustic_server --auto &#92;
&gt;  --files &quot;$(tr -d &#39;&#92;n&#39; &lt; ../whitelists/rustic_server.txt)&quot;

After the command returns, you can list the generated graphs and export them to a HTML file:
&gt;
&gt;./hound.py graph ls rustic_server
&gt;
&gt;  Name                           Nodes   Edges   Updated               Focus                                                                             
&gt; ─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── 
&gt;  RepositoryDataFlow                70     136   2025-09-06T08:56:51   The lifecycle of data within a repository (e.g., `data`, `keys`, `index`, `snaps  
&gt;  CLICommandWorkflow                32      56   2025-09-06T08:46:34   The operational logic of the `auth` subcommand, which provides an out-of-band ad  
&gt;  AuthorizationMap                  41      83   2025-09-06T08:18:05   The fine-grained access control logic centered around the `acl.rs` module. This   
&gt;  SystemArchitecture                46      91   2025-09-06T08:10:04   A high-level overview of the `rustic-server` components, their interactions, and  
&gt;  ConfigurationInfluence            44      83   2025-09-06T08:02:28   How configuration settings, from `rustic_server.toml` and command-line overrides  
&gt;
&gt;./hound.py graph export rustic_server --open

For a REST server like Rustic, a taint-style data flow graph would be nice to have. We create it with:
&gt;
&gt;./hound.py graph build rustic_server &#92;
&gt;  --graph-spec &quot;Data flows and validation from taint sources to sinks&quot; &#92;
&gt;  --iterations 4
&gt;Press enter or click to view image in full size
&gt;Data flows and validation from taint sources to sinks, already annotated by the analyzer. Modeling how user inputs flow into file system calls is useful for reasoning about input validation flaws.

Step 2: Running the Audit
&gt;
&gt;The default way to run an audit is simply:
&gt;
&gt;./hound.py agent audit rustic_server

You can also bound the runtime:
&gt;
&gt;./hound.py agent audit rustic_server --time-limit 120

Chatbot and Web UI
&gt;
&gt;To monitor audits interactively, use the --telemetry flag together with the web chatbot UI:
&gt;
&gt;./hound.py agent audit rustic_server --telemetry --session
&gt;python chatbot/run.py

Steering the Audit
&gt;
&gt;You can also steer investigations with missions or targeted investigations:
&gt;
&gt;./hound.py agent audit rustic_server --mission &quot;Focus on input validation flaws&quot;
&gt;
&gt;./hound.py agent investigate rustic_server &quot;Focus on input validation flaws&quot;

Step 3: Eliminating False Positives

As the agent works, it forms hypotheses (or beliefs) and continuously adjusts their confidence as new evidence appears.

How this works: During an audit, the Strategist creates initial beliefs with an assigned confidence level. As the Scoutexplores the codebase and gathers evidence, those confidence levels rise or fall. Over time, this process results in a collection of hypotheses, each associated with a different degree of confidence.

Once you decide the audit has progressed far enough, it is time to finalize.

The finalization step does the following:
&gt;
&gt;Adjusts confidence values based on the deeper reasoning pass
&gt;Updates the status of each hypothesis (confirmed / rejected / uncertain)
&gt;Records a concise justification that cites the relevant code lines considered

To run the finalization process via the CLI:
&gt;
&gt;./hound.py finalize rustic_server

the agent will investigate each issue one by one. For example:
&gt;
&gt;🧠 Reviewer gpt-5 enters the chamber...
&gt;  Loaded 1 file(s) from source_files
&gt;  Added 10 file(s) for next iteration
&gt;  ✓ CONFIRMED: Read-only users can create/remove locks due to access downgrade
&gt;    Justification: The code explicitly downgrades all access checks for the Locks type to Read, which allows users with only read permissions to perform write operations on locks. In src/acl.rs, Acl::is_allowed sets 
&gt;    access_type = AccessType::Read whenever tpe == TpeKind::Locks, regardless of the requested access (Append/Modify). Handlers for creating and deleting files (src/handlers/file_exchange.rs add_file/delete_file) require 
&gt;    AccessType::Append via check_auth_and_acl, but since they pass the Locks type through to is_allowed, the downgrade causes these checks to succeed for read-only users. The web router (src/web.rs) exposes POST/DELETE 
&gt;    /:repo/:tpe/:name, so POST/DELETE /:repo/locks/:name is reachable. There are no compensating guards preventing writes to locks; even the ACL tests (src/acl.rs tests) assert that Modify on Locks is allowed for users with 
&gt;    lesser privileges, confirming the behavior. This permits read-only users to create or remove lock files, enabling denial-of-service (e.g., persistent locks) or unauthorized unlocking.
&gt;
&gt;After finalization completes, you can re-run project hypotheses to see the updated confidence levels and statuses.

Step 5: Exporting the Report
&gt;
&gt;Finally, export a report:
&gt;
&gt;./hound.py report rustic_server
&gt;
&gt;This generates an HTML report with the audit details.
