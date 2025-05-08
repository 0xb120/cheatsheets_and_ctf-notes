---
author: "admin"
aliases: "Fork Bomb for Flutter"
tags: RW_inbox, readwise/articles, flutter
url: https://swarm.ptsecurity.com/fork-bomb-for-flutter/?__readwiseLocation=
date: 2025-04-24
summary: The article discusses how to reverse engineer Flutter applications, which are built on Google's open-source SDK for cross-platform development. It covers the compilation process, how to create patches, and the tools used, such as Docker and BurpSuite. The author hopes to enhance the reverse engineering of Flutter apps through the utility they created for compiling the Flutter Engine.
---
# Fork Bomb for Flutter

![rw-book-cover](https://swarm.ptsecurity.com/wp-content/uploads/2022/08/41c5fd63-res.png)

## Highlights


Architecture overview
 Flutter is an open-source SDK from Google for developing cross-platform applications. [](https://read.readwise.io/read/01js1ckmgsnny193z3q7337kdc)



Flutter is built on C, C++, Dart, and Skia. [](https://read.readwise.io/read/01js1ckycmmpb9646zc6nwzwvm)



Flutter consists of three architectural layers [](https://read.readwise.io/read/01js1cma9bjd42b7d4zpssw0sh)



[![](https://swarm.ptsecurity.com/wp-content/uploads/2022/09/fb03770a-image_2022-09-01_18-55-54.png)](https://swarm.ptsecurity.com/wp-content/uploads/2022/09/fb03770a-image_2022-09-01_18-55-54.png) [](https://read.readwise.io/read/01js1cmdk9ksa2ks53xprng46c)



**Framework** is a cross-platform layer written in the Dart language. It includes a rich set of platforms, [layouts](https://docs.flutter.dev/development/ui/layout) and foundational libraries. Many higher-level features that developers might use are implemented as packages, including platform plugins like [camera](https://github.com/flutter/plugins/tree/main/packages/camera/camera), [webview](https://github.com/flutter/plugins/tree/main/packages/webview_flutter/webview_flutter), and other functions like [http](https://github.com/dart-lang/http) and animation. [](https://read.readwise.io/read/01js1cp6ebgzak7atyg2zhx6xy)



**Engine** is a portable runtime for hosting Flutter applications that contains the required SDK for Android, iOS, or Windows; it is mostly written in C++ and provides primitives to support all Flutter applications. [](https://read.readwise.io/read/01js1cpmvr1qcwpgqd88ckn55c)



Flutter app developers write code on Dart language using the Framework. This code is executed in the Dart VM, which the Engine provides. [](https://read.readwise.io/read/01js1cq1nd3g64cb8661qf4zvf)



To compile a Flutter application, [Engine](https://github.com/flutter/engine) is used to create an [AOT AppSnapshot](https://github.com/dart-lang/sdk/wiki/Snapshots) containing precompiled machine code: the Framework source code and the developers’ source code. [](https://read.readwise.io/read/01js1cr2s85259mk4cpd56gst7)



This is approximately how the compilation process looks in AOT:
 [![](https://swarm.ptsecurity.com/wp-content/uploads/2022/08/a0c0df7b-image_2022-08-29_12-05-28.png)](https://swarm.ptsecurity.com/wp-content/uploads/2022/08/a0c0df7b-image_2022-08-29_12-05-28.png) [](https://read.readwise.io/read/01js1cthfyxsqth77w1b9y8wp8)



Next, the obtained libapp.so is combined with resources, dex files, and the libflutter.so library into a single zip archive, which is signed and made ready-to-use [release.apk](https://github.com/Impact-I/ForkForFlutter/blob/main/flutter_app/app-release.apk) [](https://read.readwise.io/read/01js1cvfzshdb4jxswrkb8t1tc)



Structure of the release.apk file with comments:
 ├── AndroidManifest.xml
 ├── assets
 │ └── flutter_assets
 │ └── AssetManifest.json
 ├── classes.dex ──── // Java (Dalvik Executable)
 ├── kotlin ──── // kotlin Metadata
 ├── lib
 │ └── arm64-v8a
 │ ├── libapp.so ──── // Dart code (App AOT Snapshot)
 │ └── libflutter.so ──── // Flutter Engine (stripped version of Dart VM)
 ├── META-INF
 ├── res
 └── resources.arsc [](https://read.readwise.io/read/01js1cwefq0y4nxd0bgmnv6mj1)



The [libflutter.so](https://github.com/Impact-I/ForkForFlutter/blob/main/flutter_app/unzip_app-release/lib/arm64-v8a/libflutter.so) file (part of the Flutter Engine) contains the required functionality for using the OS (network, file system, etc.) and a stripped version of the [DartVM](https://github.com/dart-lang/sdk/blob/2b88e9de118097d12183aa30ec9f8c6bed9d64fc/runtime/vm/clustered_snapshot.cc). This version is known as precompiled runtime, which does not contain any compiler components and is incapable of loading Dart source code dynamically. However, it handles reading of sections, deserializing, and loading instructions (binary machine code) into executable memory from the ELF file [libapp.so](https://github.com/Impact-I/ForkForFlutter/blob/main/flutter_app/unzip_app-release/lib/arm64-v8a/libapp.so). [](https://read.readwise.io/read/01js1cy1rpdxj65w8y9e7d7yxt)



~$ readelf -Ws libapp.so Symbol table '.dynsym' contains 6 entries: Num: Value Size Type Bind Vis Ndx Name 0: 0000000000000000 0 NOTYPE LOCAL DEFAULT UND 1: 0000000000001000 8 FUNC GLOBAL DEFAULT 1 _kDartBSSData 2: 0000000000002000 17792 FUNC GLOBAL DEFAULT 2 _kDartVmSnapshotInstructions 3: 0000000000007000 0x1f89c0 FUNC GLOBAL DEFAULT 3 _kDartIsolateSnapshotInstructions 4: 0000000000200000 32288 FUNC GLOBAL DEFAULT 4 _kDartVmSnapshotData 5: 0000000000208000 0x18c180 FUNC GLOBAL DEFAULT 5 _kDartIsolateSnapshotData [](https://read.readwise.io/read/01js1d2sdq2e31x7apmc1qyk6v)



All Dart code runs in an isolate. Multiple isolates can execute Dart code concurrently but cannot share any state directly and can only communicate by passing messages. [](https://read.readwise.io/read/01js1czt1dq7mz57h2wge779m4)



Let’s have a look at the [structure](https://github.com/flutter/flutter/wiki/Flutter-engine-operation-in-AOT-Mode) of the libapp.so file:
 **Isolate Instructions** — `_kDartIsolateSnapshotInstructions` — Contains the AOT code that is executed by the Dart isolate. It must live in the **text** segment.
 **Isolate Snapshot** — `_kDartIsolateSnapshotData` — Represents the initial state of the Dart heap and includes isolate specific information. Along with the VM snapshot, it helps in faster launches of the specific isolate. Should live in the **data** segment.
 **Dart VM Instructions** — `_kDartVmSnapshotInstructions` — Contains AOT instructions for common routines shared between all Dart isolates in the VM. This snapshot is typically extremely small and mostly contains stubs. It must live in the **text** segment.
 **Dart VM Snapshot** — `_kDartVmSnapshotData` — Represents the initial state of the Dart heap shared between isolates. Helps launch Dart isolates faster. Does not contain any isolate specific information. Mostly predefined Dart strings used by the VM. Should live in the **data** segment. From the VM’s perspective, this needs to be loaded in memory with READ permissions and does not need WRITE or EXECUTE permissions. In practice, this means it should end up in .rodata when putting the snapshot in a shared library. [](https://read.readwise.io/read/01js1d3wa6vm1z9g02q9hjav30)



Each Engine (libflutter.so) stores an md5 hash (Snapshot_hash) to separate the build versions. This hash is generated on the basis of major changes in the Engine source code at compile time using the [make_version.py](https://github.com/dart-lang/sdk/blob/d89e42bef5e5b8a5064790332adb8682977e4e86/tools/make_version.py#L18) script. [](https://read.readwise.io/read/01js1d4e5twapk8qnr6gfrbk3k)



To check the compatibility of libflutter.so and libapp.so, the same Snapshot_hash is stored in them. [](https://read.readwise.io/read/01js1d4m6t8gsfkypyt1y586sb)



There are three release versions in total: stable, beta and dev.
 These versions (excluding dev) can be found here: [docs.flutter.dev/development/tools/sdk/releases](https://docs.flutter.dev/development/tools/sdk/releases); [](https://read.readwise.io/read/01js1d5hf59yp3bra0d5q0akvy)



Recompilation as a reverse engineering approach
 We now know that the application code is stored in the libapp.so file, which the Engine reads. And the actual instructions for the code are in the `_kDartIsolateSnapshotInstructions` segment.
 Unfortunately, the file is divided into segments with different types of objects and has a complex structure that requires deserialization; this is confusing and hinders the reverse engineering process, making it harder to know where the required instruction begins and what function it handles.
 To understand how deserialization works we can learn the source code of libflutter.so, which is available here: [github.com/dart-lang/sdk](https://github.com/dart-lang/sdk). Then it will be possible to write a parser for the elf format, which should perform the same functions as the Engine.
 However, examining the dart-sdk source code will still take some time. Recall that Dart is constantly under development, and the functionality of DartVM is no exception. It’s worth remembering that a parser written for a Snapshot is not universal. If the developer compiles its code using a newer version of Flutter, you will have to rewrite the parser to read the new libapp.so. [](https://read.readwise.io/read/01js1d9m8xzqcqxpxg4nbz26kb)
