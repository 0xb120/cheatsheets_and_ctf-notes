---
author: Eddie Zhang, Aug 27
aliases: [CVE Hunting Made Easy]
tags: [readwise/articles, wordpress, code-review, wordpress/plugins]
url: https://projectblack.io/blog/cve-hunting-at-scale/
date: 2024-09-10
---
# CVE Hunting Made Easy

![rw-book-cover](https://projectblack.io/blog/content/images/2024/08/cve-hunting-feature-sr.png)


Vulnerable software is plentiful, and with access to source code, we can automate the discovery of high-impact CVEs by focusing on breadth rather than depth. [](https://read.readwise.io/read/01j7e93z1f1532vgbgr392v3y3)

How many serious security issues can we find just by running some SAST scans on every WordPress plugin?
[](https://read.readwise.io/read/01j7e94ssc4hzs7zs955azadr1)


## How do we get a list of all plugins and download them reliably?

As it turns out, this was a non-issue. WordPress publishes a public API that provides information about all plugins in their catalog, and it even includes a download link for each plugin. Awesome!
If you want to take a look yourself, check out some of the raw output: [WordPress Plugin API](https://api.wordpress.org/plugins/info/1.2/?action=query_plugins&request%5Bpage%5D=1&request%5Bper_page%5D=10). [](https://read.readwise.io/read/01j7e95jd3mv1pgzw8n8s5ma11)

SAST tools, in general, produce a lot of output. How do I triage and query this efficiently?
I figured the best way to handle all this output was to store it in a SQL database. This way, the results could be easily queried and searched as needed. [](https://read.readwise.io/read/01j7e96wvenbjyf7xyjw76938c)

The next part of our script:
- Runs Semgrep (unaffiliated) with the 'p/php' ruleset across each plugin.
- Stores the raw Semgrep output in a JSON file in the plugin directory.
- Parses this output and inserts it into a database.


**Couple of rules** for triaging the corpus so I wouldn't end up spending too much time [](https://read.readwise.io/read/01j7e97ycr9jq991wahz3yag0e)

- I only looked at the output for LFI (Local File Inclusion) and SQL injection rules.
- The plugin must have a active install base greater than 0.
- Vulnerabilities only exploitable by administrators will be ignored.
- A maximum of 5 minutes will be spent **triaging** each Semgrep finding that looked *interesting.*


Just because a line of code looks like it could be vulnerable doesn’t mean it actually is. Our next step is to double-check that the issues flagged by Semgrep are genuinely exploitable.

What we're effectively doing here is Sink - Source code review.[](https://read.readwise.io/read/01j7e9a2rdnbhwnz88afn3qqqb). Once we’ve identified a plugin that looks like it might be exploitable, the next step is to validate it through actual exploitation. [](https://read.readwise.io/read/01j7e9cgkm9yvxwej3s3g0ksvx)



## Takeaways
This was a fun project with a few takeaway learnings:
- Unsurprisingly, most of plugins we disclosed issues for had a small install base, around 200-2,000 users, with the largest being *quiz-master-next*, which had over 40,000 installs at the time of writing.
- There's more than one way to go CVE hunting. Usually, the focus is on depth—picking one product or codebase and reverse-engineering it thoroughly. Here, we took the opposite approach, going for breadth rather than depth. [](https://read.readwise.io/read/01j7e9fr9q5wm82jqdqbg0ksjc)
- If you're a pentester working on a WordPress site in your next test, take a closer look at the plugins installed on your customer's site. Just because there aren't any disclosed issues doesn't mean there aren't any easily discoverable vulnerabilities waiting to be found. [](https://read.readwise.io/read/01j7e9fvandt7e0pjm5rcmw74f)

