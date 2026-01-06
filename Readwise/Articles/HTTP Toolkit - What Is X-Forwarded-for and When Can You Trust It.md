---
author: HTTP Toolkit
aliases:
  - What Is X-Forwarded-for and When Can You Trust It?
tags:
  - readwise/articles
url: https://httptoolkit.com/blog/what-is-x-forwarded-for/
created: 2024-08-20
---
# What Is X-Forwarded-for and When Can You Trust It?

![rw-book-cover](https://httptoolkit.tech/apple-touch-icon.png)

## Highlights


> The X-Forwarded-For (XFF) HTTP header provides crucial insight into the origin of web requests. The header works as a mechanism for conveying the original source IP addresses of clients, and not just across one hop, but through chains of multiple intermediaries. This list of IPv4 and IPv6 addresses is helpful to understand where requests have really come from in scenarios where they traverse several servers, proxies, or load balancers.
> [View Highlight](https://read.readwise.io/read/01hsk4fvqa8ambc0w2bp0xpvkc)



> What is X-Forwarded-For used for?
>  Knowing the original source & processing path of requests has a whole load of use cases depending on what you're building.
>  • **User Authentication:** Use the header information to ensure that login attempts originate from recognized and authorized locations, and flag the login as suspect if not, triggering 2FA checks.
>  • **Load Balancing:** Evenly distribute incoming traffic across servers, to ensure optimal performance during busy periods.
>  • **Data localization:** European Union, Brazil, China and others have privacy laws about where data can be kept, and this can help identify those users who need special treatment.
>  • **Geographic Content Delivery:** CDNs use `X-Forwarded-For` to determine the user's location and serve content from the nearest server to reduce latency.
>  • **Access Control and Security:** Websites use `X-Forwarded-For` to verify the legitimacy of requests and implement access controls based on IP addresses, like a corporate intranet that only allows access to certain resources for employees coming from recognized office IP ranges.
>  • **Web Application Firewalls (WAF):** Filter incoming traffic, blocking suspicious requests from a known malicious IP address listed in `X-Forwarded-For`.
>  • **Fraud Prevention:** Financial institutions use `X-Forwarded-For` to detect and prevent fraudulent activities based on user location, e.g. identifying an unusual login attempt from a location that is inconsistent with the user's typical access patterns.
>  • **API Rate Limiting:** APIs use `X-Forwarded-For` to enforce rate limiting on a per-client basis. An API provider limits the number of requests from a specific IP address within a given time frame to prevent abuse.
>  • **Localized Advertising:** Ad platforms use `X-Forwarded-For` to customize and target ads based on the user's geographical location.
>  • **Logging and Analytics:** log to analyze user traffic patterns and behaviors for statistical purposes, like the geographical distribution of users over a specific time period.
> [View Highlight](https://read.readwise.io/read/01hsk4s53nrheev6hbzqaa7rbw)

