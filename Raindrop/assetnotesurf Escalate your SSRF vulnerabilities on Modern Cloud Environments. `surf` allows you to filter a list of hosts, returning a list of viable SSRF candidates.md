---
raindrop_id: 1548015642
raindrop_highlights:
  696a66fec87488d175793392: 830247212d135ecb1554014e7259a463
  696a671048ef4afafe949068: 7fbf6128760c254b7b93c0e502929a18
  696a671448ef4afafe94913a: 75575155ec98e65cb29f78637632db81
title: "assetnote/surf: Escalate your SSRF vulnerabilities on Modern Cloud Environments. `surf` allows you to filter a list of hosts, returning a list of viable SSRF candidates."

description: |-
  Escalate your SSRF vulnerabilities on Modern Cloud Environments. `surf` allows you to filter a list of hosts, returning a list of viable SSRF candidates. - assetnote/surf

source: https://github.com/assetnote/surf

created: 2026-01-16
sync-date: 1769114384411
tags:
  - "_index"

---
# assetnote/surf: Escalate your SSRF vulnerabilities on Modern Cloud Environments. `surf` allows you to filter a list of hosts, returning a list of viable SSRF candidates.

![](https://opengraph.githubassets.com/5dc7644d579979835cc2a989e97e5106bb550dfa8d783f6740a5a7d73b58b65d/assetnote/surf)

> [!summary]
> Escalate your SSRF vulnerabilities on Modern Cloud Environments. `surf` allows you to filter a list of hosts, returning a list of viable SSRF candidates. - assetnote/surf





Escalate your SSRF vulnerabilities on Modern Cloud Environments. `surf` allows you to filter a list of hosts, returning a list of viable SSRF candidates.
It can be installed with the following command:

go install github.com/assetnote/surf/cmd/surf@latest
Usage

Consider that you have subdomains for bigcorp.com inside a file named bigcorp.txt, and you want to find all the SSRF candidates for these subdomains. Here are some examples:

# find all ssrf candidates (including external IP addresses via HTTP probing)
surf -l bigcorp.txt
# find all ssrf candidates (including external IP addresses via HTTP probing) with timeout and concurrency settings
surf -l bigcorp.txt -t 10 -c 200
# find all ssrf candidates (including external IP addresses via HTTP probing), and just print all hosts
surf -l bigcorp.txt -d
# find all hosts that point to an internal/private IP address (no HTTP probing)
surf -l bigcorp.txt -x