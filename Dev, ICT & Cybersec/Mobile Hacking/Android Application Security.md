When performing a penetration test on mobile devices, you must always make a distinction between two different categories of data you will be dealing with.

-   **Data in motion**: these are the "dynamic" data that an app continuously exchanges with servers. The security of this data must be guaranteed by not assuming that the device is always connected to secure networks (for example, sensitive data transmitted without encryption).
-   **Data at rest**: these are the "static" data saved on the client side not in motion. It can be found within APKs, Databases, files, etc. The security of this data must be guaranteed at rest and not on the basis that the device cannot be compromised. (For example, unintentionally storing or leaking sensitive data so that it can be read by other applications on the phone or by other users).

During testing, therefore, it will be necessary to verify, in addition to traditional vulnerabilities, that both categories of data are properly protected on each platform and device. For this reason it is fundamental to use as many different OS as possible, in order to have an adequate coverage of the criticalities and vulnerabilities related to the different OS on the market, as well as the differences in implementation between the different systems.

## Android Env Setup

Requirements:
- [Java Development Kit (JDK)](https://www.oracle.com/java/technologies/downloads/)
- [Android SDK tool package](https://developer.android.com/studio#:~:text=Command%20line%20tools%20only)
- [SDK platform tools](https://developer.android.com/studio/releases/platform-tools)

>[!tip]
>Use the [android_sdk](../Tools/android_sdk.md) tools to download SDKs, build AVD and run emulators

Importing user certificates on Android 7 or higher: 

![Push burpsuite certificate inside the system certificate store](../Tools/adb.md#Push%20burpsuite%20certificate%20inside%20the%20system%20certificate%20store)

Port forwarding and network management:
![Port forwarding and network management](../Tools/android_sdk.md#Port%20forwarding%20and%20network%20management)

---

## Types of analyses

```start-multi-column
ID: ID_qh3p
Number of Columns: 2
Largest Column: standard
```

### [Static application security testing (SAST)](Static%20application%20security%20testing%20(SAST).md)

--- column-end ---

### [Dynamic application security testing (DAST)](Dynamic%20application%20security%20testing%20(DAST).md)

=== end-multi-column

---

## External interesting researches

- [Attacking Android Antivirus Applications](https://blog.scrt.ch/2023/03/29/attacking-android-antivirus-applications/)