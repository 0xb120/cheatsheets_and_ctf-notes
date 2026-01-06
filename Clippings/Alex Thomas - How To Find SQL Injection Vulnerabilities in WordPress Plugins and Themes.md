---
title: How To Find SQL Injection Vulnerabilities in WordPress Plugins and Themes
source: https://www.wordfence.com/blog/2025/08/how-to-find-sql-injection-vulnerabilities-in-wordpress-plugins-and-themes/#finding-sqli-vulnerabilities-in-wordpress-plugins-and-themes
author:
  - Alex Thomas
published: 2025-08-06
created: 2025-08-19
description: Learn how to find SQL Injection vulnerabilities in WordPress plugins and themes for the Wordfence Bug Bounty program.
tags:
  - clippings/articles
aliases:
  - How To Find SQL Injection Vulnerabilities in WordPress Plugins and Themes
---
# How To Find SQL Injection Vulnerabilities in WordPress Plugins and Themes

![](https://www.wordfence.com/wp-content/uploads/2025/08/FeaturedImage_Wordfence_262.06.png)

> [!summary]
> > This article provides a comprehensive guide on finding SQL Injection (SQLi) vulnerabilities in WordPress plugins and themes, primarily for bug bounty hunters.
> It explains that SQLi, a classic vulnerability (CWE-89), persists in WordPress due to developers not using the core API functions correctly or for complex use-cases, making it the fourth most common vulnerability in 2024.
> The most common impact is data exfiltration, and direct RCE is rare.
> The guide details different types of SQLi: In-band (Error-based, UNION-based) and Blind (Boolean-based, Time-based, Inferential Error-based), along with Second-Order SQL Injection.
> It emphasizes a general approach involving identifying database interaction points (sinks), tracing user-supplied input (sources), and checking for inadequate sanitization or query preparation.
> Key focus areas for static analysis include searching for `$wpdb` methods and `esc_sql()`, with a specific regex provided.
> Common mistakes leading to SQLi are direct concatenation of user input, incorrect assumption that XSS sanitization functions protect against SQLi, incorrect usage of `$wpdb->prepare()`, and lack of type casting for numeric values.
> The article also explains WordPress's `wp_magic_quotes()` protection and its limitations, especially concerning `php://input` and REST API endpoints which can bypass or undo this protection.
> Real-world examples from CVEs (Ultimate Member, Calendar, Duplicate Page, LearnPress) illustrate these vulnerable patterns.
> A step-by-step methodology for SQLi hunting is outlined: target selection (using WPDirectory.net), static analysis setup (VS Code regex search), creating a hit list, tracing data flow, dynamic verification with debugging, payload crafting, and diligent documentation.
> Common SQLi payloads are listed for time-based, boolean-based, UNION-based, and error-based attacks, including context-specific examples for shortcodes, JSON/Base64, ORDER BY clauses, AJAX actions, and REST API.
> Finally, it stresses the importance of writing excellent reports and focusing on high-impact vulnerabilities (unauthenticated or low-privilege access) for higher rewards in bug bounty programs like Wordfence's.

## Finding SQLi Vulnerabilities in WordPress Plugins and Themes

To find [SQL Injection](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/SQL%20Injection.md) vulnerabilities, you’ll need to know what to search for in plugin or theme code. As mentioned in the introduction, [Wordpress](../Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/Wordpress.md) core provides a number of database functions through its API. These functions are made available through the `$wpdb` class.

- Using your Integrated Development Environment (IDE), you can start by searching plugin or theme code for `$wpdb`.

- You can also search for `esc_sql()`, which is a function designed to escape quotes within strings.

- Finally, you can search for other keywords that are typically used in SQL queries like `SELECT`, `INSERT`, `ORDER BY`, `UPDATE`, `DELETE`, and so on.

However, note that earlier in this post, we mentioned that the **ideal place** to start is by **identifying sinks**.

We can take this information and develop a **regular expression** designed to get us an ideal starting search for SQLi:

`\$wpdb->(?:query|``prepare``|get_var|get_row|get_col|get_results)\s*\(|\besc_sql\s*\(`

### Common Mistakes and Vulnerability Patterns

#### 1\. Direct Concatenation of User Input into SQL Queries

The most common and dangerous pattern is **directly concatenating user-supplied input** into SQL queries without proper escaping or parameterization. This occurs when developers build dynamic queries using string concatenation.

#### 2\. Incorrect Assumption that XSS Sanitization Functions Protect Against SQL Injection

Developers might incorrectly assume that WordPress sanitization/escaping functions designed to prevent XSS (like `sanitize_text_field()`, `esc_html()`, etc.) will also protect against SQL injection or any vulnerabilities that is derived from a lack of input validation or sanitization. This is a fundamental misunderstanding.

#### 3\. Incorrect Usage of $wpdb->prepare()

Even when developers attempt to use `$wpdb->prepare()` correctly, they often make mistakes that render it ineffective.

#### 4\. Lack of Type Casting for Numeric Values

When dealing with numeric parameters, developers often fail to cast them to integers, leaving them vulnerable to SQL injection even in numeric contexts. Note that this would be rectified if `$wpdb->prepare()` was used with the `%d` placeholder.

### Understanding WordPress’s wp\_magic\_quotes() Protection

**WordPress automatically applies `wp_magic_quotes()` to superglobal data**, which escapes quotes. This means queries like the following are **NOT vulnerable** because an injection attempt like `1' OR 1=1--` becomes `1\' OR 1=1--`, resulting in the safe query: `SELECT * FROM wp_posts WHERE ID = '1\' OR 1=1--'`.

However, **this protection is limited to**:

- String contexts where values are wrapped in quotes

- Direct superglobal access (not processed/passed through functions)

**It does NOT protect against:**

- Numeric contexts without quotes: `WHERE ID = $post_id`

- ORDER BY clauses: `ORDER BY $column_name`

- Column/table names: `SELECT $column FROM $table`

- LIMIT clauses: `LIMIT $limit`

- Complex data flows where input is processed before use