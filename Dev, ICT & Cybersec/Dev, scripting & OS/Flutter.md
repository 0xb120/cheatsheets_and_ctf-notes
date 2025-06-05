**Flutter** [^flutter] is an open-source UI framework for building beautiful, **natively compiled applications** for mobile, web, desktop, and embedded devices from a single codebase. The main programming language is DART. Flutter also provide a built-in obfuscation feature.

[^flutter]: https://flutter.dev/

Flutter application are usually very large in size because the Dart framework is statically linked in the application binary. We can confirm an application is built using flutter searching for `kernel_blob.bin`, `libflutter.so` and `libapp.so` inside the APK. 

We can also extract Flutter Metadata from the application using a modified version of the flutter runtime library, using [reFlutter](https://github.com/Impact-I/reFlutter)

>[!tldr] Flutter analyses
>- https://www.guardsquare.com/blog/current-state-and-future-of-reversing-flutter-apps
>- https://www.guardsquare.com/blog/obstacles-in-dart-decompilation-and-the-impact-on-flutter-app-security
>- https://www.guardsquare.com/blog/how-classical-attacks-apply-to-flutter-apps
>- https://cryptax.medium.com/reversing-an-android-sample-which-uses-flutter-23c3ff04b847
>- https://blog.mindedsecurity.com/2024/05/bypassing-certificate-pinning-on.html
