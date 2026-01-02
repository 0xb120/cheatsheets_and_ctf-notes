---
author: Connor Ford
aliases: [Version Tracking in Ghidra]
tags: [readwise/articles, Tools]
url: https://labs.nettitude.com/blog/version-tracking-in-ghidra/
date: 2024-08-21
---
# Version Tracking in Ghidra

![rw-book-cover](https://labs.nettitude.com/wp-content/uploads/2024/08/header-2.png)

## Highlights


> When a binary is reverse engineered using [ghidra](../../Dev,%20ICT%20&%20Cybersec/Tools/ghidra.md), various annotations are applied to aid in understanding the binary’s behaviour. These annotations come in the form of comments, renamed functions, variables, arguments and more. Collectively these annotations are known as “markup” and are specific to a single binary in the Ghidra project.
>  For long running reverse engineering projects, newer versions of a binary may be released periodically, which introduces the problem of starting the reverse engineering process from scratch against the latest version.
> [View Highlight](https://read.readwise.io/read/01j5tnsa5bw3cdh6xbm1txaa4b)



> This is where Ghidra’s Version Tracking tool comes in handy. The Version Tracking tool works by using “correlators” to match specific parts of a source binary to parts of a destination binary. If a match is valid, then the markup from the source binary can be applied to the destination binary in a semi-automated way.
>  This blog post walks through using the Version Tracking tool to apply markup from an old binary to a newer one, and explains the main concepts along the way.
> [View Highlight](https://read.readwise.io/read/01j5tnsn0ay606ks1ch9bvatsd)

