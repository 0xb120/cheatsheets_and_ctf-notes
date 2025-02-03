---
author: "clearbluejar"
aliases: "Decompilation Debugging"
tags: RW_inbox, readwise/articles
url: https://clearbluejar.github.io/posts/decompilation-debugging-pretending-all-binaries-come-with-source-code/
date: 2025-01-08
---
# Decompilation Debugging

![rw-book-cover](https://clearbluejar.github.io/assets/img/2023-11-08-decompilation-debugging-pretending-all-binaries-come-with-source-code/timothy-dykes-LhqLdDPcSV8-unsplash.jpg)

## Highlights


Normally, when debugging, you have source code and data type information (aka symbols) to help navigate your application. In the world of Reverse Engineering closed source applications, you won’t have the needed information to debug your application. Don’t worry, Ghidra has you. **Decompilation debugging lets you pretend like every program comes with source.** In this post, we will see how the Ghidra Debugger integrates with the Ghidra Code Browser, allowing you to step through the pseudo-code of the program you are debugging.
> [View Highlight](https://read.readwise.io/read/01jh3d4pt4mxt7r20hxrxnt8ye)



Debugging with WinDbg[](https://clearbluejar.github.io/posts/decompilation-debugging-pretending-all-binaries-come-with-source-code/#debugging-with-windbg)
 WinDbg is a powerful tool to allow dynamic analysis of Windows applications, system Dlls, and services. There are [a](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/getting-started-with-windows-debugging) [hundred](https://codemachine.com/articles/windbg_quickstart.html) [articles](https://www.codeproject.com/Articles/6084/Windows-Debuggers-Part-1-A-WinDbg-Tutorial) to [teach](https://securityoversimplicity.wordpress.com/2019/11/03/debugging-service-using-windbg/) you [how](https://dev.to/gabbersepp/how-to-debug-an-unmanaged-application-with-windbg-2j23) to use WinDbg for debugging modern windows applications, so I won’t give you too many tips here. The most important tip I can give is to always ensure you configure and load symbols.
> [View Highlight](https://read.readwise.io/read/01jh3ddkj420930qb7txp9zzd4)



When you hit code for which you don’t have the source, most debuggers will offer you the disassembly (machine code to assembly).
> [View Highlight](https://read.readwise.io/read/01jh3djbtccycq2k74728ccj4b)



Well, nothing stops you from debugging any application with disassembly.
> [View Highlight](https://read.readwise.io/read/01jh3djhyrxyq1yea8vxjhzcpp)



Even so, it is much easier to follow program flow and understand a program with the high level representation. So, if we can’t have source for these proprietary libraries, what if we could pretend to have it?
> [View Highlight](https://read.readwise.io/read/01jh3dkd1khb4nt1sma3gfemjj)



Decompilation Debugging[](https://clearbluejar.github.io/posts/decompilation-debugging-pretending-all-binaries-come-with-source-code/#decompilation-debugging)
 Modern [SRE](https://en.wikipedia.org/wiki/Reverse_engineering#software:~:text=decades.%5B9%5D-,Software%20reverse%20engineering,-can%20help%20to) tooling provides the ability to **decompile native code into pseudo-code**.
> [View Highlight](https://read.readwise.io/read/01jh3dkte23zcp45kc7wrtdwy4)



It’s not perfect, but much easier to understand than assembly. What if you wanted to add this decompilation ability to a debug session, to more easily understand your closed source application? Well, as you probably have guessed from the title of this post, that is exactly what we are going to do.
> [View Highlight](https://read.readwise.io/read/01jh3dnw6d718174cpj6h1zhxk)



Ghidra Debugger[](https://clearbluejar.github.io/posts/decompilation-debugging-pretending-all-binaries-come-with-source-code/#ghidra-debugger)
 Let’s look at using Ghidra’s new(ish) feature, its Debugger tool.
 [![](https://clearbluejar.github.io/assets/img/2023-11-08-decompilation-debugging-pretending-all-binaries-come-with-source-code/debugger-icon.png)](https://clearbluejar.github.io/assets/img/2023-11-08-decompilation-debugging-pretending-all-binaries-come-with-source-code/debugger-icon.png)
> [View Highlight](https://read.readwise.io/read/01jh3dp7vwqxbwah2tz2qrgv40)



The Ghidra debugger is not a debugger, rather, it relies on external debugggers to control debugging targets. Ghidra’s Debugger communicates to these external debuggers via one of its “connectors”.
 [![](https://clearbluejar.github.io/assets/img/2023-11-08-decompilation-debugging-pretending-all-binaries-come-with-source-code/ghidra-connectors-debugger-code.png)](https://clearbluejar.github.io/assets/img/2023-11-08-decompilation-debugging-pretending-all-binaries-come-with-source-code/ghidra-connectors-debugger-code.png)*List of [available connectors](https://github.com/NationalSecurityAgency/ghidra/tree/master/Ghidra/Debug) for Ghidra’s Debugger*
> [View Highlight](https://read.readwise.io/read/01jh3dqm6ky3xejfkj5cdkr23n)



[![](https://clearbluejar.github.io/assets/img/2023-11-08-decompilation-debugging-pretending-all-binaries-come-with-source-code/connector-options1.png)](https://clearbluejar.github.io/assets/img/2023-11-08-decompilation-debugging-pretending-all-binaries-come-with-source-code/connector-options1.png)*Selecting one of the debugger “connectors”*
> [View Highlight](https://read.readwise.io/read/01jh3drstr0m0kga430zjdfaeq)



Ghidra can call run “IN-VM” which will start the debugger within its own process, or it has the ability to launch an agent that will run the debugger in a [separate process](https://github.com/NationalSecurityAgency/ghidra/blob/master/Ghidra/Debug/Debugger-agent-dbgeng/src/main/java/agent/dbgeng/gadp/DbgEngGadpServer.java#L89).
> [View Highlight](https://read.readwise.io/read/01jh3dv823zk3xxbd4nb411rj3)



The Puppeteer[](https://clearbluejar.github.io/posts/decompilation-debugging-pretending-all-binaries-come-with-source-code/#the-puppeteer)
 Remember, Ghidra’s Debugger is not a debugger. It allows you to perform debugging via proxy. You can direct the external debugger to start or stop the debugging target; read or modify the current program state; break on specific functions. **Ghidra’s Debugger is a puppeteer, treating externals debuggers like marionettes by directing them to start, stop, and manipulate a target application.**
 What is the point then? Why not just use the original debugger and skip the overhead? Well, Ghidra’s Debugger **combines static analysis and dynamic analysis in one view**.
> [View Highlight](https://read.readwise.io/read/01jh3dxs52aefnwmc9565v82y5)



Decompilation Debugging With Ghidra[](https://clearbluejar.github.io/posts/decompilation-debugging-pretending-all-binaries-come-with-source-code/#decompilation-debugging-with-ghidra)
 Let’s learn how to launch and debug an application on Windows. Let’s again try out the rpc-svc `server.exe` application from above. We will start with a new project, import and launch our binary, set some break points, and see how we can step through the application both in assembly (listing view) and in pseudo-code (decompiler view).
> [View Highlight](https://read.readwise.io/read/01jh3e2tjjwn9c5d5n8tsm7yxb)



Open in Debugger[](https://clearbluejar.github.io/posts/decompilation-debugging-pretending-all-binaries-come-with-source-code/#open-in-debugger)
 [![](https://clearbluejar.github.io/assets/img/2023-11-08-decompilation-debugging-pretending-all-binaries-come-with-source-code/launch-debug-1.gif)](https://clearbluejar.github.io/assets/img/2023-11-08-decompilation-debugging-pretending-all-binaries-come-with-source-code/launch-debug-1.gif)*Launch `server.exe` in Debugger Tool*
> [View Highlight](https://read.readwise.io/read/01jh3e55qr9z0evmfsmmda28ef)



The GUI appears with what seems like [1000 windows](https://github.com/NationalSecurityAgency/ghidra/blob/master/GhidraDocs/GhidraClass/Debugger/A2-UITour.md#windows). I won’t explain them all, because even after studying the tool I don’t have a full grasp on every feature, but will touch on several of the Windows necessary to understand setting a breakpoint and stepping through code.
 [![](https://clearbluejar.github.io/assets/img/2023-11-08-decompilation-debugging-pretending-all-binaries-come-with-source-code/ghidra-debugger-windows.png)](https://clearbluejar.github.io/assets/img/2023-11-08-decompilation-debugging-pretending-all-binaries-come-with-source-code/ghidra-debugger-windows.png)*Ghidra Debugger with 1000 Windows*
> [View Highlight](https://read.readwise.io/read/01jh3e5akzpxfxj1skkpdz8194)



Pretend Like Your Binary Comes with Source[](https://clearbluejar.github.io/posts/decompilation-debugging-pretending-all-binaries-come-with-source-code/#pretend-like-your-binary-comes-with-source)
 All in all, the Ghidra Debugger is pretty amazing. The Ghidra Debugger is yet another powerful tool for reverse engineering. It allows you to debug native code as if you have the source code, even for external libraries that you don’t have access to.
> [View Highlight](https://read.readwise.io/read/01jh3e9112cv79k8dftxek3bnm)



There are many more features in the Ghidra Debugger that I didn’t cover in this post, such as tracing (akin to [Time Travel Debugging](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/time-travel-debugging-overview) ) and [emulation](https://github.com/NationalSecurityAgency/ghidra/blob/master/GhidraDocs/GhidraClass/Debugger/B2-Emulation.md#emulation).
> [View Highlight](https://read.readwise.io/read/01jh3e9cn44ngpcqs00hvsg6gy)



