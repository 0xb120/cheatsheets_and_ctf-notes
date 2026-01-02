---
raindrop_id: 1321645843
raindrop_highlights:
  68c3e7848e1b546c9bca4fe6: bddc946e527532e61aa9c80444e3fa98
  68c3e797f00bf65247646bb4: a0b8acd11c99626fd71482b2379386fe
  68c3e7e322695b0c108e8f7a: 0ffba866fff5a54b2d2af0c1070b674f
  68c3e7e5ed92012ad758fde7: ab57ae55ffbad89ec1f3346e39192a8a
  68c3e7e8c2f5e25ce8bbfdaf: cd2a73657a7e760b0c337a7910df8f34
  68c3e7f7792cd23ca36528a7: 8310e80176636717ed26ea70b790e3d7
  68c3e8c7f00bf6524764ab03: eaf2844e4c456b7dbfdd21d94df9e83d
  68c3e904ed92012ad7593995: 0f2761f765ea5535f9a4c7eca0accf0b
  68c3e915639d3fd215cbea7f: 91b898a1da94289d0fc97e20596bb2c4
  68c3e96d22695b0c108ee097: a4f38e8f7ed1c5cafa6a578351ca27e5
  68c3e972792cd23ca3657833: 4a8175085ccd6662f7b4b8b2242136fb
  68c3e980f00bf6524764ce28: e24533b477cb41c136704c7c97d3ae5b
  68c3e99a792cd23ca36580a9: 000da4c7657f4f516895a18fb09a22bd
  68c3e9adf6dc05a67862ab39: 517f91c04a9a05b669df54b87b8585e6
  68c3e9bb22695b0c108ef042: e40c32fb1f8f1a70793db67099d916cd
  68c3e9d3b27d5c000ccc3b16: d03fdff89b5195892d4f8849d93af701
  68c3ea038e1b546c9bcad336: 341675f8b445d2c9bbb4e38e0c2e8aef
  68c3ea1a1274f05ffc4fb177: 12c5f454cefcfa6fb7c211f246b619cd
  68c3ea24add581f8c982ffd8: 3cd38a53a3d8cdd26e98fcec23e9000e
  68c3ea2971d9ff417ef7d9f0: 167c3ff200fce32a737b0cd07c7fc79e
  68c3ea41844ec9dfaf5dd739: 46c4b842818636519f2a9337b51ac8cd
  68c3ea488cf791bcf6407760: a279dd83bff478309ad0815676013452
  68c3ea546f5bf3d822bb950e: 70558a5051db0fccae46d0f9246acf54
  68c3ea64ed92012ad75983c7: c88295c0556d050b2e8cf8f798b67b88
  68c3ea691274f05ffc4fc288: 857362782bc022104b06394c8638bea3
  68c3ea74844ec9dfaf5de21b: 22228dbab50d6f9f5c727b1712198e01
  68c3ea7c49f0f533e4288397: 46d66a0fc1f1783b7f7479d1c42e808d
  68c3eaa382b9b2b1ba023364: 82d1b926f472199ef3c0d7747b6a7787
  68c3eaaaadd581f8c9831d63: f6c91108cc2956ed1e86db482f36e0d5
  68c3eab81959d9105a5ac9d3: c7f3b47271ad1572dd399552475b418e
  68c3eac4ed92012ad75997ee: 462438ca2301e1c7fb0615e9ee1c4465
  68c3eacb792cd23ca365c0e4: 974ebe0d9840e84cc9a954d9e088e5ef
  68c3ead0ed92012ad7599a2c: ad798ff982a44d761f20da84387fb6a7
  68c3eada1959d9105a5ad18b: 535e2ef00f6c0a1488596118b896f99f
title: "Slice: SAST + LLM Interprocedural Context Extractor"

description: |-
  Earlier this summer, Sean Heelan published a great blog post detailing his use of o3 to find a use-after-free vulnerability in the Linux kernel. The internet lit up in response, and for good reason. Since the initial release of ChatGPT in late 2022, we’ve all been wondering: Can LLMs really find complex vulnerabilities in widely used production codebases? The Linux kernel is a great research target to help answer that question.

source: https://noperator.dev/posts/slice

created: 1755648000000
type: article
tags: ["_index"]

 
  - "AI" 
  - "LLM" 
  - "SAST"

---
# Slice: SAST + LLM Interprocedural Context Extractor

![](https://noperator.dev/posts/slice.png)

> [!summary]
> Earlier this summer, Sean Heelan published a great blog post detailing his use of o3 to find a use-after-free vulnerability in the Linux kernel. The internet lit up in response, and for good reason. Since the initial release of ChatGPT in late 2022, we’ve all been wondering: Can LLMs really find complex vulnerabilities in widely used production codebases? The Linux kernel is a great research target to help answer that question.





Earlier this summer, Sean Heelan published a great blog post detailing his use of o3 to find a use-after-free vulnerability in the Linux kernel.
we’ve all been wondering: Can LLMs really find complex vulnerabilities in widely used production codebases? The Linux kernel is a great research target to help answer that question.
a few areas for future work stuck out to me:

How to decide which potentially vulnerable code to send to an LLM?
How to maintain a high signal-to-noise ratio and prevent false positives while expanding the scope of our search?
How to realistically incorporate today’s models into a researcher’s workflow?
TL;DR: I built Slice (SAST + LLM Interprocedural Context Extractor) to address these questions. We’ll show how Slice works by walking through a reproduction of discovering CVE-2025-37778.
Reproducing CVE-2025-37778
similar constraints as the original experiment:

explicitly looking for use-after-free
analyzing call graph up to a depth of 3
no tool use (in the MCP sense) or agentic frameworks
Which code to analyze?
To start: if we know we’re looking for use-after-free, then let’s just look for objects being freed and then used later on.
$ find fs/smb/server/ -name "*.c" -o -name "*.h" |
xargs sed -i 's/^#ifdef/#ifndef/g'
With that out of the way, we can build the CodeQL database and run queries against it. I pointed CodeQL at /fs/smb/server/
(that’s 58 files, 26.4K lines of code, ~204K tokens) and tested built-in queries like UseAfterFree.ql and UseAfterExpiredLifetime.ql with no luck; this wasn’t terribly surprising since many default rules for static analyzers are optimized for low false positives so they don’t generate too much noise in an automated CI/CD pipeline.
I fumbled my way through writing a query that was wide enough to find a valid cross-function UAF. As expected, this generated a lot of false positives despite my attempts to keep the noise down.
$ codeql database create smb-server \

    --language=cpp \

    --source-root=fs/smb/server/ \

    --build-mode=none  # important!



# so our query can use cpp stuff

$ cat > qlpack.yml << EOF

name: uaf

dependencies:

  codeql/cpp-all: "*"

EOF

$ codeql pack install



$ codeql query run uaf.ql \

    --database=smb-server \

    --output uaf.bqrs



# here's the bug we want to find

$ codeql bqrs decode --format=csv uaf.bqrs |

    xsv search -s free_func krb5_authenticate |

    xsv search -s use_func smb2_sess_setup |

    xsv flatten

usePoint          user

object            user

free_func         krb5_authenticate

free_file         smb2pdu.c

free_func_def_ln  1580

free_ln           1606

use_func          smb2_sess_setup

use_file          smb2pdu.c

use_func_def_ln   1668

use_ln            1910



# yikes...good thing this post is about using AI for triage!

$ codeql bqrs decode --format=csv uaf.bqrs | xsv count

1722
Since CodeQL results only give us a bit of information to go on (namely, line numbers and symbol names from source code), we can use Tree-sitter to fetch the rest of the code context that we’d want to send to an LLM for deeper analysis.
The following slice query subcommand runs a CodeQL query on an already-built CodeQL database, enriches the results with Tree-Sitter, and returns only those results with the specified maximum call depth:
$ slice query \
--call-depth 3 \
--database smb-server \
--query uaf.ql \
--source fs/smb/server/ \
>query.json
$ jq '.results | map(select(.query |
.free_func == "krb5_authenticate"
and
.use_func == "smb2_sess_setup"
))[0]' query.json |
cut -c -80
{
"query": {
"object": "user",
"free_func": "krb5_authenticate",
"free_file": "smb2pdu.c",
"free_func_def_ln": 1580,
"free_ln": 1606,
"use_func": "smb2_sess_setup",
"use_file": "smb2pdu.c",
"use_func_def_ln": 1668,
"use_ln": 1910
},
"source": {
"free_func": {
"def": " 1580 static int krb5_authenticate(struct ksmbd_work *work,\n 158
"snippet": "ksmbd_free_user(sess->user);"
},
"use_func": {
"def": " 1668 int smb2_sess_setup(struct ksmbd_work *work)\n 1669 {\n 16
"snippet": "if (sess->user && sess->user->flags & KSMBD_USER_FLAG_DELAY_SE
},
"inter_funcs": []
},
"calls": {
"valid": true,
"reason": "Target function can reach source function",
"chains": [
[
"smb2_sess_setup",
"krb5_authenticate"
]
],
"details": "Reverse call: smb2_sess_setup calls krb5_authenticate",
"min_depth": 1,
"max_depth": 1
}
}
$ jq '.results | length' query.json
217
217 results sounds more reasonable than 1722, but still quite high.
How to raise the signal-to-noise ratio?
Rather than trying to manually triage 217 potential use-after-free vulnerabilities, we’ll instead make two passes with an LLM:

Triage with a relatively small model. Without yet worrying about any security implications
Analyze the syntactically valid UAF more deeply with a large model. What conditions might actually lead to accessing freed memory? Is this actually exploitable? How might it be fixed?
We can use the slice filter subcommand for both steps
The following pipeline passes the CodeQL query output forward into two filter commands, adding .triage and .analyze keys to each element in the .results array.
$ {
slice query \
--call-depth 3 \
--database smb-server \
--query uaf.ql \
--source fs/smb/server/ |
slice filter \
--prompt-template triage.tmpl \
--model gpt-5-mini \
--reasoning-effort high \
--concurrency 100 |
tee triage.json |
slice filter \
--prompt-template analyze.tmpl \
--model gpt-5 \
--reasoning-effort high \
--concurrency 20 \
>analyze.json
} 2>&1 | grep 'token usage statistics' | grep -oE 'call.*'
calls=217 prompt_tok=394621 comp_tok=425909 reason_tok=398464 cost_usd=1.75
calls=9 prompt_tok=31425 comp_tok=90766 reason_tok=69184 cost_usd=1.64
$ for step in triage analyze; do
echo -n "$step results: "
jq '.results | length' "$step.json"
done
triage results: 9
analyze results: 1
Hey, that’s the bug we wanted to see!
Sure, but is it consistent?

Yes. Across 10 consecutive runs3, it finds the correct bug every time.
Future work
Support other languages. CodeQL and Tree-Sitter have robust support for many compiled and interpreted languages like Go, Python, JavaScript, etc.
Analyze large codebases. This experiment started with a predetermined target within the Linux kernel, but it’d be powerful to be able to automatically summarize independent components of a large codebase and identify high-value research targets.
Rank analysis results.
Add more vuln classes. Only requires a new CodeQL query and prompt template.
Output SARIF. The standard for static analysis tools.
Dynamically generate queries. Use LLMs to generate just-in-time CodeQL queries on the fly.