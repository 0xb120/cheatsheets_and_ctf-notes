Session Hijacking consists of taking over the identity of a user, bypassing authentication mechanisms.

Remembering that the server generates a session token which it sends in the HTTP header with the cookies, if an attacker managed to replicate it, he could pretend to be another user. The most common session attacks are:

- [Cross-Site Request Forgery (CSRF)](Cross-Site%20Request%20Forgery%20(CSRF).md) aka Session Abuse
- [Stealing cookies and session tokens with XSS](Cross-Site%20Scripting%20(XSS).md#Steal%20of%20cookies%20and%20session%20tokens)
- Stealing tokens with [Open Redirection](Open%20Redirection.md)
- Session harvesting
	- sniffing (MITM/MITB)
	- from web logs or session data (eg. `PHP.ini`, `Manager` [^manager], `StateServer` [^asp-net-sessions], etc.)
	- from database access
- Session prediction
- [Session Fixation](Session%20Fixation.md)
- [SQL Injection](SQL%20Injection.md) and exfiltration of credentials and session ids

[^manager]: [The Manager Component](https://tomcat.apache.org/tomcat-6.0-doc/config/manager.html), tomcat.apache.org
[^asp-net-nessions]: [Introduction to ASP.NET Sessions](https://www.c-sharpcorner.com/UploadFile/225740/introduction-of-session-in-Asp-Net/), c-sharpcorner.com

# Session Prediction

The practice of guessing a Session ID in order to bypass authentication. There are no special practices for exploiting these vulnerabilities other than by analyzing tokens and their encryption mechanisms.
