# DAST basics
The mobile application is analyzed as it runs on the device. Reviews include forensic analysis of the file system, assessment of the network traffic between the application and server and an assessment of the application’s inter-process communication (IPC).

>[!hint]
>Test the application both online and offline in order to evaluate if it behaves correctly.

>[!warning] 
>Rooted/Jailbroken device are almost mandatory

Main tasks include:
- Network Traffic Analysis
    - Insecure network communication
    - Man in the Middle (MiTM) required
- Backend Analysis
    - VA/PT on API and WebServices (classic Web Application Penetration Test)
- Client-side testing, e.g.
    - Sensitive Data Exposure
        - Inside logs
        - Inside non-censored activities
    - Insecure Configuration
    - Insecure Cryptography
    - Rooting detection

Key security testing areas:
- **Local Data Storage**, e.g. data leak to cloud storage, backups, logs, keyboard cache, db or local files
- **Authentication and Authorization**, e.g. bypassing, insecure or wrongly implemented framework
- **Interaction** with the **Mobile Platform**, e.g. IPC APIs misuse, sensitive data or functionality exposure to other apps running on the device.
- **Code Quality** and **Exploit Mitigation** e.g. injection and memory management issues, cross-site scripting
- **Anti-Tampering** and **Anti-Reversing**

# Security mechanism bypasses

## General application patching

Disassembling the app in order to remove the code responsible for the security check, and then [rebuild and sign the «patched» app](../Tools/APKTool.md#Rebuild%20files%20into%20APK).

>[!tip]
>If you can't re-build the APK once decompiled because of broken/misspelled resources, decompile only the application code using the `apktool d --no-res whos_that_pokemon.apk` command

Inject debug statements inside the smali code:
```java
const-string v1, "TEST"
invoke-static {v1, v2}, Landroid/util/Log;->i(Ljava/lang/String;Ljava/lang/String;)I
```

Dalvik opcodes:

```cardlink
url: http://pallergabor.uw.hu/androidblog/dalvik_opcodes.html
title: "Dalvik opcodes"
host: pallergabor.uw.hu
```

## Network Interception and Certificate Pinning Bypass

>[!warning] 
>**Certificate Pinning**: validation of the certificate given by the back-end on the base of a public key embedded on the application. If Implemented, it blocks the analysis of HTTPS streams via interception proxy: the certificate of the proxy does not match the one embedded in the app.

- Check the existence of `network_security_config.xml` and patch it or add it to trust user certificates/remove certificate pinning

	If you don't have root access or the application implements `network_security_config.xml`, you can [patch your APK](Android%20Dynamic%20application%20security%20testing%20(DAST).md#General%20application%20patching), add 
	```java
	android:networkSecurityConfig="@xml/network_security_config"
	```
	
	to the **`<application>`** element in your application `AndroidManifest.xml` file and create an `network_security_config.xml` that trusts user certificates and HTTP traffic.

	```xml
	<base-config cleartextTrafficPermitted="true">
	    <trust-anchors>
	        <certificates src="system" />
	        <certificates src="user" />
	    </trust-anchors>
	</base-config>
	```

	[^net-sec-conf]: [network_security_config.xml](https://developer.android.com/privacy-and-security/security-config), android.com

- Install the proxy certificate into the device trust store (system certificate store or user certificate store based on the Android version and root availability). See [Push burpsuite certificate inside the system certificate store](../Tools/adb.md#Push%20burpsuite%20certificate%20inside%20the%20system%20certificate%20store)
- MITM proxy
- **VPN service**: sometime HTTP traffic is not intercepted because the application ignores proxy specifications (eg. OKHTTP3 + No Proxy). In this cases a VPN service like https://github.com/celzero/rethink-app 
	1. Change DNS settings to "System DNS"
	2. Add a HTTP(S) CONNECT proxy
	3. Start the "VPN"
- **Runtime Manipulation**: runtime execution modification in order to invalidate certificate pinning validation
    - [Xposed!](../Tools/Xposed!.md) framework with [TrustMeAlready](../Tools/Xposed!.md#TrustMeAlready) or [SSLUnpinning - Certificate Pinning Bypass](../Tools/Xposed!.md#SSLUnpinning%20-%20Certificate%20Pinning%20Bypass) (root required)
    - [Frida](../Tools/Frida.md)
    - [objection](../Tools/objection.md) (non-root, see the [following video](https://www.youtube.com/watch?v=qaJBWcueCIA&ab_channel=CorSecure))
    - Flutter-specific: https://blog.mindedsecurity.com/2024/05/bypassing-certificate-pinning-on.html
- **Improper Cert Management**: the server certificate might be stored in unsafe locations, (Documents directory, application files) therefore replaced with our own proxy certificate
- **VPN** + **DNS Spoofing** using [dnsmasq](https://thekelleys.org.uk/dnsmasq/doc.html) and **Transparent proxy** 
	```txt title:"dnsmaq configuration"
	address=/hextree.io/192.168.178.37
	address=/ht-api-mocks-lcfc4kr5oa-uc.a.run.app/192.168.178.37
	log-queries
	```
	Docker:
	```sh title:docker
	docker pull andyshinn/dnsmasq
	docker run --name my-dnsmasq --rm -it -p 0.0.0.0:53:53/udp \
	 -v D:\tmp\proxy\dnsmasq.conf:/etc/dnsmasq.conf andyshinn/dnsmasq.conf andyshinn/dnsmasq
	```

## Bypassing Anti-Root

>[!warning]
> **Anti-Root Mechanism**: the application checks if the device is rooted by looking at different file-system areas. If implemented, it might block a rooted/jailbroken device to run the app or it might hide some sensitive functionalities.

Evasion techniques:
- **App patching**: based on disassembling the app in order to remove the code responsible for root checks and then rebuild the «patched» app
    - **Smali** code **knowledge** is required
    - **disassembly tools** + [APKTool](../Tools/APKTool.md)
- **Runtime Manipulation**: runtime execution modification in order to invalidate root checks
    - **Rooted device** required
    - **Magisk Hide**
    - **Frida**

---

# General vulnerabilities

## Exported components

Verify for each exported component what it does and if exposes private information or vulnerable behaviors.
Some examples are:
- [Content Providers vulnerabilities](Content%20Providers.md#Example%20vulnerabilities)
- PIN/Authentication bypass using exported activities
- Intent redirection [^intent-redir-1][^intent-redir-2][^intent-redir-3]

[^file-read-content-provider-vault]: [Arbitrary file interaction using an exported content provider](https://0xbro.red/disclosures/disclosed-vulnerabilities/digital-private-vault/#arbitrary-file-interaction-using-an-exported-content-provider), 0xbro.red
[^file-read-content-provider-element]: [Element Android (<1.6.12) Sensitive file disclosure via share activity](https://www.shielder.com/advisories/element-android-sensitive-file-disclosure/), shielder.com
[^intent-redir-1]: [Attacking Android Antivirus Applications](https://blog.scrt.ch/2023/03/29/attacking-android-antivirus-applications/), blog.scrt.ch
[^intent-redir-2]: [Element Android (<1.6.12) Intent Redirection](https://www.shielder.com/advisories/element-android-intent-redirection/), shielder.com
[^intent-redir-3]: [Element Android CVE-2024-26131, CVE-2024-26132 - Never Take Intents From Strangers](https://www.shielder.com/blog/2024/04/element-android-cve-2024-26131-cve-2024-26132-never-take-intents-from-strangers/), shielder.com

## WebViews Attacks

- [File access](WebViews.md#File%20access)
- [JavaScript Enabled](WebViews.md#JavaScript%20Enabled)
- [JavaScript Bridge](WebViews.md#JavaScript%20Bridge)
- [In the wild WebView attacks](WebViews.md#In%20the%20wild%20WebView%20attacks)

## Arbitrary File Write to Remote Code Execution

Different techniques how to [escalate an arbitrary file write vulnerability in Android apps to arbitrary code execution](../../Readwise/Tweets/@LiveOverflow%20on%20Twitter%20-%20I'm%20Looking%20for%20Differen....md): 

- Method 1: Overwriting native libraries (NDK) bundled with the .apk
- Method 2: Overwriting .dex files in large apps thanks to multidex feature
- Method 3: When the app itself implements some custom code execution that can be abused

Good gadget's candidate for this are [FileProviders](Content%20Providers.md#read%20or%20write%20permissions) 

Examples:
- [RCE in Adobe Acrobat Reader for Android](../../Readwise/Articles/hulkvision.github.io%20-%20RCE%20in%20Adobe%20Acrobat%20Reader%20for%20Android.md), hulkvision.github.io


---

# Tools

## DAST Tools
- [adb](../Tools/adb.md#Dynamic%20analysis)
- [Frida](../Tools/Frida.md)
- [objection](../Tools/objection.md)
- [drozer](../Tools/drozer.md)
- [Xposed!](../Tools/Xposed!.md) + [Modules](../Tools/Xposed!.md#Modules)
- [Magisk](https://github.com/topjohnwu/Magisk) (on [Genymotion](https://support.genymotion.com/hc/en-us/articles/360011385178-How-to-install-Xposed-or-Magisk-Edxposed-with-Genymotion-Device-image-PaaS-))
- [WebView Remote Debugger](chrome://inspect/#devices)
- [Koodous](https://koodous.com/)
