---
Category:
  - Misc
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags: [esoteric-lang, malbolge]
---
>[!quote]
> *Download the provided file*


# Set up

- Download the provided file

# Information Gathering

```bash
┌──(kali㉿kali)-[~/Downloads]
└─$ cat inferno.txt 
RCdgXyReIjdtNVgzMlZ4ZnZ1PzFOTXBMbWwkakdGZ2dVZFNiYn08eyldeHFwdW5tM3Fwb2htZmUrTGJnZl9eXSNhYFleV1Z6PTxYV1ZPTnJMUUpJTkdrRWlJSEcpP2MmQkE6Pz49PDVZenk3NjU0MzIrTy8uJyYlJEgoIWclJCN6QH59dnU7c3JxdnVuNFVxamlubWxlK2NLYWZfZF0jW2BfWHxcW1pZWFdWVVRTUlFQMk5NRktKQ0JmRkU+JjxgQDkhPTw1WTl5NzY1NC0sUDAvby0sJUkpaWh+fSR7QSFhfXZ7dDpbWnZvNXNyVFNvbm1mLGppS2dgX2RjXCJgQlhXVnpaPDtXVlVUTXFRUDJOR0ZFaUlIR0Y/PmJCJEA5XT08OzQzODFVdnUtMiswLygnSysqKSgnfmZ8Qi8= 

┌──(kali㉿kali)-[~/Downloads]
└─$ cat inferno.txt | base64 -d
D'`_$^"7m5X32Vxfvu?1NMpLml$jGFggUdSbb}<{)]xqpunm3qpohmfe+Lbgf_^]#a`Y^WVz=<XWVONrLQJINGkEiIHG)?c&BA:?>=<5Yzy765432+O/.'&%$H(!g%$#z@~}vu;srqvun4Uqjinmle+cKaf_d]#[`_X|\[ZYXWVUTSRQP2NMFKJCBfFE>&<`@9!=<5Y9y7654-,P0/o-,%I)ih~}${A!a}v{t:[Zvo5srTSonmf,jiKg`_dc\"`BXWVzZ<;WVUTMqQP2NGFEiIHGF?>bB$@9]=<;4381Uvu-2+0/('K+*)('~f|B/
```

# The Bug

-

# Exploitation

![inferno_1.png](../../zzz_res/attachments/inferno_1.png)

# Flag

>[!success]
>`HTB{!1t_1s_just_M4lb0lg3_l4ngu4g3!}`