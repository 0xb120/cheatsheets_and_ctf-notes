---
Category:
  - Web
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags: [authentication-bypass, broken-auth]
---
>[!quote]
> Who needs session integrity these days?

# Set up

# Information Gathering

![Pasted image 20210818171438.png](../../zzz_res/attachments/Pasted_image_20210818171438.png)

- Create an account
- login

![Pasted image 20210818171613.png](../../zzz_res/attachments/Pasted_image_20210818171613.png)

![Pasted image 20210818172630.png](../../zzz_res/attachments/Pasted_image_20210818172630.png)

`eyJ1c2VybmFtZSI6InRlc3QifQ%3D%3D` = `{"username":"test"}`

# The Bug

# Exploitation

`{"username":"admin"}` = `eyJ1c2VybmFtZSI6ImFkbWluIn0=`

![Pasted image 20210818172900.png](../../zzz_res/attachments/Pasted_image_20210818172900.png)

# Flag

`HTB{s3ss10n_1nt3grity_1s_0v3r4tt3d_4nyw4ys}`