---
Category: Web
Difficulty: Easy
Platform: HackTheBox
Retired: false
Status: 3. Complete
Tags:
  - broken-auth
  - brute-force
  - SQL-Injection
---
>[!quote]
> *Who is lucky enough to be included in the phonebook?*

# Set up

-

# Information Gathering

Home page:

![Pasted image 20210505212220.png](../../zzz_res/attachments/Pasted_image_20210505212220.png)

Authentication bypass using `*`:`*`

![Pasted image 20210505224026.png](../../zzz_res/attachments/Pasted_image_20210505224026.png)

Hidden folder:

![Pasted image 20210505212636.png](../../zzz_res/attachments/Pasted_image_20210505212636.png)

![Pasted image 20210505212748.png](../../zzz_res/attachments/Pasted_image_20210505212748.png)

User disclosed containing a `'`

![Pasted image 20210505224420.png](../../zzz_res/attachments/Pasted_image_20210505224420.png)

# The Bug

Because the login form is vulnerable to “SQL Injection” using the `*` wildcard, it is possible to brute-force the credentials for the disclosed user.

# Exploitation

![Pasted image 20210505232332.png](../../zzz_res/attachments/Pasted_image_20210505232332.png)

![Pasted image 20210505232919.png](../../zzz_res/attachments/Pasted_image_20210505232919.png)

# Flag

>[!success]
>`HTB{d1rectory_h4xx0r_is_k00l}`

