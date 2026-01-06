---
author: Phil Nash
aliases:
  - A Comprehensive Guide to the Dangers of Regular Expressions in JavaScript
tags:
  - readwise/articles
url: https://www.sonarsource.com/blog/vulnerable-regular-expressions-javascript
created: 2024-08-20
---
# A Comprehensive Guide to the Dangers of Regular Expressions in JavaScript

![rw-book-cover](https://www.sonarsource.com/app-icon.png)

## Highlights


> We'll investigate real-life examples of vulnerable regular expressions from outage reports and open source.
> [View Highlight](https://read.readwise.io/read/01he02m6s6pppbgneb12n1wc8n)



> We'll see what can go wrong with seemingly innocent regular expressions like `/\s*,\s*/` or `/^(.+\.)*localhost$/`.
> [View Highlight](https://read.readwise.io/read/01he02mfyttvjvp0egqkbe59ff)



> Due to the way that many regular expression engines work it is possible to write an expression that, with the right input, will cause the engine to take a long time to evaluate. In JavaScript, this will occupy the main thread and halt the event loop until the expression has been completely evaluated.
> [View Highlight](https://read.readwise.io/read/01he02n8mgzfynrgrh99sehz2t)



> In the Stack Overflow outage, the offending regular expression was `/^[\s\u200c]+|[\s\u200c]+$/`.
> [View Highlight](https://read.readwise.io/read/01he02rskpzvkgdvvxdykcjqcn)



> Put together, the expression looks for one or more whitespace characters at the start of a line or one or more whitespace characters at the end of a line. It was being used to trim whitespace from the beginning or end of a line.
>  This works great if the string begins or ends with a whitespace character. However, if a string ends with a lot of space characters and then a non-space character it will cause an issue.
> [View Highlight](https://read.readwise.io/read/01he02v7sx2e57nfjgfpm20dpp)



> The engine backtracks to where it started the match, discards the first "a" and starts again with the second "a". Now it matches the next three characters, meets the "b" and fails the match.
>  It then backtracks again, starting with the third "a" and repeats for the fourth and fifth as well.
> [View Highlight](https://read.readwise.io/read/01he02y47rbfvdex3hzf7amcnc)



> In [node-fetch](https://www.npmjs.com/package/node-fetch), a function to check whether an origin was trustworthy used a regular expression to aid in detecting whether a URL is trustworthy. One of the tests used the regular expression `/^(.+\.)*localhost$/`.
> [View Highlight](https://read.readwise.io/read/01he0326veqcgt6fh4tfax23cj)



> problem is that both the `+` and `*` quantifiers are greedy and will try to match as much as they can. This initially causes the wildcard to match everything in a string, before backtracking to match the period.
> [View Highlight](https://read.readwise.io/read/01he035gbh568vq7vmaz46xwpt)



> Consider the string "a.a.a.a.a.a.a". The expression will find the last period and then go on to check whether the group exists zero or more times before looking for the ending, the literal "localhost". It doesn't find it, so the first attempt at matching fails and these are the characters considered:
>  Now the backtracking starts, the first group matches all but one of the "a." strings and then the `*` quantifier causes that group to match the last "a." string.
>  The last character doesn't match "localhost", so we backtrack again. Now the options start to build up. We match the first four "a." strings, and the next two can either be matched together or in two groups.
>  This fails, so we backtrack again. Now we match the first three "a." strings and the last three can be matched in four different ways.
> [View Highlight](https://read.readwise.io/read/01he037nzs2dm29ey9r84v4ypm)

