---
author: "@7h3h4ckv157 on Twitter"
aliases:
  - Tweets From 7h3h4ckv157
  - HexHTTP
tags:
  - readwise/tweets
  - tools
url: ?__readwiseLocation=
created: 2025-03-29
---
# Tweets From 7h3h4ckv157

![rw-book-cover](https://pbs.twimg.com/profile_images/1837821520875343872/dIYON0Hz.jpg)


[HExHTTP](https://github.com/c0dejump/HExHTTP) is a tool designed to perform tests on HTTP headers and analyze the results to identify vulnerabilities and interesting behaviors.
 Source: https://t.co/h9IqdkkFdQ
 Author: c0dejump 
 ![](https://pbs.twimg.com/media/GiCCuSebsAADsMg.jpg) [](https://twitter.com/7h3h4ckv157/status/1882648384475213933) ^d53888

## Install

```sh
git clone https://github.com/c0dejump/HExHTTP.git
```

### Usage and run

```sh
Usage: hexhttp.py [-h] [-u URL] [-f URL_FILE] [-H CUSTOM_HEADER] [-A USER_AGENT] [-F] [-a AUTH] [-b] [-hu HUMANS] [-t THREADS] [-l LOG] [-L LOG_FILE] [-v] [-p CUSTOM_PROXY]

HExHTTP is a tool designed to perform tests on HTTP headers.

options:
  -h, --help            show this help message and exit
  -u, --url URL         URL to test [required]
  -f, --file URL_FILE   File of URLs
  -H, --header CUSTOM_HEADER
                        Add a custom HTTP Header
  -A, --user-agent USER_AGENT
                        Add a custom User Agent
  -F, --full            Display the full HTTP Header
  -a, --auth AUTH       Add an HTTP authentication. Ex: --auth admin:admin
  -b, --behavior        Activates a simplified version of verbose, highlighting interesting cache behaviors
  -hu, --humans HUMANS  Performs a timesleep to reproduce human behavior (Default: 0s) value: 'r' or 'random'
  -t, --threads THREADS
                        Threads numbers for multiple URLs. Default: 10
  -l, --log LOG         Set the logging level (DEBUG, INFO, WARNING, ERROR, CRITICAL)
  -L, --log-file LOG_FILE
                        The file path pattern for the log file. Default: logs/
  -v, --verbose         Increase verbosity (can be used multiple times)
  -p, --proxy CUSTOM_PROXY
                        Add a custom proxy. Ex: http://127.0.0.1:8080


docker run --rm -it --net=host -v "$PWD:/hexhttp/" hexhttp:latest -u 'https://target.tld/'

# Scan only one domain
» ./hexhttp.py -u 'https://target.tld/'

# Scan a list of domains with behavior feature
» ./hexhttp.py -b -f domains.lst

# if the application is very sensitive (waf or not)
» ./hexhttp.py -u 'https://target.tld/' -hu r

# Add custom User-Agent
» ./hexhttp.py -u 'https://target.tld/' --user-agent "User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64) Firefox/123.0-BugBounty"

# Use a custom Header and authentication
» ./hexhttp.py --header 'Foo: bar' --auth 'user:passwd' -u 'https://target.tld/' 

# Loop on domains, grep for vulnerabilities only and send result with notify (from projectdiscovery)
» for domain in $(cat domains.lst); do ./hexhttp.py -u "$domain" | grep -Eio "(INTERESTING|CONFIRMED)(.*)PAYLOAD.?:(.*){5,20}$" | notify -silent; done

```