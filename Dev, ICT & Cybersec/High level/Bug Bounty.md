# Bug Bounty 101

A bug bounty program is a deal offered by many websites, organizations, and software developers by which individuals can receive recognition and compensation [^bb-compensation] for reporting bugs, especially those pertaining to security exploits and vulnerabilities. [^bb]

[^bb]: [Bug Bounty Program](https://en.wikipedia.org/wiki/Bug_bounty_program); wikipedia.org
[^bb-compensation]: [Intigriti - The Major Bug Bounty Debate Which Department Should Pay for Rewards](../../Readwise/Articles/Intigriti%20-%20The%20Major%20Bug%20Bounty%20Debate%20Which%20Department%20Should%20Pay%20for%20Rewards.md)

## Most famous Bug Bounty programs

- [HackerOne](https://www.hackerone.com/)
- [Bugcrowd](https://www.bugcrowd.com/)
- [Intigriti](https://www.intigriti.com/)
- [Wordfence](https://www.wordfence.com/) [^wordfence-program]
- [Patchstack](https://patchstack.com/articles/bug-bounty-guidelines-rules)
- [Google Mobile VRP](https://bughunters.google.com/about/rules/android-friends/6618732618186752/google-mobile-vulnerability-reward-program-rules)

[^wordfence-program]: [Catalin Cimpanu - Risky Biz News Clop Is Coming After Your SysAid Servers](../../Readwise/Articles/Catalin%20Cimpanu%20-%20Risky%20Biz%20News%20Clop%20Is%20Coming%20After%20Your%20SysAid%20Servers.md#^a6b8e1)


### Patchstack vs Wordfence

- https://www.wordfence.com/wordfence-vs-patchstack/
- https://patchstack.com/bug-bounty/
- https://patchstack.com/articles/bug-bounty-guidelines-rules/
- https://www.wordfence.com/threat-intel/bug-bounty-program/


|                                         | Wordfence                                                                                                                                                                                | Patchstack                                                                                                                                                                                                                                                                     |
| --------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Standard Scope                          | Plugins with 50k+ active installations (standard)<br>Plugins with 15k+ active installations (resourceful)<br>Plugins with 1k+ active installations (1337)                                | Plugins with 50+ active installations                                                                                                                                                                                                                                          |
| Time constraints                        | Last updated in 2 years                                                                                                                                                                  | No restrictions on the last version’s age                                                                                                                                                                                                                                      |
| Standard in-scope vulns.                | CVSS ≥ Medium                                                                                                                                                                            | 50 - 500 active installs: Only unauth., subscriber, customer roles and CVSS is ≥6.4<br>500+: any vuln                                                                                                                                                                          |
| Plugins in scope for high impact vulns. | Plugins with 1k+ active installations                                                                                                                                                    | 50+, zeroday cash bonus for plugins with 1k+ active installations                                                                                                                                                                                                              |
| High-impact in-scope vuln.              | Arbitrary PHP File Upload or Read<br>Arbitrary PHP File Deletion<br>Arbitrary Options Update<br>Remote Code Execution<br>Authentication Bypass to Admin<br>Privilege Escalation to Admin | Vulnerability leads to a full site compromise (ability to upload & access a functional backdoor) and exploitable with Unauthenticated (none), or non-higher than Subscriber/Customer roles.                                                                                    |
| High privileges vulnerabilities         | OoS. Administrator, Editor, and Shop Manager roles, along with any other role that has the `unfiltered_html` capability fall into this category.                                         | Superadmin / Admin, CVE awarded but no XP given                                                                                                                                                                                                                                |
| Bounties                                | Use [Bounty Estimator](https://www.wordfence.com/threat-intel/bug-bounty-program/#rewards)                                                                                               | Monthly competitions. Guaranteed **minimum** bounty pool of $8,800. Each month, the total prize pool will be paid out based on the results from the final leaderboard of 21 researchers. Bounties will be paid only if researcher has more than 0 AXP on the particular month. |


