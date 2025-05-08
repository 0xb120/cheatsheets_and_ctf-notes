---
author: "Jack Maginnes"
aliases: "Uncovering a 0-Click RCE in the SuperNote Nomad E-Ink Tablet"
tags: RW_inbox, readwise/articles
url: https://www.prizmlabs.io/post/remote-rootkits-uncovering-a-0-click-rce-in-the-supernote-nomad-e-ink-tablet?ref=blog.exploits.club?__readwiseLocation=
date: 2025-04-28
summary: Researchers discovered a serious security flaw in the SuperNote Nomad E-ink Tablet that allows for a remote installation of a rootkit without any user interaction. They found that a misconfiguration in the device's file-sharing system could be exploited to overwrite important files with malicious updates. After notifying the manufacturer, they agreed to fix the vulnerability in a future update.
---
# Uncovering a 0-Click RCE in the SuperNote Nomad E-Ink Tablet

![rw-book-cover](https://static.wixstatic.com/media/82b748_624e8f67c9d947258767a24d625667cf~mv2.avif/v1/fill/w_980,h_551,al_c,q_85/82b748_624e8f67c9d947258767a24d625667cf~mv2.avif)

## Highlights


What follows is a blog post detailing how we were able to chain a vulnerability and a handful of misconfigurations into a remotely installable, 0-click rootkit. A malicious attacker on the same network as the victim could fully compromise the target device without any user-interaction. [](https://read.readwise.io/read/01js1fpx32q7assd9fhkazz3w7)



we found port 60002 open and listening. Nmap was unable to identify the service directly, so we decided to investigate this mysterious port a bit further by grabbing a firmware image for the device from Ratta Software's ["Updates" page](https://support.supernote.com/en_US/change-log/how-to-update-your-supernote). [](https://read.readwise.io/read/01js1fr7204c83fcmp8dpghtd2)



This led us to the *SuperNoteLauncher.apk* [](https://read.readwise.io/read/01js1frt891afc6m7r9kprq31k)



Looking for cross-references, we eventually tracked down where it was being used: *com.ratta.supernote.wifip2p.receive* [](https://read.readwise.io/read/01js1fvg387vtt0n79mc54px75)



when the port is open, a handful of functions are triggered after something is received over the *ServerSocket*. Specifically, the code of interest resides in the *DeviceThread* class. [](https://read.readwise.io/read/01js1fvsgbn8b9gyq26cgkpems)



*DeviceThread* implements *Runnable* and its r*un()* function is a behemoth. That said, a quick glance at it indicates that we are likely dealing with a custom HTTP server based on the way error messages are communicated back to the client. [](https://read.readwise.io/read/01js1fvyd5pax0atjx3ewrwh11)



but what piqued our interest most was the class names: *RecieverManager* and *WiFiP2PService*. Unauthenticated device-to-device file sharing sounds like a potentially great attack surface. [](https://read.readwise.io/read/01js1fyba4qh3m5vp6yaa5zh79)



Given that we at-least know how to interact with the server, at this point we went ahead and mocked up a python client that just sends a POST request with an attached file and all the necessary headers...andddd the file showed up on the device in the INBOX directory! [](https://read.readwise.io/read/01js1fyy64rrcjhjx3y923p205)



Investigating A Path Traversal:
 After figuring out how to get files into the INBOX directory, we did what all good security researchers would do: added some "dot-dot-slashes" to our payload to see what would happen. [](https://read.readwise.io/read/01js1fzmr2xt09h7vwny9h06m3)



headers = {"version": "1", "content-length": "1234", "name": "../../../../../../../../sdcard/EXPORT/testfile.txt", "devicename": "testdevice"} And what do you know? It worked! [](https://read.readwise.io/read/01js1g0yyatxc2ze517j0x3f0w)



"*(1)"* appended to the end of our filename? Yeah, that is going to prove to be quite the problem during exploitation here in a second. [](https://read.readwise.io/read/01js1g2q2vpp4dsr6wns46xj9c)



Exploit Plan: [](https://read.readwise.io/read/01js1g3ngr2kq3z1t4s3bdbjyw)



• SuperNote's instruction on the Firmware Update page specifically ask you to put the downloaded zip file into the EXPORT directory
 • A manual firmware update will automatically start after a hotplug event or reboot if a valid update image is found in the EXPORT directory
 • A researcher [poked at the previous generation of SuperNote devices](https://github.com/TA1312/supernote-a5x) and found that firmware images were signed with publicly available debug keys and the bootloader was unlocked by default...nice. [](https://read.readwise.io/read/01js1g48kd32bshhcdcwjampcb)



we figured we could do the following:
 1. Create a backdoored firmware image and sign it with the publicly available debug keys
 2. Use the arbitrary write to get the download directly into the "EXPORT" directory
 3. An update would automatically be initiated during normal device usage, installing our malicious rootkit. [](https://read.readwise.io/read/01js1g4pvj60gmz2tkn65gbqz8)



The Problem
 Remember that "*(1)*" appended to our filename? Yeah, here is where it comes back to bite us. [](https://read.readwise.io/read/01js1g5ay179xnrp7qqd02g7gp)



the service that scans for the update in the EXPORT directory [](https://read.readwise.io/read/01js1g64y8dsehq1f6cpxbb3t9)



filename has to be exactly *update.zip*...not *update(1).zip*. [](https://read.readwise.io/read/01js1g6g3m3papkpwtwqtdd3hb)



we need to figure out *why* the extra number gets appended to our path traversal payload and how we can get rid of it. [](https://read.readwise.io/read/01js1g6w4pzeeg26rfbjbt3hav)



we finally determined the root cause of the issue [](https://read.readwise.io/read/01js1g82wmmjvmg6vy83b257w8)



• The server first creates a file in its application directory under the "*/*receiver*_file_cache/file_name*" path
 • It then copies the incoming stream of data into that newly created file in "receiver*_file_cache/file_name*" path
 • When it has reached the end of the incoming stream of data, it then creates a new file, named `INBOX/file_name`
 • It copies the contents from "receiver*_file_cache/file_name"* over to "*INBOX/file_name"*
 • Finally, it deletes the cached file. [](https://read.readwise.io/read/01js1g875p9y63t5n85xdatkvt)



The problem arises due to the continued use of the attacker supplied filename for *all* operations. [](https://read.readwise.io/read/01js1g99ewzb06cfq33sawt0xf)



• The server receives and creates the "*/*receiver*_file_cache/../../../../sdcard/EXPORT/update.zip*" file, thinking this should be the cache file
 • Then after it has finished receiving the data, it goes to create the INBOX file, but actually attempts to create "*/INBOX/../../../../sdcard/EXPORT/update.zip*" again [](https://read.readwise.io/read/01js1ga3wwgm4zcyk99ztpwm70)



Finding A Naming Issue Bypass Via A "Race Condition" [](https://read.readwise.io/read/01js1gb0brjaqfcgst91h27m57)



We need to figure out how to get our actual update file into EXPORT with the right name. [](https://read.readwise.io/read/01js1gb7gw6bpq1sq5r3gg0t8n)



What *did* end up working was taking advantage of a logical "race condition"...technically it's not *actually* a vulnerability or a traditional race condition, but we can leverage it to our advantage. [](https://read.readwise.io/read/01js1gbwyqvkjmyz33mt6rc1hj)



You see, a legitimate update file is 1.1GB large. That means, it takes quite a bit of time to transfer. [](https://read.readwise.io/read/01js1gczbvds55zc0wadg83g9x)



Revised Exploit Strategy: [](https://read.readwise.io/read/01js1gdn7wdcvfk8h1aek783gz)



• Create a very small, dummy file named `update.zip`
 • Create a legitimate `update.zip` file that contains our malicious backdoor, signed with the publicly available development keys
 • Send the the very small, dummy file first
 • Send the legitimate `update.zip` file immediately after. [](https://read.readwise.io/read/01js1ge13dqa5z4fghnggr0db2)



if we do that, the following happens on device [](https://read.readwise.io/read/01js1ge9cdy0deqz358whvth67)



• *`EXPORT/update.zip` is created by the dummy file during the caching / initial reception step*
 • **`EXPORT/update(1).zip` is created by the real update during the caching / initial reception step**
 • *`EXPORT/update(2).zip` is created by the dummy file during the copy step*
 • *`EXPORT/update.zip` is deleted by the dummy file after completion of the copy step*
 • **`EXPORT/update.zip` is created by the real update during copy step**
 • **`EXPORT/update(1).zip` is deleted by the real update after completion of copy step** [](https://read.readwise.io/read/01js1gewpa25x4pmwvxw6dkwyx)



Because our dummy file is so much smaller than the real update file, it completes the caching, copy, and delete steps before the valid update is done being recieved. [](https://read.readwise.io/read/01js1gfaeycqr5bdt5xb7bj2qq)



Backdooring The Firmware Image
 We won't go too in-depth on this step, but we found the development keys needed after a bit of Googling around. From there, we created a backdoor using [flashable-android-rootkit](https://github.com/ng-dst/flashable-android-rootkit) and writing simple C reverse shell payload. [](https://read.readwise.io/read/01js1gfytvknpqe2c70qg5x366)



To re-package the firmware, we followed the recommended method from the [previous research](https://github.com/TA1312/supernote-a5x?tab=readme-ov-file#build-and-sign-your-own-updatezip) and used [Multi Image Kitchen](https://forum.xda-developers.com/t/kitchen-windows-multi-image-kitchen-repack-android-partitions.4326387/) [](https://read.readwise.io/read/01js1gg7y8vtfj8mz6beqtgq9k)

