---
author: "noreply@blogger.com (Christian Cotignola (@b4dsheep))"
aliases: "Bypassing Certificate Pinning on Flutter-Based Android Apps. A New Guide."
tags: RW_inbox, readwise/articles
url: https://blog.mindedsecurity.com/2024/05/bypassing-certificate-pinning-on.html?__readwiseLocation=
date: 2025-04-24
summary: One of the preliminary activities when analyzing mobile application, more usually than not, is to be able to sniff HTTP/S traffic via a MitM proxy. This is quite straightforward in the case of naive applications, but can be quite challenging when applications use certificate pinning techniques. In this post I'll try to explain the methodology I used to make this possible for a Flutter-based Android sample application in a reliable way.IntroductionIt was indeed the need to bypass a certificate validation on a Flutter framework during a mobile application penetration testing activity for a customer of ours, that led to this research. As a first approach, as usual, we tried some of the specific exploits/bypasses we found on the web. Alas, in this case, they failed.Some of the main concepts that are going to be explained, actually, overlap in what those articles contain; what it differs is the technique used for identifying and hooking at runtime the routine used for certificate verification.While minimizing effort ...
---
# Bypassing Certificate Pinning on Flutter-Based Android Apps. A New Guide.

![rw-book-cover](https://blogger.googleusercontent.com/img/a/AVvXsEiLZOgVn35LqPDpfHUk6wcqLOjrvV6JioR2CPitS6-4WhuB6jSaudX8wXAwTaHk3U7ebucLS-R5vO6GRTF2dC-cjzUcm-qv1IQLhkB-zDUYLavj06fTkgqAbwsix1lSP2klyN8rqETtdsCcQFAv_FOKE9AGSNQ0TOyi3q5sxBSkyE_lhziAZbmO1DDnroxg=s72-c)

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

