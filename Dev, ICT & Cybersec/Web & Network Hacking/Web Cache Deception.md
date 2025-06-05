>[!question] What is Web Cache Deception?
>-  Web cache deception is an advanced technique whereby attacker causes the application to store some sensitive content belonging to another user in the cache, and the attacker then retrieves this content from the cache.

>[!info] Pre-requisites:
>- [Web Cache](Web%20Cache.md)

Web Cache Deception can be achieved [using request smuggling vulnerabilities](Exploiting%20HTTP%20Request%20Smuggling.md#Using%20request%20smuggling%20to%20perform%20web%20cache%20poisoning), but it can be obtained also through [Cross-Site Request Forgery (CSRF)](Cross-Site%20Request%20Forgery%20(CSRF).md) forcing a user into clicking a link containing an attacker known cache-buster as well as a specific extension ignored by the web-server but cached when requested. [^deception1]

[^deception1]: [Rachid.A - A web cache deception chained to a CSRF, the recipe](../../Readwise/Articles/Rachid.A%20-%20A%20web%20cache%20deception%20chained%20to%20a%20CSRF,%20the%20recipe.md#^972f41)

Some examples are:
```http
https://example.com/private_info.js?cachebuster=1
https://example.com/private_info/.css?cachebuster=1
https://example.com/private_info/;.png?cachebuster=1
```

Once the attack vector has been found, simply send the link containing its cache-buster to the victim. When he clicks on the link, his personal information — _the nature of which will depend on the web app_ — will be **accessible** to the attacker **via this same link**.

## Web Cache Deception vulnerabilities
- [ChatGPT Account Takeover - Wildcard Web Cache Deception](../../Readwise/Articles/Harel%20Security%20Research%20-%20ChatGPT%20Account%20Takeover%20-%20Wildcard%20Web%20Cache%20Deception.md)