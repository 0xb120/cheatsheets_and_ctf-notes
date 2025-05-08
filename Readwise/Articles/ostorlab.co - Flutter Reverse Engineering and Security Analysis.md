---
author: "ostorlab.co"
aliases: "Flutter Reverse Engineering and Security Analysis"
tags: RW_inbox, readwise/articles, Flutter
url: https://blog.ostorlab.co/flutter-reverse-engineering-security-analysis.html?__readwiseLocation=
date: 2025-04-24
summary: Flutter is a popular framework by Google for building cross-platform mobile, web, and desktop applications, and it can be analyzed for security vulnerabilities. Tools for this analysis include static and dynamic analysis methods, though some tools may not support the latest versions of Flutter. The article discusses how to patch the Flutter runtime and extract information to improve security in Flutter applications.
---
# Flutter Reverse Engineering and Security Analysis

![rw-book-cover](https://blog.ostorlab.co/favicon-32x32.png)

## Highlights


Article on Static and Dynamic analysis techniques for Reverse engineering Flutter Applications. The article goes over runtime patching, custom ABI interception and traffic interception. [](https://read.readwise.io/read/01jqtyx8a53e2gm9ym2qh238hv)



Tools and Techniques [](https://read.readwise.io/read/01jqtyzvq47vhyfxmnvjt5nws6)



Static analysis tools, such as [Doldrums](https://github.com/rscloura/Doldrums), parse the file by re-implementing the file format. Doldrums only partially supports older versions of Flutter (2.10 and 2.12, the latest release at the time of writing of this article is 3.10), and is no longer maintained [](https://read.readwise.io/read/01jqtz091qermcy1h6k8vpawfs)



Dynamic analysis tools, on the other hand, like [reFlutter](https://github.com/Impact-I/reFlutter) patch the Flutter runtime to collect information during execution. [](https://read.readwise.io/read/01jqtz0k815zjv3wt3egj5prg3)



Despite its powerful approach to analyzing Flutter applications, reFlutter no longer supports the latest versions of the runtime. [](https://read.readwise.io/read/01jqtz0t7h90w4w0e7dvkgkvzm)



Structure Flutter Application
 Flutter applications, on all platforms, consist of a wrapper application, the Flutter runtime, and the Flutter app. [](https://read.readwise.io/read/01jqtz1022nczmwd79n9f0s30y)



`libapp.so` is the application code, `counter` is the wrapper, and `libflutter_linux_gtk.so` is the runtime. [](https://read.readwise.io/read/01jqtz15tmrgk5nhqjyp6fqhyn)



├── counter ├── data │ ├── flutter_assets │ │ ├── AssetManifest.json │ │ ├── FontManifest.json │ │ ├── fonts │ │ │ └── MaterialIcons-Regular.otf │ │ ├── NOTICES.Z │ │ ├── packages │ │ │ └── cupertino_icons │ │ │ └── assets │ │ │ └── CupertinoIcons.ttf │ │ ├── shaders │ │ │ └── ink_sparkle.frag │ │ └── version.json │ └── icudtl.dat └── lib ├── libapp.so └── libflutter_linux_gtk.so [](https://read.readwise.io/read/01jqtz198tv0360n13crvsg7tp)



The wrapper application provides an entry point that coordinates with the underlying operating system for access to services like rendering surfaces, accessibility, and input, and manages the message event loop. [](https://read.readwise.io/read/01jqtz1m5xy1mjztx4snkzvhhx)



The Flutter runtime is responsible for loading the Flutter app and offers a set of functionalities to the application. It comprises the Dart Virtual Machine (VM) and the Flutter engine [](https://read.readwise.io/read/01jqtz21mf2qd38qhx05nf8xfq)



The engine is responsible for tasks such as rasterizing composited scenes, providing the low-level implementation of Flutter’s core API, handling file and network I/O, accessibility support, the plugin architecture, and a Dart runtime and compile toolchain. The runtime is fixed, and It doesn't include any application-specific code. [](https://read.readwise.io/read/01jqtz2wcc3d13ymxmd9048h90)



The Flutter app is a standalone library but does not load as a standard one. Instead, it follows a custom format containing the application layout, code, and objects. [](https://read.readwise.io/read/01jqtz34j85efp11vy76xr2nn0)



Setup and Runtime Patching [](https://read.readwise.io/read/01jqtz3dza185mzvxqpq0qmq22)



To analyze a Flutter application, our goal is to rely on a **robust** and **maintainable** approach. [](https://read.readwise.io/read/01jqtz3t8wck71xdepf6dkhye7)



Instead of reimplementing the file format, we will modify the Dart SDK to extract the necessary information from the Flutter application at runtime. [](https://read.readwise.io/read/01jqtz4cjkbpfwwpbts2wh6va3)



When you compile a Flutter application, two files are generated within your application directory: [](https://read.readwise.io/read/01jqtz4jdp0qe5pxg7h2wnj4p0)



• `libFlutter.so`: This file serves as a shared library containing the Flutter engine, responsible for rendering the UI, handling input events, and managing the application lifecycle.
 • `libapp.so`: This file contains the actual application code and logic that make up the Flutter application. [](https://read.readwise.io/read/01jqtz4q9zx8xtj345arbhfnga)



Both files include a `snapshot_hash` to distinguish build versions. The `snapshot_hash` serves as a unique identifier for the build versions. [](https://read.readwise.io/read/01jqtz8amgx8f4tv5n8xk1tcp7)



Traffic Interception
 Flutter traffic is an oddball that requires custom care. [](https://read.readwise.io/read/01jqtze7zgswyn3bb8hfp9kvzd)



While in vanilla Android or iOS applications, it is possible to intercept traffic and bypass TLS pinning using a variety of methods, like changing the network security policy to accept the custom certificate, hooking the `SSL_read` and `SSL_write` to intercept traffic before encryption or dumping the TLS session key to decrypt traffic. [](https://read.readwise.io/read/01jqtzexngq24a7rzvskbfp1rd)



Flutter does not use the native TLS stack, and therefore it is not possible to set a proxy or intercept traffic. The TLS library is compiled statically in the Flutter runtime, and it is therefore difficult to identify it and patch it dynamically. [](https://read.readwise.io/read/01jqtzf7r7qas1zzgkw1tf20m1)



To intercept traffic on Flutter, the most robust solution is to again patch the Flutter runtime to disable TLS certificate validation and set a custom proxy. [](https://read.readwise.io/read/01jqtzg847vs5q76e6pe7vs3cz)



This approach is already seen in the reFlutter tool and continues to work in the current versions of the Flutter runtime. [](https://read.readwise.io/read/01jqtzjhtqhfyqnfv7fr8qxcwq)



After those changes, we use the proxy to intercept the traffic without installing any certificate on the device. **However, we need to make sure the invisible proxy mode is used**. [](https://read.readwise.io/read/01jqtzk1x7p008v1p899axejce)

