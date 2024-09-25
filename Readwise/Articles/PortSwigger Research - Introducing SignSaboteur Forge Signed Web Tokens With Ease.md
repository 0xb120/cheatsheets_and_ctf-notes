---
author: PortSwigger Research
aliases:
  - "Introducing SignSaboteur: Forge Signed Web Tokens With Ease"
tags:
  - readwise/articles
url: https://portswigger.net/research/introducing-signsaboteur-forge-signed-web-tokens-with-ease
date: 2024-08-22
---
# Introducing SignSaboteur: Forge Signed Web Tokens With Ease

![rw-book-cover](https://portswigger.net/cms/images/78/6b/2275-twittercard-twitter-gr.png)

## Highlights

# SignSaboteur
> SignSaboteur is a Burp Suite extension for editing, signing, verifying, and attacking signed tokens. It supports different types of tokens, including Django, Flask, and Express.
> [View Highlight](https://read.readwise.io/read/01j5x21mpynfjzge7j68ndr3dd)
> #tools 


> The extension provides automatic detection and in-line editing of tokens within HTTP request / response pairs and WebSocket messages, signing of tokens and automation of brute force attacks.
> [View Highlight](https://read.readwise.io/read/01j5x223e159n2ysvtb351yfvf)



> You can modify the signed tokens in the Proxy and Repeater message editors. There are a number of built-in handy editors for JSON, timestamps and HEX strings.
> [View Highlight](https://read.readwise.io/read/01j5x2297jebjfhh0cz9zax39v)



# Methodology
>  To exploit a signed token, you usually need to discover the secret key. These may be disclosed in source code, configuration files, documentation pages, and error messages.
>  These papers provide a good overview of the different attacks:
>  - [Authentication Bypass in Apache Airflow CVE-2020-17526](https://ian.sh/airflow)
>  - [Baking Flask cookies with your secrets](https://blog.paradoxis.nl/defeating-flasks-session-management-65706ba9d3ce)
>  - [Redash Insecure default configuration](https://github.com/getredash/redash/security/advisories/GHSA-g8xr-f424-h2rv)
>  - [Remote Code Execution on a Facebook server](https://blog.scrt.ch/2018/08/24/remote-code-execution-on-a-facebook-server/)
>
>  To detect and exploit signed web tokens, we recommend the following methodology:
>  - Observe request/response cookie, url, body parameters for potential signed tokens: base64 encoded or hex strings of known length common for HMAC
>  - If a signed web token contains a timestamp, observe whether the signature changes when the token expires
>  - Try to identify any frameworks the target is using
>  - If the framework was identified, search for common secret keys and salts, for example hardcoded values or configuration files, including docker environment variables
>  - Update your wordlists with found information
>  - Try to brute force the token with different derivation techniques
>  - If you discover the secret key try the following attacks:
>  - If the token structure is known, change id to high privilege account, usually **1**
>  - If the token structure is unknown, brute force parameters names with a commonly used list: **id**, **user_id**, **username**, etc. Note, that SignSaboteur extension supports some of the mentioned techniques
>  - If you couldn't find the secret key, try to re-sign a message with a default key and send it to the application. If the server accepts your forged message that means that a vulnerability exists. Follow the steps mentioned for a known secret key.
>  - If the server accepts your forged message, visit all pages with a new token and also try to get access to administrative functionality.
> [View Highlight](https://read.readwise.io/read/01j5x2an4djqxpp1d81tmxgs50)



# Small demo
>  This short GIF demonstrates how to find the unknown secret key of a Flask test application, modify the session token, and re-sign it.
> [View Highlight](https://read.readwise.io/read/01j5x2dk2jc76pb3b6207rffs9)

