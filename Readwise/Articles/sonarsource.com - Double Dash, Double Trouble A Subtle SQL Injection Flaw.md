---
author: sonarsource.com
aliases:
  - "Double Dash, Double Trouble: A Subtle SQL Injection Flaw"
tags:
  - RW_inbox
  - readwise/articles
url: https://www.sonarsource.com/blog/double-dash-double-trouble-a-subtle-sql-injection-flaw/?__readwiseLocation=?__readwiseLocation=
date: 2025-07-07
---
# Double Dash, Double Trouble: A Subtle SQL Injection Flaw

![rw-book-cover](https://assets-eu-01.kc-usercontent.com:443/886afe32-410a-0136-0267-0f7515a29063/ea023d19-b35a-43a6-9306-3e030ba28b82/Double%20Dash%2C%20Double%20Trouble_Landscape_blog-header.png)

## Highlights


Usually, string parameters are the ones that can cause trouble because they can alter the syntax of a query if they're not properly escaped. But are there also other data types that might alter the syntax of a query? [](https://read.readwise.io/read/01jxeyf9t2692f4ggn46993xyz)



SQL Injection via line comment creation [](https://read.readwise.io/read/01jxeygw15j22zkyqdm3v00cve)



In the right circumstances, an attacker can inject into SQL queries and execute malicious statements. For the attack to work, a prepared statement of a certain structure must be used, which we will detail below. In addition, the library has to use the simple query protocol when communicating with the PostgreSQL server. [](https://read.readwise.io/read/01jxeyhgptm17y6r5qdp86v5r4)



Technical Details[](https://www.sonarsource.com/blog/double-dash-double-trouble-a-subtle-sql-injection-flaw/#technical-details)
 To understand what type of character combinations can change a parsing context in SQL, let's look at a bunch of syntax constructs that change the parsing state for longer sequences [](https://read.readwise.io/read/01jxeyjjyr56pt90tfgfjmb012)



**Strings.** The classic, breaking out of strings leads to SQL injection because strings can contain almost any characters that can now become SQL syntax. [](https://read.readwise.io/read/01jxeyjze3cmqfszwshm8arv6b)



**Identifiers.** Similar to strings. Less likely to be injected into since user input is mostly used for values, not for column names. [](https://read.readwise.io/read/01jxeyka1v2am2epb87y5y587j)



**Comments.** There are line comments, starting with `--`, which comment out the rest of the line. Some Databases also support `#` as the start of a line comment. There are also multi-line or block comments, starting with `/*` and ending with `*/`. [](https://read.readwise.io/read/01jxeym1ejnbysd8mfwq8r6j08)



Looking further at comment syntax, we can see an interesting difference between different database implementations: MySQL requires a trailing space after the two dashes (`--`) that start a line comment. [](https://read.readwise.io/read/01jxeyn5m5pjt4ywwbagbgez2p)



*"[...] the* `*--*` *start-comment sequence is accepted as such, but must be followed by a whitespace character such as a space or newline. The space is intended to prevent problems with generated SQL queries that use constructs [...]"* [](https://read.readwise.io/read/01jxeyn8qg4eb57kmpexd5hgxe)



example of such a problem [](https://read.readwise.io/read/01jxeyng5d4x9x685nx2f2j6wx)



Let's use this prepared statement as an example:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='143'%20width='1850'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/886afe32-410a-0136-0267-0f7515a29063/4982cccb-0c6e-4919-b2cd-d821017bf48b/SQL%20syntax%20viz%20-%20Step%201.png?w=1850&h=143&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/886afe32-410a-0136-0267-0f7515a29063/4982cccb-0c6e-4919-b2cd-d821017bf48b/SQL%20syntax%20viz%20-%20Step%201.png?w=1850&h=143&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
 The update statement is supposed to charge a user account and has two parameters. `$1` is replaced with the charge, and `$2` is replaced with the account ID. [](https://read.readwise.io/read/01jxeypaxr2vyefvgscv6xjw3z)



the statement looks like this:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='143'%20width='1850'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/886afe32-410a-0136-0267-0f7515a29063/ed5dedce-d1c0-4475-9c3d-aefbccaff649/SQL%20syntax%20viz%20-%20Step%202.png?w=1850&h=143&auto=format&fit=crop)
 <img data-gatsby-image-ssr="" data-main-image="" style="opacity:0" sizes="(min-width: 1850px) 1850px, 100vw" decoding="async" loading="lazy" src="https://assets-eu-01.kc-usercontent.com:443/886afe32-410a-0136-0267-0f7515a29063/ed5dedce-d1c0-4475-9c3d-aefbccaff649/SQL%20syntax%20viz%20-%20Step%202.png?w=1850&amp;h=143&amp;auto=format&amp;fit=crop" srcSet="https://assets-eu-01.kc-usercontent.com:443/886afe32-410a-0136-0267-0f7515a29063/ed5dedce-d1c0-4475-9c3d-aefbccaff649/SQL%20syntax%20viz%20-%20Step%202.png?w=463&amp;h=36&amp;auto=format&amp;fit=crop 463w,https://assets-eu-01.kc-usercontent.com:443/886afe32-410a-0136-0267-0f7515a29063/ed5dedce-d1c0-4475-9c3d-aefbccaff649/SQL%20syntax%20viz%20-%20Step%202.png?w=925&amp;h=72&amp;auto=format&amp;fit=crop 925w,https://assets-eu-01.kc-usercontent.com:443/886afe32-410a-0136-0267-0f7515a29063/ed5dedce-d1c0-4475-9c3d-aefbccaff649/SQL%20syntax%20viz%20-%20Step%202.png?w=1850&amp;h=143&amp;auto=format&amp;fit=crop 1850w" alt=""/>
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
 The balance of the account with ID `acc-1337` will be decreased by 42. However, what happens when the first parameter is negative? The filled-in statement would look like this:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='851'%20width='1877'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/886afe32-410a-0136-0267-0f7515a29063/890ca22f-48a0-4beb-b6ba-6e0b7c980179/SQL%20syntax%20viz%20-%20Step%203.png?w=1877&h=851&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/886afe32-410a-0136-0267-0f7515a29063/890ca22f-48a0-4beb-b6ba-6e0b7c980179/SQL%20syntax%20viz%20-%20Step%203.png?w=1877&h=851&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
 The syntax has become ambiguous! [](https://read.readwise.io/read/01jxeypw9mqnmpk6x4zs0m1y4g)



To avoid this ambiguity, MySQL requires a whitespace after the `--` start-comment sequence. But what about other databases? [](https://read.readwise.io/read/01jxeyrwwj9zs9fx9x1n425zmy)



Looking Closer at PostgreSQL[](https://www.sonarsource.com/blog/double-dash-double-trouble-a-subtle-sql-injection-flaw/#looking-closer-at-postgresql)
 To answer this question, we examined PostgreSQL client libraries more closely because there are plenty of open source ones. [](https://read.readwise.io/read/01jxeysstsk66ra4964xysnjcc)



PostgreSQL supports two query modes: simple and extended. In the simple mode, an SQL string is sent to the database, and the result is returned. If there are user-controlled parameters in the query, the client has to insert them into the query string before sending it. On the other hand, there's the extended query mode that sends a prepared statement and its parameter values separately. This means that the values are never interpolated into the query because the database treats them separately, which in turn means that parameter values can never alter the syntax of a query. [](https://read.readwise.io/read/01jxeyv6rfqx1aexp3v9xwztvj)



When running in simple query mode, libraries have to interpolate parameter values into the query themselves. Let's take a look at how PgJDBC, the most popular PostgreSQL driver for Java, handled the query `SELECT 1-?` for a parameter value of `-1`. This is the query that is sent to the database:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='296'%20width='1624'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/886afe32-410a-0136-0267-0f7515a29063/1f1fe4ef-0e8c-4ade-b849-48ee76ca1895/sqli-query.png?w=1624&h=296&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/886afe32-410a-0136-0267-0f7515a29063/1f1fe4ef-0e8c-4ade-b849-48ee76ca1895/sqli-query.png?w=1624&h=296&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
 Looking at the result, we can see that PostgreSQL indeed parses the `--` sequence as the start of a line comment:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='123'%20width='930'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/886afe32-410a-0136-0267-0f7515a29063/38d3a06f-440e-4144-adf1-00533e3b49c4/sqli-result.png?w=930&h=123&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/886afe32-410a-0136-0267-0f7515a29063/38d3a06f-440e-4144-adf1-00533e3b49c4/sqli-result.png?w=930&h=123&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}} [](https://read.readwise.io/read/01jxeywq9ncz1cc7j60yk5swvk)



Gauging the Impact[](https://www.sonarsource.com/blog/double-dash-double-trouble-a-subtle-sql-injection-flaw/#gauging-the-impact)
 After confirming that it is possible to alter the syntax of a prepared statement by causing the creation of a line comment, we wanted to know if it's just possible to comment out parts of a query or if attackers could even inject new syntax. [](https://read.readwise.io/read/01jxeyxbd8909v8x26j0f7b2me)



We experimented with several queries until we realized another fact about PostgreSQL: multi-line string literals are supported! [](https://read.readwise.io/read/01jxeyxqywyc40xzjhy0sp911m)



Let's consider the example query from the beginning again:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='143'%20width='1850'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/886afe32-410a-0136-0267-0f7515a29063/4982cccb-0c6e-4919-b2cd-d821017bf48b/SQL%20syntax%20viz%20-%20Step%201.png?w=1850&h=143&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/886afe32-410a-0136-0267-0f7515a29063/4982cccb-0c6e-4919-b2cd-d821017bf48b/SQL%20syntax%20viz%20-%20Step%201.png?w=1850&h=143&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
 When a charge of `-1` and an account ID of `foo``**\n**``bar` are passed as the parameter values, the resulting interpolated query looks like this:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='207'%20width='1850'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/886afe32-410a-0136-0267-0f7515a29063/4ce330e6-0031-4f64-ae6a-3d2e4f89df2f/SQL%20syntax%20viz%20-%20Step%204.png?w=1850&h=207&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/886afe32-410a-0136-0267-0f7515a29063/4ce330e6-0031-4f64-ae6a-3d2e4f89df2f/SQL%20syntax%20viz%20-%20Step%204.png?w=1850&h=207&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
 When the PostgreSQL database parses this query, it will ignore the comment, resulting in this:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='207'%20width='1850'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/886afe32-410a-0136-0267-0f7515a29063/75b2f75b-5575-4dd1-acfe-0f6d4abf80ff/SQL%20syntax%20viz%20-%20Step%205.png?w=1850&h=207&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/886afe32-410a-0136-0267-0f7515a29063/75b2f75b-5575-4dd1-acfe-0f6d4abf80ff/SQL%20syntax%20viz%20-%20Step%205.png?w=1850&h=207&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}} [](https://read.readwise.io/read/01jxeyyhda6bfbw9ahfff840v1)



since the account ID is user-controlled, an attacker could provide a value that modifies the query without causing syntax errors. Here, it comes in handy for the attacker that PostgreSQL strings can be multi-line:
 ![](data:image/svg+xml;charset=utf-8,%3Csvg%20height='335'%20width='1850'%20xmlns='http://www.w3.org/2000/svg'%20version='1.1'%3E%3C/svg%3E)
 ![](https://assets-eu-01.kc-usercontent.com:443/886afe32-410a-0136-0267-0f7515a29063/a779d34e-6bc9-48d4-aff1-d2365995e3d3/SQL%20syntax%20viz%20-%20Step%206.png?w=1850&h=335&auto=format&fit=crop)
 ![](https://assets-eu-01.kc-usercontent.com:443/886afe32-410a-0136-0267-0f7515a29063/a779d34e-6bc9-48d4-aff1-d2365995e3d3/SQL%20syntax%20viz%20-%20Step%206.png?w=1850&h=335&auto=format&fit=crop)
 const t="undefined"!=typeof HTMLImageElement&&"loading"in HTMLImageElement.prototype;if(t){const t=document.querySelectorAll("img[data-main-image]");for(let e of t){e.dataset.src&&(e.setAttribute("src",e.dataset.src),e.removeAttribute("data-src")),e.dataset.srcset&&(e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset"));const t=e.parentNode.querySelectorAll("source[data-srcset]");for(let e of t)e.setAttribute("srcset",e.dataset.srcset),e.removeAttribute("data-srcset");e.complete&&(e.style.opacity=1,e.parentNode.parentNode.querySelector("[data-placeholder-image]").style.opacity=0)}}
 With this, it is clear that attackers can inject malicious SQL statements. Luckily, the requirements are quite high, and such queries are likely not very widespread. [](https://read.readwise.io/read/01jxeyznpbaf7h3vsr0hye7xdg)

