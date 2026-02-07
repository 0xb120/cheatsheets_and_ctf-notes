---
raindrop_id: 1543277460
raindrop_highlights:
  696d113d185cd356a6180eef: 539eae7672d88905e0f9d0209f325c4c
  696d114b7a2cb19be6c03d03: 6c70c4b092f59da21696127d2d7bb5b2
  696d11575e4bb6c2b4bc380f: 591e18608efa3802f93f4b67f895e6d5
  696d116ebbae731ea930e1ef: 7060c1a38a01062be120f064a4361cf4
  696d117901e5cf94d629db4c: cd2586252fae316896b596badd6bc486
  696d118085dd85ae92054296: 28b842adf01c3c746ec24d6161f397ba
  696d119f5e4bb6c2b4bc4975: b4c9be3dee942314d04f090313db4041
  696d11a2dd2f2cd092409949: e7a8395b4d805d396bc12d827b7bca19
  696d11dadde790a5bfa7db05: 61081f4e97c584e788f80de86a0230fc
  696d11edf22bdc89515acb96: 06d483340f66c6e6d58a0780bd86b5a1
  696d1219dafedffded12d84a: 82d95e584f523424199114cded995fdd
  696d1247dd2f2cd09240c210: eb1f6c90b0955d7ada9150ba14c81df4
  696d1258dd2f2cd09240c6b5: ef7b42971682c0680ebc38bb65cc17bd
  696d125e508a8738f42dc02e: 60ef26023dac369efb6cfb72c3b0d302
  696d1278dd2f2cd09240cde8: bd82a2857a560f7a9f388d94654e3b64
  696d1288bbae731ea931299c: 89f003b6cb685bdd1348cba089a7eb03
  696d128dfdfd5d4f771a1b9b: d7a81288da37e0ff9c266dd83774d89e
  696d12b35e4bb6c2b4bc905f: a6de7bac03d1fe9e44d73cffd2bd7b78
  696d1321508a8738f42df370: a4751f689cfd65b4f450113087f86422
  696d13407ea25d943089ce87: 9503b6ba19c62e16a2c81311061b9744
title: Multiple XSS in Meta Conversion API Gateway Leading to Zero-Click Account Takeover | Youssef Sammouda (sam0) personal blog
description: Introduction
source: https://ysamm.com/uncategorized/2025/01/13/capig-xss.html
created: 2026-01-14
sync-date: 1769114383319
tags:
  - _index
  - vuln-research-blog
  - tech-blog
---
# Multiple XSS in Meta Conversion API Gateway Leading to Zero-Click Account Takeover | Youssef Sammouda (sam0) personal blog

![](https://rdl.ink/render/https%3A%2F%2Fysamm.com%2Funcategorized%2F2025%2F01%2F13%2Fcapig-xss.html)

> [!summary]
> Introduction





Bug #1: Trusting event.origin and Turning It Into a Script Loader
The first issue exists entirely on the client side, inside capig-events.js. It only triggers under a specific condition: when the page has an opener window.

If window.opener is present, the script registers a message event listener to receive configuration data for IWL.

window.addEventListener("message", function (event) {
    var data = event.data;

    if (
        !localStorage.getItem("AHP_IWL_CONFIG_STORAGE_KEY") &&
        !localStorage.getItem("FACEBOOK_IWL_CONFIG_STORAGE_KEY") &&
        data.msg_type === "IWL_BOOTSTRAP"
    ) {
        if (checkInList(g.pixels, data.pixel_id) !== -1) {
            localStorage.setItem("AHP_IWL_CONFIG_STORAGE_KEY", {
                pixelID: data.pixel_id,
                host: event.origin,
                sessionStartTime: data.session_start_time
            });
            startIWL();
        }
    }
});
When a message with type IWL_BOOTSTRAP is received, the script checks that the provided pixel_id exists in an internal list. What it does not check is where the message came from.

Instead of validating event.origin against an allowlist, the script stores it and treats it as a trusted host.

Later, during initialization, this stored origin is used to dynamically load another JavaScript file:

<origin>/sdk/<pixel_id>/iwl.js
If they can make the browser accept a script from that origin, they gain arbitrary JavaScript execution in the context of the embedding site.

This is a classic example of a dangerous pattern: taking unvalidated origin data from postMessage and turning it into code execution.
Why This Is Not Immediately Trivial? CSP and COOP
Content Security Policy (CSP) and Cross-Origin-Opener-Policy (COOP) appear to limit the impact.
On www.meta.com, CSP does not allow arbitrary external scripts. Additionally, most pages deploy:

Cross-Origin-Opener-Policy: same-origin-allow-popups


This prevents simple opener-based exploitation.
CSP Bypass
In logged-out states, certain Meta pages (notably under /help/) relax CSP to include third-party analytics providers such as:

*.THIRD-PARTY.com
*.THIRD-PARTY.net
A subdomain takeover, XSS, or file upload on any CSP-allowed third-party domain would be sufficient to host an attacker script capable of sending the required message to trigger the exploit.
Cross-Origin-Opener-Policy: same-origin-allow-popups Bypass
When executed inside Facebook’s Android WebView, window.name reuse combined with window.open() allowed attacker-controlled pages to regain access to the opener.

window.name = "test";
window.open(target, "test");
This bypass only ensured that the current window would have an opener object; however, the window becomes its own opener. In this case, the cross-window message must be sent from one of the page’s iframes.
Iframe Hijacking
Likely, meta.com was loading third-party content such as ads or social plugins hosted in iframes. We exploited a vulnerability in one of these third-party components to completely hijack the iframe and later send the malicious postMessage from there.
This allowed the attacker-controlled iframe to interact with the parent window (www.meta.com) and deliver a malicious postMessage payload
Attack Flow Summary
First, we visit the attacker’s website while logged into Facebook, inside the Facebook App WebView:

0) User opens a url inside Facebook application

1) The script running on the attacker’s website executes the opener bypass
   and redirects the user to https://www.meta.com/help/

2) On the help page, the script inside the hijacked iframe redirects to a
   (*.THIRD-PARTY.com) page (allowed by Meta’s CSP). The script there sends the
   IWL_BOOTSTRAP postMessage to the parent window (www.meta.com)

3) At the same time, the attacker-controlled (*.THIRD-PARTY.com) hosts a
   JavaScript file named iwl.js under /sdk/{pixel_id}/iwl.js containing
   the XSS payload

4) The script is loaded from the attacker-controlled third-party website
   and executed in the context of www.meta.com
Since this occurs in a logged-out state on www.meta.com and inside the Facebook App WebView, the objective is to escalate this into Facebook account takeover.

We identified an endpoint on www.facebook.com that whitelists www.meta.com in CORS.
From there, it becomes possible to issue authenticated requests, read CSRF tokens from responses, and perform privileged actions