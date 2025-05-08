---
author: Chris Hosking
aliases:
  - Re-Assessing Risk | Subdomain Takeovers as Supply Chain Attacks
tags:
  - readwise/articles
url: https://www.sentinelone.com/blog/re-assessing-risk-subdomain-takeovers-as-supply-chain-attacks/?__readwiseLocation=
date: 2025-04-24
summary: Subdomain takeovers occur when attackers exploit improperly configured or unused subdomains, which can lead to serious supply chain risks. These vulnerabilities can arise from outdated DNS records or deleted cloud resources, allowing attackers to seize control and potentially harm an organization’s reputation. To mitigate these risks, security teams should proactively monitor and manage their DNS configurations and cloud resources.
---
# Re-Assessing Risk | Subdomain Takeovers as Supply Chain Attacks

Subdomain takeovers: a known issue that may not always appear critical at first glance. Within some specific scenarios though, they could rise into a drastic [Supply Chain](../../Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/Supply%20Chain.md) threat. [](https://read.readwise.io/read/01jrwqf1f21sfmax3n4zxbga6c)


## Dangling DNS

A [Subdomain takeover](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Domain%20Attacks.md#Subdomain%20takeover) happens when an attacker gains control of an organization’s improperly configured or unused subdomain. [](https://read.readwise.io/read/01jrwqfbhwvqzxbas21fn6qznt)

There’s a number of ways this can happen, the simplest of which is when [DNS - Domain Name System](../../Dev,%20ICT%20&%20Cybersec/Services/DNS%20-%20Domain%20Name%20System.md) records, specifically canonical name (CNAME) records, have been set up to point to a specific subdomain, and either:
- The subdomain expires and is available for registration by an attacker, or
- The attacker sets up virtual hosting for the subdomain before you can. [](https://read.readwise.io/read/01jrwqgcfy7tagfz7p2jca3av3)

This scenario is known as “Dangling DNS” [](https://read.readwise.io/read/01jrwqh8tex97pc8e8qkgzex0r)

Alongside canonical name (CNAME) records, the same risk is present for other DNS records: NS, MX, A and AAA records. [](https://read.readwise.io/read/01jrwqhkdjn2rzzmtj35ds4d89)

In some instances, Dangling DNS can be the result of SaaS choices and migrations, [](https://read.readwise.io/read/01jrwqk9vsr8h94scrcq1h16av) with a CNAME pointing to an SaaS provided subdomain. For example, `support.YourBiz.com` points to `YourBiz.zendesk.com`, and `careers.YourBiz.com` points to `YourBiz.smartjobboard.com`. [](https://read.readwise.io/read/01jrwqkntx2dfqacp44rtkxjrv)

If a business stops using a SaaS provider but fails to remove or update the associated CNAME records, or if a service subscription payment lapse, attackers may be able to register the subdomain with that provider, potentially gaining control over it. [](https://read.readwise.io/read/01jrwqm36v0wehp9vwgwv72tsq)

### Dangling DNS From Cloud Providers

Another common example of Dangling DNS involves DNS records pointing to a deprovisioned or deleted cloud resource. [](https://read.readwise.io/read/01jrwqnq2kbczr9kqk49nfqrdc)

Similar to the SaaS examples from above, consider this: If you use cloud object storage to host a static website on a subdomain like `Amazing.YourBiz.Com` and later delete the storage bucket (intentionally or not) without updating the corresponding DNS records, this opens a door for subdomain takeovers.

![](https://www.sentinelone.com/wp-content/uploads/2025/04/404_error_missing_AWS_bucket.jpg) [](https://read.readwise.io/read/01jrwqprzz6nvtza1wfqtq9ztj)

An attacker could then register their own S3 bucket with the same subdomain, in an environment they control, and upload content of their own. [](https://read.readwise.io/read/01jrwqq6d7gpmm164z9d50yygx)

In [Microsoft’s documentation](https://learn.microsoft.com/en-us/azure/security/fundamentals/subdomain-takeover), they breakdown subdomain takeover risks into the below types:
- Loss of control resulting in negative press
- Cookie harvesting from unsuspecting visitors
- Phishing campaigns leveraging authentic-looking subdomains
- Other classic web attacks: cross-site scripting (XSS), cross-site request forgery (CSRF), CORS bypass, and more. [](https://read.readwise.io/read/01jrwqsh1kpkrnb1khtzd9x2h0)

## Where is the Supply Chain Risk? 

Researchers conducted an [in-depth investigation](https://labs.watchtowr.com/8-million-requests-later-we-made-the-solarwinds-supply-chain-attack-look-amateur/) into the nature of these requests and the risks they pose. [^1] [](https://read.readwise.io/read/01jrwqwfexq672cxnvaz55415r)

[^1]: [Benjamin Harris - 8 Million Requests Later, We Made the SolarWinds Supply Chain Attack Look Amateur](Benjamin%20Harris%20-%208%20Million%20Requests%20Later,%20We%20Made%20the%20SolarWinds%20Supply%20Chain%20Attack%20Look%20Amateur.md)

