---
title: "DOM-based Extension Clickjacking: Your Password Manager Data at Risk"
source: https://marektoth.com/blog/dom-based-extension-clickjacking/
author:
  - Marek Tóth
published: 2025-08-09
created: 2026-02-08
description: Security Researcher | Ethical Hacker | Web Application Security
tags:
  - clippings/articles
aliases:
  - DOM-based Extension Clickjacking
---
# DOM-based Extension Clickjacking: Your Password Manager Data at Risk

![](https://marektoth.com/images/dombased-image1.png)

> [!summary]+
> > This research introduces **DOM-based Extension Clickjacking**, a new attack technique where malicious scripts manipulate and hide UI elements injected into the DOM by browser extensions (e.g., password managers), making them clickable unknowingly to the user.
> 
> All 11 tested password managers were initially vulnerable in their default configurations, potentially putting **tens of millions of users at risk**.
> 
> **Impacts include:**
> > -   **Stealing credit card details and personal data** (1-2 clicks on an attacker's website, not domain-specific).
> > -   **Stealing login credentials (username, password, TOTP) and even passkeys** (via \"signed assertion hijacking\") when an attacker can execute JavaScript on a trusted domain (e.g., via XSS, subdomain takeover, or custom content in web applications like Shopify). Password managers autofill credentials across subdomains by default.
> 
> **As of January 2026, several major password managers remain vulnerable:** 1Password, LastPass (login credentials), Bitwarden (overlay methods), iCloud Passwords (login credentials), Enpass, LogMeOnce. Others like NordPass, ProtonPass, RoboForm, Dashlane, Keeper, and KeePassXC-Browser have released fixes for the described methods.
> 
> **Recommendations for users:** Ensure automatic updates, disable manual autofill features if possible, set autofill to only exact URL matches, configure extension site access to \"on click,\" and disable autofill suggestions injected into the DOM.

Is [Clickjacking](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Clickjacking.md) still an exploitable vulnerability nowadays? Many bug bounty programs have this vulnerability listed in the "out of scope" section, and in better cases they accept it but don't reward it. The reason is that there are many protections available today that significantly reduce the impact of this vulnerability. It can be said that a common web clickjacking vulnerability has already been solved and can be easily defended against.

The result of my research is that clickjacking is still a security threat, but it's necessary to shift from web applications to browser extensions, which are more popular nowadays (password managers, crypto wallets and others).

A **single click anywhere** on a attacker controlled website could **allow attackers to steal users' data** (credit card details, personal data, login credentials including TOTP). The new technique is general and can be applied to other types of extensions.

## Key Information

- A new clickjacking technique where a malicious script manipulates UI elements that browser extensions inject into the DOM by making them invisible using javascript.

- A single **click anywhere on the attacker's website could leak credit card details** including security codes (6 out of 9 were vulnerable) or exfiltrate stored personal information (8 out of 10 vulnerable).

- All password managers filled credentials not only to the "main" domain, but also to all subdomains. An attacker could easily find XSS or other vulnerabilities and **steal the user's stored credentials with a single click (10 out of 11), including TOTP (9 out of 11)**. In some scenarios, passkey authentication could also be exploited (8 out of 11).

- For Chromium-based browser users it is recommended to configure site access to "on click" in extension settings. This configuration allows users to manually control autofill functionality.

- The described technique is general and I only tested it on 11 password managers. Other DOM-manipulating extensions are probably also vulnerable (password managers, crypto wallets, notes etc.).

## Introduction

### Intrusive Web Elements

While browsing websites, users often encounter intrusive web elements that block immediate access to target content. These elements require users to click to close them.

- Cookie consent banners - 1 click (accept/decline cookies)

- Newsletter popup, login dialog - 1 click (closing dialog)

- Web push notifications - 1 click (accept/decline)

- Cloudflare challenge pages / Captcha pages - 1 click (Verify you are human)  
	##### 2 clicks if verification fails, 4 or more clicks if captcha needs to be solved

Because users expect to interact with these elements, **I will use fake intrusive web elements in a clickjacking exploit** to trick the user into clicking.

### Clickjacking (Web Application)

Clickjacking (or "UI redressing") is a client-side attack technique that exploits visual layering to hijack user clicks intended for legitimate interface elements. This attack uses invisible frames (iframes) to deceive users who believe they are clicking on legitimate website elements, while actually performing actions that benefit the attacker.

To mitigate security impact, security headers X-Frame-Options or Content-Security-Policy are used.

```html
X-Frame-Options: DENY 
X-Frame-Options: SAMEORIGIN
X-Frame-Options: ALLOW-FROM https://example.com

Content-Security-Policy: frame-ancestors 'none';
Content-Security-Policy: frame-ancestors 'self';
Content-Security-Policy: frame-ancestors https://example.com
```

Additionally, the SameSite Lax or Strict cookie attribute can be set to prevent cookie usage in cross-site iframes.

```html
SameSite=Lax            cookies in cross-site POST request or cross-site iframes are blocked
SameSite=Strict         cookies only for same-site requests
SameSite=None           allowed cookies for cross-site requests
```

Default value is **SameSite=Lax if SameSite isn't specified.**

### Password Managers

Password managers are widely used as browser extensions to simplify website authentication.

Password managers have autofill functionality that can be of 2 types:

- Automatic autofill - credentials are **automatically filled in** (0-click)

- Manual autofill - **user interaction is required** to fill in credentials (selecting from a dropdown menu)

## Browser Extension Clickjacking

Clickjacking vulnerability in browser extensions works similarly to web applications. Through clicking, the user unknowingly performs an action that causes their browser extension to execute malicious activity such as data exfiltration, functionality deactivation, stored note deletion and others.

Browser extension clickjacking can currently be categorized into 2 types:

- **IFRAME-based** - publicly described type (web\_accessible\_resources)

- **DOM-based - new described type**

## IFRAME-based

The iframe-based type **loads an external resource via an iframe**.

A publicly documented clickjacking technique for browser extensions **exploits misconfigured web\_accessible\_resources in the manifest.json file.**

> **manifest.json is the main configuration file of a browser extension**. It contains basic information such as the extension’s name, version, and description, as well as settings that define what scripts, icons, and permissions the extension uses. Without this file, the browser cannot recognize or run the extension.  
>   
> ```
> Chromium-based path:  
> chrome-extension://**<extension\_ID>**/manifest.json
> 
> Mozilla Firefox path:  
> moz-extension://**<extension\_ID>**/manifest.json
> 
> Local device path:  
> %LocalAppData%\\Google\\Chrome\\User Data\\Default\\Extensions\\**<extension\_ID>**\\<version>\\manifest.json
> ```
> 

In the web\_accessible\_resources part, developers explicitly define files (HTML, scripts, styles, images) that should be accessible from web pages outside the extension interface itself. If developers don't specify sufficient restrictions, attackers can abuse these resources.

### Example

Although this isn't a new technique and information has been available for several years, some extensions still have this security vulnerability.

Due to incorrect web\_accessible\_resources definition, it was possible to load the entire password manager UI in an iframe.

![](attachments/Marek%20Tóth%20-%20DOM-based%20Extension%20Clickjacking%20Your%20Password%20Manager%20Data%20at%20Risk.png)

![](attachments/Marek%20Tóth%20-%20DOM-based%20Extension%20Clickjacking%20Your%20Password%20Manager%20Data%20at%20Risk2.png)

**With 4 clicks, it was possible to share all items from the password manager** to an attacker's account.

Because the victim was required to perform several clicks (select all, share items...), a fake Cloudflare captcha page was created as proof-of-concept. By completing the captcha, the attacker gets all data from the password manager.

### Manifest

With Manifest V3, resource access is now explicitly controlled through the matches parameter, requiring developers to define the exact origins from which resources may be loaded.

```json
"web_accessible_resources": [
  {
    "resources": ["image.png", "script.js"],
    "matches": ["https://example.com/*"]
  }
]
```

## DOM-based Extension Clickjacking

A new clickjacking technique where **a malicious script manipulates UI elements that browser extensions inject into the DOM**.

The principle is that a browser extension injects elements into the DOM, which an attacker can then make invisible using javascript.
![](attachments/Marek%20Tóth%20-%20DOM-based%20Extension%20Clickjacking%20Your%20Password%20Manager%20Data%20at%20Risk3.png)
To change visibility is used opacity:0 to various elements or overlay UI components.

In my case, focusing on password managers, the manual autofill feature was used to increase impact.

**Password Manager Exploit Steps:**

- Create an intrusive element (cookie consent, cloudflare captcha etc.)

- Create a form (login, personal data... )

- Set transparency for the form (opacity: 0.001)

- Use focus() for the form input → the autofill dropdown menu will appear

- Make the UI invisible with **DOM-based Extension Clickjacking**

- Victim accepts/rejects cookies = clicks on the invisible UI  
	→ data will be filled into the newly created form (2.)  
	→ attacker gets data from the form values

![](https://marektoth.com/images/steps.gif)

DOM-based Extension Clickjacking can be divided into several types/categories. Each manipulates DOM elements differently, but the **result is always the same - the UI is invisible but clickable**.

```json
DOM-based Extension Clickjacking
├── Extension Element
│       ├── Root Element
│       └── Child Element
├── Parent Element
│       ├── BODY
│       └── HTML
└── Overlay
        ├── Partial Overlay
        └── Full Overlay
```