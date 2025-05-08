---
author: novasecio
aliases:
  - "Recon for Bug Bounty: 8 Essential Tools for Performing Effective Reconnaissance"
tags:
  - readwise/articles
  - Tools
url: https://blog.intigriti.com/hacking-tools/recon-for-bug-bounty-8-essential-tools-for-performing-effective-reconnaissance
date: 2024-10-22
---
# Recon for Bug Bounty: 8 Essential Tools for Performing Effective Reconnaissance

## Amass
[Amass](https://github.com/owasp-amass/amass) is an in-depth attack surface management open-source tool developed by OWASP that can be used to gather assets using both passive as well as active enumeration methods. It combines all these different sources and enumeration methodologies to help you find every host in your list of targets.

To perform a quick subdomain scan for a root domain (utilizing only passive sources), you can use the following command:
`amass enum -d example.com -passive`

> [View Highlight](https://read.readwise.io/read/01jasm8py7s15nf0t5bd732188)


## Eyewitness
After you've enumerated your list of targets, hosts and subdomains, it's time to filter out the non-resolving hosts. [Eyewitness](https://github.com/RedSiege/EyeWitness) is an incredible tool to help you not only with probing live hosts and screenshotting them but also perform basic technology fingerprinting. This enables you to quickly fly over your list of targets and assess each host independently.

> [View Highlight](https://read.readwise.io/read/01jasmafry4x6vv4hfd68bq0hn)


## HTTPX
[HTTPX](https://github.com/projectdiscovery/httpx) (which relies on the Wappalyzer package) for fingerprinting your list of targets.

> [View Highlight](https://read.readwise.io/read/01jasmb8fphxqr5836g5976tt8)


## GAU
GAU (short for GetAllUrls) is an open-source tool that can help you quickly fetch URLs, links and other indexed files from the WaybackMachine (Internet Archive) and other archiving and indexing engines!

It's also easy to use, to return links of a list of targets, you can use the command below:
` cat targets.txt | gau`

> [View Highlight](https://read.readwise.io/read/01jasmbmkv40eefws9ft6xw3pt)

## LinkFinder
Javascript files are a goldmine for bug bounty hunters, and this is often because they contain several references to links and endpoints that could previously not be found through other content discovery methods.

[LinkFinder](https://github.com/GerbenJavado/LinkFinder) is a simple tool that performs an exceptional job of finding new links, URLs and other referenced files and endpoints in javascript code.

Linkfinder is also easy to use, here's how to quickly analyze a javascript file:
`python3 linkfinder.py -i https://example.com/app.js`

> [View Highlight](https://read.readwise.io/read/01jasmd6h508h0nm15700sfa4g)

