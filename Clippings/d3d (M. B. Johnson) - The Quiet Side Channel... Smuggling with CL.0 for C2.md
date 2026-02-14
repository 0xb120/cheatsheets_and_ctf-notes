---
title: The Quiet Side Channel... Smuggling with CL.0 for C2
source: https://blog.malicious.group/the-quiet-side-channel-smuggling-with-cl-0-for-c2/
author:
  - d3d (M. B. Johnson)
published: 2025-07-25
created: 2026-02-08
description: Most people think HTTP smuggling requires complex header tricks or broken protocol parsing. But sometimes, the most effective exploits aren’t based on complexity — they’re based on trust. In this paper, I’ll show how a simple misalignment in expectations between front-end and back-end servers can be quietly exploited
tags:
  - clippings/articles
---
# The Quiet Side Channel... Smuggling with CL.0 for C2

![](https://blog.malicious.group/content/images/2025/07/sidechannel.png)

> [!summary]+
> > This paper details how a simple misalignment in expectations between front-end and back-end servers can be exploited to build an undetectable HTTP C2 channel, without relying on complex header tricks or broken protocol parsing.
> 
> **Prerequisites:** A solid understanding of Request Smuggling, Desync gadgets, Cache Poisoning vulnerabilities (especially CL.0 variations), and Python for PoC development.
> 
> **Discovery Process:**
> 1.  Identify targets vulnerable to CL.0 request smuggling (e.g., using subdomain enumerators and `tlsx` for major cloud/CDN providers like Akamai).
> 2.  Verify global cache poisoning potential.
> 3.  Assess cache stability and sensitivity.
> 
> **Target Testing:**
> *   **Option 1:** Burp Suite + HTTP Request Smuggler bApp (requires specific configuration to disable Live Audit and modify Live passive crawl for efficiency).
> *   **Option 2:** Custom tooling (more streamlined for bulk analysis, outputs to Discord and a `cl0.log` file).
> 
> **Testing Attributes:** For a reliable communication channel, vulnerable machines must allow global poisoning and caching data on 3xx redirects. The paper demonstrates how repeated malformed POST requests using `nameprefix1` can cause a server to include a smuggled path (e.g., `/robots.txt`) in a `Location` header, leading to global cache poisoning (a temporary DoS).
> 
> **Stealth Improvement:** The attack is enhanced by using a GET-based gadget instead of POST, as GET requests with `Content-Length` headers are often tolerated by proxies but honored by backends, making the traffic less suspicious in logs and harder to detect.
> 
> **Development (C2 Channel):**
> *   A Python PoC tool (`CacheC2Channel`) demonstrates operationalizing the technique.
> *   Instead of `/robots.txt`, the script smuggles an encoded/encrypted message. The client receives a 3xx redirect with the message in the `Location` header but intentionally avoids following the redirect to maintain stealth.
> *   The tool supports both sending (via `-s` option with a file like `PoC.txt`) and listening (via `-l` option) for these smuggled messages.
> *   This widespread vulnerability was found across high-profile `.mil`, `.gov`, and `.cn` domains, highlighting its impact and the need for a deep understanding of HTTP desync.

In this paper, I’ll show how a simple misalignment in expectations between front-end and back-end servers can be quietly exploited to build an undetectable channel.

## Prerequisites

To get the most out of this research, it's helpful to have a solid grasp of Request Smuggling, Desync gadgets and Cache Poisoning vulnerabilities, especially `CL.0` (*malformed content length*) variations along with some familiarity with Python development for the PoC.

## Discovery

The initial step in this research was identifying inconsistencies in how different servers and intermediaries interpret and process HTTP requests.

By observing response behavior and using tooling to fuzz combinations of headers, I uncovered a parsing discrepancy that laid the foundation for a new smuggling primitive.

Before diving into the technical details, let’s walk through the high-level steps of the discovery process to ensure a shared understanding:

1. Identify a target that is vulnerable to `CL.0` request smuggling.
2. Verify whether the vulnerability leads to global cache poisoning.
3. Assess the stability and sensitivity of the cache while it remains poisoned.

While this may sound straightforward, each step requires careful attention to detail.

### Finding Targets

To narrow our focus, we’ll target infrastructure operating behind major cloud and CDN providers such as Akamai (`akamaiedge.net`), Azure (`azureedge.net`), and Oracle Cloud (`oraclecloud.com`).

we’ll limit ourselves to endpoints within their respective namespaces—easily discovered using tools like `chaos-client`, `subfinder`, `bbot`, or similar subdomain enumerators.

At this stage, we can start identifying which companies are utilizing specific instances within the `akamaiedge.net` network. To do this, we’ll use the `tlsx` tool from [ProjectDiscovery](https://projectdiscovery.io/?ref=blog.malicious.group) to extract and analyze TLS certificate data from servers listening on port 443.

This is where things start to get interesting—we can now begin mapping which companies are leveraging specific instances within the `akamaiedge.net` network.

Next, I’ll parse all domains from the `tlsx.log` output and save them into a `domains.txt` file to prepare for the following step.

With the company domains separated, let's verify which have active DNS with web services we can take a poke at using the tool `httpx`.

After running `httpx` we now have a list of both HTTP and HTTPS endpoints for domains that we know are using the `akamaiedge.net` services.

### Testing Targets

***Option 1 -** Burp Suite + HTTP Request Smuggler bApp:*

select `Request Smuggler` and select the `cl.0` option.

Once the `cl.0` window opens, it will overwhelm you will choices to pick from. To narrow this down, each option (minus a few) is a different `cl.0` smuggling gadget. For this paper I will be using `nameprefix`, `nameprefix2`, `options` and `head`

On pressing `Ok` the attack probe will start.

***Option 2 -** Custom tooling:*

So now that we have a means of discovering these bugs, let's take a closer look at one of the gadgets to get a better idea of what is going on.

### Testing Attributes

Once we've identified potential positives, we need to manually review each hit to confirm not only that the technique is effective, but also that the behavior aligns with what we need for establishing a reliable communication channel.

For building a communications channel, we need to find vulnerable machines that have the following attributes:

- Desync allows for global poisoning
- Desync allows caching data on redirects (3xx)

The first point is obvious since we require global caching, but the reason for requiring the 3xx redirects is so that we can poison a redirection which shows up in the `Location` header for the client

The following image is of a `nameprefix1` gadget, which is a typical `POST` request but using another request as its content, while using a malformed `Content-Length` header.

![](https://blog.malicious.group/content/images/2025/05/image-19.png)

When executing the request above, the `redacted.tld` domain responds with a `Location: https://www.redacted.tld` header. This indicates that the non-`www` version consistently redirects to the `www` subdomain, completely disregarding the POST body I included—as shown below.

![](https://blog.malicious.group/content/images/2025/05/image-20.png)

However, what happens if I sent the exact same request, 3 to 5 times in a row?

![](https://blog.malicious.group/content/images/2025/05/image-21.png)

After hitting the send button several times, the server shifted from ignoring the POST content to fully adopting the smuggled endpoint—thanks to the `CL.0` gadget—and included it in the `Location` header as part of the redirect.

While we've confirmed the behavior locally, the real question is whether it impacts the global cache.

To test this, one of my go-to techniques is spinning up a VM on DigitalOcean or Linode to simulate a normal user—looping a `curl` request every few seconds.

![](https://blog.malicious.group/content/images/2025/05/image-22.png)

from a logging and detection perspective, the attack currently relies on sending a `POST` request to trigger the desynchronization

a `GET`\-based gadget—if viable—would likely appear less suspicious in logs and could evade basic anomaly detection more easily especially if the client never follows the 3xx redirect.

Using the `GET` method with a `Content-Length` header may seem invalid at first, since `GET` requests typically do not include a body. However, many web servers, CDNs, and proxies do not strictly enforce this part of the HTTP specification.

![](https://blog.malicious.group/content/images/2025/07/image-6.png)

This leniency can lead to discrepancies between how front-end and back-end systems parse the same request.

- A proxy (like a CDN) may assume `GET` requests never have a body and may **ignore the `Content-Length`** header entirely, forwarding the request as-is.

- Meanwhile, a backend server may honor the `Content-Length` and **wait for a body**, leading to a desynchronization in state between the two systems.

This inconsistent parsing behavior is exactly what enables request smuggling attacks using `GET` — even though it's uncommon