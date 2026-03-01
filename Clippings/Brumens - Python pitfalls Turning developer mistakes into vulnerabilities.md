---
title: "Python pitfalls: Turning developer mistakes into vulnerabilities"
source: https://www.yeswehack.com/learn-bug-bounty/python-pitfalls-turning-developer-mistakes
author:
  - Brumens
published: 2026-02-27
created: 2026-03-01
description: Abusing unexpected Python behaviours to achieve path traversal and even RCE via path manipulation issues in os.path.join, URL handling quirks in urljoin, class pollution problems in Python object handling, and more besides.
tags:
  - clippings/articles
---
# Python pitfalls: Turning developer mistakes into vulnerabilities

![](https://cdn.sanity.io/images/d51e1jt0/production/3536f4c82f8765dcb58990f777e410caaff4074e-1200x628.jpg?rect=2,0,1196,628&w=1200&h=630&fit=crop)

> [!summary]+
> > This article by Alex Brumen explores common Python pitfalls that turn developer mistakes into vulnerabilities, targeting pentesters, security researchers, and Bug Bounty hunters.
> It details six main pitfalls:
> - `os.path.join` and `pathlib.joinpath`: These functions can be exploited for path traversal due to their behavior of truncating earlier path segments when an absolute path is provided as user input. This can lead to arbitrary file reads/writes and RCE.
> - `pickle.loads` and `PyYAML's load`: Both functions, when used with untrusted input and without safe loaders, are vulnerable to insecure deserialization (CWE-502), allowing arbitrary code execution (RCE).
> - `Python3 class pollution`: Arises from insecure object merging, leading to unintended attribute/variable overwrites and potential code injection (CWE-94).
> - `urllib.parse.urljoin`: This function discards the entire base URL if an absolute URL is provided as input, making it vulnerable to SSRF, path traversal, and open redirects.
> For each pitfall, the article provides example code, visual workflows, real-world CVEs, and links to practical labs on the YesWeHack Dojo platform.
> The key mitigation strategy emphasized is to thoroughly understand function documentation and implement appropriate sanitization for user-controlled input.

## Goal of this research

The research objective was to identify [Python](../Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/Python.md) functions that are frequently found in production applications, before crafting payloads that weaponize the aforementioned ‘Python pitfalls’.

## Black-box testing for Python bugs

Most real-world engagements involve attacking applications in a black-box environment. That’s why it’s essential to develop a strategy for fingerprinting the Python pitfalls we’re about to explore. Before doing so, you need to understand the core concepts of what to observe and extract from an application’s responses.

- Time delays
- Content
- Errors
- Reflected values
- Process result
- Verbose messages
- Syntax

## Commonly exploitable Python functions

During the research, I discovered six different Python pitfalls. Five originate from built-in libraries, and one comes from a widely used external library.

- [`os.path.join`](https://docs.python.org/3/library/os.path.html#os.path.join) (built-in)
- [`pathlib.joinpath`](https://docs.python.org/3/library/pathlib.html#pathlib.PurePath.joinpath) `(built-in)
- [`pickle.loads`](https://docs.python.org/3/library/pickle.html#pickle.loads) (built-in)
- [`urllib.parse.urljoin`](https://docs.python.org/3/library/urllib.parse.html#urllib.parse.urljoin) (built-in)
- [`PyYAML's load function`](https://pyyaml.org/wiki/PyYAMLDocumentation) (external library)
- [`Python3 class pollution`](https://blog.abdulrah33m.com/prototype-pollution-in-python/) (built-in)

## os.path.join & path traversal

Python’s [`os.path.join`](https://docs.python.org/3/library/os.path.html#os.path.join) function joins one or more path segments and returns a single, concatenated path.

`os.path.join` does not prevent path traversal or enforce path containment, and will happily discard earlier path segments if an absolute path appears later in the input.

The following code snippet demonstrates this pitfall:

```python
import os

payload = "/etc/passwd"

file = os.path.join("/user/uploads/", payload)

with open(file, "r") as f:

   print(f.read()) # print the “/etc/passwd” file content
```

Example:
- [OpenSource](../CTFs/OpenSource.md)
- [Analysis of CVE-2023-37474 in CopyParty](../../Readwise/Articles/0xdf%20-%20Analysis%20of%20CVE-2023-37474%20in%20CopyParty.md)
## pathlib.joinpath & path traversal

The [`pathlib.joinpath`](https://docs.python.org/3/library/pathlib.html#pathlib.PurePath.joinpath) method is very similar to `os.path.join`. Calling it is equivalent to appending each provided path segment in sequence.

Just like `os.path.join`, you might expect `joinpath` to sanitise input or enforce safe normalisation. But that is not the case. If any segment is an absolute path, it discards the earlier parts and continues from the absolute path. It also accepts dot-dot-slash sequences, which means user-controlled input can still lead to path traversal if you rely on it for safety.

```python
from pathlib import Path

payload = "/etc/passwd"

file = Path("/var/www/html").joinpath("files", payload)

with open(file, "r") as f:

   print(f.read()) # print the “/etc/passwd” file content
```

## pickle.loads & RCE

Python’s pickle module is well known for being dangerous when used with user-controlled input, especially the function [`pickle.loads`](https://docs.python.org/3/library/pickle.html#pickle.loads). While this behaviour can be considered a pitfall, using this function with untrusted input introduces a direct vulnerability known as [deserialization of untrusted data (CWE-502)](https://cwe.mitre.org/data/definitions/502.html), which can result in arbitrary code execution and full RCE.

Examples:
- [Pickle](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Insecure%20Deserialization%20&%20Object%20Injection.md#Pickle)
## PyYAML & insecure deserialisation

Moving beyond built-in modules, when developers handle YAML-based data they most often turn to [`PyYAML`](https://pypi.org/project/PyYAML/). This external library makes it easy to serialise and deserialise YAML data.

However, there is an important catch in the documentation. When deserialising YAML data, developers often rely on [`PyYAML`’s load function](https://pyyaml.org/wiki/PyYAMLDocumentation). If a safe loader is not explicitly specified, the application may become vulnerable to insecure deserialisation (CWE-502), in a manner similar to `pickle.loads`.

Example code:

```python
import yaml

user_data = "!!python/object/apply:print ['pwned']"

result = yaml.load(user_data, Loader=yaml.Loader)
```

In the code above, the user `input !!python/object/apply:print ['pwned']` instructs `PyYAML`’s loader (`yaml.Loader`) to invoke a Python function during deserialisation, resulting in `code: print('pwned')` being executed by the application.

Examples:
- [Ophiuchi](../CTFs/Ophiuchi.md)
- [YAML](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Insecure%20Deserialization%20&%20Object%20Injection.md#YAML)

## Python3 & class pollution

Python can be vulnerable to [Class Pollution](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Class%20Pollution.md) . Unlike the other pitfalls covered here, this is not tied to a specific function but instead relates to how Python handles variables in memory and how they can be unexpectedly overwritten if improperly managed. Class pollution typically arises in code that merges objects insecurely.

## Python urllib.parse.urljoin

Python’s [`urllib.parse.urljoin`](https://docs.python.org/3/library/urllib.parse.html#urllib.parse.urljoin) function constructs a final URL by combining a base URL with one or more URL components.

`urllib.parse.urljoin` discards the entire base URL when an absolute URL is provided as input, ignoring everything that came before it.

```python
from urllib.parse import urljoin

payload = "http://evil.com/"

print(urljoin("http://example.com/", payload))  # output: http://evil.com/
```