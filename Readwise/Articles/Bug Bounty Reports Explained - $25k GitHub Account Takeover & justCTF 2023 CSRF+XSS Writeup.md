---
author: Bug Bounty Reports Explained
aliases:
  - $25k GitHub Account Takeover & justCTF 2023 CSRF+XSS Writeup
tags:
  - readwise/articles
url: https://www.youtube.com/watch?v=GLXMGinQyFk&ab_channel=BugBountyReportsExplained
created: 2024-11-15
---
# $25k GitHub Account Takeover & justCTF 2023 CSRF+XSS Writeup

![rw-book-cover](https://i.ytimg.com/vi/GLXMGinQyFk/maxresdefault.jpg?sqp=-oaymwEmCIAKENAF8quKqQMa8AEB-AH-CYAC0AWKAgwIABABGEwgRyhlMA8=&rs=AOn4CLA_UaCEnH2gFIE9f3_khhIOxO-8Dg)

## Highlights


The mistake was made a few lines above this, here, when the **code was not parsed, but tokenized**.
> [View Highlight](https://read.readwise.io/read/01jcr6ndcsnzh6r5pzyfd9611y)


If you have a simple code like this, the outcome would be the same. But it becomes confusing when you introduce HTML namespaces, because **every HTML tag has a namespace**.
> [View Highlight](https://read.readwise.io/read/01jcr6nw1vd4rzx794vf9xvpyk)

![](attachments/tokenizer-vs-parser.png)

Most of them, in most websites that we use, are just in the HTML namespace. But within the HTML, you can also embed objects from a math or SVG namespace. And the thing is, the same tag can behave differently in HTML namespace and in SVG namespace. For example, the text area tag in the HTML namespace can only contain text inside.
> [View Highlight](https://read.readwise.io/read/01jcr6pjfm33jd980edj6nqr8y)

![](attachments/livedomng-1.png)

But this stops being true when we wrap this in the SVG tag, **which changes the namespace**. 

![](attachments/livedomng-2.png)

And this is when we finally get to the difference between the tokenizer and the parser. The parser will apply these namespace rules to this, and we'll see the script tag being inside. The tokenizer will ignore the namespace, so it will still not see the XSS payload here. The browser, of course, parses our HTML, so this code will look innocent to the sanitizer, but it will execute in the browser, thus giving you the XSS.
> [View Highlight](https://read.readwise.io/read/01jcr6r59qsy22ej4qd95jx43n)

![](attachments/Bug%20Bounty%20Reports%20Explained%20-%20$25k%20GitHub%20Account%20Takeover%20&%20justCTF%202023%20CSRF+XSS%20Writeup-poc.png)

