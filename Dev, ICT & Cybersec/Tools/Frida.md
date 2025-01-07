---
Description: Dynamic instrumentation toolkit for developers, reverse-engineers, and security researchers.
URL: https://frida.re/
---

>[!summary] 
>It’s [Greasemonkey](https://addons.mozilla.org/en-US/firefox/addon/greasemonkey/) for native apps, or, put in more technical terms, it’s a dynamic code instrumentation toolkit. It lets you inject snippets of JavaScript or your own library into native apps on Windows, macOS, GNU/Linux, iOS, Android, and QNX. Frida also provides you with some simple tools built on top of the Frida API. These can be used as-is, tweaked to your needs, or serve as examples of how to use the API.

# Documentation and CodeShare

- https://learnfrida.info/
- https://frida.re/docs/home/
- https://codeshare.frida.re/

# Frida server

Frida server can be used only if you have root privileges! [Patch APKs with frida using objection](objection.md#Patch%20APKs%20with%20frida%20using%20objection) if you do not have them.

>[!info] Download
>https://github.com/frida/frida/releases

```bash
adb push ..\..\frida\frida-server-16.0.11-android-x86 /data/local/tmp
adb shell
su
chmod +x /data/local/tmp/frida-server-16.0.11-android-x86
/data/local/tmp/frida-server-16.0.11-android-x86

netstat -tulnp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program Name
tcp        0      0 127.0.0.1:27042         0.0.0.0:*               LISTEN      7846/frida-server-16.0.11-android-x86
```

Attach to the frida server using
```sh
$ frida -U -f com.example.package -l script.js
```
### Expose the frida server on a public port

```bash
/data/local/tmp/frida-server-16.0.11-android-x86 -l 0.0.0.0:27042
 # netstat -tulnp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program Name
tcp        0      0 0.0.0.0:27042           0.0.0.0:*               LISTEN      4990/frida-server-16.0.11-android-x86
```

# Frida REPL

The Frida REPL (Read-Eval-Print-Loop) is a JavaScript interpreter, and so we can directly run JavaScript statements. You can also specify built-ins non-JavaScript commands specifying them with `%` (eg. `%help`).

# Script examples

## Instantiating classes and objects

```js
Java.perform(() => {
    console.log("Started frida script!")
    let FlagClass = Java.use("io.hextree.fridatarget.FlagClass")
    console.log(FlagClass.flagFromStaticMethod())
    
    let FlagInstance = FlagClass.$new()
    console.log(FlagInstance.flagFromInstanceMethod())

    console.log(FlagInstance.flagIfYouCallMeWithSesame("sesame"))
})
```

## Trace active activities

```js
Java.perform(() => {
    let ActivityClass = Java.use("android.app.Activity");
    ActivityClass.onResume.implementation = function() {
        console.log("Activity resumed:", this.getClass().getName());
        // Call original onResume method
        this.onResume();
    }
})
```
## Hooking a custom function

Original source: https://github.com/kush412/CTF_WriteUps/blob/master/UMDCTF2023/UMDCTF2023.md
```js
setTimeout(function () { // avoid java.lang.ClassNotFoundException

    Java.performNow(function () {
	    // YOUR CODE HERE
        let MainActivity = Java.use("com.example.whosthatpokemon.MainActivity"); // Select the desired activity
        MainActivity['getGuessString'].implementation = function () { // Redefine the "getGuessSting" method
            console.log("Hooked getGuessString");
            let result = this['getGuessString']();
            console.log("Result is: "+result);
            return result;
        }
    });
}, 1000);

> frida -U -f com.example.whosthatpokemon -l .\test\foo.js
> Spawned `com.example.whosthatpokemon`. Resuming main thread!
[Android Emulator 5554::com.example.whosthatpokemon ]-> 
Hooked getGuessString
Result is: asd
```

## Hooking an internal function (areEqual)

```js
setTimeout(function () {
  // avoid java.lang.ClassNotFoundException

  Java.performNow(function () {
    //YOUR CODE HERE
    let Intrinsics = Java.use("kotlin.jvm.internal.Intrinsics");
    Intrinsics["areEqual"].overload('java.lang.Object', 'java.lang.Object').implementation = function (first, second) {
        console.log(`Intrinsics.areEqual is called: first=${first}, second=${second}`);
        let result = this["areEqual"](first, second);
        console.log(`Intrinsics.areEqual result=${result}`);
        return true;
    };
  });
}, 1000);

> frida -U -f com.example.whosthatpokemon -l .\test\foo2.js
> Spawned `com.example.whosthatpokemon`. Resuming main thread!
[Android Emulator 5554::com.example.whosthatpokemon ]-> Intrinsics.areEqual is called: first=asd, second=Terrapulseonic
Intrinsics.areEqual result=false
```

## Android SSL Pinning Bypass scripts

- https://codeshare.frida.re/@akabe1/frida-multiple-unpinning/
- https://codeshare.frida.re/@pcipolloni/universal-android-ssl-pinning-bypass-with-frida/

# frida-trace

## trace function calls

To make it work, we need to **instrument each function** we want to trace.

We can use the syntax `-j classname!methodname`, using wildcards, to specify everything we want to include, and `-J classname!methodname`, suing wildcards, to specify anything we want to exclude:

```bash
frida-trace -U -j 'io.hextree.Class!button_click_handler' FridaTarget
frida-trace -U -j 'io.hextree.*!*' FridaTarget
frida-trace -U -j 'io.hextree.*!*password*' FridaTarget # all functions that include the word "password"
frida-trace -U -j 'io.hextree.*!*' -J '*AnnoyingClass!*' FridaTarget 
```

## trace native libraries

```bash
frida-trace -U -I 'libhextree.so' 'io.hextree.*!*' FridaTarget
```

# External resources and writeup
- [Frida on Java applications and applets in 2024](https://security.humanativaspa.it/frida-on-java-applets-in-2024/), humanativaspa.it