---
author: Guardsquare
aliases: The Current State & Future of Reverse Engineering Flutter™ Apps
tags:
  - _inbox
  - readwise/articles
  - Flutter
url: https://www.guardsquare.com/blog/current-state-and-future-of-reversing-flutter-apps?__readwiseLocation=
date: 2025-04-24
summary: Reverse engineering Flutter apps is currently challenging due to the constantly changing Dart snapshot format and the way Dart frameworks are linked. While there are tools to extract information from Dart snapshots, they must be updated with each new Flutter release. Despite built-in obfuscation options, some metadata remains accessible, making it possible to identify common framework functions in the code.
---
# The Current State & Future of Reverse Engineering Flutter™ Apps

![rw-book-cover](https://www.guardsquare.com/hubfs/The-Current-State-and-Future-of-Reversing-Flutter%E2%84%A2-Apps.jpg)

## Highlights


open source Flutter game called [NyaNya Rocket!](https://github.com/CaramelDunes/nyanya_rocket) as an example throughout this post. Although this is an open source game, we will analyze it as if we don’t have access to the source code. [](https://read.readwise.io/read/01jqv710yk9vvss8jqas2g89d1)



we have prepared a [Github repo](https://github.com/Guardsquare/flutter-re-demo) with all applications and scripts! [](https://read.readwise.io/read/01jqv71ag43yswqej81bpqvdqt)



We identify three main obstacles that currently slow down Flutter reverse engineering:
 1. The Dart AOT snapshot format is changing a lot with each update.
 2. All Dart frameworks are statically linked in the application binary.
 3. The Dart code relies on the Dart VM to be executed. [](https://read.readwise.io/read/01jqv726n9jbxqb4zmcjwcwkxt)



Dart language is still young and evolving. Because of that, the format of the Dart snapshot, which contains all the compiled machine code and data for a Flutter application, keeps changing, too. [](https://read.readwise.io/read/01jqv736cy81pq5k51972favaj)



The second obstacle is caused by all Dart frameworks used by an app being statically linked into the Dart snapshot [](https://read.readwise.io/read/01jqv74qkd56n234wkz62nyrj6)



three main consequences [](https://read.readwise.io/read/01jqv74tknw9nv94j719f88q8y)



The size of the Dart snapshot is way bigger [](https://read.readwise.io/read/01jqv74z5eh7ctaby7q8m4ycf5)



It can be hard to distinguish application code from framework code [](https://read.readwise.io/read/01jqv753rke63d0abrbc59aq5w)



It is not directly possible to guess what a function is doing by looking at framework function calls [](https://read.readwise.io/read/01jqv75hmf8r1770hqn3jx9fwb)



The third obstacle is the dependency of Dart code on the Dart VM to be executed. [](https://read.readwise.io/read/01jqv760dhzeb3mn8n3ga31htr)



There are no direct references from a static data definition to the places it is used. [](https://read.readwise.io/read/01jqv76bdt8t2s48zfgryx6fjn)



The Dart VM uses a custom register layout and ABI. [](https://read.readwise.io/read/01jqv76mxa9qp0wbq6znsw3deq)



If you are interested in the internals of the Dart VM from a reverse engineering perspective, I advise reading [the blogpost series from Andre Lipke](https://blog.tst.sh/reverse-engineering-flutter-apps-part-1/). [](https://read.readwise.io/read/01jqv772277dhrc78f6m8wvqe9)



Extracting Information from a Flutter Snapshot [](https://read.readwise.io/read/01jqv77x9khx8h8h1z9vpajbnm)



What information could be retrieved from a Flutter snapshot and what is the state-of-the-art for doing so? [](https://read.readwise.io/read/01jqv78a34sscvrkgjsjbb1hqw)



• The compiled code that will be used to run the application. It includes not only application-specific code but also the code of all frameworks used by the application.
 • All strings or static data used by the Flutter app.
 • A lot of metadata that is used by the Flutter engine to make a Flutter app run:
 • Some is mandatory, like the definition of Dart objects and the Dart VM object pool.
 • Some is optional, like the class/function names, but can be useful to, for instance, print symbolicated stack traces when a crash occurs. [](https://read.readwise.io/read/01jqv79fr3zh9a6hwkpkrkpyjw)



State-of-the-art Approaches for Flutter Metadata Extraction [](https://read.readwise.io/read/01jqv7azvntektzpehq0m3yp6x)



3 ways this information can be extracted:
 • Using a Dart snapshot parser
 • Using a modified version of the Flutter runtime library
 • Using debug information [](https://read.readwise.io/read/01jqv7bjdc4vv42n2zdx97568c)



If you search for Dart snapshot parsers online, you will find several of them including [darter](https://github.com/mildsunrise/darter) and [Doldrums](https://github.com/rscloura/Doldrums). [](https://read.readwise.io/read/01jqv7c0v14r4mhw13swabaryh)



The second approach is the one chosen by e.g. the [reFlutter](https://github.com/Impact-I/reFlutter) project. Rather than trying to parse the Dart snapshot, it modifies the Flutter runtime library to make the application dump information at runtime. [](https://read.readwise.io/read/01jqv7dwm9rvkpj8s51nrnaxkt)



A third approach to extract this information is to leverage the debug information that is generated when building a Flutter application with the`--split-debug-info`flag. Using this flag will generate a DWARF file that can be easily parsed and which contains class/function names and their associated offset in the`libapp.so`. [](https://read.readwise.io/read/01jqv7tb8sc396d4yd756pwz2y)



What about Flutter’s Built-in Obfuscation? [](https://read.readwise.io/read/01jqv7xvcagvhb0cch9rze8xd6)



Flutter has a built-in option that automatically [obfuscates Dart code](https://docs.flutter.dev/deployment/obfuscate) inside the Flutter app. When this option is enabled, most module/class/function names are replaced by random names. [](https://read.readwise.io/read/01jqv7y57rmskqq54qj56th8nf)



this is not game over, since no [obfuscation](https://www.guardsquare.com/what-is-code-obfuscation) is applied to the code itself, classical binary diffing tools such as [BinDiff](https://www.zynamics.com/bindiff/manual/) or [Diaphora](https://github.com/joxeankoret/diaphora) can be used to recover the original name of functions [](https://read.readwise.io/read/01jqv7yrh27jbpwr0vr3m3mpc8)

