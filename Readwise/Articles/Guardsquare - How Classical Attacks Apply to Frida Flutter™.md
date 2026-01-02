---
author: Guardsquare
aliases:
  - How Classical Attacks Apply to Frida Flutter™
tags:
  - RW_inbox
  - readwise/articles
  - Flutter
url: https://www.guardsquare.com/blog/how-classical-attacks-apply-to-flutter-apps?__readwiseLocation=
date: 2025-04-24
---
# How Classical Attacks Apply to Frida Flutter™

![rw-book-cover](https://www.guardsquare.com/hubfs/Blog%20assets/Blog%20featured%20images/How-classic-attacks-apply-to-FlutterTM-apps_GUARDSQUARE.jpg)

## Highlights


In this post, we will start by looking at the game we will target. Then we will investigate how three common techniques used in cheats apply to Flutter applications:
 • Static tampering of data
 • Static tampering of code
 • Dynamic tampering of code via hooking [](https://read.readwise.io/read/01jqvav3r2ywz5jg4hn90wrtqs)



Understanding Our Target [](https://read.readwise.io/read/01jqvav7ag063j0ah3e516wfd8)



The first step in attacking an application is getting an overall understanding of what the app is doing. Only after gaining this level of clarity is it possible to decide which kind of attack to implement. [](https://read.readwise.io/read/01jqvavjw41pq22dpt367e97fm)



Tampering with Flutter static objects [](https://read.readwise.io/read/01jqvaw3dncdcmer95scvmsxxy)



One popular attack that we see on mobile applications consists in repackaging the application while changing some parts. [](https://read.readwise.io/read/01jqvawnbbgj7nqdvw12j8hn1d)



In this first section, we will discuss the easiest type of tampering which requires very little experience: asset or data tampering. [](https://read.readwise.io/read/01jqvax82kqg5ba27rav9vywqp)



All mobile applications use some data, which is typically very easily understandable by an attacker, such as graphical assets or strings. Depending on the application architecture, this data can be stored in asset files and/or directly in the executable binary, but in any case, the attacker can quickly locate and patch them [](https://read.readwise.io/read/01jqvaxn6easnwwzgbyvy5jtsv)



• For asset files, the attacker can just replace files in the`res`folder of the APK.
 • For strings in DEX files, since DEX files are disassembled into`smali`files (which are text files with assembly instructions in them), the attacker only has to search for the target strings in the smali files and replace them by the strings they want.
 • For strings in native library code, it’s almost the same: the attacker can find the strings in the binary and patch them. However there are some restrictions; the replacement string length, for example, must be smaller or equal to the original string length. If the replacement string was larger than the original one, other data in the binary would be altered, which could cause the application to crash. [](https://read.readwise.io/read/01jqvaynszx4mvhqnfvwd8yz2j)



Data Tampering: The Flutter case [](https://read.readwise.io/read/01jqvayxvvvr2nvq3j4qwwsm1d)



Since Flutter apps store assets in the same way as regular mobile apps, patching asset files in a Flutter application is as easy as patching them on any mobile application. [](https://read.readwise.io/read/01jqvaz5hpj0em7504swjhrst4)



When it comes to data (e.g. strings) patching in Flutter application, it is important to remember that all Dart code is compiled into a native library (`libapp.so`) and that all Dart objects (including the strings) are serialized and stored in this binary. [](https://read.readwise.io/read/01jqvb0gn86ekf9gg28nz3m5d5)



Therefore, patching strings in a Flutter application is exactly the same as patching a string in a native library of a standard mobile application. [](https://read.readwise.io/read/01jqvb0rp450zamjjh9gth9nz1)



Let’s consider the following`Supa Sonic`puzzle. We can search for this string in the native library:
 $ strings libapp.so | grep "Supa Sonic"
 {"name":"Supa Sonic","gameData":"{...}”,"arrows":[0,1,0,0]} [](https://read.readwise.io/read/01jqvb24vqs6pj91a1jg9f36n8)



To patch this library, no advanced reverse engineering tool is needed. We can just use a simple Python script:
 def change_puzzle_arrows(libapp_path, puzzle_name, nb_arrows, output_path):
 with open(libapp_path, "rb") as fp:
 orig_data = fp.read()
 puzzle_string_index = orig_data.index(puzzle_name)
 puzzle_arrows_string_index = puzzle_string_index +
 orig_data[puzzle_string_index:].index(b"\"arrows\"")
 Patched_arrows = 
 F"\"arrows\":[{nb_arrows[0]},{nb_arrows[1]},{nb_arrows[2]},{nb_arrows[3]}]"
 .encode("ascii")
 patched_data = orig_data[:puzzle_arrows_string_index] + patched_arrows + \
 orig_data[puzzle_arrows_string_index + len(patched_arrows):]
 with open(output_path, "wb") as fp:
 fp.write(patched_data) [](https://read.readwise.io/read/01jqvb6g1nb6zyrjhfe977x3cg)



In this example, we patched a string, but the same technique works exactly in the same way when patching any static data (i.e. every Dart serialized object) included in an application. However, since Flutter data is stored in a native library, the patched data must have the same length as the original data. Thus, in the previous example, we can’t change the number of arrows to 10 or more because it would add digits which would change the string size. [](https://read.readwise.io/read/01jqvb92a3htd93rsg5h6jxz44)



Tampering with Flutter code [](https://read.readwise.io/read/01jqvba0rde2wdnx3m46trcqfn)



The goal of code tampering is to modify the application logic by changing one or several instructions directly in the compiled binary:
 • When patching a DEX file, we have a lot more flexibility since the DEX file can be disassembled into smali file and then reassembled into a DEX file. Thus there are no restrictions on the code injected or tampered with.
 • When patching a native library, we directly patch the binary code. For this reason, we need to ensure we are not corrupting other parts of the code while modifying a function. In practice, this means that when patching native libraries, most of the time, only a limited number of assembly instructions are changed. [](https://read.readwise.io/read/01jqvbbcf35vtgehejgnksqhmj)



But, even only patching a couple of assembly instructions, can have a huge impact on the application. [](https://read.readwise.io/read/01jqvbbn2at0qgb5e1h93s1gdx)



We can replace a function call with a very small stub by replacing the two first instructions of a function [](https://read.readwise.io/read/01jqvbd6mam754v7wemz87n81t)



As an example, if the application has a function`was_hit`, which performs various checks and returns 1 if you were hit and 0 otherwise, an attacker can patch the prolog of this function with these two assembly instructions so all checks are bypassed and the function always returns 0. [](https://read.readwise.io/read/01jqvbdj3sgm7mn3796s1jy3yf)



We can also force a branch to be always (or never) taken [](https://read.readwise.io/read/01jqvbdqnra1da3bfh7cgns2e6)



Patching an`if`statement has many uses because it allows bypassing any application logic checks, such as checking if the user has won or if the user has paid for a feature. [](https://read.readwise.io/read/01jqvbe3fq5t2e02v0n0zt09f0)



[shell-storm](http://shell-storm.org/online/Online-Assembler-and-Disassembler/?inst=MOV+X0%2C+1%0D%0ARET&arch=arm64&as_format=inline#assembly) allows you to assemble code directly from your browser, and there is a really good [IDA Pro plugin](https://github.com/gaasedelen/patching) to do it very easily inside IDA. [](https://read.readwise.io/read/01jqvbfn994qa6pgjvea3ydjs1)



Code Tampering: The Flutter case [](https://read.readwise.io/read/01jqvbgfnr7ffrjytbxtppr72f)



Since Dart code is compiled into the`libapp.so`native library, patching it is exactly the same as patching any native part of a mobile application. As a result, the main challenge is the reverse engineering part. [](https://read.readwise.io/read/01jqvbh56k5jwhh25tpbsw11m4)



The first step is finding the relevant functions. [](https://read.readwise.io/read/01jqvbk9839qdb501f9308adm6)



two main possibilities:
 1. The Flutter application is not built using the`obfuscate`built-in option. In this case, as explained in [our first blog post](https://www.guardsquare.com/blog/current-state-and-future-of-reversing-flutter-apps), it is easy to recover all function names and addresses. [](https://read.readwise.io/read/01jqvbkm66cs8fhd3bc0bhncdz)



Application-specific function names won’t be accessible if the application is built using the`obfuscate`built-in option. Thus it will require other standard reverse engineering techniques to find them. [](https://read.readwise.io/read/01jqvbm76yf643wqsz1ffj4tyq)



it could be done using dynamic analysis to trace all functions called [](https://read.readwise.io/read/01jqvbmkgbwyfmbtnfdjqczyyp)



The second step is where the real reverse engineering takes place. [](https://read.readwise.io/read/01jqvbnrcc0dqa16w5n1m0114s)



Since Flutter apps are compiled to native code, we decided to try code tampering as we would do on the native components of a mobile application.
 We demonstrated that it works in exactly the same way as this type of attack is usually done:
 • First the attacker needs to understand the code logic to find out which instructions need to be patched. (We explained in a [previous post](https://www.guardsquare.com/blog/obstacles-in-dart-decompilation-and-the-impact-on-flutter-app-security) how efficient reverse engineering can be done on Flutter applications.)
 • Then, the attacker needs to replace the instruction and repackage the modified application. [](https://read.readwise.io/read/01jqvbt2pkmagzs7g1p4ep9r9m)



Hooking a Flutter application [](https://read.readwise.io/read/01jqvbtbqgt4hc7pxvvv8f14k5)



Patching code is powerful but it has some limitations:
 • Since patching is done at the assembly level, it means that patches themselves must be written in assembly.
 • Patching one instruction is easy, but if the patch logic requires more assembly instructions, there is a risk that it will overwrite other instructions/functions that will change the application behavior unexpectedly [](https://read.readwise.io/read/01jqvbv0efq09jx4eynmy851fd)



When prototyping a patch, it has a lot of overhead. This means you’ll need to repackage and re-install the app each time you want to try something new. [](https://read.readwise.io/read/01jqvbv8807d8sny3wxejanj1v)



To overcome these limitations, attackers can use another type of attack to implement more complex patches: hooking. [](https://read.readwise.io/read/01jqvbvhd2hpkdcq5rc2r9490r)



Hooking has virtually unlimited possibilities. [](https://read.readwise.io/read/01jqvbx252tde6vxpyqetjvmeh)



For changing application behavior [](https://read.readwise.io/read/01jqvbx9gr85ybjqh81ypz53b2)



To ease the reverse engineering process [](https://read.readwise.io/read/01jqvbxcprjqpf0am8fsv693rd)



Hooking can be performed at various levels, for instance by redirecting calls to imported functions to attacker code, or by directly patching binary code with a trampoline which will jump to attacker code. [](https://read.readwise.io/read/01jqvbxtq3br7p4fzvh7165gvz)



Let’s focus on inline hooking, which relies on patching code after it has been loaded into memory. The core idea of inline hooking is to replace the first assembly instructions of a function with a trampoline [](https://read.readwise.io/read/01jqvbydkbcwvzrn195d4qpfb8)



The attacker code will:
 • Ensure that the original instructions patched by the trampoline instructions are executed to ensure the application code will work as expected.
 • Do whatever the attacker wants, like logging function parameters, changing function parameters or replacing the function call. [](https://read.readwise.io/read/01jqvbyv59ev518cn2h3v1aeyp)



Hooking: The Flutter case [](https://read.readwise.io/read/01jqvbz09kp0nwat19mxr5azfc)



Hooking Dart code works similarly to classical native code hooking. Well known hooking frameworks, such as Frida, can put their trampoline code and they can inject attacker code when the function is called or when it returns. [](https://read.readwise.io/read/01jqvbze7zs6rpwhaybyxsh1tr)



Dart code has two specificities that we discussed in [our previous blog post](https://www.guardsquare.com/blog/obstacles-in-dart-decompilation-and-the-impact-on-flutter-app-security) that we must take into account:
 • It uses a custom stack and a custom calling convention, which means it is not possible to access the function parameters directly from within a hook
 • The function parameters are Dart objects, thus they must be parsed to get their actual value [](https://read.readwise.io/read/01jqvc05wpd4nbjqngefswzjqq)



Because of this, Frida is not able to (out of the box) retrieve the function parameters. [](https://read.readwise.io/read/01jqvc0j65wpagfc0pamf6s99k)



Let’s first see what the Frida script looks like:
 let OFFSET_APPLY_TILE_EFFECT = 0x458d10
 let APPLY_TILE_EFFECT_ENTITY_PARAMETER_INDEX = 2;
 let ENTITY_TYPE_OFFSET = 1;
 let ENTITY_TYPE_CAT_VALUE = 1166;
 let ENTITY_POSITION_OFFSET = 7;
 let BOARD_POSITION_X_OFFSET = 7;
 let BOARD_POSITION_Y_OFFSET = 0xf;
 function reset_cat_position(){
 var base_address = Module.findBaseAddress("libapp.so");
 Interceptor.attach(base_address.add(OFFSET_APPLY_TILE_EFFECT), {
 onEnter: function () {
 let entity = dart_get_arg(this.context, APPLY_TILE_EFFECT_ENTITY_PARAMETER_INDEX);
 let entity_type = entity.add(ENTITY_TYPE_OFFSET).readInt() * 2
 if (entity_type == ENTITY_TYPE_CAT_VALUE){
 let entity_position = 
 get_pointer_with_heap_bit(entity, ENTITY_POSITION_OFFSET, this.context);
 let entity_position_x = 
 entity_position.add(BOARD_POSITION_X_OFFSET).readInt();
 let entity_position_y = 
 entity_position.add(BOARD_POSITION_Y_OFFSET).readInt();
 if ((entity_position_x > 1) || (entity_position_y > 1)){
 console.log(
 `Resetting position of cat (${entity}): 
 (${entity_position_x}, ${entity_position_y})`
 );
 entity_position.add(BOARD_POSITION_X_OFFSET).writeInt(0);
 entity_position.add(BOARD_POSITION_Y_OFFSET).writeInt(0);
 }
 }
 }
 });
 } [](https://read.readwise.io/read/01jqvc6wxc2trcnfc6m4wfhk1a)



Understanding cheat creation [](https://read.readwise.io/read/01jqvc71q3w4y9p7zcm6wy8pds)



The first step is to recover the address of the targeted function (`OFFSET_APPLY_TILE_EFFECT`), which is trivial if the build is not obfuscated. Otherwise, the attacker could perform dynamic analysis to trace all function calls and have a short list of functions that are called at each game update. [](https://read.readwise.io/read/01jqvc88xpzp4pxpgbj5b7h8e5)



Once the function is found, it is possible to start reverse engineering it. [](https://read.readwise.io/read/01jqvc8hkyy7z1b221yaz2krs4)



here is the beginning of the function:
 **![14-Classical attacks on Flutter apps](https://www.guardsquare.com/hs-fs/hubfs/Blog%20assets/Blog%20internal%20images/14-Classical%20attacks%20on%20Flutter%20apps.png?width=700&name=14-Classical%20attacks%20on%20Flutter%20apps.png)**
 Since all these checks seem related to the position of the entity, it leaks some information on the`Entity`structure:
 • The position object of the entity seems to be located at offset 7 of the entity (which gives us`ENTITY_POSITION_OFFSET`)
 • Inside the position object, the vertical position seems to be located at offset`0xf`(which gives us`BOARD_POSITION_Y_OFFSET`)
 • Inside the position object, the horizontal position seems to be located at offset`0x7`(which gives us`BOARD_POSITION_X_OFFSET`) [](https://read.readwise.io/read/01jqvc9brra0x7nkvx44184z4a)



All these guesses can be verified by hooking the`applyTileEffect`function and logging the value of the data stored at these offsets [](https://read.readwise.io/read/01jqvc9rmheerdpsx1p0ae3jhj)

