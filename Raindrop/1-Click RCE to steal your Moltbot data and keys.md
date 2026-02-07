---
raindrop_id: 1573898205
raindrop_highlights:
  69833df0a24b819084dc2cb9: 9cd918f26a3ad1cac672094a8d8d7c4b
  69833dfd52107c984d82badd: c5294c390b07856727d5905485a62bd4
  69833e015adc1d5b0c624fa5: d3e0c162b408fd3e8b68aa0e841293ac
  69833e096064010a58eb585f: ba93126d236ba83a62f9847e309a765e
title: "1-Click RCE to steal your Moltbot data and keys"

description: |-
  null

source: https://depthfirst.com/post/1-click-rce-to-steal-your-moltbot-data-and-keys

created: 2026-02-02
sync-date: 1770460623194
tags:
  - "_index"

 
  - "Inoreader" 
  - "HackerNews"

---
# 1-Click RCE to steal your Moltbot data and keys

![](https://cdn.prod.website-files.com/691d9445275873e3d3fc4279/697f824bc8f0ea823f8c9535_Whisk_1358248e38462f0921e418ad52a19889eg.png)

> [!summary]
> null





Pivoting to Bypass localhost Network Restrictions
Most users run OpenClaw on localhost. As a result, their OpenClaw is inaccessible from the internet. Even if an attacker has a valid auth token, they can’t access a victim’s local OpenClaw. 

However, I found a bug to bypass this otherwise frustrating restriction.
Regularly, attacker.com can’t make arbitrary client-side requests to localhost. This is because Same Origin Policy (SOP) prevents separate origins (sites) from fully interacting with each other.
While browsers apply SOP to http connections, they do not to WebSocket ones. It’s a WebSocket server’s responsibility to validate a request's origin and decide whether to accept the connection. I found that OpenClaw’s WebSocket server fails to validate the WebSocket origin header, accepting requests from any site.

This allows me to perform Cross-Site WebSocket Hijacking (CSWSH). When the victim visits attacker.com, I can run JavaScript on the victim’s browser to open a connection to ws://localhost:18789 . The browser acts as a pivot point between attacker.com and the victim’s otherwise inaccessible localhost.