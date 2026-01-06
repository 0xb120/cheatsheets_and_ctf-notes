---
author: Adan
aliases:
  - "GitHub Actions and the Pinning Problem: What 100 Security Projects Reveal"
tags:
  - readwise/articles
url: https://medium.com/@adan.alvarez/github-actions-and-the-pinning-problem-what-100-security-projects-reveal-54a3a9dcc902?__readwiseLocation=
created: 2025-04-24
---
# GitHub Actions and the Pinning Problem: What 100 Security Projects Reveal

![rw-book-cover](https://miro.medium.com/v2/resize:fit:1200/1*VJdUP9IAo_lQy5kOPdV--A.png)

A targeted attack on Coinbase ended up impacting other popular GitHub repositories and [GitHub Actions](../../Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/GitHub%20Actions.md) as well. It showed a larger issue: many GitHub Actions can be silently changed, putting your workflows at risk. [](https://read.readwise.io/read/01jra41wp9knaph64zts551h8y)

Read more about the incident in these great posts from Wiz and Unit 42: 
- [https://www.wiz.io/blog/new-github-action-supply-chain-attack-reviewdog-action-setup](https://www.wiz.io/blog/new-github-action-supply-chain-attack-reviewdog-action-setup) 
- [https://unit42.paloaltonetworks.com/github-actions-supply-chain-attack/](https://unit42.paloaltonetworks.com/github-actions-supply-chain-attack/) [](https://read.readwise.io/read/01jra4249pzkhyn7tejy2va1a6)

The main takeaway? **Pin your GitHub Actions** [^1]. That means referencing a specific commit SHA, not a tag like `@v1` or `@main`, so the code you execute will always be the same. [](https://read.readwise.io/read/01jra42n3rac867jjpdt62stb4)

However, you might still be vulnerable if those actions rely on other actions that aren’t pinned. [](https://read.readwise.io/read/01jra439x6smz0xnk5ab2pgrz5) [^2]

Using the [Open Source Security Index](https://opensourcesecurityindex.io/) [^3], I took the top 100 most popular & fastest growing open source security projects [](https://read.readwise.io/read/01jra484aqpwyxeptzvjmdschn) and scanned them using a [custom script](https://github.com/adanalvarez/MyScripts/tree/main/GitHub) that:
- Parses all GitHub Actions used in workflows
- Checks whether actions are pinned (SHA) or not
- Recursively checks dependencies of those actions
- Outputs trusted/untrusted dependency patterns [](https://read.readwise.io/read/01jra49b7yfx2qf3nnrm2t957k)

Only 7 out of 100 repositories have all actions pinned. [](https://read.readwise.io/read/01jra49qd36jnzb2ydb8359265)

51 repositories don’t pin any of their actions.
Over half of the analyzed projects are using fully unpinned workflows. What this means is that the code they run in the GitHub workflows could change at any time. [](https://read.readwise.io/read/01jra4aqmq09jd5zynswx650na)

42 repositories had a mix of pinned and unpinned actions.
This may suggest that some teams are choosing to pin only the actions they don’t fully trust. [](https://read.readwise.io/read/01jra4b6dmv47r3pfydan229kb)

The most popular actions are the official ones from GitHub:
![](https://miro.medium.com/v2/resize:fit:700/1*99AHWb3-wz_Kl-qiv-e3gg.png) [](https://read.readwise.io/read/01jra4dd92ymgcv8zwrm3weraw)

## Want to Pin Your Actions?

Here’s how you can start:
1. Fix existing workflows
	- [**SecureRepo (UI tool)**](https://lnkd.in/d7k7ESmV) — An Easy web that will allow you to do a PR to improve the security and pin your actions
	- **CLI tools to automate the process:**`[mheap/pin-github-action](https://github.com/mheap/pin-github-action) [suzuki-shunsuke/pinact](https://github.com/suzuki-shunsuke/pinact)`
	- [**Renovate helper action**](https://lnkd.in/dwdAv9i8) [](https://read.readwise.io/read/01jra4et0qm3yxrt57qp2m2hxh)
2. Keep pinned actions up to date
	- Use [**Dependabot**](https://docs.github.com/en/code-security/supply-chain-security/keeping-your-dependencies-updated-automatically) or [**Renovate**](https://docs.renovatebot.com/) to track and update pinned SHAs when the action is updated. [](https://read.readwise.io/read/01jra4f1y85x46ybwd3g1eed4y)
3. Make sure all new actions are pinned
	Use security scanning tools to enforce best practices:
	- [Semgrep custom rule](https://lnkd.in/daG2RXBg)
	- [Datadog’s SAST for GitHub Actions](https://lnkd.in/di9Shng2) [](https://read.readwise.io/read/01jra4f97m72hgdfz2jexfpnw3)

[^1]: [Action Pinning](Yaron%20Avital%20-%20Unpinnable%20Actions%20How%20Malicious%20Code%20Can%20Sneak%20Into%20Your%20GitHub%20Actions%20Workflows.md#Action%20Pinning)

[^2]: [Unpinnable Actions](Yaron%20Avital%20-%20Unpinnable%20Actions%20How%20Malicious%20Code%20Can%20Sneak%20Into%20Your%20GitHub%20Actions%20Workflows.md)

[^3]: [Passive information gathering (OSINT)](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Passive%20information%20gathering%20(OSINT).md#^890ecf)
