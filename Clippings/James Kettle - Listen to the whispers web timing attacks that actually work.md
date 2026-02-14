---
title: "Listen to the whispers: web timing attacks that actually work"
source: "https://portswigger.net/research/listen-to-the-whispers-web-timing-attacks-that-actually-work"
author:
  - "James Kettle"
published: 2024-08-07
created: 2026-02-08
description: "Websites are riddled with timing oracles eager to divulge their innermost secrets. It's time we started listening to them. In this paper, I'll unleash novel attack concepts to coax out server secrets"
tags:
  - "clippings/articles"
  - "_inbox"
---
# Listen to the whispers: web timing attacks that actually work

![](https://portswigger.net/cms/images/39/7f/e41b-twittercard-listen-to-the-whispers-twitter.png)

> [!summary]+
> > This article, \"Listen to the whispers: web timing attacks that actually work,\" by James Kettle, argues that web timing attacks, often considered theoretical, are now practical and powerful due to advancements.
>
> The key innovation is the \"single-packet attack\" (using HTTP/2), which eliminates network jitter by sending two requests in one TCP packet and observing response order. This technique is refined by using an extra ping frame to mitigate the \"sticky request-order problem\" and make attacks portable across targets.
>
> The author details how to maximize signal and minimize server noise for effective attacks, leveraging short code paths, caching, and multiplying workload.
>
> Three main applications are presented:
> - **Hidden attack surface discovery:** Using a time-augmented Param Miner to find hidden parameters/headers by detecting subtle response time differences, even identifying DNS lookups and exceptions.
> - **Server-side injection:** Detecting blind JSON and server-side parameter pollution, especially where traditional OAST or sleep-based methods are blocked, by observing execution flow differences.
> - **Reverse proxy misconfigurations (Scoped SSRF):** Detecting proxies that only route to internal systems (bypassing external pingback detection) by observing DNS caching effects or delays from invalid DNS labels.
>
> Exploiting scoped SSRF can lead to firewall bypasses, discovering invisible routes, front-end rule bypasses, and front-end impersonation attacks by manipulating internal headers. The research emphasizes using a combination of timing and visible information for successful exploitation.
>
> Defensively, developers are advised to assume attackers can observe execution flow and branch taking, especially with performance optimizations. Mitigation strategies include rate limiting and WAFs breaking up multi-request packets.

Websites are riddled with timing oracles eager to divulge their innermost secrets.

In this paper, I'll unleash novel attack concepts to coax out server secrets including masked misconfigurations, blind data-structure injection, hidden routes to forbidden areas, and a vast expanse of invisible attack-surface.

### Background

Web timing attacks are notorious for two things; making big promises, and failing to deliver. Examples are often theoretical, and even where a technique is dubbed 'practical' everyone knows it'll stop working as soon as you try to apply it outside a lab environment.

Before this research, the smallest time gap I'd personally exploited was 30,000μs. Now, it's 200μs. This was made possible by massive advancements in timing-attack accuracy, and enables multiple powerful new techniques.

Three key attack techniques stood out as providing valuable findings on a diverse range of live systems: discovering hidden attack surface, server-side injection vulnerabilities, and misconfigured reverse proxies. In this paper, I'll explore each of these in depth.

### Hidden attack surface

Vulnerabilities often lurk out of sight in disused and forgotten features that get overlooked by developers and security testers alike. As such, vulnerability discovery journeys often start with the detection of a hidden parameter, cookie, or HTTP header.

At its core, discovering these hidden inputs involves guessing potential parameter names and observing if they change the response. Parameters that don't alter the response may remain undetected, alongside any associated vulnerabilities.

When I tried this concept out, I anticipated two problems. First, I expected many of the techniques to fail completely. Second, I suspected that any valid results I encountered would be hidden in a morass of false positives.

The biggest challenge came from neither. It's that timing attacks are too powerful. They can detect so much that it's incredibly easy to misunderstand what you've detected. They're incredibly good at detecting 'something', but that something isn't necessarily what you're trying to detect.

This video shows what initially looks like a potential remote code execution vulnerability due to an 'exec' parameter causing a visible response delay. This delay turns out to be an indicator of a WAF doing additional processing on more suspicious requests. We then see that the delay stacks when the parameter is repeated, unless the request body is over a certain size threshold. Ultimately this leads to the discovery of a complete WAF bypass. This bypass discovery was completely unexpected to me, but it's since been found by others and is now implemented in the [nowafpls](https://github.com/assetnote/nowafpls) tool. It remains a beautiful demonstration of how timing analysis can reveal insights into the target's control flow.

#### Proving the concept

To avoid being misled by false assumptions, I decided to focus on specific parameters that provide a clear security impact without any time-consuming manual investigation and a straightforward way to gather additional corroborating evidence.

IP address spoofing via HTTP headers fulfilled these requirements perfectly. It's a relatively common misconfiguration and directly enables various exploits including rate-limit bypasses, forged logs, and even [access control](https://portswigger.net/web-security/access-control) bypasses in some cases.

Conveniently, if you place a domain inside a spoofed header, vulnerable servers will often perform an in-band DNS lookup to resolve it, causing an easily detectable delay.

### Server-side injection

Triggering and spotting exceptions is a foundational part of testing for server-side injection vulnerabilities, from [SQLi](https://portswigger.net/web-security/sql-injection) to [OS command injection](https://portswigger.net/web-security/os-command-injection). This makes timing analysis a perfect match for server-side injection detection.

#### Blind JSON injection

Timing comes into its own when looking for the injection underclass; vulnerabilities that allow manipulation of data structures and formats, but stop shy of full code execution. This includes injection into formats like JSON, XML, CSV, and server-side query parameters and HTTP headers. Many of these bugs are rarely spoken of because they're so hard to detect.

They're hard to exploit too, but sometimes you can combine timing information with visible features to gain extra insight into what's happening behind the scenes. For example, I spotted one target where an invalid JSON escape sequence made the response come back 200us (0.2ms) faster:

| Parameter | Response | Time |
| --- | --- | --- |
| key=a\\"bb | "error": {     "message": "Invalid Key: a\\"bb"   } | 24.3ms |
| key=a"\\bb | "error": {     "message": "Invalid Key: a"\\bb"   } | 24.1ms |

There's a clue in the response formatting - the invalid syntax we injected hasn't altered the formatting in the response. I would expect a JSON formatter to fail when run on invalid syntax, or at least return visibly different output.

Also, lengthy inputs got redacted in the response:

| Parameter | Response | Time |
| --- | --- | --- |
| key=aaa…a"bbb | "error": {     "message": "Invalid Key: \*\*\*\*bbb"   } | 24.3ms |

This feature provides a second clue: when our invalid JSON sequence got redacted, the timing difference disappeared! Taken together, this strongly suggests that the delay is happening due to a component parsing the response being sent to us. My best guess is that it's some kind of error logging system. I was pretty pleased about figuring this out from a 0.2ms time differential but with no clear path to an exploit, I decided to move on.

#### Blind server-side parameter pollution

My most prolific probe was for blind server-side parameter pollution. This worked by comparing the response times for reserved URI characters like ? and #, with non-reserved characters like !.

In some cases, sending an encoded # made the response come back faster:

| Request | Response | Time |
| --- | --- | --- |
| /path?objectId=57%23 | Can't parse parameter | 180ms |
| /path?objectId=57%21 | Can't parse parameter | 430ms |

This could be due to the fragment breaking a server-side path and getting a speedy static response from the back-end, or the application's HTTP client simply refusing to send a HTTP request containing a raw #. Of course, it's crucial not to assume which way around the delay will land - on other targets, the encoded # made the response arrive slower.

Server-side parameter pollution was the most common type of injection discovery by a huge margin, so I think it's a promising area for further research. For more information on this attack class, check out [server-side parameter pollution](https://portswigger.net/web-security/api-testing/server-side-parameter-pollution), and [Attacking Secondary Contexts in Web Applications](https://www.youtube.com/watch?v=hWmXEAi9z5w).

#### Bug doppelgangers

Gathering enough information for an exploit based purely on timing evidence is often tricky and time-consuming. Testing each idea on a regular, non-blind vulnerability typically involves a single repeater request, whereas with many of these, you're potentially looking at a 30-second Turbo Intruder attack.

One thing that can help here is 'bug doppelgangers' - non-blind variations of the target bug class. Param Miner will report these, and they're great for learning how to interpret and exploit these bugs in a less challenging environment.

### Reverse proxy misconfigurations

The single biggest breakthrough in this research was when I realized I could use timing to detect a widely overlooked type of SSRF.

#### Scoped SSRF

Although successful, this detection technique had a major blind spot - scoped SSRF.

After I published the research, someone from Google asked if I'd found any vulnerabilities in their systems, strongly implying that they had been vulnerable. Shortly later, Ezequiel Pereira posted [$10k host header](https://web.archive.org/web/20200411123311/https://sites.google.com/site/testsitehacking/10k-host-header) in which he exploited an open proxy belonging to Google that I'd failed to detect. My scanning method had failed because Google's proxy was configured to only route requests to their own systems, so my server never received a DNS lookup.

This was a hint at a really common scenario, where companies allow request forwarding to arbitrary subdomains:

| Host header | Full SSRF | Scoped SSRF |
| --- | --- | --- |
| random.example.com | 404 Not Found | 404 Not Found |
| random.notexample.com | 404 Not Found | 403 Forbidden |

I don't think there's an established name for this type of SSRF, so I'll call it scoped SSRF. This restriction can be implemented via an internal DNS server, simple hostname validation, a firewall blocking outbound DNS, or a tight listener config. The outcome is always the same - you've got a bug with an impact close to full SSRF, but it can't be detected using pingback/OAST techniques.

#### Detecting scoped SSRF

To detect scoped SSRF, we need to answer the question "Did the server try to connect to the specified hostname?". Timing is perfectly suited for this. Consider a server at www.example.com that issues the following responses:

| Host header | Response | Time |
| --- | --- | --- |
| foo.example.com | 404 Not Found | 25ms |
| foo.bar.com | 403 Forbidden | 20ms |

These two responses show that it's doing some kind of validation on the Host header, but there isn't sufficient information to tell if it's an open proxy. If you rely on the response content, you'll end up with both false positives and false negatives.

The following request pair is what proves the issue - the faster second response is evidence of DNS caching:

| Host header | Response | Time |
| --- | --- | --- |
| abc.example.com | 404 Not Found | 25ms |
| abc.example.com | 404 Not Found | 20ms |

Some DNS systems don't cache failed DNS lookups, but I found an alternative solution for this - sending an overlong 64-octet DNS label, leading to the DNS client refusing to issue the lookup and a faster response:

| Host header | Response | Time |
| --- | --- | --- |
| aaa{62}.example.com | 404 Not Found | 25ms |
| aaa{63}.example.com | 404 Not Found | 20ms |

#### Firewall bypass

The simplest exploit is where you can see the target from outside but can't directly access it.

On one company, sonarqube.redacted.com resolved to a public IP address, but attempting to access it triggered a connection reset from a firewall. My probes had identified app.redacted.com as a reverse proxy and, using that, I was able to route around the firewall and access the internal SonarQube instance.

| Entry point | Host header | Result |
| --- | --- | --- |
| sonarqube.redacted.com | sonarqube.redacted.com | \--reset-- |
| app.redacted.com | sonarqube.redacted.com | 200 OK |

#### Firewall bypass - invisible route variant

There's a common variation where the internal system doesn't have a convenient public DNS record to let you know it exists:

There are a huge number of pre-prod, staging, and development servers exposed to anyone applying this technique. If you get lucky, they'll have debugging enabled or test credentials configured, making them soft targets. These systems may even have real target data, or reused keys from production.

The most interesting targets I found were pre-launch systems still under active development. In particular, I discovered an admin console with apparently-public access on a really cool US government system, which I'm gutted I can't provide any details about. I reported the issue and the system went 'live' a few months later, but the admin console is nowhere in sight.

#### Front-end rule bypass

Some targets are publicly accessible, but sit behind front-end servers that enforce inconvenient security rules that block attacks or restrict access to valuable endpoints. The classic way to handle these is by talking directly to the back-end, but that's often impossible due to firewalls.

Reverse proxies provide a compelling alternative - go around the barrier:

#### Front-end impersonation attacks

The most spectacular and surprising exploits happen when there's a trust relationship between the front-end and back-end. It's common knowledge that you can use headers like X-Forwarded-For to spoof your IP address. What's less appreciated is that this is part of a much broader and more powerful bug class. This type of attack has no established name, so I'll call it a front-end impersonation attack.

Front-end systems often add HTTP headers onto requests before forwarding them to the back-end. These contain additional information that the back-end might find useful, such as the user's remote IP address, and the originating protocol. More complex deployments sometimes use custom headers to transmit critical authentication information. Back-end servers trust these headers implicitly.

If an attacker attempts to spoof these headers, the front-end will typically overwrite them. This header overwriting behavior is the single brittle line of defense against front-end impersonation attacks.

![](https://portswigger.net/cms/images/c7/8b/a600-article-impersonation.png)

The easiest way to bypass this defense is to simply talk directly with the back-end, but this is usually impossible due to network firewalls. Another approach is [HTTP request tunneling](https://portswigger.net/web-security/request-smuggling/advanced/request-tunnelling), which I used to [completely compromise New Relic's core internal API](https://portswigger.net/research/http2#guessing) using a header called "Service-Gateway-Is-Newrelic-Admin". You can also try [obfuscating headers](https://www.intruder.io/research/practical-http-header-smuggling) to smuggle them past the front-end.

Misconfigured proxies offer an elegant alternative way to bypass header-overwriting defenses and perform front-end impersonation attacks. To try this out,

- Use Param Miner's 'Detect scoped SSRF' scan to detect a reverse proxy
- Run 'Exploit scoped SSRF' to find alternative routes to internal systems
- On each alternate route, run 'Guess headers' to find useful headers

#### Chaining

Finally, scoped SSRF via reverse proxies offers some great exploit chaining opportunities.

If you're able to take over a subdomain on the target company and point the DNS record to an arbitrary IP address, you can use this to upgrade a scoped SSRF into a full SSRF and hit arbitrary IP addresses. This is a lot like chaining a traditional SSRF with an open redirect.

Since reverse proxies let you pick your back-end, they're great for HTTP request smuggling. I didn't have time to properly explore this concept. In short, I think you'll find that, while it should be easy to find back-ends that are vulnerable to request smuggling, cross-user exploitation will often be impossible because no legitimate users will be sharing your front-end/back-end connection. To prove the impact, you'll need to pursue tunneling-based exploits like front-end impersonation and header disclosure.

### Takeaways

It's not just about the exploits. At their core, web timing attacks are about answering difficult questions.

With the single-packet attack, web timing attacks have become 'local', portable, and feasible.

Timing oracles are everywhere. Whatever you're testing, timing murmurs are always present, waiting for you to listen.