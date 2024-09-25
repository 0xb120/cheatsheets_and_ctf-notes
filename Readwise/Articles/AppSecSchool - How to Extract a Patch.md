---
author: AppSecSchool
tags:
  - readwise/articles
url: https://www.youtube.com/watch?v=wEVcv88BoJ4&ab_channel=AppSecSchool
---
# How to Extract a Patch?

![rw-book-cover](https://i.ytimg.com/vi/wEVcv88BoJ4/maxresdefault.jpg)

Tags: #patch-diffing 

## Notes
Easy way to do patch diffing using git and grep

## Highlights

> It is essential to meticulously read every detail in the advisory because this will guide us on what to look for ([View Highlight](https://read.readwise.io/read/01hb3kvbyzh1sm82haweeq4kpy))

> We clone the project using [git](../../Dev,%20ICT%20&%20Cybersec/Tools/git.md) clone. Since it's a big project, it will take a while. Now, after cloning, we go into the directory and run git tag to list all the tags. With a flood of results, we will use grep to filter out required tags by grepping for 11.0.0, which promptly displays our two tags, 11.0.0-M10 and 11.0.0-M11 ([View Highlight](https://read.readwise.io/read/01hb3kxrgbmve89we7e17qqxvx))

> Let's run git diff with our version numbers to extract all the changes between these two versions. ([View Highlight](https://read.readwise.io/read/01hb3kyep39pvka38bj8fpfpym))

> I like to then extract the files that changed using grep +++. ([View Highlight](https://read.readwise.io/read/01hb3kzjk66esweyv9qvq0nqzb))
