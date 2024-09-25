When performing a penetration test on mobile devices, you must always make a distinction between two different categories of data you will be dealing with.

-   **Data in motion**: these are the "dynamic" data that an app continuously exchanges with servers. The security of this data must be guaranteed by not assuming that the device is always connected to secure networks (for example, sensitive data transmitted without encryption).
-   **Data at rest**: these are the "static" data saved on the client side not in motion. It can be found within APKs, Databases, files, etc. The security of this data must be guaranteed at rest and not on the basis that the device cannot be compromised. (For example, unintentionally storing or leaking sensitive data so that it can be read by other applications on the phone or by other users).

During testing, therefore, it will be necessary to verify, in addition to traditional vulnerabilities, that both categories of data are properly protected on each platform and device. For this reason it is fundamental to use as many different OS as possible, in order to have an adequate coverage of the criticalities and vulnerabilities related to the different OS on the market, as well as the differences in implementation between the different systems.

## Android Env Setup

https://developer.android.com/codelabs/basic-android-kotlin-compose-first-app#0

![Android setup](Android%20setup.canvas)


Requirements:
- [Java Development Kit (JDK)](https://www.oracle.com/java/technologies/downloads/)
- [Android SDK tool package](https://developer.android.com/studio#:~:text=Command%20line%20tools%20only)
- [SDK platform tools](https://developer.android.com/studio/releases/platform-tools)

>[!tip]
>Use the [android_sdk](../Tools/android_sdk.md) tools to download SDKs, build AVD and run emulators

Install custom certificates inside the system certificate store
- [Android >= 7 & Android <14](../Tools/adb.md#Android%20>=%207%20&%20Android%20<14)
- [Android >14](../Tools/adb.md#Android%20>14%20[%20certs-android-14])

Port forwarding and network management:
![Port forwarding and network management](../Tools/android_sdk.md#Port%20forwarding%20and%20network%20management)

Eventually, you can use [appetize](https://appetize.io/) to run and preview mobile apps with the browser

---

## Types of analyses

### [Static application security testing (SAST)](Static%20application%20security%20testing%20(SAST).md)


### [Dynamic application security testing (DAST)](Dynamic%20application%20security%20testing%20(DAST).md)

---

# External interesting researches and vulnerabilities

- [Attacking Android Antivirus Applications - Intent redirection](https://blog.scrt.ch/2023/03/29/attacking-android-antivirus-applications/)
- [TikTok uXSS and 1-Click RCE](https://dphoeniixx.medium.com/tiktok-for-android-1-click-rce-240266e78105)
- [Attacking Android browsers via intent scheme URLs](https://www.mbsd.jp/Whitepaper/IntentScheme.pdf)
- [XSS + unsecure deeplink causing arbitrary application installation](https://ssd-disclosure.com/ssd-advisory-galaxy-store-applications-installation-launching-without-user-interaction/)
- [One-click/Open-redirect to own Samsung S22](https://starlabs.sg/blog/2023/06-the-old-the-new-and-the-bypass-one-clickopen-redirect-to-own-samsung-s22-at-pwn2own-2022/)
- [Android File Write to RCE](../../Readwise/Tweets/@LiveOverflow%20on%20Twitter%20-%20I'm%20Looking%20for%20Differen....md)