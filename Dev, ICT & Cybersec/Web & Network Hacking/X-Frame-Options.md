>[!summary]
>The header provides the website owner with control over the use of iframes or objects so that inclusion of a web page within a frame can be confirmed or not.

`X-Frame-Options` can have different values:

- **deny**: the site cannot be imbedded within any iframe (`X-Frame-Options: deny`)
- **sameorigin**: framing can be restricted to the same origin (`X-Frame-Options: sameorigin`)
- **allow-from**: only specific URLs can embed the page using iframe (`X-Frame-Options: allow-from https://normal-website.com`)

X-Frame-Options is not implemented consistently across browsers. However, when properly applied in conjunction with [Content Security Policy (CSP)](Content%20Security%20Policy%20(CSP).md) as part of a multi-layer defense strategy it can provide effective protection against clickjacking attacks.