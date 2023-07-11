---
Description: Android Application Identifier for Packers, Protectors, Obfuscators and Oddities - PEiD for Android
URL: https://github.com/rednaga/APKiD
---

PKiD gives you information about how an APK was made. It identifies many compilers, packers, obfuscators, and other weird stuff.

>[!tldr]- Installation
>It is suggested to install the docker environment - I had dependency problems with the pip one

Useful writeups:
- [Android Compiler Fingerprinting](http://hitcon.org/2016/CMT/slide/day1-r0-e-1.pdf)
- [Detecting Pirated and Malicious Android Apps with APKiD](http://rednaga.io/2016/07/31/detecting_pirated_and_malicious_android_apps_with_apkid/)
- [APKiD: PEiD for Android Apps](https://github.com/enovella/cve-bio-enovella/blob/master/slides/bheu18-enovella-APKID.pdf)
- [APKiD: Fast Identification of AppShielding Products](https://github.com/enovella/cve-bio-enovella/blob/master/slides/APKiD-NowSecure-Connect19-enovella.pdf)


```bash
$ docker/apkid.sh /mnt/hgfs/VM-Shared/apks/possible/Calculator\ -\ photo\ vault_10.7.1_apkcombo.com.apk 
basename: extra operand ‘photo’
Try 'basename --help' for more information.
[+] APKiD 2.1.4 :: from RedNaga :: rednaga.io
[*] /input/Calculator - photo vault_10.0.7_apkcombo.com.apk!classes.dex
 |-> anti_debug : Debug.isDebuggerConnected() check
 |-> anti_vm : Build.BOARD check, Build.FINGERPRINT check, Build.MANUFACTURER check, Build.MODEL check, Build.PRODUCT check, Build.TAGS check, SIM operator check, network operator name check, subscriber ID check
 |-> compiler : r8
[*] /input/Calculator - photo vault_10.0.7_apkcombo.com.apk!classes2.dex
 |-> anti_vm : Build.FINGERPRINT check, Build.MANUFACTURER check, Build.TAGS check
 |-> compiler : r8 without marker (suspicious)
[*] /input/Calculator - photo vault_10.6.6_apkcombo.com.apk!classes.dex
 |-> anti_vm : Build.BOARD check, Build.FINGERPRINT check, Build.MANUFACTURER check, Build.MODEL check, Build.PRODUCT check, SIM operator check, network operator name check, possible VM check
 |-> compiler : r8
[*] /input/Calculator - photo vault_10.6.6_apkcombo.com.apk!classes2.dex
 |-> anti_debug : Debug.isDebuggerConnected() check
 |-> anti_vm : Build.BOARD check, Build.FINGERPRINT check, Build.HARDWARE check, Build.MANUFACTURER check, Build.MODEL check, Build.PRODUCT check, Build.TAGS check, SIM operator check, network operator name check, possible VM check
 |-> compiler : r8 without marker (suspicious)
[*] /input/Calculator - photo vault_10.6.6_apkcombo.com.apk!classes3.dex
 |-> anti_vm : Build.BOARD check, Build.FINGERPRINT check, Build.MANUFACTURER check, Build.MODEL check, Build.PRODUCT check, network operator name check, possible VM check
 |-> compiler : r8 without marker (suspicious)
[*] /input/Calculator - photo vault_10.7.1_apkcombo.com.apk!classes.dex
 |-> anti_vm : Build.BOARD check, Build.FINGERPRINT check, Build.MANUFACTURER check, Build.MODEL check, Build.PRODUCT check, SIM operator check, network operator name check, possible VM check
 |-> compiler : r8 without marker (suspicious)
[*] /input/Calculator - photo vault_10.7.1_apkcombo.com.apk!classes2.dex
 |-> anti_debug : Debug.isDebuggerConnected() check
 |-> anti_vm : Build.BOARD check, Build.FINGERPRINT check, Build.HARDWARE check, Build.MANUFACTURER check, Build.MODEL check, Build.PRODUCT check, Build.TAGS check, SIM operator check, network operator name check, possible VM check
 |-> compiler : r8 without marker (suspicious)
[*] /input/Calculator - photo vault_10.7.1_apkcombo.com.apk!classes3.dex
 |-> anti_vm : Build.BOARD check, Build.FINGERPRINT check, Build.MANUFACTURER check, Build.MODEL check, Build.PRODUCT check, SIM operator check, network operator name check, possible VM check
 |-> compiler : r8 without marker (suspicious)
[*] /input/Obsidian_1.4.6_apkcombo.com.apk!classes.dex
 |-> anti_debug : Debug.isDebuggerConnected() check
 |-> anti_vm : Build.FINGERPRINT check, Build.MANUFACTURER check, Build.MODEL check, Build.PRODUCT check
 |-> compiler : r8 without marker (suspicious)
[*] /input/owncloud.apk!classes.dex
 |-> anti_vm : Build.FINGERPRINT check, Build.MANUFACTURER check
 |-> compiler : r8 without marker (suspicious)
[*] /input/owncloud.apk!classes2.dex
 |-> compiler : r8 without marker (suspicious)
[*] /input/owncloud.apk!classes3.dex
 |-> compiler : r8 without marker (suspicious)
[*] /input/Pass Safe_1.2.12a_apkcombo.com.apk!classes.dex
 |-> anti_vm : Build.FINGERPRINT check, Build.MANUFACTURER check, Build.MODEL check, Build.PRODUCT check, Build.TAGS check, possible Build.SERIAL check
 |-> compiler : r8 without marker (suspicious)
[*] /input/Pass Safe_1.2.12_apkcombo.com.apk!classes.dex
 |-> anti_vm : Build.FINGERPRINT check, Build.MANUFACTURER check, Build.MODEL check, Build.PRODUCT check, possible Build.SERIAL check
 |-> compiler : r8 without marker (suspicious)
[*] /input/PAYEER_2.5.0_apkcombo.com.apk!classes.dex
 |-> anti_debug : Debug.isDebuggerConnected() check
 |-> anti_vm : Build.FINGERPRINT check, Build.MANUFACTURER check, Build.MODEL check, Build.PRODUCT check, Build.TAGS check
 |-> compiler : r8
[*] /input/PAYEER_2.5.0_apkcombo.com.apk!classes2.dex
 |-> compiler : r8 without marker (suspicious)
[*] /input/Private Photo Vault_3.5.1_apkcombo.com.apk!classes.dex
 |-> anti_vm : Build.FINGERPRINT check, Build.MANUFACTURER check, Build.MODEL check, Build.PRODUCT check, Build.TAGS check, SIM operator check, possible Build.SERIAL check
 |-> compiler : r8 without marker (suspicious)
[*] /input/Private Photo Vault_3.5.1_apkcombo.com.apk!classes2.dex
 |-> anti_debug : Debug.isDebuggerConnected() check
 |-> anti_vm : Build.HARDWARE check, Build.MANUFACTURER check, Build.MODEL check, Build.PRODUCT check, Build.TAGS check, network operator name check, possible VM check
 |-> compiler : r8 without marker (suspicious)
[*] /input/Private Photo Vault_3.5.1_apkcombo.com.apk!classes3.dex
 |-> compiler : r8 without marker (suspicious)
[*] /input/Private Photo Vault_3.5.1_apkcombo.com.apk!classes4.dex
 |-> anti_vm : Build.MANUFACTURER check
 |-> compiler : r8 without marker (suspicious)
[*] /input/Smart Switch My Phone_19.0_apkcombo.com.apk!classes.dex
 |-> anti_vm : Build.BOARD check, Build.FINGERPRINT check, Build.MANUFACTURER check, Build.MODEL check, Build.PRODUCT check, SIM operator check, network operator name check, possible Build.SERIAL check, possible VM check
 |-> compiler : r8
[*] /input/Smart Switch My Phone_19.0_apkcombo.com.apk!classes2.dex
 |-> anti_debug : Debug.isDebuggerConnected() check
 |-> anti_vm : Build.FINGERPRINT check, Build.HARDWARE check, Build.MANUFACTURER check, Build.MODEL check, Build.PRODUCT check, Build.TAGS check, SIM operator check, network operator name check, possible VM check, subscriber ID check
 |-> compiler : r8 without marker (suspicious)
[*] /input/Smart Switch My Phone_19.0_apkcombo.com.apk!assets/audience_network.dex
 |-> anti_debug : Debug.isDebuggerConnected() check
 |-> anti_vm : possible Build.SERIAL check
 |-> compiler : unknown (please file detection issue!)
[*] /input/Smart Switch My Phone_19.0_apkcombo.com.apk!classes3.dex
 |-> compiler : r8 without marker (suspicious)
[*] /input/SplashID Safe Password Manager_8.3.8_apkcombo.com (2).apk!classes.dex
 |-> anti_vm : Build.FINGERPRINT check, possible Build.SERIAL check
 |-> compiler : r8
[*] /input/SplashID Safe Password Manager_8.3.8_apkcombo.com (2).apk!classes2.dex
 |-> anti_vm : Build.FINGERPRINT check, Build.MANUFACTURER check, Build.MODEL check, Build.PRODUCT check, Build.TAGS check, network operator name check
 |-> compiler : r8 without marker (suspicious)
[*] /input/SplashID Safe Password Manager_8.3.8_apkcombo.com (2).apk!classes3.dex
 |-> anti_vm : possible Build.SERIAL check
 |-> compiler : r8 without marker (suspicious)
[*] /input/Tracki_1.0.329-PROD_apkcombo.com.apk!assets/audience_network.dex
 |-> anti_vm : possible Build.SERIAL check
 |-> compiler : unknown (please file detection issue!)
[*] /input/Tracki_1.0.329-PROD_apkcombo.com.apk!classes.dex
 |-> anti_vm : Build.FINGERPRINT check, Build.MANUFACTURER check, Build.MODEL check, Build.PRODUCT check
 |-> compiler : r8
[*] /input/Tracki_1.0.329-PROD_apkcombo.com.apk!classes2.dex
 |-> anti_debug : Debug.isDebuggerConnected() check
 |-> anti_vm : Build.FINGERPRINT check, Build.HARDWARE check, Build.MANUFACTURER check, Build.MODEL check, Build.PRODUCT check, Build.TAGS check, SIM operator check, network operator name check, possible VM check
 |-> compiler : r8 without marker (suspicious)
[*] /input/Tracki_1.0.329-PROD_apkcombo.com.apk!classes3.dex
 |-> anti_debug : Debug.isDebuggerConnected() check
 |-> anti_vm : Build.HARDWARE check, Build.MODEL check, Build.PRODUCT check, Build.TAGS check, possible Build.SERIAL check, possible VM check
 |-> compiler : r8 without marker (suspicious)
[*] /input/Tracki_1.0.329-PROD_apkcombo.com.apk!classes4.dex
 |-> anti_vm : Build.BOARD check, Build.FINGERPRINT check, Build.HARDWARE check, Build.MANUFACTURER check, Build.MODEL check, Build.PRODUCT check, Build.TAGS check, emulator file check, possible Build.SERIAL check, possible VM check
 |-> compiler : r8 without marker (suspicious)
[*] /input/Tracki_1.0.329-PROD_apkcombo.com.apk!classes5.dex
 |-> compiler : r8 without marker (suspicious)
```