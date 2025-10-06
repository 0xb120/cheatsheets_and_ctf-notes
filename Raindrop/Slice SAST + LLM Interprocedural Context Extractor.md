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

created: Wed Aug 20 2025 02:00:00 GMT+0200
type: article
tags:
  - "_index"

 
  - "AI" 
  - "LLM" 
  - "SAST"

---
# Slice: SAST + LLM Interprocedural Context Extractor

![](https://noperator.dev/posts/slice.png)

> [!summary]
>Earlier this summer, Sean Heelan published a great blog post detailing his use of o3 to find a use-after-free vulnerability in the Linux kernel. The internet lit up in response, and for good reason. Since the initial release of ChatGPT in late 2022, we’ve all been wondering: Can LLMs really find complex vulnerabilities in widely used production codebases? The Linux kernel is a great research target to help answer that question.





Earlier this summer, Sean Heelan published a great blog post detailing his use of o3 to find a use-after-free vulnerability in the Linux kernel.

we’ve all been wondering: Can LLMs really find complex vulnerabilities in widely used production codebases? The Linux kernel is a great research target to help answer that question.

a few areas for future work stuck out to me:
&gt;
&gt;How to decide which potentially vulnerable code to send to an LLM?

How to maintain a high signal-to-noise ratio and prevent false positives while expanding the scope of our search?

How to realistically incorporate today’s models into a researcher’s workflow?

TL;DR: I built Slice (SAST + LLM Interprocedural Context Extractor) to address these questions. We’ll show how Slice works by walking through a reproduction of discovering CVE-2025-37778.

Reproducing CVE-2025-37778

similar constraints as the original experiment:
&gt;
&gt;explicitly looking for use-after-free
&gt;analyzing call graph up to a depth of 3
&gt;no tool use (in the MCP sense) or agentic frameworks

Which code to analyze?

To start: if we know we’re looking for use-after-free, then let’s just look for objects being freed and then used later on.

$ find fs/smb/server/ -name &quot;*.c&quot; -o -name &quot;*.h&quot; |
&gt;xargs sed -i &#39;s/^#ifdef/#ifndef/g&#39;

With that out of the way, we can build the CodeQL database and run queries against it. I pointed CodeQL at /fs/smb/server/

(that’s 58 files, 26.4K lines of code, ~204K tokens) and tested built-in queries like UseAfterFree.ql and UseAfterExpiredLifetime.ql with no luck; this wasn’t terribly surprising since many default rules for static analyzers are optimized for low false positives so they don’t generate too much noise in an automated CI/CD pipeline.

I fumbled my way through writing a query that was wide enough to find a valid cross-function UAF. As expected, this generated a lot of false positives despite my attempts to keep the noise down.

$ codeql database create smb-server &#92;
&gt;
&gt;    --language=cpp &#92;
&gt;
&gt;    --source-root=fs/smb/server/ &#92;
&gt;
&gt;    --build-mode=none  # important!
&gt;
&gt;
&gt;
&gt;# so our query can use cpp stuff
&gt;
&gt;$ cat &gt; qlpack.yml &lt;&lt; EOF
&gt;
&gt;name: uaf
&gt;
&gt;dependencies:
&gt;
&gt;  codeql/cpp-all: &quot;*&quot;
&gt;
&gt;EOF
&gt;
&gt;$ codeql pack install
&gt;
&gt;
&gt;
&gt;$ codeql query run uaf.ql &#92;
&gt;
&gt;    --database=smb-server &#92;
&gt;
&gt;    --output uaf.bqrs
&gt;
&gt;
&gt;
&gt;# here&#39;s the bug we want to find
&gt;
&gt;$ codeql bqrs decode --format=csv uaf.bqrs |
&gt;
&gt;    xsv search -s free_func krb5_authenticate |
&gt;
&gt;    xsv search -s use_func smb2_sess_setup |
&gt;
&gt;    xsv flatten
&gt;
&gt;usePoint          user
&gt;
&gt;object            user
&gt;
&gt;free_func         krb5_authenticate
&gt;
&gt;free_file         smb2pdu.c
&gt;
&gt;free_func_def_ln  1580
&gt;
&gt;free_ln           1606
&gt;
&gt;use_func          smb2_sess_setup
&gt;
&gt;use_file          smb2pdu.c
&gt;
&gt;use_func_def_ln   1668
&gt;
&gt;use_ln            1910
&gt;
&gt;
&gt;
&gt;# yikes...good thing this post is about using AI for triage!
&gt;
&gt;$ codeql bqrs decode --format=csv uaf.bqrs | xsv count
&gt;
&gt;1722

Since CodeQL results only give us a bit of information to go on (namely, line numbers and symbol names from source code), we can use Tree-sitter to fetch the rest of the code context that we’d want to send to an LLM for deeper analysis.

The following slice query subcommand runs a CodeQL query on an already-built CodeQL database, enriches the results with Tree-Sitter, and returns only those results with the specified maximum call depth:

$ slice query &#92;
&gt;--call-depth 3 &#92;
&gt;--database smb-server &#92;
&gt;--query uaf.ql &#92;
&gt;--source fs/smb/server/ &#92;
&gt;&gt;query.json
&gt;$ jq &#39;.results | map(select(.query |
&gt;.free_func == &quot;krb5_authenticate&quot;
&gt;and
&gt;.use_func == &quot;smb2_sess_setup&quot;
&gt;))[0]&#39; query.json |
&gt;cut -c -80
&gt;{
&gt;&quot;query&quot;: {
&gt;&quot;object&quot;: &quot;user&quot;,
&gt;&quot;free_func&quot;: &quot;krb5_authenticate&quot;,
&gt;&quot;free_file&quot;: &quot;smb2pdu.c&quot;,
&gt;&quot;free_func_def_ln&quot;: 1580,
&gt;&quot;free_ln&quot;: 1606,
&gt;&quot;use_func&quot;: &quot;smb2_sess_setup&quot;,
&gt;&quot;use_file&quot;: &quot;smb2pdu.c&quot;,
&gt;&quot;use_func_def_ln&quot;: 1668,
&gt;&quot;use_ln&quot;: 1910
&gt;},
&gt;&quot;source&quot;: {
&gt;&quot;free_func&quot;: {
&gt;&quot;def&quot;: &quot; 1580 static int krb5_authenticate(struct ksmbd_work *work,&#92;n 158
&gt;&quot;snippet&quot;: &quot;ksmbd_free_user(sess-&gt;user);&quot;
&gt;},
&gt;&quot;use_func&quot;: {
&gt;&quot;def&quot;: &quot; 1668 int smb2_sess_setup(struct ksmbd_work *work)&#92;n 1669 {&#92;n 16
&gt;&quot;snippet&quot;: &quot;if (sess-&gt;user &amp;&amp; sess-&gt;user-&gt;flags &amp; KSMBD_USER_FLAG_DELAY_SE
&gt;},
&gt;&quot;inter_funcs&quot;: []
&gt;},
&gt;&quot;calls&quot;: {
&gt;&quot;valid&quot;: true,
&gt;&quot;reason&quot;: &quot;Target function can reach source function&quot;,
&gt;&quot;chains&quot;: [
&gt;[
&gt;&quot;smb2_sess_setup&quot;,
&gt;&quot;krb5_authenticate&quot;
&gt;]
&gt;],
&gt;&quot;details&quot;: &quot;Reverse call: smb2_sess_setup calls krb5_authenticate&quot;,
&gt;&quot;min_depth&quot;: 1,
&gt;&quot;max_depth&quot;: 1
&gt;}
&gt;}
&gt;$ jq &#39;.results | length&#39; query.json
&gt;217

217 results sounds more reasonable than 1722, but still quite high.

How to raise the signal-to-noise ratio?

Rather than trying to manually triage 217 potential use-after-free vulnerabilities, we’ll instead make two passes with an LLM:
&gt;
&gt;Triage with a relatively small model. Without yet worrying about any security implications

Analyze the syntactically valid UAF more deeply with a large model. What conditions might actually lead to accessing freed memory? Is this actually exploitable? How might it be fixed?

We can use the slice filter subcommand for both steps

The following pipeline passes the CodeQL query output forward into two filter commands, adding .triage and .analyze keys to each element in the .results array.

$ {
&gt;slice query &#92;
&gt;--call-depth 3 &#92;
&gt;--database smb-server &#92;
&gt;--query uaf.ql &#92;
&gt;--source fs/smb/server/ |
&gt;slice filter &#92;
&gt;--prompt-template triage.tmpl &#92;
&gt;--model gpt-5-mini &#92;
&gt;--reasoning-effort high &#92;
&gt;--concurrency 100 |
&gt;tee triage.json |
&gt;slice filter &#92;
&gt;--prompt-template analyze.tmpl &#92;
&gt;--model gpt-5 &#92;
&gt;--reasoning-effort high &#92;
&gt;--concurrency 20 &#92;
&gt;&gt;analyze.json
&gt;} 2&gt;&amp;1 | grep &#39;token usage statistics&#39; | grep -oE &#39;call.*&#39;
&gt;calls=217 prompt_tok=394621 comp_tok=425909 reason_tok=398464 cost_usd=1.75
&gt;calls=9 prompt_tok=31425 comp_tok=90766 reason_tok=69184 cost_usd=1.64
&gt;$ for step in triage analyze; do
&gt;echo -n &quot;$step results: &quot;
&gt;jq &#39;.results | length&#39; &quot;$step.json&quot;
&gt;done
&gt;triage results: 9
&gt;analyze results: 1

Hey, that’s the bug we wanted to see!

Sure, but is it consistent?
&gt;
&gt;Yes. Across 10 consecutive runs3, it finds the correct bug every time.

Future work

Support other languages. CodeQL and Tree-Sitter have robust support for many compiled and interpreted languages like Go, Python, JavaScript, etc.

Analyze large codebases. This experiment started with a predetermined target within the Linux kernel, but it’d be powerful to be able to automatically summarize independent components of a large codebase and identify high-value research targets.

Rank analysis results.

Add more vuln classes. Only requires a new CodeQL query and prompt template.

Output SARIF. The standard for static analysis tools.

Dynamically generate queries. Use LLMs to generate just-in-time CodeQL queries on the fly.
