---
author: "blackbird-eu"
aliases: "Creating Custom Wordlists for Bug Bounty Targets: A Complete Guide"
tags: RW_inbox, readwise/articles
url: 
date: 2025-02-04
---
# Creating Custom Wordlists for Bug Bounty Targets: A Complete Guide

![rw-book-cover](https://blog.intigriti.com/icon.svg)

## Highlights


Everyone understands the importance of custom wordlists in bug bounties, and how they can be deployed in targeted bruteforcing attacks to help discover new hidden endpoints.
[View Highlight](https://read.readwise.io/read/01jk8r30gg38hyjtqjrvhg1y7f)



A custom wordlist isn't only about collecting random targeted keywords, it also consists of the following 3 main lists:
 • Company-specific keywords
 • Technology-specific keywords
 • Generic and commonly occurring keywords
[View Highlight](https://read.readwise.io/read/01jk8r38jvah68h1z50xhrg98j)



Company-specific terms are keywords used within the app to define endpoint names, parameters, and features.
[View Highlight](https://read.readwise.io/read/01jk8r3w8z4vk6n95mehp0yper)



Technology-specific keywords are based on the tech stack and frameworks that a target is using. These keywords are crucial as they can help you identify files and application routes on the server that are specific to a certain technology.
[View Highlight](https://read.readwise.io/read/01jk8r4kg40qdyqy41az5wphz3)



Finally, a good custom wordlist also consists of commonly occurring keywords. These are often keywords that appear across many web applications, regardless of the company or technology stack. Think of the `/api` endpoint or the `/assets` directory, for example.
[View Highlight](https://read.readwise.io/read/01jk8r586c7gyr27khq8e1dd98)



Step 1: Generating a company-specific wordlistExtracting in-page keywords
[View Highlight](https://read.readwise.io/read/01jk8r5rcca9fev3zmyzsdq983)



$ cewl https://example.com --header "Cookie: PHPSESSID=7a9b4c2d8e3f1g5h6i7j8k9l0m1n2o3p" -d 5 -m 4
[View Highlight](https://read.readwise.io/read/01jk8r6ghc4tsbt6hdk8r4gbx3)



Extracting URL keywords
 Another way to build a list of target-specific keywords is by tokenizing URLs. Using this method, we can transform our list of URLs gathered from our proxy intercepting tool into keywords that we can use for content discovery.
[View Highlight](https://read.readwise.io/read/01jk8r7fz2yp0ww86xgg9wnc8q)



Here's a quick usage example of Tok:
 $ cat /path/to/urls.txt | tok
[View Highlight](https://read.readwise.io/read/01jk8r81wha9x5n3wrhffcv06e)



Extracting keywords from JavaScript files
 We can now even go a step further and use a tool like [getjswords](https://github.com/m4ll0k/BBTz/blob/master/getjswords.py) to generate even more custom keywords derived from JavaScript files.
[View Highlight](https://read.readwise.io/read/01jk8r8efd8fc2xfh534k4yc1w)



Here's a quick usage guide on using getjswords:
 $ cat /path/to/js-urls.txt | python3 getjswords.py
[View Highlight](https://read.readwise.io/read/01jk8r8mpsh3s28aq1agj83652)



**Step 2: Crafting a technology-specific wordlist**
 Technology-specific wordlists are helpful to add coverage for the underlying technology stack that your target consists of. For this reason, we must fingerprint all technologies used by our target and use public wordlists or curate our own.
[View Highlight](https://read.readwise.io/read/01jk8r9fsgzq1x8pp0zh22n0zb)



Once you've fingerprinted all used technologies, [browse through SecList's wordlist files](https://github.com/danielmiessler/SecLists/tree/master/Discovery/Web-Content) and select the wordlists that match your fingerprinted technologies and services.
[View Highlight](https://read.readwise.io/read/01jk8ra1m2vgaass2n5me0feg8)



Step 3: Including commonly occurring keywords
 The last step in generating our custom wordlist is adding commonly occurring keywords and terminologies that are not bound to a specific target.
[View Highlight](https://read.readwise.io/read/01jk8rahn7qrb2t25ahqh9tb6p)



examples:
 • Assets folders (`assets`, `storage`, `files`, ...)
 • API endpoints (`api`, `v1`, `graphql`, ...)
 • Authentication-related paths (`login`, `signin`, `oauth`, `sso`, ...)
 • Admin and other authentication panels (`admin`, `dashboard`, `console`, ...)
 • Profile and account components (`profile`, `settings`, `account`, ...)
 • Development paths (`dev`, `staging`, `test`, `profiler`, `debug`, ...)
 • Common parameters (`id`, `userId`, `page`, ...)
 [JHaddix's all.txt is a great starting point](https://gist.github.com/jhaddix/86a06c5dc309d08580a018c66354a056),
[View Highlight](https://read.readwise.io/read/01jk8rb90tgyysdngndqedjxa5)



Step 4: Combining all lists
 The latest step consists of combining all the lists that you've generated so far together and using them for content discovery.
[View Highlight](https://read.readwise.io/read/01jk8rbpgx1eztjjcmj7xkrhc0)



**Use a tool like** [**wl**](https://github.com/s0md3v/wl) **to convert your wordlist to match the naming convention of your target!**
[View Highlight](https://read.readwise.io/read/01jk8rc883tyykbp54da1e0dbc)

