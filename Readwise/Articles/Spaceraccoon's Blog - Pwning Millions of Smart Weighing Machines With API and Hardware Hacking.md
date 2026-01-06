---
author: Spaceraccoon's Blog
aliases:
  - Pwning Millions of Smart Weighing Machines With API and Hardware Hacking
tags:
  - readwise/articles
url: https://spaceraccoon.dev/pwning-millions-smart-weighing-machines-api-hardware-hacking/?__readwiseLocation=
created: 2025-04-01
---
# Pwning Millions of Smart Weighing Machines With API and Hardware Hacking

Internet-Connected… Weighing Machines??? [](https://read.readwise.io/read/01jq3j4db0pnbxe27jkdde48df)

I realised to my horror that people have now decided that connecting *weighing machines* to the internet is a good idea [](https://read.readwise.io/read/01jq3j4skgg11ec4av8b8a20wd). 

Many of them were made by the same OEM, and even if they were made by different OEMs with marginally different codebases, a quick peek at the associated Android applications revealed that many of them used the same common libraries, such as `com.qingniu.heightscale`, presumably because it would take way less effort to write a compatible library from scratch. [](https://read.readwise.io/read/01jq3j54kmdv08zbyy8qywn57a)

While the BLE protocol-related code was interesting and allowed me to figure out the right opcodes to communicate with these devices over Bluetooth, most of them have been reverse-engineered and documented by the [openScale project](https://github.com/oliexdev/openScale). [](https://read.readwise.io/read/01jq3j5zaqwhm8abzj2c5zavdf)

## We Need to Go Deeper

 If your goal is to hack not just one but all of the devices, a key target is the *user-device association flow*. [](https://read.readwise.io/read/01jq3j6k2kcmd45pw76pqvbpnc) When you first buy a smart device and take it out of the box, you often need to **login to a mobile application and scan a QR code or pair with the device over Bluetooth**. [](https://read.readwise.io/read/01jq3j6yssw8rdynwskm589sja)

This can be a tricky process to secure. Starting from the factory, each device needs a unique device identifier/secret so you don’t accidentally pair with another device B when scanning the QR code of device A. *The least-secure way* of doing this is using a static string such as a UUID, MAC address, or serial number. [](https://read.readwise.io/read/01jq3j82y83wcm1gek2hnwvp3d)

A more secure option would be to generate cryptographic keys like public/private key pairs. [](https://read.readwise.io/read/01jq3j9ef0fpy432radqmkej1a)

So the typical flow would go:
 1. User installs mobile app and logs in with their user account.
 2. Through the app, user connects to the hardware device.
 3. Hardware device’s secret is sent to the mobile app.
 4. Mobile app sends both the user’s secret (e.g., session token) and the device’s secret to the server.
 5. Server confirms the authenticity of the secrets and associates the user account to the hardware device.
 6. User can now control and fetch data from the hardware device remotely over internet. [](https://read.readwise.io/read/01jq3ja1hs4ztsj3qx8jptwvtc)

What could go wrong?

### [SQL Injection](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/SQL%20Injection.md) in OEM (BT-WAF Bypass)

I enumerated the available API endpoints on the mobile application, including an interesting `api/ota/update` endpoint. I thought this would allow me to get my hands on firmware to further understand the device. Thanks to the decompiled Java code for the Android mobile app, I was able to easily reconstruct the required JSON body parameters. [](https://read.readwise.io/read/01jq3jbe6em9g4bfetmhpszn5e)

Unfortunately, while exploring the API endpoints, I discovered that there were several endpoints that suffered from basic SQL injections. [](https://read.readwise.io/read/01jq3jbty2pdrqtdw8nvwgcsb1)

A `/api/device/getDeviceInfo` endpoint allowed looking up serial numbers for devices, which were used as *both identifiers and authentication secrets* by this manufacturer. [](https://read.readwise.io/read/01jq3jdr2g2gc78xnq1mt9x33x) The serial number itself was used in a `/api/device/bindv2` endpoint that would bind, or associate, the requesting user’s account with the device referenced by the serial number! [](https://read.readwise.io/read/01jq3je3h00g5kfhczm87q6nqb)

Here’s the initial payload body for the vulnerable endpoint: [](https://read.readwise.io/read/01jq3jfa353q71b69dpbd21n5e)
 ```json
{
 "serialnumber":"'001122334455"
 }
 ```

With a lot of trial and error, I eventually managed to land on this bypass for BT-WAF: [](https://read.readwise.io/read/01jq3jfetnmq8nzqv6y1ns55t5)
 ```json
{
 "serialnumber":"'or\n@@version\nlimit 1\noffset 123#"
}
```

There are two key bypass gadgets here:
1. `@@version` always evaluates to true and can be used instead of the more obvious `1=1`.
2. `\n` newlines can break up a statement instead of spaces. [](https://read.readwise.io/read/01jq3jghrqsfaa0wkc8vyc719r)

With this, I was now able to leak the device information, including the serial number used as authentication secrets, of any device! [](https://read.readwise.io/read/01jq3jh0njb6zeajgmpn631hsz)

### From Day Zero to Zero Day

Book with No Starch Press! I wrote [“From Day Zero to Zero Day”](https://nostarch.com/zero-day) for newcomers looking to enter the rarefied world of vulnerability research. From code review to reverse engineering to fuzzing, I go through the “how”, not just the “what”, of hunting zero days - stuff that I wish I could’ve learned from the beginning. Available for [early access](https://nostarch.com/zero-day) now at all your usual channels including [Amazon](https://www.amazon.com/Day-Zero/dp/1718503946), out in June 2025! [](https://read.readwise.io/read/01jq3hnz5nzx0br5f1326b7bc6)

