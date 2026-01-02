---
author: Daniel Stenberg
aliases:
  - CVSS Is Dead to Us
tags:
  - readwise/articles
url: ?__readwiseLocation=
date: 2025-03-06
---
# CVSS Is Dead to Us

![rw-book-cover](https://daniel.haxx.se/blog/wp-content/uploads/2023/08/9-8-critical.jpg)

In the curl project we have **given up** trying to use CVSS to get a severity score and associated severity level. [](https://read.readwise.io/read/01jnnqzrajmg9evwpvwb8mrafs)
We instead work hard to put all our knowledge together and give a rough indication about the severity by dividing it into [one out of four levels](https://curl.se/dev/vuln-disclosure.html#severity-levels): **low**, **medium**, **high**, **critical**. [](https://read.readwise.io/read/01jnnr00n7eqft0cnhbahh2fzw)

We believe that because we are not tied to any (flawed and limited) calculator and because we are intimately familiar with the code base and how it is used, we can assess and set a better security severity this way. It serves our users better. [](https://read.readwise.io/read/01jnnr2m3b8gnj7kz7bkvesg8e)

However, the CVE system itself it built on the idea that every flaw has a CVSS score. When someone like us creates CVE entries without scores, that leaves something that apparently is considered a gaping sore in the system that someone needs to “fix”. [](https://read.readwise.io/read/01jnnr5fxnmj64yk216gq33vjt)

A while ago this new role was added to the CVE ecosystem called ADPs [](https://read.readwise.io/read/01jnnr60s8kxkfveyxkfx8179g) who would get all the CVEs, edit them and then publish them all themselves to the world with their additions. [](https://read.readwise.io/read/01jnnr6tv3rmrgnkk5jy37bp58)

However NVD kind of drowned themselves by this overwhelming work and it has instead been replaced by CISA who is an “ADP” and is thus allowed to *enrich* CVE entries in the database that they think need “improvement”. [](https://read.readwise.io/read/01jnnr7e0rkdn8vtjc650f7p53)

Exactly in the same way this system was broken before when NVD did it, this new system is broken when CISA does it. [](https://read.readwise.io/read/01jnnr8d3p8pqf9acwtdrpyp18)

One positive change that the switch to CISA from NVD brought is that now they host their additional data in [GitHub repository](https://github.com/cisagov/vulnrichment). [](https://read.readwise.io/read/01jnnrh7w6nkk98p73y05gns63)

A problem with the initial bad score getting published is of course that a certain number of websites and systems are really slow or otherwise bad at *updating* that information after they initially learned about the critical score. [](https://read.readwise.io/read/01jnnrmhx4809d972crhmmysm0)

I don’t think switching to CVSS 4.0 or updates to this system is ultimately going to help us. The problem is grounded in the fact that a single one-dimensional score is just too limited. [](https://read.readwise.io/read/01jnnrrwnrk0vbfetnf0fe2j9d)

CVSS is short for [Common Vulnerability Scoring System](https://en.wikipedia.org/wiki/Common_Vulnerability_Scoring_System) and is according to Wikipedia *a technical standard for assessing the severity of vulnerabilities in computing systems*. [](https://read.readwise.io/read/01jnnnz15g8jjqp01h2dbkbvsq)

The CVSS scoring is really designed for when you know exactly when and how the product is used and how an exploit of the flaw affects it. [](https://read.readwise.io/read/01jnnpa0v0h2q1vr3f8w7byb66)

