**Flutter** [^flutter] is an open-source UI framework for building beautiful, **natively compiled applications** for mobile, web, desktop, and embedded devices from a single codebase. The main programming language is [[DART]]. Flutter also provide a built-in obfuscation feature.

[^flutter]: https://flutter.dev/

Flutter application are usually very large in size because the **Dart framework is statically linked** in the application binar[^1]. We can confirm an application is built using flutter searching for `kernel_blob.bin`, `libflutter.so` and `libapp.so` inside the APK. 

We can also extract Flutter Metadata from the application using a modified version of the flutter runtime library, using [reFlutter](https://github.com/Impact-I/reFlutter)

>[!tldr] Flutter analyses
>- https://www.guardsquare.com/blog/current-state-and-future-of-reversing-flutter-apps
>- https://www.guardsquare.com/blog/obstacles-in-dart-decompilation-and-the-impact-on-flutter-app-security
>- https://www.guardsquare.com/blog/how-classical-attacks-apply-to-flutter-apps
>- https://cryptax.medium.com/reversing-an-android-sample-which-uses-flutter-23c3ff04b847

## Attacking Flutter Applications

- [b4dsheep - Bypassing Certificate Pinning on Flutter-Based Android Apps. A New Guide.](../../Readwise/Articles/b4dsheep%20-%20Bypassing%20Certificate%20Pinning%20on%20Flutter-Based%20Android%20Apps.%20A%20New%20Guide..md)


[^1]: [Flutter architecture and compilation process](../../Readwise/Articles/admin%20-%20Fork%20Bomb%20for%20Flutter.md)
