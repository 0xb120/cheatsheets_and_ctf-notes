---
author: blackbird-eu
aliases:
  - 7 Overlooked Recon Techniques to Find More Vulnerabilities
tags:
  - readwise/articles
url: https://www.intigriti.com/researchers/blog/hacking-tools/7-overlooked-recon-techniques-to-find-more-vulnerabilities
date: 2025-01-13
---
# 7 Overlooked Recon Techniques to Find More Vulnerabilities


## Virtual host (VHost) enumeration

Most hosts and servers deployed today serve **multiple** applications. For example, multiple (sub)domains may point to a single host that has a running reverse proxy server (such as Nginx). This proxy server will determine based on the host header what application to serve. [](https://read.readwise.io/read/01jhg4je2scztvd9a98n1kzxqc)

```nginx
# Nginx reverse-proxy configuration
server {
    server_name app.example.com api.example.com;
    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
    }

    # ...
}

server {
    server_name app-stg.example.com api-stg.example.com;
    location / {
        proxy_pass http://localhost:8081;
        proxy_set_header Host $host;
    }

    # ...
}
```

Fuzz them using [ffuf](../../Dev,%20ICT%20&%20Cybersec/Tools/ffuf.md) or similar:
```sh
$ ffuf -u https://example.com -H "Host: FUZZ.example.com" -w /path/to/wordlist
```


## JavaScript file monitoring

This recon technique is one of the most overlooked ones as it requires a system that's **always up and monitoring**, yet so effective as it can help you get notified when your target application gets updated and new API endpoints, app routes and input parameters have been referenced.
There are several open-source tools [^1] that you can make use of, some of which even provide support for notifications on your favorite messaging app.
[](https://read.readwise.io/read/01jhg4n0dnk33ppyyg3sqpd5an)


## Finding related assets with favicon hashes

Some hosts are set up for development or administrative purposes only and have no (sub)domain pointing to them. They do have a few similarities that can help us track these down, and this is often the public IP or IP/CIDR range they are in.

In some cases, especially when your target is a smaller company, it may not have a reserved IP space, or the company regularly deploys hosts in various regions across the world. We can identify hosts that belong to that specific target using the f**avicon hash**.

Favicons are the small icons that appear on the web browser's tab next to your page title. These icons can be used to find more related assets [^2] (such as subdomains, IPs, hosts, etc). [](https://read.readwise.io/read/01jhg4p1s7540fv54ca2ngc44e)



[^1]: [JSMON](https://github.com/robre/jsmon) - a javascript change monitoring tool for bugbounties

[^2]: [Finding more subdomains using favicons](novasecio%20-%20Complete%20Guide%20to%20Finding%20More%20Vulnerabilities%20With%20Shodan%20and%20Censys.md#Finding%20more%20subdomains%20using%20favicons)
