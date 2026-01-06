---
author: Erik
aliases:
  - Last Week in Security (LWiS) - 2025-05-19
  - Commit Stomping
tags:
  - readwise/articles
url: https://blog.badsectorlabs.com/last-week-in-security-lwis-2025-05-19.html?__readwiseLocation=
created: 2025-07-07
---
# Last Week in Security (LWiS) - 2025-05-19

![rw-book-cover](https://blog.badsectorlabs.com/images/social.jpg)

## Highlights


[Commit Stomping - Manipulating Git Histories to Obscure the Truth](https://blog.zsec.uk/commit-stomping/) - "[git](../../Dev,%20ICT%20&%20Cybersec/Tools/git.md)’s distributed and trust-based design can be turned into a technique for deception." This post gives some more detail to `git commit "stomping"` following the release of [RepoMan](https://github.com/ZephrFish/RepoMan). [](https://read.readwise.io/read/01jvryhgprq8410978nqk8nxqx) #tools 

# Commit Stomping

### What is Commit Stomping?

**Commit Stomping** is a technique where an attacker manipulates the timestamps of Git commits to alter the historical timeline of a repository. By changing when a commit appears to have happened, an attacker can blend malicious code into a project's history, making it difficult to detect during security reviews or forensic investigations.

### How It Works

The technique exploits legitimate Git features that allow users to rewrite history or specify dates. Key methods include:

- **Environment Variables:** Using `GIT_AUTHOR_DATE` and `GIT_COMMITTER_DATE` to spoof the time a commit was authored or committed.
    
- **Git Commands:** modifying history using commands like `git commit --amend`, `git rebase`, and `git filter-branch` to backdate or alter specific commits.