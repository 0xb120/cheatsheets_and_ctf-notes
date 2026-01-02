---
Category:
  - B2R
Difficulty: Medium
Platform: HackTheBox
Status: 3. Complete
tags: [Android, RCE, SSTI, code-review, command-injection, hardcoded-key, insecure-credentials, parameter-injection, reversing, Linux]
---
# Resolution summary

>[!summary]
>- Enumerate and discover services.
>- Download the APK file and analyze it.
>- Get key token through APK and get user credentials.
>- Get the Shell by user credentials.
>- Further information gathering to discover another user credentials.
>- Login to monitor the target process.
>- Discover key scripts and code audit to discover power extraction vulnerabilities.

# Walkthrough

[https://rustlang.rs/posts/HTB-Catch/](https://rustlang.rs/posts/HTB-Catch/)