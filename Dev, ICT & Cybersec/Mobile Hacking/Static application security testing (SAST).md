The analysis is done on the provided files or decompiled source code. **No app installation or execution.**

The most commonly and easily applied analysis method are conducted on the **static code**. It can be:
- **raw** (when source code provided)
- **decompiled**
- **compiled** (object)

Main tasks include:
- Reverse Engineering
- Source Code Analysis
- Security Setting Analysis
- Vulnerable or insecure code patterns searches
- Archive Analysis

---

# Reverse Engineering

Reversing is the process to **convert the compiled applications into human-readable source code**. Usually it is performed in order to understand the internal application functionality and search for vulnerabilities. Furthermore, reversing is a step in order to patch APK from security implementations.

General reversing steps:
1. Retrieve the APK
2. Retrieve the DEX from the APK
3. Convert the DEX to JAR
4. Decompile JAR file

Possible retrieved findings:
- **Hard-Coded Keys** or **Credentials**
- Critical **internal process** identification
- **Pinning**
- **Authentication**
- **Native Code** & **3rd Party Libraries**
    - **insecure native code** (.so)
    - **Public known vulnerabilities** in 3rd party lib

APK retrieving:
- Provided by client
- From filesystem via [adb](../Tools/adb.md) inside `/data/app` or `/data/app-private` 
	```bash
	$ adb shell pm list packages  
	$ adb shell pm path <package_name>
	$ adb pull <APK_path>
	```
- Share application feature in Android (File Manager > Apps > Select an application > Share)
- APK converters/external stores (unsafe, chose a trusted application!)
	- https://apkcombo.com/
	- https://apps.evozi.com/apk-downloader/?id=com.netflix.Speedtest
- Android Market API on GitHub

## Identify frameworks and components
Modern APK are not programmed using plain Java. They are built using **frameworks** or **modern programming languages**, and it is essential to identify them in order to better analyze the application.

### Kotlin

Application written using **Kotlin** [^kotlin] can be recognized because they contain `.kt` files inside the APK archive as well as a `kotlin` folder. It supports both R8 and ProGuard for obfuscation and code shrinking [^kotlin-obf] . 

[^kotlin]: https://kotlinlang.org/
[^kotlin-obf]: https://developer.android.com/build/shrink-code

Eventually you can also find references to annotations like `@kotlin.Metadata`, `kotlin/Metadata`, etc.


### Flutter

**Flutter** [^flutter] is an open-source UI framework for building beautiful, **natively compiled applications** for mobile, web, desktop, and embedded devices from a single codebase. The main programming language is DART. Flutter also provide a built-in obfuscation feature.

[^flutter]: https://flutter.dev/

Flutter application are usually very large in size because the Dart framework is statically linked in the application binary. We can confirm an application is built using flutter searching for `kernel_blob.bin`, `libflutter.so` and `libapp.so` inside the APK. 

We can also extract Flutter Metadata from the application using a modified version of the flutter runtime library, using [reFlutter](https://github.com/Impact-I/reFlutter)

>[!tldr] Flutter analyses
>- https://www.guardsquare.com/blog/current-state-and-future-of-reversing-flutter-apps
>- https://www.guardsquare.com/blog/obstacles-in-dart-decompilation-and-the-impact-on-flutter-app-security
>- https://www.guardsquare.com/blog/how-classical-attacks-apply-to-flutter-apps
>- https://cryptax.medium.com/reversing-an-android-sample-which-uses-flutter-23c3ff04b847

### ReactNative

### Apache Cordova

## Manifest and source code analysis

Search for:
- Over privileged apps
- Insecure configurations
    - Exported components
    - Insecure Android attributes (`allowBackup`, `debuggable`, `sharedUserId`)
- Dangerous permissions which can affect user privacy or charge consumption
- Usage of system-level permissions
- Dangerous code
    - SQL statements
    - Log enabled
    - Insecure protocols and deeplink
    - Weak cryptographic functions or key leaks
    - Weak WebView configuration
- Check Configuration of Cryptographic Standard Algorithms
    - `Cipher`
    - `Mac`
    - `MessageDigest`
    - `Signature`
    - `AndroidKeyStore`
    - `Key`, `PrivateKey`, `PublicKey`, `SecretKeySpec`, `KeyInfo`
    - Few others in the `java.security.*` and `javax.crypto.*` packages.
    - Check Secure Random Number Generation
        - Locate uses of `java.util.Random`
    - Check way of generating and storing key material.
        - Locate uses of the cryptographic primitives in the code.
        - Some of the most frequently used classes and interfaces

---

# Tools

```start-multi-column
ID: ID_fpde
Number of Columns: 2
Largest Column: standard
```

## SAST Tools
SAST can be performed via automatic open source tool which automatically decompile the app and enforce controls or through a manual effort.

### Automatic Tools
- [MobSF](../Tools/MobSF.md)
- [semgrep rules](https://github.com/mindedsecurity/semgrep-rules-android-security/tree/main/rules)
- [mariana-trench](https://github.com/facebook/mariana-trench) (a security focused static analysis tool for Android and Java applications)
- MARA Framework
- [qark](../Tools/qark.md) (last commit on 2019)

### General analyses tool

- [apkid](../Tools/apkid.md)
- [fsmon](https://github.com/nowsecure/fsmon)
- [pidcat](https://github.com/JakeWharton/pidcat)
- [apkurlgrep](https://github.com/ndelphit/apkurlgrep) (extract endpoints from APK files)

--- column-end ---

### Android Reverse Engineering Tools
- [APKTool](../Tools/APKTool.md)
- [Bytecode Viewer](../Tools/Bytecode%20Viewer.md)
- [jadx](../Tools/jadx.md)
- [dex2jar](../Tools/dex2jar.md)
- [JD-GUI](../Tools/JD-GUI.md)
- [APKLab](https://github.com/APKLab/APKLab) (VSCode extension)
- [simplify](../Tools/simplify.md) (de-obfuscator)
- [ClassNameDeobfuscator](https://github.com/HamiltonianCycle/ClassNameDeobfuscator) (de-obfuscator)

=== end-multi-column

