---
raindrop_id: 1538014864
raindrop_highlights:
  6963ef800c546c9599da79cc: 0881ad3bb0b7d6f8b4200fbd1808c3a3
  6963ef8dd815af5e327e4b97: e1b49c18d7e0aa8c2765f25d785c89f7
  6963ef9b86ce7738d8e1f197: 0ce2f84f2d307049b46be95f3000cf46
title: "Gixy-Next: NGINX Configuration Security & Hardening Scanner"

description: |-
  Open source NGINX security, hardening, and configuration compliance scanner that statically analyzes nginx.conf for security issues and misconfigurations.

source: https://gixy.io/

created: 2026-01-11
sync-date: 1769114384440
tags:
  - "_index"

 
  - "Tools"

---
# Gixy-Next: NGINX Configuration Security & Hardening Scanner

![](https://gixy.io/imgs/gixy.jpg)

> [!summary]
> Open source NGINX security, hardening, and configuration compliance scanner that statically analyzes nginx.conf for security issues and misconfigurations.





Gixy-Next (Gixy) is an open-source NGINX configuration security scanner and hardening tool that statically analyzes your nginx.conf to detect security misconfigurations, hardening gaps, and common performance pitfalls before they reach production.
Quick start

Gixy-Next (the gixy or gixy-next CLI) is distributed on PyPI. You can install it with pip or uv:

# pip
pip3 install gixy-next

# uv
uv pip install gixy-next


You can then run it:

# gixy defaults to reading /etc/nginx/nginx.conf
gixy

# But you can also specify a path to the configuration
gixy /opt/nginx.conf
Web-based scanner

Instead of downloading and running Gixy-Next locally, you can use this webpage and scan a configuration from your web browser (locally, using WebAssembly).