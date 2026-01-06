---
author: b4dsheep
aliases:
  - Bypassing Certificate Pinning on Flutter-Based Android Apps. A New Guide.
tags:
  - readwise/articles
url: https://blog.mindedsecurity.com/2024/05/bypassing-certificate-pinning-on.html?__readwiseLocation=
created: 2025-04-28
---
# Bypassing Certificate Pinning on Flutter-Based Android Apps. A New Guide.

![rw-book-cover](https://blogger.googleusercontent.com/img/a/AVvXsEiLZOgVn35LqPDpfHUk6wcqLOjrvV6JioR2CPitS6-4WhuB6jSaudX8wXAwTaHk3U7ebucLS-R5vO6GRTF2dC-cjzUcm-qv1IQLhkB-zDUYLavj06fTkgqAbwsix1lSP2klyN8rqETtdsCcQFAv_FOKE9AGSNQ0TOyi3q5sxBSkyE_lhziAZbmO1DDnroxg=s72-c)

One of the foundational steps in analyzing a mobile application is establishing a Man-in-the-Middle (MitM) position to inspect HTTP/S traffic. While standard tools and techniques often suffice for Java or Kotlin-based Android apps, [Flutter](../../Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/Flutter.md) introduces a unique set of challenges that render traditional bypass methods ineffective.

This guide outlines a reliable methodology to intercept traffic and bypass certificate pinning in Flutter-based Android applications, using a custom testbed available on [GitHub](https://github.com/mindedsecurity/flutter-cert-check-bypass/).

---

## 1. Why Flutter is Different

To understand why standard tools (like [Xposed!](../../Dev,%20ICT%20&%20Cybersec/Tools/Xposed!.md) or standard [Frida](../../Dev,%20ICT%20&%20Cybersec/Tools/Frida.md) Java hooks) fail, we must look at the Flutter architecture.

- **Non-Standard Compilation:** Flutter apps are written in Dart. Unlike standard Android apps that run on the Dalvik/ART VM, **Flutter code is compiled to native machine code** (AOT) [^1] using the Android NDK.
    
- **Native Networking:** Flutter uses the `dart:io` library for I/O operations. It **manages sockets and connections directly rather than relying on the Android OS's high-level HTTP stack**.
    
- **Proxy Ignorance:** By default, the `dart:io` **HttpClient does not honor the Android system's proxy** settings.
    

### The APK Structure

When you unpack a Flutter APK, the core logic resides in the `resources/lib` folder:

| **Shared Object** | **Function**                                                               |
| ----------------- | -------------------------------------------------------------------------- |
| **libapp.so**     | Contains the **compiled Dart code specific to your application**.          |
| **libflutter.so** | Contains the **Flutter engine** and core functionalities (written in C++). |

**The Strategy:** Since the real SSL checks are implemented in native code within `libflutter.so`, we must reverse engineer this library and hook the native functions, rather than Java classes.

---

## 2. Phase I: Forcing Traffic Redirection

Before bypassing the certificate check, we must first force the application to route traffic through our proxy (e.g., Burp Suite), as it ignores system settings.

### The IPTables Approach

We can use `iptables` [^2] on a rooted device to redirect outgoing traffic from the app to a local port, and then reverse-map that port to our analysis machine.

**Prerequisites:**

- A rooted Android device connected via USB.
    
- Burp Suite running on the host machine.
    

**Step-by-Step Redirection:**

1. Access the Shell:
    
    Run an [adb](../../Dev,%20ICT%20&%20Cybersec/Tools/adb.md) shell with root privileges: 
    
    `adb shell su`
    
2. Set the Redirect Rule:
    
    Redirect all TCP traffic destined for port 443 to the local port 8080:
    
    ```sh
    iptables -t nat -A OUTPUT -p tcp -d destination-host.domain --dport 443 -j DNAT --to-destination 127.0.0.1:8080
    ```
    
3. Reverse Port Mapping:
    
    Map the Android device's port 8080 to the host machine's port 8080:
    
    `adb reverse tcp:8080 tcp:8080`
    

> **Critical Note:** You must configure your Burp Proxy listener to support **Invisible Proxying**. This allows Burp to accept non-proxy-aware connections that are being forcefully redirected to it.

---

## 3. Phase II: Bypassing the SSL Check

With traffic flowing to Burp, the **TLS handshake will fail** because the application implements Certificate Pinning. The Flutter engine relies on **BoringSSL** (Google's fork of OpenSSL) to handle these checks.

### Reverse Engineering `libflutter.so`

We need to find the function responsible for verifying the certificate chain.

1. **Decompile:** Unpack the APK and open `resources/lib/arm64-v8a/libflutter.so` in a disassembler like IDA Pro or [ghidra](../../Dev,%20ICT%20&%20Cybersec/Tools/ghidra.md).
    
2. **Locate Anchors:** The specific validation function utilizes two string literals located in the `.rodata` section:
    
    - `"ssl_client"`
        
    - `"ssl_server"`
        
3. **Trace Cross-References (XREFS):** Analyze where these strings are referenced. They will lead you to the function that initializes the SSL session.
    

Through this analysis, we identify the target validation function: `ssl_crypto_x509_session_verify_cert_chain`.

### The Frida Hook

To bypass the check, we do not need to rewrite the encryption logic; we simply need to force this verification function to report success.

We can write a [Frida](../../Dev,%20ICT%20&%20Cybersec/Tools/Frida.md) script to intercept calls to `ssl_crypto_x509_session_verify_cert_chain` and immediately return `0x01` (true).

**The Logic:**

```js
// Pseudo-code representation of the hook
Interceptor.attach(Module.findExportByName("libflutter.so", "ssl_crypto_x509_session_verify_cert_chain"), {
    onLeave: function(retval) {
        // Force return value to 1 (True)
        retval.replace(0x1);
    }
});
```

---
## Highlights


Bypassing Certificate Pinning on Flutter-based Android Apps. A new guide. [](https://read.readwise.io/read/01jqv0kkch82001mj0c0ew19bs)



One of the preliminary activities when analyzing mobile application, *more usually than not*, is to be able to sniff HTTP/S traffic via **a MitM proxy**. [](https://read.readwise.io/read/01jqv0kxqbmz0r6dk714exvaf6)



In this post I'll try to explain the methodology I used to make this possible for a **Flutter-based** **Android** sample application in a reliable way. [](https://read.readwise.io/read/01jqv0mcp8301ts2m0y1h1txkd)



in order to give some testbed, we created a basic Flutter application which fetches data from the URL [https://dummy.restapi.example.ltd/api/v1/employees](https://dummy.restapiexample.com/api/v1/employees), using code that correctly implements **[certificate pinning](https://owasp.org/www-community/controls/Certificate_and_Public_Key_Pinning)**.
The entire application code, the build and the [Frida](https://frida.re/) script are available on Github at the following link:
[https://github.com/mindedsecurity/flutter-cert-check-bypass/](https://github.com/mindedsecurity/flutter-cert-check-bypass/) [](https://read.readwise.io/read/01jqv0sxpe0q461m56808ey99s)



Why Flutter is so Different [](https://read.readwise.io/read/01jqv0wdhcwxwtya25tg1mevvh)



Flutter is a Google's open source portable toolkit able to create natively compiled applications for mobile, web, and desktop from a single codebase. [](https://read.readwise.io/read/01jqv0x1fb7e03ck8zq7fehy64)



Flutter applications are written in Dart, whilst the framework engine is built mainly in C++ [](https://read.readwise.io/read/01jqv0x8ywag0egttpehy5c0vh)



compilation obviously differs on the target platform: for Android, the engine is compiled using the Android's NDK [](https://read.readwise.io/read/01jqv0xqc5p1ej8c81ezxkganm)



more blazoned SSL pinning bypass [methods](https://mas.owasp.org/MASTG/techniques/android/MASTG-TECH-0012/), would not work, as the real checks are implemented in "native code", thus results compiled in machine code and many details may change among different framework builds. [](https://read.readwise.io/read/01jqv0ydjc21z6fprjkmzfggk2)



Flutter, at low level, uses **dart:io** library for managing socket, file and other I/O support in non browser-based applications. [](https://read.readwise.io/read/01jqv0z22ban0gr3vdbpajq263)



As **dart:io** documentation [says](https://api.dart.dev/stable/3.2.6/dart-io/HttpClient-class.html), by default the HttpClient uses the proxy configuration available from the environment, [](https://read.readwise.io/read/01jqv0zpemm84bqscjvwpxd4fc)



**Flutter Based APK Structure**
The **resource/lib** folder inside the APK contains the compiled Dart code (compiled to native binary code using the Dart VM's Ahead-of-Time compilation) for the Flutter application. [](https://read.readwise.io/read/01jqv11d5vgmmnqq3vpqkevzxy)



the two main shared objects are:
• **libapp.so**: which contains the compiled native code that corresponds to the Dart code of your Flutter app. This native code is executed when the Flutter app is launched on a device.
• **libflutter.so**: containing the native code of the Flutter engine, which powers the Flutter framework and enables the execution of Flutter apps on Android devices. This library includes the core functionalities of the Flutter framework [](https://read.readwise.io/read/01jqv11n1par9r817d7z01qs5w)



**Forcing Traffic Forwarding to the Proxy**
We know that the Flutter application will not honor the proxy settings at system level, thus we need a finer technique for fundamental part. [](https://read.readwise.io/read/01jqv14fa6vscff0xem6hyztat)



we have the possibility to set an *iptables* rule for the redirection of the desired HTTP traffic table throughout the a Burp Pro proxy instance [](https://read.readwise.io/read/01jqv14w4jdtm32jqq0g0t40e0)



• connect the android device via USB and run an adb shell as root. 
**adb shell su**
• setup the redirection: 
**iptables -t nat -A OUTPUT -p tcp -d destination-host.domain --dport** **443 \****-j DNAT --to-destination 127.0.0.1:8080**
• reverse map the port 8080 of the android device over the proxy running on my laptop on port 8080: 
**adb reverse tcp:8080 tcp:8080** [](https://read.readwise.io/read/01jqv17qky7sckta54ax74stv8)



it is also important to point out that we need to set up our Burp listening server as [invisible](https://portswigger.net/burp/documentation/desktop/tools/proxy/invisible) [](https://read.readwise.io/read/01jqv19w3v1spx5ydxhmrr73dg)



**Diving into BoringSSL Source Code**Flutter Engine relies on **[*boringssl*](https://github.com/google/boringssl)** library for SSL/TLS implementation, which is the Google's fork of the well known **[*openssl*](https://www.openssl.org/).** [](https://read.readwise.io/read/01jqv1eax2nmvfmakcx7ngsg32)



To bypass the check was only needed to make this function return always ***true***. [](https://read.readwise.io/read/01jqv1ftja4fk3xdcsdwaghx8f)



**Reverse-Engineering on libflutter.so**
At this point, we unpack the application's apk and open in IDA the ***resources/lib/arm64-v8a/libflutter.so*** shared object [](https://read.readwise.io/read/01jqv1gfjbm2czx95kx7s8y62v)



**Reverse-Engineering on libflutter.so**
At this point, we unpack the application's apk and open in IDA the ***resources/lib/arm64-v8a/libflutter.so*** shared object; [](https://read.readwise.io/read/01jqv1gne2hx9jk4b1sh0mbvzc)



Reverse-Engineering on libflutter.so [](https://read.readwise.io/read/01jqv1j5b31w0aka1h12qappvk)



At this point, we unpack the application's apk and open in IDA the ***resources/lib/arm64-v8a/libflutter.so*** shared object [](https://read.readwise.io/read/01jqv1j8tt9p07nkptsvf8mzn8)



the function at a certain point uses two string literals: "*ssl_client*" and "*ssl_server*". Those are both stored in the **.rodata** section of ***libflutter.so**.* So, the strategy used was to analyze the *XREFS* (cross-references) of those strings to find a function which locally addressed both. [](https://read.readwise.io/read/01jqv1kczgba9xzggvy6jgyeke)



**Frida Hooking Script**
After identifying the native subroutine that should have been hooked, it is possible to implement a simple Frida script to intercept calls to the function  ***ssl_crypto_x509_session_verify_cert_chain*** and force its return value to ***true*** (i.e., 0x01). [](https://read.readwise.io/read/01jqv1kxgbbbrcxz5m7zxqtpsm)



Frida Hooking Script [](https://read.readwise.io/read/01jqv1m0y39rabm9wfyj22bd38)



After identifying the native subroutine that should have been hooked, it is possible to implement a simple Frida script to intercept calls to the function  ***ssl_crypto_x509_session_verify_cert_chain*** and force its return value to ***true*** (i.e., 0x01). [](https://read.readwise.io/read/01jqv1m4ap1edfqstcxgge1d73)

[^1]: [Flutter compilation process](admin%20-%20Fork%20Bomb%20for%20Flutter.md#Flutter%20compilation%20process)

[^2]: [iptables](../../Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/Linux%20command%20cheatsheet.md#iptables)
