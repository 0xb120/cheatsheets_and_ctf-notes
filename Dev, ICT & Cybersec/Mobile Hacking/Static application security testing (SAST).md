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

Reversing steps:
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
- APK converters (unsafe, chose a trusted application!)
	- https://apps.evozi.com/apk-downloader/?id=com.netflix.Speedtest
- Android Market API on GitHub

---

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
    - Insecure protocols
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
- MARA Framework
- [Droidstat-x](https://github.com/integrity-sa/droidstatx) (code review)
- Qark
- Androwarn

### Manual Tools
- [adb](../Tools/adb.md#Static%20analysis)
- [drozer](../Tools/drozer.md)

--- column-end ---

## Android Reverse Engineering Tools
- [APKTool](../Tools/APKTool.md)
- [Bytecode Viewer](../Tools/Bytecode%20Viewer.md)
- [jadx](../Tools/jadx.md)
- [dex2jar](../Tools/dex2jar.md)
- [JD-GUI](../Tools/JD-GUI.md)
- [simplify](../Tools/simplify.md)
- [ClassNameDeobfuscator](https://github.com/HamiltonianCycle/ClassNameDeobfuscator)

=== end-multi-column

