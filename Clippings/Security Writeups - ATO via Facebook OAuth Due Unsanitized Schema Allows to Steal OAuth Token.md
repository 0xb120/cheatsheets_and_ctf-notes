---
title: ATO via Facebook OAuth Due Unsanitized Schema Allows to Steal OAuth Token
source: https://blog.sicks3c.io/posts/ato-via-facebook-oauth-due-unsanitized-schema-allows-to-steal-oauth-token/
author:
  - Security Writeups
published: 2023-09-01
created: 2026-02-14
description: Exploiting OAuth flow to Takeover accounts via Login with FB
tags:
  - clippings/articles
  - _inbox
  - Android
---
# ATO via Facebook OAuth Due Unsanitized Schema Allows to Steal OAuth Token

![](https://neelbhatt.com/wp-content/uploads/2018/01/fb13.png)

> [!summary]+
> > This article details a 0-day vulnerability discovered in 2023 by mainteemoforfun and the author, affecting websites using Facebook's \\"Login With Facebook\\" feature.
    > The exploit involved manipulating the `redirect_uri` parameter in the OAuth flow. By appending an extra character to the `clientID` in the `fb://clientID` scheme (e.g., `fb://clientID_a`), the authentication token could be redirected to an attacker-controlled endpoint, bypassing Facebook's validation.
    > A proof-of-concept involved a malicious Android app with a custom intent filter designed to capture the redirected token.
    > The vulnerability was responsibly disclosed to Facebook. Reports via HackerOne resulted in approximately 15K USD in bounties for 5 high/medium severity findings, while Bugcrowd reports were mostly deemed inapplicable.
    > Facebook acknowledged the issue, which was already known and subsequently patched.
    > Key takeaways emphasize the importance of innovative testing for subtle vulnerabilities in OAuth implementations, the severe real-world implications of validation oversights (account hijacking), and continuous vigilance in evolving authentication mechanisms.

## Deep Dive into an OAuth Exploit: A 0-Day Case Study

an in-depth exploration of **mobile authentication mechanisms**. Our efforts culminated in the discovery of a striking **0-day vulnerability** back in **2023** that has since been patched.

This vulnerability enabled us to potentially hijack user sessions on websites utilizing **Facebook’s “Login With Facebook”** feature. By manipulating the **`redirect_uri`** parameter in the [OAuth](../Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/OAuth%202.0.md) flow, an attacker could redirect authentication tokens to a host under their control.

## Understanding the Vulnerability

### OAuth Basics and the Attack Surface

**OAuth** is the de facto standard for delegated authorization. Typically, during an OAuth transaction, the client application registers a trusted **`redirect_uri`** (often in a custom scheme format, such as `fb://clientID`). This URI is used by the authorization server (**Facebook**, in this case) to send back the authentication token after a user successfully logs in.

When testing OAuth implementations, it is common to experiment with modifying the **`redirect_uri`** parameter to see if the endpoint strictly enforces whitelist validations.

### The Exploit Mechanism

We began by modifying the **`redirect_uri`** in the OAuth flow. Our initial test—attempting to change the **clientID** directly (e.g., `fb://anotherClientID`) produced a “mismatch” error, as expected.

By appending an extra character to the **clientID** (e.g., `fb://clientIDa`), we observed that the authentication token was redirected to the newly specified scheme. This minor change bypassed the strict check, allowing the token to be delivered to an endpoint not originally whitelisted by the service.

To rule out any anomalies, we repeated the process several times. The results were consistent

## Proof-of-Concept

To demonstrate the impact of this vulnerability, I developed a small [Android 101](../Dev,%20ICT%20&%20Cybersec/Mobile%20Hacking/Android%20101.md) application with a custom intent filter designed to capture the redirected token.

```xml
<intent-filter>
    <action android:name="android.intent.action.MAIN" />
    <category android:name="android.intent.category.LAUNCHER" />
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="fbClientIDa" />
</intent-filter>
```

When the OAuth flow was initiated, the token was unexpectedly sent to this malicious app instead of the legitimate one.