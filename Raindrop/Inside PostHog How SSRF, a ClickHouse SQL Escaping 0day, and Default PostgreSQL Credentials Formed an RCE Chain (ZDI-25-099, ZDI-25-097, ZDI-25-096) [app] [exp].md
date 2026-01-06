---
raindrop_id: 1494730902
raindrop_highlights:
  6957c3004a1bc6fca9844490: 3d4b5fe24c76c265955565cc17e7fc7c
title: "Inside PostHog: How SSRF, a ClickHouse SQL Escaping 0day, and Default PostgreSQL Credentials Formed an RCE Chain (ZDI-25-099, ZDI-25-097, ZDI-25-096) [app] [exp]"
description: "null"
source: https://mdisec.com/inside-posthog-how-ssrf-a-clickhouse-sql-escaping-0day-and-default-postgresql-credentials-formed-an-rce-chain-zdi-25-099-zdi-25-097-zdi-25-096/
created: 1766042692631
type: article
tags:
  - _index
---
# Inside PostHog: How SSRF, a ClickHouse SQL Escaping 0day, and Default PostgreSQL Credentials Formed an RCE Chain (ZDI-25-099, ZDI-25-097, ZDI-25-096) [app] [exp]

![](https://mdisec.com/wp-content/uploads/2025/12/posthog-e1765557833411.webp)

> [!summary]
> null





The Following request is being send to the test_slack_webhook endpoint. As you can see in the response, localhost is not allowed to be added as a webhook endpoint.
>
>POST /api/user/test_slack_webhook/ HTTP/2
>Host: us.posthog.com
>Accept-Encoding: gzip, deflate, br
>Priority: u=1, i
>
>{
>  "webhook": "http://localhost/"
>}
