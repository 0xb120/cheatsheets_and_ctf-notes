---
raindrop_id: 1349555064
raindrop_highlights:
  68d5b49b420977e81663a836: 14a6220821a948f20bc7c85a43c0e3cf
  68d5b4a4c09d890017688a4f: 4514d3e4ceb8e011ebfa9e213be616b8
  68d5b55af5019387c3337802: e6192e951b598c2e476b0ea841481e16
  68d5b576d522af76a3265ec5: 8c32eaef75ee5843f23586b238d3dcf5
  68d5b58408ae98bc807ce22d: edc6715a9b64a5f0219cdafc75c7fa1f
  68d5b589420977e81663c7ad: 50000e5da3583c0e75232f3932120603
  68d5b59791c768ef7e5f5c4f: f0f0b2c66834261c04b9381f186a53a9
  68d5b5aed522af76a326658b: 0d21e4ae0c078bc2b9c6bccb09261f73
  68d5b5bf6ce16004f4c6d084: ed878ff12a6dc3261cd7f24e9f94b83c
  68d5b5c9f5019387c3338676: 32b59563d4b1a7bacd5bfaeb655f03e4
  68d5b6e1420977e81663f31c: 1606b5f18a5cfa7c00ad056b8903c5fa
  68d5b6ec5aebcc0fd31e25f1: 2cb571001c3370e9abc46c9268997d99
  68d5b843c09d89001768fbc0: ccfe7b939642ce3cf1a88761c79890a3
  68d5b84e5430135b7f7f4e91: 13c530eee997ec8edd9e6f306756d34a
  68d5b866b81b9bf2807fbfd3: 9f6b0b2c66707643dcc9bcca383a9dad
  68d5ba8b15d4063d77525cfe: 50e57df99c5064192585992a2bebf76a
  68d5bacc3d2379b5b115b773: 1c6ec5784b56d635c10b36e7aa520677
  68d5badbb3703b68c12a8a4d: 0b4b932531ccb40dc226f54d011a04bb
  68d5bae7f5019387c3342352: 61573fe92d0d1df75d5fcc77c1b94fea
  68d5bb14c8cc0b1170ee1043: 9620b5a47cbe392cc936c0649ad1e7bb
  68d5bb1aa2be5671f21d5cb9: fc2345648e879aee91a47aa2092a406b
  68d5bb20e77c5209d0147a8c: e2be810de1ef6fb96eb172a3ff355af9
  68d5bb25c09d8900176957b3: 664e4ccb5dfeef45f97a88f92cc066c6
  68d5bb3291c768ef7e600b2b: a9af81a3f49626312bb4c2928eabbcdd
title: "Code auditing 101 -"

description: |-
  Topics covered This post explores the evolution from manual code review to automated security testing, covering:
  The reality of manual code review and its limitations Understanding vulnerabilities vs weaknesses How SAST tools work under the hood Taint analysis and data flow tracking Sink-to-source vs source-to-sink methodologies Mitigation strategies: whitelisting vs blacklisting Dealing with false positives in practice Choosing and implementing SAST tools at scale The complementary relationship between manual and automated testing It’s 3 AM. You’re on your fifth cup of coffee, eyes bloodshot, staring at line 2,847 of a 10,000-line pull request. Somewhere in this maze of curly braces and semicolons lurks a SQL injection vulnerability that could bring down your entire application. Welcome to the glamorous world of manual code review!

source: https://blog.rodolpheg.xyz/posts/code-auditing--101/

created: 1758299696280
type: article
tags:
  - "_index"

 
  - "tech-blog" 
  - "SAST"

---
# Code auditing 101 -

![](https://blog.rodolpheg.xyz/img/link_logo_orange_darkmode.png)

> [!summary]
> Topics covered This post explores the evolution from manual code review to automated security testing, covering:
The reality of manual code review and its limitations Understanding vulnerabilities vs weaknesses How SAST tools work under the hood Taint analysis and data flow tracking Sink-to-source vs source-to-sink methodologies Mitigation strategies: whitelisting vs blacklisting Dealing with false positives in practice Choosing and implementing SAST tools at scale The complementary relationship between manual and automated testing It’s 3 AM. You’re on your fifth cup of coffee, eyes bloodshot, staring at line 2,847 of a 10,000-line pull request. Somewhere in this maze of curly braces and semicolons lurks a SQL injection vulnerability that could bring down your entire application. Welcome to the glamorous world of manual code review!





Code auditing 101
This post explores the evolution from manual code review to automated security testing
Manual review has its charm. You develop an intimate relationship with the codebase, understanding not just what the code does, but why that junior developer thought using eval() on user input was a stellar idea. You become fluent in reading between the lines, spotting patterns like the infamous “TODO: Add authentication” comment that’s been there since 2019, variable names like temp, temp2, and tempFinal (spoiler: it’s never final), or that one function that’s 500 lines long because “it works, don’t touch it.”
Vulnerabilities vs Weaknesses: Know Your Enemy
In the security world, we throw around terms like “vulnerability” and “weakness” as if they’re interchangeable. Spoiler alert: they’re not.
Weakness: The Potential Problem
A weakness is like leaving your window unlocked. It’s a flaw in your code that could be exploited, but there might be other factors preventing actual exploitation.
Vulnerability: The Actual Problem
A vulnerability is when that unlocked window is on the ground floor, facing a dark alley, with a “Rob Me” sign. It’s a weakness that can actually be exploited in your specific context
The relationship? Every vulnerability is a weakness, but not every weakness is a vulnerability.
When to use sink-to-source:

Security audits with time constraints
Focusing on critical vulnerabilities only
When you know which sinks are most dangerous in your app
Compliance checks for specific vulnerability types
When to use source-to-sink:

Development phase (catch everything early)
When you need comprehensive coverage
For understanding all data flows in your application
When building security test cases
The Bottom Line: Practical SAST Strategy
Start small - don’t scan your entire monolith on day one. Pick one service, tune it, learn from it. Embrace the hybrid approach by using source-to-sink for comprehensive coverage during development, then switch to sink-to-source for focused security audits, while keeping manual review for business logic and architectural decisions.
Manage your expectations because SAST won’t find everything. It’s terrible at authentication flaws (“Is this login secure?” 🤷), business logic bugs (“Should users order -1 pizzas?” 🍕), and race conditions where timing is everything.
Use SAST for what it’s good at: finding known vulnerability patterns at scale. Configure it properly to reduce noise by excluding test files, adding custom rules, and tuning sensitivity levels. Use manual review for what humans do best: understanding context, business logic, and architectural decisions. Combine both approaches iteratively - let SAST do the first pass, manually review the interesting findings, then dig deeper into areas SAST can’t reach.
References & Training Resources
Essential Reading
OWASP Code Review Guide - The comprehensive guide to secure code review
OWASP Source Code Analysis Tools - Extensive list of SAST tools
Static Code Analysis | OWASP - Fundamentals of static analysis
Technical Deep Dives
How to Find More Vulnerabilities - Source Code Auditing Explained - Practical vulnerability hunting techniques
Performing a Secure Code Review - Step-by-step methodology
Vulnerability.Codes

vulnerability.codes
Real code snippets with actual vulnerabilities
Language-specific vulnerability examples
Great for understanding how vulnerabilities look in different contexts
Hands-On Training Platforms
OWASP Juice Shop

juice-shop.herokuapp.com
OWASP WebGoat

owasp.org/www-project-webgoat
DeepReview.app

deepreview.app/leaderboard
Competitive code review challenges
Real-world vulnerability scenarios
Leaderboard to track your progress against other security researchers