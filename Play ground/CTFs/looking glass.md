---
Category:
  - Web
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags:
  - RCE
  - command-injection
---
>[!quote]
> *We've built the most secure networking tool in the market, come and check it out!*


# Set up

-

# Information Gathering

![Pasted image 20210818165923.png](../../zzz_res/attachments/Pasted_image_20210818165923.png)

![Pasted image 20210818170154.png](../../zzz_res/attachments/Pasted_image_20210818170154.png)

Request sent:

```
POST / HTTP/1.1
Host: 188.166.173.208:32110
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 52
Origin: <http://188.166.173.208:32110>
Connection: close
Referer: <http://188.166.173.208:32110/>
Upgrade-Insecure-Requests: 1

test=ping&ip_address=188.166.173.208&submit=Test
```

# Exploitation

![Pasted image 20210818170501.png](../../zzz_res/attachments/Pasted_image_20210818170501.png)

![Pasted image 20210818170810.png](../../zzz_res/attachments/Pasted_image_20210818170810.png)

# Flag

>[!success]
>`HTB{I_f1n4lly_l00k3d_thr0ugh_th3_rc3}`

