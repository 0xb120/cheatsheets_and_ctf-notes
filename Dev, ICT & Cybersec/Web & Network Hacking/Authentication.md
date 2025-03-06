# Authentication 101

>[!question] What is authentication? 
>**Authentication** is the process of verifying the identity of a given user or client. In other words, it involves making sure that they really are who they claim to be

There are three authentication factors into which different types of authentication can be categorized:

- Something you **know**, such as a password or the answer to a security question. These are sometimes referred to as "knowledge factors".
- Something you **have**, that is, a physical object like a mobile phone or security token. These are sometimes referred to as "possession factors".
- Something you **are** or **do**, for example, your biometrics or patterns of behavior. These are sometimes referred to as "inherence factors".

>[!important] Authorization vs Authentication
>Authentication is the process of verifying that a user really is who they claim to be, whereas authorization involves verifying whether a user is allowed to do something.


## HTTP Authentication

The HTTP-based login is displayed as a browser pop-up. It does not require cookie design, sessions or even programming, the whole process is handled by the Web Server. It can be found as a barrier for test pages or access zones for a few people or as additional protection to the normal Web App login.

### Basic Authentication

Given a list of credentials in a file, access will require that the data entered matches that in the file. Typically, the data is encoded in Base64, but some web servers or external services encode the data differently, for example using bcrypt.
Authentication data are saved within `.htpasswd` and `.htaccess` files.

`User --> WEB SERVER --> Searches within .HTACCESS --> Asks user for password --> Checks within .HTPASSWD --> Log in / Error`

**IMPORTANT**: It is necessary that HTTP BA is performed in an HTTPS context. Since encryption is performed on the server side, the data exchange between client and server takes place in plain text, and a MITM attack could easily reveal the password used.

### Digest Authentication

Similar to HTTP BA, but data transfer takes place after it has already been encrypted, so it is immune to MITM

## Web App Authentication

Authentication based on a Web App requires good knowledge and mastery of programming, since it is no longer the Web Server that takes care of it, but the Web platform itself.
There is no precise scheme for this process, but roughly speaking there are three main categories:

- Use of "hand-designed" schemas: this is more crude but also more customizable. Typical of this category are [SQL Injection](SQL%20Injection.md) attacks.
- Use of schemas provided by frameworks: For example, use of Lavarel, CodeIgniter, etc. The vulnerability grip is given by the version and model in use.
- Use of external schemas, such as Google or Facebook: authentication is handled by third parties, who then pass API codes to the main app. The web app does not handle authentication.

## Multi-factor authentication

>[!summary]
>Multi-factor authentication is when two or more different types of authentication are required to login (eg. something you know + something you have)

Verifying biometric factors is impractical for most websites. However, it is increasingly common to see both mandatory and optional two-factor authentication (2FA) based on **something you know** and **something you have**. This usually requires users to enter both a traditional password and a temporary verification code from an out-of-band physical device in their possession.

>[!warning]
Two-factor authentication is demonstrably more secure than single-factor authentication. However, as with any security measure, it is only ever as secure as its implementation. Poorly implemented two-factor authentication can be beaten, or even bypassed entirely, just as single-factor authentication can.


## Other authentication mechanisms & third-party auth

In addition to the basic login functionality, most websites provide supplementary functionality to allow users to manage their account. For example, users can typically change their password or reset their password when they forget it. These mechanisms can also introduce vulnerabilities that can be exploited by an attacker.

Furthermore, many websites now uses third-party authentication mechanism, like [OAuth 2.0](../Dev,%20scripting%20&%20OS/OAuth%202.0.md), STS, etc. Also this implementations can be equally vulnerable to classic one and may be exploited by attackers.

---

# Common attacks

- [Password-based login attacks](Authentication%20Attacks.md#Password-based%20login%20attacks)
- [Multi-factor authentication attacks](Authentication%20Attacks.md#Multi-factor%20authentication%20attacks)
- [Other authentication mechanisms](Authentication%20Attacks.md#Other%20authentication%20mechanisms)
