---
Category:
  - B2R
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags:
  - GTFObins
  - OpenNetAdmin
  - code-review
  - command-injection
  - credentials-reuse
  - hardcoded-credentials
  - privesc/nano
  - port-forwarding
  - ssh-keys-cracking
  - Linux
---
## Improved skills

- Apache configuration review
- Port Forwarding
- SSH keys cracking
- Exploiting GTFO binaries

## Used tools

- nmap
- dirbuster
- searchsploit
- metasploit
- pspy64
- ssh2john
- john

---

## Introduction & Foothold

Let’s start as every time with an **nmap** scan:

```bash
root@kali:~/Documents/CTF/Machine/OpenAdmin# nmap -sV -A -O  10.10.10.171
Starting Nmap 7.80 ( https://nmap.org ) at 2020-02-11 11:28 CET
Nmap scan report for 10.10.10.171
Host is up (0.060s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 4b:98:df:85:d1:7e:f0:3d:da:48:cd:bc:92:00:b7:54 (RSA)
|   256 dc:eb:3d:c9:44:d1:18:b1:22:b4:cf:de:bd:6c:7a:54 (ECDSA)
|_  256 dc:ad:ca:3c:11:31:5b:6f:e6:a4:89:34:7c:9b:e5:50 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.80%E=4%D=2/11%OT=22%CT=1%CU=31767%PV=Y%DS=2%DC=T%G=Y%TM=5E4281D
OS:C%P=x86_64-pc-linux-gnu)SEQ(SP=105%GCD=1%ISR=107%TI=Z%CI=Z%II=I%TS=A)SEQ
OS:(SP=105%GCD=1%ISR=107%TI=Z%CI=Z%TS=A)OPS(O1=M54DST11NW7%O2=M54DST11NW7%O
OS:3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST11NW7%O6=M54DST11)WIN(W1=7120%W2=
OS:7120%W3=7120%W4=7120%W5=7120%W6=7120)ECN(R=Y%DF=Y%T=40%W=7210%O=M54DNNSN
OS:W7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%D
OS:F=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O
OS:=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W
OS:=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%R
OS:IPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 1720/tcp)
HOP RTT      ADDRESS
1   61.58 ms 10.10.14.1
2   61.83 ms 10.10.10.171

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 26.18 seconds
```

We can see that only two services are exposed: a non-vulnerable SSH version, and an Apache web server on port 80 with a default page.

In order to enumerate all the possible sub-pages of the site and find an entry point, let’s start a **Dirbuster** scan. Among the first results of the tool execution we find these records:

- `/` 
- `/index.php` 
- `/icons/` 
- `/music/` 
- `/ona/`

Visiting each page, arrived at http://10.10.10.171/ona/ we realize that in front of us we have a **vulnerable version of OpenNetAdmin**, a web application for the network administration. Let’s use **searchsploit** to find out if there are exploits fitting this particular version… the [47772](https://www.exploit-db.com/exploits/47772) seems to be what we need, a **Command Injection** on **OpenNetAdmin 18.1.1**. We just have to download it, import it into **metasploit**, set the correct parameters and run it.

[Offensive Security's Exploit Database Archive](https://www.exploit-db.com/exploits/47772)

## Lateral movement to Jimmy

Analyzing the contents of the `/etc/passwd` file we notice that the main users on the box are actually two: **Jimmy** and **Joanna**.

After having take a look at the machine and launching *LinEnum.sh* without success, I decided to check better the processes of the machine via **pspy64**.

![pspy64.png](../../zzz_res/attachments/pspy64.png)

By observing the various processes, one particular line attracts my attention:

`2020/02/14 22:35:20 CMD: UID=33 PID=7150 | sh -c php -l /opt/ona/www/local/config/database_settings.inc.php`

I decide to analyze the contents of the **database_settings.inc.php** file:

```php
<?php

$ona_contexts=array (
  'DEFAULT' =>
  array (
    'databases' =>
    array (
      0 =>
      array (
        'db_type' => 'mysqli',
        'db_host' => 'localhost',
        'db_login' => 'ona_sys',
        'db_passwd' => 'n1nj4W4rri0R!',
        'db_database' => 'ona_default',
        'db_debug' => false,
      ),
    ),
    'description' => 'Default data context',
    'context_color' => '#D3DBFF',
  ),
);
```

**We have a password!** Let’s try using it to log in as **jimmy**… it works!

## Lateral movement to Joanna

Now that we have a shell as jimmy, our goal is to become **joanna**, as this user will take us to the root.

Since we have new privileges compared to before, we will be able to enumerate the machine in more depth, as for the `/var/www/internal/folder`, which seems to suggest the *presence of a web page that can only be visited from inside* the machine, so before we could not find it via **dirbuster**.

![internal.png](../../zzz_res/attachments/internal.png)

The confirmation is obtained from the *Apache configuration file*, which tells us that on **port 52846** there is a virtual host running as **joanna**, exposing the pages contained in `/var/www/internal/`.

![sites-available.png](../../zzz_res/attachments/sites-available.png)

At this point, there are different ways to go over the obstacle:

1. SSH Port Forwarding
2. local **curl**
3. Write a PHP shell in the `/var/www/internal/` directory

>[!info]
>In this writeup we will deal with the port forwarding approach.

I used the OpenAdmin box as an **SSH tunnel** on the Kali **local port 12345**, and through **SwitchyOmega** I used port 12345 as a **proxy** so that when I visited http://127.0.0.1/52864 I was actually visiting locally the OpenAdmin machine.

![internal_browser.png](../../zzz_res/attachments/internal_browser.png)

In this way we can interact with the web application as if it was an exposed application.

Having access to the **/var/www/internal/folder**, it was possible to **review the sources contents** in search of any vulnerabilities or bypass techniques. The **index.php** page, for example, to log in a user, checks the supplied input with a hard-coded hash:

```php
<?php
            $msg = '';

            if (isset($_POST['login']) && !empty($_POST['username']) && !empty($_POST['password'])) {
              if ($_POST['username'] == 'jimmy' && hash('sha512',$_POST['password']) == '00e302ccdcf1c60b8ad50ea50cf72b939705f49f40f0dc658801b4680b7d758eebdc2e9f9ba8ba3ef8a8bb9a796d34ba2e856838ee9bdde852b8ec3b3a0523b1') {
                  $_SESSION['username'] = 'jimmy';
                  header("Location: /main.php");
              } else {
                  $msg = 'Wrong username or password.';
              }
            }
         ?>
```

It is therefore easy to bypass the control, either by deciding to **crack the hash** (which turns out to be a SHA512 containing the password *Revealed*), or, since we have access to the source in read/write, replacing the control with a control at will.

Once the control is bypassed, we will find ourselves on the **main.php** page with **Joanna**’s private RSA certificate shown on the video.

![joanna_RSA.png](../../zzz_res/attachments/joanna_RSA.png)

Let’s copy it into the **joanna.txt** file and give it to **ss2john** in order to generate a hash that can be cracked with **john**:

![joanna_ssh_bruteforce.png](../../zzz_res/attachments/joanna_ssh_bruteforce.png)

We have Joanna’s password!

## Privilege Escalation

The Privilege Escalation process is the easiest and fastest I’ve ever seen. By running the `sudo -l` command we notice how the user **joanna** can open the **/opt/priv** file via the **nano** text editor as root user.

![privesc_1.png](../../zzz_res/attachments/privesc_1.png)

We just have to run the command and, within nano, use the **code execution feature** to be able to run commands as root!

![privesc_2.png](../../zzz_res/attachments/privesc_2.png)

# Trophy

>[!quote]
> It's never too late to start.
>
>\- Me, Myself and I

>[!success]
**User.txt**
c9b2cf07d40807e62af62660f0c81b5f

>[!success]
>**Root.txt**
>2f907ed450b361b2c2bf4e8795d5b561

