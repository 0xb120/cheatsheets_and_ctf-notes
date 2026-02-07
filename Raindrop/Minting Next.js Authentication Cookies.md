---
raindrop_id: 1544822261
raindrop_highlights:
  6980e32f9e83455b3cefe9ff: ffe6c60bdfca8d46d35ad96031267a2d
  6980e33c1a703ee110ee98a1: 510b5c9961c9446f8c3ff93c813992cd
  6980e34baf90b17cb678cc5f: f7ebc5549167bb573dd2d87f97212a5a
  6980e3501a703ee110ee9dbb: bb5763004f4750c6fcd031e6246e652d
  6980e35b97934345acf638f2: 7b55e7c9bc4a5b6f283020c8519890aa
title: "Minting Next.js Authentication Cookies"

description: |-
  null

source: https://embracethered.com/blog/posts/2026/minting-next-auth-nextjs-auth-cookies-react2shell-threat/

created: 2026-01-15
sync-date: 1770460623518
tags:
  - "_index"

 
  - "tech-blog" 
  - "NextJS"

---
# Minting Next.js Authentication Cookies

![](https://embracethered.com/blog/images/2026/next-auth.png)

> [!summary]
> null





There is another critical environment variable, typically named NEXTAUTH_SECRET (or in the latest version just AUTH_SECRET), that needs rotation and serves as the standalone secret used to encrypt and authenticate next-auth session cookies.
Imagine an adversary exploited your application, and ran the env command. By doing so, the attacker likely retrieved OAuth (e.g. Entra ID,…) client_id and client_secret among other environment variables.
However, for a typical Next.js app that uses the next-auth library (also called Auth.js), that is not a sufficient mitigation.
The NEXTAUTH_SECRET is all you need
The next-auth code that creates the authentication cookies is in this GitHub repo. The salt used for minting these cookies is the cookie name itself. So, it’s pretty easy for an adversary to derive it as well.