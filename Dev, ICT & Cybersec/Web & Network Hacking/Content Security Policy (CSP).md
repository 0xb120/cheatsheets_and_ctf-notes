>[!summary]
>Content Security Policy (CSP) is a detection and prevention mechanism that provides mitigation against attacks such as [Cross-Site Scripting (XSS)](Cross-Site%20Scripting%20(XSS).md) [^1] and [Clickjacking](Clickjacking.md). CSP is usually implemented in the web server as a return header of the form: `Content-Security-Policy: policy` where policy is a string of policy directives separated by semicolons.

[^1]: https://portswigger.net/web-security/cross-site-scripting/preventing#:~:text=Mitigating%20XSS%20using%20content%20security%20policy%20(CSP)

The CSP provides the client browser with information about permitted resources that a page can load or from which can be embedded in frames.

Some examples:
- `Content-Security-Policy: frame-ancestors 'self';`: CSP whitelists frames to the same domain only
- `Content-Security-Policy: frame-ancestors normal-website.com;`: framing restricted to a named site
- `Content-Security-Policy: script-src 'self';`: scripts are allowed only from the same origin ([SOP](Same-origin%20policy%20(SOP).md))
- `Content-Security-Policy: script-src https://scripts.normal-website.com;`: scripts are allowed only from a specific domain
- `Content-Security-Policy: img-src 'self';`: images are allowed only from the same origin ([SOP](Same-origin%20policy%20(SOP).md))
- `Content-Security-Policy: img-src https://scripts.normal-website.com;`: images are allowed only from a specific domain

In addition to whitelisting specific domains, content security policy also provides two other ways of specifying trusted resources: nonces and hashes:

- The CSP directive can specify a nonce (a random value) and the same value must be used in the tag that loads a script. If the values do not match, then the script will not execute. To be effective as a control, the nonce must be securely generated on each page load and not be guessable by an attacker.
- The CSP directive can specify a hash of the contents of the trusted script. If the hash of the actual script does not match the value specified in the directive, then the script will not execute. If the content of the script ever changes, then you will of course need to update the hash value that is specified in the directive.

## CSP Evasion

- [Evading CSP with DOM-based dangling markup](https://portswigger.net/research/evading-csp-with-dom-based-dangling-markup)
- [Bypassing CSP with policy injection](https://portswigger.net/research/bypassing-csp-with-policy-injection)
- [AngularJS CSP bypass](../Dev,%20scripting%20&%20OS/AngularJS.md#AngularJS%20CSP%20bypass)


