>[!Question] What is Authentication?
>[Authentication 101](Authentication.md)

# Authentication Bypass Attacks

- Exploit general misconfigurations on the Authorization or Authentication logic to access restricted sites area
	- [HTTP Verb Tampering](HTTP%20Verb%20Tampering.md)
	- [Mass Assignment](Mass%20Assignment.md)
- [SQL Injection leading to authentication bypass](SQL%20Injection.md#Authentication%20Bypass)
	- Injection inside parameters, headers, cookies, etc. leading to direct bypass
	- Credential leaks
	- Token leaks
	- Overwriting other user's data
- [Session Attacks and Session Prediction](Session%20Attacks%20and%20Session%20Prediction.md)

---

## Password-based login attacks

The security of the website would be compromised if an attacker is able to either obtain or guess the login credentials of another user.

Techniques:
- Enumerate or brute-force usernames from online resources ([Passive information gathering (OSINT)](Passive%20information%20gathering%20(OSINT).md))
- Username enumerations using login/registration/password reset (look to status codes, error messages, response times, leaked info)
- Plain [Password Attacks](Password%20Attacks.md)
	- [Password Guessing Attacks](Password%20Attacks.md#Password%20Guessing%20Attacks)
	- [Common Network Service Brute-Forces](Password%20Attacks.md#Common%20Network%20Service%20Brute-Forces)
- Exploit flawed protection implementation
	- Account locking mechanisms disclosing registered users
	- Flawed brute-force protections resetting the counter every valid login or for any different IP
-  [Type Juggling (aka type confusion)](Type%20Juggling%20(aka%20type%20confusion).md) on logins
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

---

## Multi-factor authentication attacks

- SIM swapping to intercept SMS-based verification codes
- [Business logic vulnerabilities](Business%20logic%20vulnerabilities.md) (eg. browsing the site before submitting the 2FA code, editing server responses to bypass client side validation, etc.)
- Brute-forcing 2FA codes
- Exploiting flawed two-factor verification logic (eg. log in using own credentials but then change the value of the account identifier)
- Exploiting other vulnerabilities (eg. [Host Header attacks](Host%20Header%20attacks.md)) or 3non-standard headers (eg. `X-Forwarded-for` [^xff], etc.) to intercept 2FA codes

[^xff]: [HTTP Toolkit - What Is X-Forwarded-for and When Can You Trust It](../../Readwise/Articles/HTTP%20Toolkit%20-%20What%20Is%20X-Forwarded-for%20and%20When%20Can%20You%20Trust%20It.md)

---
## Other authentication mechanisms attacks

- "Keep me log in" cookie brute-force or steal
- Password reset attacks
	- [Password Reset Poisoning](Password%20Reset%20Poisoning.md)
	- Predictable password reset token (eg. [Sandwich Attack](../../Readwise/Articles/Nathan%20Touati%20-%20Sandwich%20Attacks%20From%20Reset%20Password%20to%20Account%20Takeover.md) - brute force UUIDv1 token included in a limited range, [Time-sensitive attacks](Race%20Condition.md#Time-sensitive%20attacks), etc.)
	- Leaking password reset token
- Exploit application features (eg. password reset) to brute-force other user's password
- [Access control vulnerabilities](Access%20control%20vulnerabilities.md) in password reset logic
- [OAuth 2.0](../Dev,%20scripting%20&%20OS/OAuth%202.0.md) and [OAuth 2.0 attacks](OAuth%202.0%20attacks.md)

>[!tip] Use Race Condition techniques to discover time-sensitive attacks
>Using [Race Condition](Race%20Condition.md) technique may allow you to perform [Time-sensitive attacks](Race%20Condition.md#Time-sensitive%20attacks) and eventually bypass authentication, as in the case of timestamp usage when generating tokens, etc.

---

# Common protections

- Locking the account that the remote user is trying to access if they make too many failed login attempts (this protection do not stop password stuffing attacks or password spraying attacks)
- Blocking the remote user's IP address if they make too many login attempts in quick succession (this protection may be bypassed if the server has some misconfiguration that reset the IP's error count: eg. `X-Forwarded-for`, valid logins, etc.)
- User rate limiting
- [Captcha](../Dev,%20scripting%20&%20OS/Captcha.md)
- https://portswigger.net/web-security/authentication/securing