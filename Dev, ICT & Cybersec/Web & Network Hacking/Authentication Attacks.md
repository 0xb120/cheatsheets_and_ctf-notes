# Authentication 101

>[!info] What is authentication? 
>**Authentication** is the process of verifying the identity of a given user or client. In other words, it involves making sure that they really are who they claim to be

There are three authentication factors into which different types of authentication can be categorized:

- Something you **know**, such as a password or the answer to a security question. These are sometimes referred to as "knowledge factors".
- Something you **have**, that is, a physical object like a mobile phone or security token. These are sometimes referred to as "possession factors".
- Something you **are** or **do**, for example, your biometrics or patterns of behavior. These are sometimes referred to as "inherence factors".

>[!important] Authorization vs Authentication
>Authentication is the process of verifying that a user really is who they claim to be, whereas authorization involves verifying whether a user is allowed to do something.

# Authentication types

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

Furthermore, many websites now uses third-party authentication mechanism, like OAuth, STS, etc. Also this implementations can be equally vulnerable to classic one and may be exploited by attackers.

---

# Common attacks

## Password-based login attacks

The security of the website would be compromised if an attacker is able to either obtain or guess the login credentials of another user.

Techniques:
- Enumerate or brute-force usernames online ([Passive information gathering (OSINT)](Passive%20information%20gathering%20(OSINT).md))
- Username enumerations using login/registration/password reset (look to status codes, error messages, response times)
- [Password Attacks](Password%20Attacks.md)
- Exploit flawed protection
	- Account locking disclosing registered users
	- Flawed brute-force protections resetting the counter every valid login or for any different IP
- [Session Attacks (CSRF, session stealing, etc.)](Session%20Attacks%20(CSRF,%20session%20stealing,%20etc.).md)
-  [Type Juggling](Type%20Juggling.md)
```http
POST /login HTTP/1.1
Host: 0a690029038e65bfc04ed2d1007d0057.web-security-academy.net
Cookie: session=Zb1D8NGHIvy10kSH26Xrwex4kxtvUPHZ
Content-Length: 1194
Sec-Ch-Ua: "Not?A_Brand";v="8", "Chromium";v="108", "Google Chrome";v="108"
Sec-Ch-Ua-Platform: "Windows"
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36
Content-Type: text/plain;charset=UTF-8
Accept: */*
Origin: https://0a690029038e65bfc04ed2d1007d0057.web-security-academy.net
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://0a690029038e65bfc04ed2d1007d0057.web-security-academy.net/login
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

{"username":"carlos","password":["123456",
"password",
"12345678",
"qwerty",
"123456789",
"12345",
"1234",
"111111",
"1234567",
...
"moscow"
],"":""}
```

## Multi-factor authentication attacks

- SIM swapping to intercept SMS-based verification codes
- [Business logic vulnerabilities](Business%20logic%20vulnerabilities.md) (eg. browsing the site before submitting the 2FA code, editing server responses to bypass client side validation, etc.)
- Brute-forcing 2FA codes
- Exploiting flawed two-factor verification logic (eg. log in using own credentials but then change the value of the account identifier)
- Exploiting other vulnerabilities (eg. [Host Header attacks](Host%20Header%20attacks.md)) or non-standard headers (eg. `X-Forwarded-for`, etc.) to intercept 2FA codes

## Other authentication mechanisms

- "Keep me log in" cookie brute-force or steal
- [Password Reset Poisoning](Password%20Reset%20Poisoning.md)
- Exploit application features (eg. password reset) to brute-force other user's password
- [Session Attacks (CSRF, session stealing, etc.)](Session%20Attacks%20(CSRF,%20session%20stealing,%20etc.).md)
- [Access control vulnerabilities](Access%20control%20vulnerabilities.md) in password reset logic

---

# Common protections

- Locking the account that the remote user is trying to access if they make too many failed login attempts (this protection do not stop password stuffing attacks or password spraying attacks)
- Blocking the remote user's IP address if they make too many login attempts in quick succession (this protection may be bypassed if the server has some misconfiguration that reset the IP's error count: eg. X-Forwarded-for, valid logins, etc.)
- User rate limiting
- https://portswigger.net/web-security/authentication/securing