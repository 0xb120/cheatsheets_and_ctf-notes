---
author: "Guardsquare"
aliases: "Dart Decompilation and the Impact on Flutter™ App Security"
tags: RW_inbox, readwise/articles, Flutter
url: https://www.guardsquare.com/blog/obstacles-in-dart-decompilation-and-the-impact-on-flutter-app-security?__readwiseLocation=
date: 2025-04-24
summary: This article examines the challenges of decompiling Dart code and its implications for Flutter app security. It highlights issues like the custom Dart VM stack and non-standard function calling conventions that complicate reverse engineering. The authors demonstrate how to improve decompiled code by reintroducing cross-references and making the code easier to understand.
---
# Dart Decompilation and the Impact on Flutter™ App Security

![rw-book-cover](https://www.guardsquare.com/hubfs/Blog%20assets/Blog%20featured%20images/Obstacles-in-Dart-decompilation-and-the-impact-on-Flutter%E2%84%A2--app-security.jpg)

## Highlights


Why does the decompiled code look weird? [](https://read.readwise.io/read/01jqv9zvvw4zasn71rax3zf5yw)



All the strange looking artifacts that you can see are linked to the third obstacle that we discussed in the previous post: the Dart code depends on the Dart VM to be executed. [](https://read.readwise.io/read/01jqva0fnemycvpqh4xt3mw7q6)



The three main issues that we can see in previous image are linked to the **three following characteristics of the Dart code**:
 1. All Dart **objects are accessed through the object pool**. Thus, the decompiled code only contains the index of the object accessed and does not reference the object itself.
 2. The Dart VM **uses a custom stack** indexed by register`X15`, which prevents reverse engineering tools from correctly identifying local stack variables out of the box.
 3. Dart decompiled code uses a non-standard ARM64 ABI where most of the parameters of function calls are pushed to the Dart VM stack. Thus, IDA Pro isn’t able to identify function parameters and it considers that functions have no parameters. [](https://read.readwise.io/read/01jqva1x3cy2v98ad8y1td6vt3)



Dart code characteristic 1: Object pool indirection [](https://read.readwise.io/read/01jqva2arttmqe5z3gngxvd83h)



The first issue is closely linked to the way Dart snapshots work. When you compile a Flutter application, all Dart objects are serialized and stored inside the Dart snapshot. Thus the Dart code can’t access them directly. When the Flutter runtime loads your application, these objects are deserialized and stored on the Flutter heap. Because of that, Dart code can’t know the addresses of these deserialized Dart objects at compilation time, thus it needs to access them using an indirection at runtime. [](https://read.readwise.io/read/01jqva3g8bbfp840sgpm27y4ad)



This is where the Dart object pool comes into play. At compilation time, a reference to each Dart object is stored in a big array called the Dart object pool, which is itself serialized [](https://read.readwise.io/read/01jqva3z59vavwrd7r437rmvd3)



At the same time, in the Dart decompiler code, all direct accesses to these objects are replaced by indirect access through the object pool using their corresponding object pool index. [](https://read.readwise.io/read/01jqva4791krr245ktw5506zhm)



The following figure demonstrates what is happening at runtime:
 • Steps 1 and 2 are performed while the Flutter engine loads the Flutter application
 • Steps 3 and 4 are performed each time the Dart code wants to access a Dart object. As you can see, the Dart code never uses the serialized Dart objects. [](https://read.readwise.io/read/01jqva4k00zabxp8871red53d7)



The usage of this object pool technique has the following impacts on Dart code: 
 • There are no direct cross-references between code and data, all these links are hidden by the object pool indirection.
 • It explains why we can only see the index of the Dart object in the decompiled code instead of the Dart object itself. [](https://read.readwise.io/read/01jqva4zyfw2f5qm7j10gywbfj)



Dart code characteristic 2: Custom stack [](https://read.readwise.io/read/01jqva5s7y1feh0d6bwp7ty9g2)



The second issue is caused by the Dart SDK having its own stack, which is indexed by the`X15`register instead of the classical ARM64 `SP` register. [](https://read.readwise.io/read/01jqva63jdhx67gcqqp81m132w)



Most reverse engineering tools are able to deduce a lot of things by analyzing the stack operations and accesses. [](https://read.readwise.io/read/01jqva6hxjhbbzpy95d06qkkxm)



Because the register used for stack operations by the Dart VM is not the classical ARM64 `SP`register, all these analysis passes don’t work on Dart code. This is the reason why, in the decompiled code, there are no local variables detected and functions don’t have any parameters. [](https://read.readwise.io/read/01jqva6w9y9mvgs7nt2pndav9d)



Dart code characteristic 3: Custom ABI [](https://read.readwise.io/read/01jqva7j5pvse2n5yg317pdv6d)



The third issue is caused by the fact that the Dart SDK does not use the standard ARM64 calling convention. Rather than using registers`X0-X7`to pass arguments to called functions, it pushes most arguments on the custom Dart VM stack. [](https://read.readwise.io/read/01jqva7y39d6enw6wtktb4vyfs)



One important thing to keep in mind is that all this information is public since the Dart SDK is open source. For instance, all special register values and purposes can be found [here](https://github.com/dart-lang/sdk/blob/b80e682cbf313691650cc9c45302811deb710aa0/runtime/vm/constants_arm64.h). Additionally while trying to get more information on the second and third issues, we found [an open issue](https://github.com/dart-lang/sdk/issues/39083) on the Dart SDK GitHub repo which is pushing for the adoption of standard ARM64 ABI and usage of `SP`instead of `X15`. If these changes are implemented, they will directly fix the second and third issues identified. [](https://read.readwise.io/read/01jqva9813vh6sh47tbk73g38r)

