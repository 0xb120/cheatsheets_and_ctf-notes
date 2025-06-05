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
	- [apkd](../../Readwise/Articles/httpsgithub.comkiber-io%20-%20GitHub%20-%20kiber-ioapkd%20APK%20downloader%20from%20few%20sources.md) #tools - APK Downloader is a tool that allows you to easily download APK files from popular app stores
- Android Market API on GitHub

## Identify frameworks and components

Modern APK are not programmed using plain Java. They are built using **frameworks** or **modern programming languages**, and it is essential to identify them in order to better analyze the application:

- [Kotlin](../Dev,%20scripting%20&%20OS/Kotlin.md)
- [Flutter](../Dev,%20scripting%20&%20OS/Flutter.md)
- [ReactNative](../Dev,%20scripting%20&%20OS/ReactNative.md)
- Apache [Cordova](../Dev,%20scripting%20&%20OS/Cordova.md)


## Manifest and source code analysis

Search for:
- Over privileged apps
- Insecure configurations
    - Exported components [^element-android-cve]
    - Insecure Android attributes (`allowBackup`, `debuggable`, `sharedUserId`)
- Dangerous permissions which can affect user privacy or charge consumption
- Usage of system-level permissions
- Dangerous code
    - SQL statements
    - Log enabled
    - Insecure protocols and deeplink
    - Weak cryptographic functions or key leaks
    - [Weak WebView configuration](Android%20Dynamic%20application%20security%20testing%20(DAST).md#WebView%20attacks%20[%20web-view-attacks])
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

[^element-android-cve]: [Element Android (<1.6.12) Sensitive file disclosure via share activity](https://www.shielder.com/advisories/element-android-sensitive-file-disclosure/), shielder.com

# Tools

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

### Android Reverse Engineering Tools
- [APKTool](../Tools/APKTool.md)
- [Bytecode Viewer](../Tools/Bytecode%20Viewer.md)
- [jadx](../Tools/jadx.md)
- [dex2jar](../Tools/dex2jar.md)
- [JD-GUI](../Tools/JD-GUI.md)
- [APKLab](https://github.com/APKLab/APKLab) (VSCode extension)
- [simplify](../Tools/simplify.md) (de-obfuscator)
- [ClassNameDeobfuscator](https://github.com/HamiltonianCycle/ClassNameDeobfuscator) (de-obfuscator)

### Diffing

- [Source code analyses tools, rules and resources](../Services/HTTP%20&%20HTTPS.md#Source%20code%20analyses%20tools,%20rules%20and%20resources)