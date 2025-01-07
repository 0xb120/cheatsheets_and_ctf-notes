---
author: Admin
aliases:
  - "When WAFs Go Awry: Common Detection & Evasion Techniques for Web Application Firewalls"
tags:
  - readwise/articles
url: https://www.mdsec.co.uk/2024/10/when-wafs-go-awry-common-detection-evasion-techniques-for-web-application-firewalls/
date: 2024-10-30
---
# When WAFs Go Awry: Common Detection & Evasion Techniques for Web Application Firewalls

![rw-book-cover](https://www.mdsec.co.uk/wp-content/uploads/2019/11/tim-van-der-kuip-CPs2X8JYmS8-unsplash.jpg)

## Highlights


techniques for identifying WAFs can be found at the comprehensive [Awesome WAF](https://github.com/0xInfection/Awesome-WAF) resource from 0xInfection.
> [View Highlight](https://read.readwise.io/read/01jbewrr838ymekken48s5h4mg)



## Fuzzing
 A common approach to determining the payloads which are (and are not) being blocked by a WAF solution is fuzzing. This approach involves testing a set of payloads against the WAF, for example those from [SecLists](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing), [FuzzDB](https://github.com/fuzzdb-project/fuzzdb/tree/master/attack), and [Payloads All The Things](https://github.com/swisskyrepo/PayloadsAllTheThings), and observing the WAF’s responses.
 All responses from the different payloads fuzzed should be recorded and analysed to determine what is (and is not) being blocked by the WAF.
 However, this approach runs the risk that the source IP address for the fuzzing might be blocked by the WAF, either temporarily or permanently. Using the [IP Rotate](https://portswigger.net/bappstore/2eb2b1cb1cf34cc79cda36f0f9019874) Burp Suite extension, random User-Agent strings, and dynamic request latency can be helpful to avoid such blocks.
> [View Highlight](https://read.readwise.io/read/01jbewsb18m0hrae36qf3yp6d7)



## Reversing Regexes
 Perhaps the most efficient method for bypassing WAFs is reversing the regular expressions (or regexes) and rules which are used to detect common attacks.
> [View Highlight](https://read.readwise.io/read/01jbewt1zr46hhfpq9p1jb9t8n)



More techniques can be found at the comprehensive [Awesome WAF](https://github.com/0xInfection/Awesome-WAF) resource from 0xInfection.
> [View Highlight](https://read.readwise.io/read/01jbewtf8ehcje8fgw0yttjz4g)



## Obfuscation and Encoding Techniques
> [View Highlight](https://read.readwise.io/read/01jbewtavdv8qezavvk85d6b1v)



### Alternative Character Sets
> [View Highlight](https://read.readwise.io/read/01jbewvbn3k0xy7ay5zxdkzcxh)



It might be possible to obfuscate payloads using a character encoding which is not supported by the WAF, but which the web server can interpret correctly (Burp Suite’s [Hackvertor](https://portswigger.net/bappstore/65033cbd2c344fbabe57ac060b5dd100) extension can be used to do this).
> [View Highlight](https://read.readwise.io/read/01jbewvjxb562sb0wajb6v7h0k)



### Unicode Compatibility Bugs
 Unicode compatibility bugs can also be used to evade detection by a WAF by changing the charset parameter or `Accept-Charset` header to a higher Unicode encoding (e.g., UTF-32) which is then decoded by the browser using a lower Unicode encoding (e.g., UTF-8).
> [View Highlight](https://read.readwise.io/read/01jbewvst5wq1gyn3bqy9ka971)

