---
author: cryptax
aliases:
  - Reversing an Android Sample Which Uses Flutter
tags:
  - RW_inbox
  - readwise/articles
  - Flutter
url: https://cryptax.medium.com/reversing-an-android-sample-which-uses-flutter-23c3ff04b847?__readwiseLocation=
created: 2025-04-24
---
# Reversing an Android Sample Which Uses Flutter

![rw-book-cover](https://miro.medium.com/v2/resize:fit:1200/1*uifnkrWZPEmc586QKl3TmA.png)

## Highlights


Reverse engineering Flutter-based Android appsHow do I detect the app uses Flutter?
 There are two cases.
 • If the app is in **debug mode**, you are lucky. Unzip the APK and look for the code in `./assets/flutter_assets/kernel_blob.bin` [1]
 • If the app is in **release** **mode** (which is the case for the suspicious sample), you will find `libflutter.so` in `./lib/` subdirectories. [](https://read.readwise.io/read/01jqtzwgk9aj8w7kwqp22y1df7)



Where is the Dart code? [](https://read.readwise.io/read/01jqv05aw8sbbzy3vd60dgcfza)



In Flutter release apps, the app payload is located in `./lib/<platform>/libapp.so`[2]. [](https://read.readwise.io/read/01jqv05j2t57b3dr4x8t2dbkqk)



When we inspect `libapp.so`, we see a VM “snapshot” and an isolate snapshot. Snapshots are the serialized state of an isolate, frozen at a given moment.
 ![](https://miro.medium.com/v2/resize:fit:700/1*SsOasZCwCMHcql9W_NjLaw.png) [](https://read.readwise.io/read/01jqv07sftcy7wtct6fay2carn)



more precisely, the Dart code of the app is `_kDartIsolateSnapshotInstruction` (and Data). [](https://read.readwise.io/read/01jqv0d79nbg785zptrvhvsf17)



[reFlutter](https://github.com/Impact-I/reFlutter) [7]: this framework operates differently. The idea is to patch the sample and use a patched version of the Flutter library. Then, to write Frida hooks and dynamically analyze calls to the patched library. [](https://read.readwise.io/read/01jqv0f118ys5kkz6c7r29698y)

