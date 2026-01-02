---
Category:
  - Mobile
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags: [Android, certificate-pinning-bypass, frida, reversing]
---
>[!quote]
> *This app has stored my credentials and I can only login automatically. I tried to intercept the login request and restore my password, but this seems to be a secure connection. Can you help bypass this security restriction and intercept the password in plaintext?*


# Set up

- 1. Install this application in an API Level 29 or earlier (i.e. Android 10.0 (Google APIs)).

# Information Gathering

Application perform certificate pinning

![Untitled](../../zzz_res/attachments/Pinned%205461569accbf4ba19c27981f1f7ea046.png)

# The Bug

-

# Exploitation

Download the Frida certificate pining bypass script from [here](https://codeshare.frida.re/@pcipolloni/universal-android-ssl-pinning-bypass-with-frida/)

```bash
# Already pushed and started frida server
$ adb push burpca-cert-der.crt /data/local/tmp/cert-der.crt

$ frida -U -f com.example.pinned -l unpin.js --no-pause
     ____
    / _  |   Frida 15.1.17 - A world-class dynamic instrumentation toolkit
   | (_| |
    > _  |   Commands:
   /_/ |_|       help      -> Displays the help system
   . . . .       object?   -> Display information about 'object'
   . . . .       exit/quit -> Exit
   . . . .
   . . . .   More info at https://frida.re/docs/home/
   . . . .
   . . . .   Connected to Google Nexus 5X (id=192.168.164.104:5555)
Spawned `com.example.pinned`. Resuming main thread!
[Google Nexus 5X::com.example.pinned ]->
[.] Cert Pinning Bypass/Re-Pinning
[+] Loading our CA...
[o] Our CA Info: CN=PortSwigger CA, OU=PortSwigger CA, O=PortSwigger, L=PortSwigger, ST=PortSwigger, C=PortSwigger
[+] Creating a KeyStore for our CA...
[+] Creating a TrustManager that trusts the CA in our KeyStore...
[+] Our TrustManager is ready...
[+] Hijacking SSLContext methods now...
[-] Waiting for the app to invoke SSLContext.init()...
[o] App invoked javax.net.ssl.SSLContext.init...
[+] SSLContext initialized with our custom TrustManager!
```

![Untitled](../../zzz_res/attachments/Pinned%205461569accbf4ba19c27981f1f7ea046%201.png)

![Untitled](../../zzz_res/attachments/Pinned%205461569accbf4ba19c27981f1f7ea046%202.png)

# Flag

>[!success]
>`HTB{trust_n0_1_n0t_3v3n_@_c3rt!}`

# Video

<iframe width="660" height="415" src="https://www.youtube.com/embed/CJR_BSIStmE" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

