---
author: "ProjectDiscovery"
aliases: "Building a Fast One-Shot Recon Script for Bug Bounty"
tags: RW_inbox, readwise/articles
url: https://projectdiscovery.io/blog/building-one-shot-recon?__readwiseLocation=
date: 2025-05-05
summary: This article explains how to create a fast one-shot reconnaissance script for bug bounty testing. It covers essential steps like asset discovery, subdomain enumeration, and HTTP server scanning. The script helps organize and analyze data collected during the reconnaissance process.
---
# Building a Fast One-Shot Recon Script for Bug Bounty

![rw-book-cover](https://projectdiscovery.ghost.io/content/images/2024/11/Blog-Post--Building-a-Fast-One-Shot-Recon-Script-for-Bug-Bounty.png)

## Highlights


Introduction
In this article we are going to build a fast one-shot recon script to collect the bulk of the information we need to serve as a starting point for our bug bounty testing. [](https://read.readwise.io/read/01jtfrwvd8drhvsn3wv99txgw7)



many of the top application security testers and bug bounty hunters take advantage of the power of human-aided automation. Human-aided automation is where one is intellectually honest about the state of AI/ML in the current day, and builds a framework in a way that compliments both the strengths and weaknesses of computers and humans accordingly. [](https://read.readwise.io/read/01jtfrz3z50mag3yv27tvaydmj)



1. The Skeleton [](https://read.readwise.io/read/01jtfrzn0ptr5qxzehtaq6jnr1)



The skeleton will enable us to insert our code-logic in a modular way as well as take care of the file and directory organization. [](https://read.readwise.io/read/01jtfs0gha61jx6e6b6v6v7k3h)



edit scan.sh: and build your skeleton, this should:
• read and check that a valid scope directory exists
• create directories such as the scan directory for each scan
• calculate how long the scan took
• notify the user that the scan is done [](https://read.readwise.io/read/01jtfs3rg1d4njctdb6chywcw1)



2. Asset Discovery [](https://read.readwise.io/read/01jtfsh89tjm3e5h868r801pns)



Asset discovery often requires a human to review the information, this step may be lightly automated using a tool such as Uncover and the Whoxy API. [](https://read.readwise.io/read/01jtfsj0qm53j5cvbrr3kbhaef)



Root Domains
The first step in reconnaissance is asset discovery. Start by identifying what companies you would like to define as in-scope and obtain as many root domains as possible. [](https://read.readwise.io/read/01jtfsjw5frssvtfwf3t2xdh9d)



Censys / Shodan
Search for the Company Common Name on [Censys](https://search.censys.io/) or Shodan and you may identify IP addresses of assets that do not have existing DNS records. [](https://read.readwise.io/read/01jtfska00aba8tv4490ryneff)



Perform reverse DNS lookups on the IP’s you discover through these search engines and see if you can identify IPs, ASN’s, root domains, or other unlinked company owned assets. [](https://read.readwise.io/read/01jtfskw39w7pxgyps483wep3c)



3. Subdomain Enumeration [](https://read.readwise.io/read/01jtfskzmcgaebkrv6hmg2wsma)



Once you have gathered a list of your root domains, it is time to perform subdomain enumeration. [](https://read.readwise.io/read/01jtfsm7fknm0xng02fmbx9and)



DNS Bruteforcing
The first and oldest technique is DNS bruteforcing. DNS Bruteforcing has become exponentially faster than previous years. My go-to tools for DNS bruteforcing are puredns or shuffledns. [](https://read.readwise.io/read/01jtfsmkjzhr7fhtcah49ngwbn)



Shuffledns is a Golang wrapper for Massdns and makes it extremely easy to perform DNS bruteforcing at very high speed. [](https://read.readwise.io/read/01jtfsn1y5bx85svqhyv35g9xz)



We will add subfinder and shuffledns as our first two functions. [](https://read.readwise.io/read/01jtfsq1cgp7fqqbckn2x8txpw)



This is a good start, however, we need to account for wildcard DNS records. [](https://read.readwise.io/read/01jtfsv2w514yjbstpjcq5y72t)



A wildcard record may exist for: *.api.example.com [](https://read.readwise.io/read/01jtfsvne067jad5kx20sxv622)



As such, anything.api.example.com will resolve to the same addresses. This can lead to false positives or lead to us discovering subdomains that don't really exist [](https://read.readwise.io/read/01jtfsw997jrs3xy6d5j0fnzcv)



To remedy this, we can use the awesome wildcard detection that comes with Puredns. [](https://read.readwise.io/read/01jtfswh3wberhrxr10x7ep0x1)



To test this, we’re going to run puredns resolve on our existing subs domains. [](https://read.readwise.io/read/01jtfswym01ej1xqnmx3pxwqty)



We are going to add some more passive sources to our script, and then we should end up with something like this:
![](https://projectdiscovery.io/_next/image?url=https%3A%2F%2Flh6.googleusercontent.com%2FTjzInScPuon9SGZzrDM1aBua-3umO5n6Zu7tlSwXWXtlLNg7sjx0E4TLmX-rtmht256gBJ87l-tTRkmuSQyHAp5HecainP7dQx_EAqmkrKm-zaK_hrvY7gPsYP5eBNiqw7_6F5LUFP57KVVuUiS-mGiXK2Yv06tcaPKe72vt_ajQl0bspg0PGTJ5gw&w=3840&q=75)
If we run this scan with these added functions, it should generate the following files:
• dns.json
• ips.txt
• resolved.txt
• roots.txt
• subs.txt
In the next section, we will explore using these files to further enumerate. [](https://read.readwise.io/read/01jtfsysbznecffjp0tm256t1w)



4. HTTP Server Enumeration [](https://read.readwise.io/read/01jtfsz6s8jjd77bb66s1hqz95)



we will make use of nmap to scan the IPs that we have discovered through resolution of the discovered subdomains. [](https://read.readwise.io/read/01jtft171qxxz1x08ec1pm2djx)



If we want to just discover HTTP servers from the Nmap results, we can do that using an nmap parser, such as nmaptocsv. [](https://read.readwise.io/read/01jtft1m8zxrxatds5pdtr0c0q)



a handy tool called “tew”. [](https://read.readwise.io/read/01jtft2ambd8a1691xrcz0hvp4)



tew -x nmap.xml 2tew -x nmap.xml | httpx [](https://read.readwise.io/read/01jtft2j7s48e5ya58pt57pn49)



5. Solving the Vhosting Problem [](https://read.readwise.io/read/01jtft2t9cmywjapn04t41ahk2)



To solve this issue, I added a —vhost flag to tew which allows it to import DNS data from DNSx JSON output and resolve them accordingly. [](https://read.readwise.io/read/01jtft3t1dd66r7rd89ctc2yxt)



tew -x nmap.xml -dnsx dns.json —vhost | httpx -json -o http.json [](https://read.readwise.io/read/01jtft4c66efeacdpm7qrfjdyj)



![](https://projectdiscovery.io/_next/image?url=https%3A%2F%2Flh4.googleusercontent.com%2F2fOz7a8jphs0jfPLYa7Z5_Bgq-pHPcNqFLCI-xSgXbbk6i0h1X2rWC-hnbV1nFtE1qCqCX4AkQn0pxsS21NGH6Y61dIG6oac-Z3QNXYPnzabVsGM9blNXpQfaw0l40lKxQc3v9pekWhS_auhvyJ4ABLYzqBJZ0LZnbXm2ZKkOpvlAztd-yuV7FU0ag&w=3840&q=75)
As you can see, the hostnames from our DNSx output have been automatically resolved to the IP addresses and open ports from our Nmap XML output file. [](https://read.readwise.io/read/01jtft5hp0nnx2pr3tw2kebwj5)



Our one shot script is coming along nicely! Our scan functions should look like the following:
![](https://projectdiscovery.io/_next/image?url=https%3A%2F%2Flh3.googleusercontent.com%2Fwt1lYI633p9vkxIq-KvK6AiM3578Ev7TpywsWzsW9U5BlPycueF7hEfrscXEGac_QeDVbVJXFmnDVoCzKYcbt-ZYhDNadBO5Uk90JpYaDzYBH2qy8GdOlFzWcJo32Z-fJg_sJXz0cJzVCBvGS5RaQ0a_XVzKOlz0EZ4Xa7qR5OvYbwPrldfiZnNSaA&w=3840&q=75) [](https://read.readwise.io/read/01jtft6v6ck7gqyqqra0v83pjr)



6. HTTP Crawling
The next piece of our script will be crawling our discovered URLs. [](https://read.readwise.io/read/01jtft70r6r8yxvm26cnbg3g1k)



We will be using Gospider for crawling, however, we could easily use something else such as hakrawler or xnLinkFinder. [](https://read.readwise.io/read/01jtft7knbz7ms51ek9sbyzw96)



gospider -S http.txt —json | grep “{” | jq -r ‘.output’ [](https://read.readwise.io/read/01jtft7pxc90qqc54vf9pv709v)



7. HTTP Response capture [](https://read.readwise.io/read/01jtft7zdy68jrg5fpawpmcc8q)



Inspired by Tomnomnom, capturing the HTTP responses from HTTP index pages can serve as a good starting point for hunting. [](https://read.readwise.io/read/01jtft8dtygeh906hbatqgv9ap)



HTTPx incorporates this feature and we can modify our script to store responses as it performs its enumeration. [](https://read.readwise.io/read/01jtft8sprwqbpn3dd1c25d2yd)



![](https://projectdiscovery.io/_next/image?url=https%3A%2F%2Flh3.googleusercontent.com%2FhygJNH6RkNZsAqA8PA2rNiNyXJlRuTCIewrHdyTjhx201BQnMioW5ZQifGXSSVr0jIiSj_jvUjV2Ka-lBWPwkiDI-O-oqc32rn_YRjbybuxma7fcOdLT5neLhCnhoKElgWuUy3MGl1_FGbP2ZVvdmuICBO_5ybdC2CrA9IwNbhsM4andUjOOoZYYzA&w=3840&q=75)
This will create a directory called “responses”. Inside, will be all of the HTTP responses.
![](https://projectdiscovery.io/_next/image?url=https%3A%2F%2Flh5.googleusercontent.com%2FhbwQNAxQEm5Jo1kQ4Ha5dFjFk19DU4eb8-byZ8pvZQor65Qnu2dvCuRpeJgyZLCfcxU_GwyfngEP8dF5KiHxK8bGrtzZZRQJbDQTSTpR3yj27reweaGWwYLknG7O9MF-rhPcSTGT48cHlusmV-JXCkXnSiBQiPUsgC6Imz6gA8Y2QgEHaP2dgJYe0A&w=3840&q=75) [](https://read.readwise.io/read/01jtft99q1yagvb08bx7r0hhs6)



After updating our scan script, our scan functions should look like this:
![](https://projectdiscovery.io/_next/image?url=https%3A%2F%2Flh5.googleusercontent.com%2FOTQUDp7oZqrA3JHWeoDGv5kOnANx7jO4wXj3yc0_Rq-XUOK09h0yzVOtgZramWiQ5i7EEyjtKnRXU2ZQCWG0rmldHaXHls7_W8Ni1Ej7qRkBnGJehVSXqy9TYahL2H3amlib6PK3yveQcVn_MCqDm_r2us1vsb0Jg5KbtV8CErPMEO6R2DM8hmRJAA&w=3840&q=75) [](https://read.readwise.io/read/01jtfta84xrm8dpdjk5f3tywbe)



8. Javascript Scraping [](https://read.readwise.io/read/01jtftadkzydfg6bj8h453x730)



cat crawl.txt | grep “\.js” | httpx -sr -srd js [](https://read.readwise.io/read/01jtftaww71kef84ke1w56q569)



Now that we have a directory full of javascript files, we can explore these using grep or trufflehog. [](https://read.readwise.io/read/01jtftbcdwyam95dmen25h8szf)



This is how the scan function should look:
![](https://projectdiscovery.io/_next/image?url=https%3A%2F%2Flh6.googleusercontent.com%2F5i04sXvMpFLkwgcwaDKuHnOUo5cjnrrbsidRA1G3P95I-M5s6ZABTlFNuA80uOgaujGQkVHH_21ZnhILPEJ4szcOCK9inPAbAlX9IanotdLfZBqKmmo9ENmny_wAn7q3POnIE0YH3I-9Pu0zPD6cCWG2GBP3leEpqYCt0w4_vCdUSr--zvPcYh7Q8w&w=3840&q=75) [](https://read.readwise.io/read/01jtftbktar9axwp2jfjtntjh8)



Exploring the collected data
After our scan is complete, we will see some files generated akin to the following:
![](https://projectdiscovery.io/_next/image?url=https%3A%2F%2Flh3.googleusercontent.com%2Fck6ljmW9rGCtMMyij_nXNkUuxcCwZGsvgJA5ox0iGnExHwbu1SZTtXI6kfLqPPzBV-0QzVrPsJGJb8gkhJSKDPNMskvrR4keFEyXrm3D1cnrtJ_OB6NTaRoXGMrePn0Zvsfkk8kQ3x1Dn7MAhK4adbpZbLui8aPVeixYZpFsKN3cGTN3_43ApMRqRw&w=3840&q=75) [](https://read.readwise.io/read/01jtftcnycqyxbf0j4qs1fnbgz)



Hack like Tomnomnom [](https://read.readwise.io/read/01jtftdfe043p8csehdy9mj4ky)



I will cover some of those methods here. This article is also a helpful reference: [https://mavericknerd.github.io/knowledgebase/tomnomnom/session1/](https://mavericknerd.github.io/knowledgebase/tomnomnom/session1/)
[https://tomnomnom.com/talks/wwwww.pdf](https://tomnomnom.com/talks/wwwww.pdf) [](https://read.readwise.io/read/01jtftdzcq8s9svhvd4xwhbhf8)



To recursively grep through your data, the following grep command will show line numbers and filenames as well as be case insensitive. [](https://read.readwise.io/read/01jtftg50c54dva1fm3hz91hpx)



grep -Hrni ‘set-cookie’ —-color=always | batcat [](https://read.readwise.io/read/01jtftg8r7x2816vhngsr0sh6q)



gf [](https://read.readwise.io/read/01jtftgwew7b6w338nw8n8yywp)



Gf allows you to define your grep patterns in JSON files and refer to them with an alias. A list of example rules can be found here: https://github.com/tomnomnom/gf/tree/master/examples [](https://read.readwise.io/read/01jtfth8vmznj6qfz45kqkx8dv)



gf aws-keys [](https://read.readwise.io/read/01jtfthdq6ya3yqb13vg03gp3w)



tok [](https://read.readwise.io/read/01jtfthmb2phwz69yc83a2awq9)



Tok is a hugely powerful tokenizer tool, this means that it extracts words from given files and removes special characters and spaces. [](https://read.readwise.io/read/01jtfthys5bfm1c4ph1rz607g0)



find -type f | tok | sort | uniq -c | sort -rn | head -n 40 [](https://read.readwise.io/read/01jtftj4vyme6j4d1jg2z6bz32)



Using the above one-liner, we extract all the words from the HTTP responses we stored earlier and we sort them in ascending order of frequency of occurrence. [](https://read.readwise.io/read/01jtftk0nhgp6y0vm9xp5bv9bb)



These words can be added to a wordlist for further enumeration, or grepped for to identify context. [](https://read.readwise.io/read/01jtftk9vsktabf9r4sczp010m)



find -type f | tok | sort -u -V | vim - [](https://read.readwise.io/read/01jtftkdh849449zhp1v6paysc)



qsreplace + ffuf [](https://read.readwise.io/read/01jtftkq9esxehj7txdvw2qxe7)



Using qsreplace and ffuf, we can look for vulnerabilities such as path traversal or SQL Injection vulnerabilities. [](https://read.readwise.io/read/01jtftm0ae40cf299rfa4fkzx0)



Simply set qsreplace to your payload of choice and then set the regex match (-mr ‘REGEX’) to a relevent regex. [](https://read.readwise.io/read/01jtftmhf2s9z8x7cy62jk9qjm)



function fuzz() { 2 payload=“$1” 3 regex=“$2” 4 5 cat crawl.txt | grep “?” | qsreplace “$payload” | ffuf -u “FUZZ” -w - -mr “$REGEX” 6} 7 8fuzz “../../../../etc/passwd” “^root:” 9 10# Or: 11 12cat crawl.txt | grep "?" | qsreplace ../../../../etc/passwd | ffuf -u 'FUZZ' -w - -mr '^root:' [](https://read.readwise.io/read/01jtftmpwvbnwpa9pfcpzh87bb)

