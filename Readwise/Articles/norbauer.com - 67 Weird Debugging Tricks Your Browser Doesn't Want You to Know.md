---
author: norbauer.com
aliases:
  - 67 Weird Debugging Tricks Your Browser Doesn't Want You to Know
tags:
  - readwise/articles
url: https://alan.norbauer.com/articles/browser-debugging-tricks
created: 2024-08-20
---
# 67 Weird Debugging Tricks Your Browser Doesn't Want You to Know

![rw-book-cover](https://alan.norbauer.com/articles/browser-debugging-tricks/twitter-card.jpg)

## Highlights


> Advanced Conditional Breakpoints
>  By using expressions that have side effects in places you wouldn’t expect, we can squeeze more functionality out of basic features like conditional breakpoints.
> [View Highlight](https://read.readwise.io/read/01hfbw3cycsg65xgv46t4rjs5d)



> Logpoints / Tracepoints
>  For example, we can `console.log` in breakpoints. Logpoints are breakpoints that log to the console without pausing execution.
> [View Highlight](https://read.readwise.io/read/01hfbw3kkh76eqre8kn384aeta)



> Watch Pane
>  You can also use `console.log` in the watch pane.
> [View Highlight](https://read.readwise.io/read/01hfbw5569gmrv50gs0zpd39c2)



> to execute an expression after DOM mutation, set a DOM mutation breakpoint
> [View Highlight](https://read.readwise.io/read/01hfbw6hbae06m2rb58gacqvgy)



> And then add your watch expression, e.g. to record a snapshot of the DOM: `(window.doms = window.doms || []).push(document.documentElement.outerHTML)`. Now, after any DOM subtree modification, the debugger will pause execution and the new DOM snapshot will be at the end of the `window.doms` array.
> [View Highlight](https://read.readwise.io/read/01hfbw6qcp8q6tb2neqdbyxtv6)



> How can you find the source of the unpaired show call? Use `console.trace` in a conditional breakpoint in the show method, run your code, find the last stack trace for the show method and click the caller to go to the code
> [View Highlight](https://read.readwise.io/read/01hfbw8c0myfyar3t5gw82sjpr)



> Break on Number of Arguments
>  Only pause when the current function is called with 3 arguments: `arguments.callee.length === 3`
>  Useful when you have an overloaded function that has optional parameters.
> [View Highlight](https://read.readwise.io/read/01hfbwb00cp0xasbhw1m4jfr4f)



> [Break on Function Arity Mismatch](https://alan.norbauer.com/articles/browser-debugging-tricks#break-on-function-arity-mismatch)
>  Only pause when the current function is called with the wrong number of arguments: `(arguments.callee.length) != arguments.length`
>  ![Conditional Breakpoint - arity check](https://alan.norbauer.com/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fconditional-breakpoint-arity-check.70c0a60c.gif&w=1920&q=75)
>  Useful when finding bugs in function call sites.
> [View Highlight](https://read.readwise.io/read/01hfbwebaqtrfbjrzpb99h5n6j)



> [Break on Sample](https://alan.norbauer.com/articles/browser-debugging-tricks#break-on-sample)
>  Only break on a random sample of executions of the line, e.g. only break 1 out of every 10 times the line is executed: `Math.random() < 0.1`
> [View Highlight](https://read.readwise.io/read/01hfbwfvnecb24a8sk65ggss5v)



> [Automatic Instance IDs](https://alan.norbauer.com/articles/browser-debugging-tricks#automatic-instance-ids)
>  Automatically assign a unique ID to every instance of a class by setting this conditional breakpoint in the constructor: `(window.instances = window.instances || []).push(this)`
>  Then to retrieve the unique ID: `window.instances.indexOf(instance)` (e.g. `window.instances.indexOf(this)` when in a class method)
> [View Highlight](https://read.readwise.io/read/01hfbwj5ggq4tjtrg4zjbsc9tv)



> [monitor() class Calls](https://alan.norbauer.com/articles/browser-debugging-tricks#monitor-class-calls)![Chrome](https://alan.norbauer.com/_next/static/media/chrome.2d2a19fd.svg)
>  You can use Chrome’s `monitor` command line method to easily trace all calls to class methods. E.g. given a class `Dog`
>  1class Dog {2 bark(count) {3 /* ... */4 }5}
>  1class Dog {2 bark(count) {3 /* ... */4 }5}
>  If we want to know all calls made to all instances of `Dog`, paste this into the command line:
>  1var p = Dog.prototype;2Object.getOwnPropertyNames(p).forEach((k) => monitor(p[k]));
>  1var p = Dog.prototype;2Object.getOwnPropertyNames(p).forEach((k) => monitor(p[k]));
>  and you’ll get output in the console:
>  > function bark called with arguments: 2
>  > function bark called with arguments: 2
>  You can use `debug` instead of `monitor` if you want to pause execution on any method calls (instead of just logging to the console).
> [View Highlight](https://read.readwise.io/read/01hfbwkz7bj55mkfe768cdw9jy)



> [Call and Debug a Function](https://alan.norbauer.com/articles/browser-debugging-tricks#call-and-debug-a-function)
>  Before calling the function you want to debug in the console, call `debugger`. E.g. given:
>  1function fn() {2 /* ... */3}
>  1function fn() {2 /* ... */3}
>  From your console:
>  > debugger; fn(1);
>  > debugger; fn(1);
>  And then “Step into next function call” to debug the implementation of `fn`.
> [View Highlight](https://read.readwise.io/read/01hfbwmwva0w0d9wky7feng6m3)



> [Debugging Property Reads](https://alan.norbauer.com/articles/browser-debugging-tricks#debugging-property-reads)
>  If you have an object and want to know whenever a property is read on it, use an object getter with a `debugger` call. For example, convert `{configOption: true}` to `{get configOption() { debugger; return true; }}` (either in the original source code or using a conditional breakpoint)
> [View Highlight](https://read.readwise.io/read/01hfbwphg1m3jg9rfmrt3ppvsz)



> [Use copy()](https://alan.norbauer.com/articles/browser-debugging-tricks#use-copy)![Chrome](https://alan.norbauer.com/_next/static/media/chrome.2d2a19fd.svg)![Firefox](https://alan.norbauer.com/_next/static/media/firefox.583d9a58.svg)
>  You can copy interesting information out of the browser directly to your clipboard without any string truncation using the `copy()` console API. Some interesting things you might want to copy:
>  • Snapshot of the current DOM: `copy(document.documentElement.outerHTML)`
>  • Metadata about resources (e.g. images): `copy(performance.getEntriesByType("resource"))`
>  • A large JSON blob, formatted: `copy(JSON.parse(blob))`
>  • A dump of your localStorage: `copy(localStorage)`
> [View Highlight](https://read.readwise.io/read/01hfbwq5gaq13xgxj3t3xzkvnr)



> [Record Snapshots of the DOM](https://alan.norbauer.com/articles/browser-debugging-tricks#record-snapshots-of-the-dom)
>  To grab a copy of the DOM in its current state:
>  1copy(document.documentElement.outerHTML);
>  1copy(document.documentElement.outerHTML);
>  To record a snapshot of the DOM every second:
>  1doms = [];2setInterval(() => {3 const domStr = document.documentElement.outerHTML;4 doms.push(domStr);5}, 1000);
>  1doms = [];2setInterval(() => {3 const domStr = document.documentElement.outerHTML;4 doms.push(domStr);5}, 1000);
>  Or just dump it to the console:
>  1setInterval(() => {2 const domStr = document.documentElement.outerHTML;3 console.log("snapshotting DOM: ", domStr);4}, 1000);
> [View Highlight](https://read.readwise.io/read/01hfbwrqwvxjw4vg3ecdhg5q05)



> [Get Event Listeners](https://alan.norbauer.com/articles/browser-debugging-tricks#get-event-listeners)![Chrome](https://alan.norbauer.com/_next/static/media/chrome.2d2a19fd.svg)
>  In Chrome you can inspect the event listeners of the currently selected element: `getEventListeners($0)`
> [View Highlight](https://read.readwise.io/read/01hfbwsqbjqwz9m1x1428frxgr)



> [Monitor Events for Element](https://alan.norbauer.com/articles/browser-debugging-tricks#monitor-events-for-element)![Chrome](https://alan.norbauer.com/_next/static/media/chrome.2d2a19fd.svg)
>  Debug all events for selected element: `monitorEvents($0)`
>  Debug specific events for selected element: `monitorEvents($0, ["control", "key"])`
> [View Highlight](https://read.readwise.io/read/01hfbwt10kr1yxaf4tq3y2400z)

