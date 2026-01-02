---
author: 0xdf hacks stuff
aliases:
  - Python Tools and Scripts W/ UV CheatSheet
tags:
  - RW_inbox
  - readwise/articles
  - tools
url: https://0xdf.gitlab.io/cheatsheets/uv?__readwiseLocation=
date: 2025-05-08
---
# Python Tools and Scripts W/ UV CheatSheet

![rw-book-cover](https://0xdf.gitlab.io/img/uv-cs-cover.png)

## Highlights


UV is the hot new tool among Python developers. [](https://read.readwise.io/read/01jtjqptndwj60brahtmxydxgd)



It addresses a ton of issues in the Python ecosystem, from packaging, project management, tool installation, and virtual environment management. [](https://read.readwise.io/read/01jtjqq409ryrd6m4txwgh8ykd)



This post is all about how to use UV to install and run Python applications and scripts. [](https://read.readwise.io/read/01jtjqqgv36a0jde7hpfc2qz1j)



I released a [video showing a lot of this in action](https://www.youtube.com/watch?v=G36QXtBXKBQ), and it’ll remain the more detailed explanation of this tool [](https://read.readwise.io/read/01jtjqqs6pfdbzj7xna70438f0)



This post is going to show how I now use it to replace `pipx` for installing tools as well as `venv` for running Python scripts. [](https://read.readwise.io/read/01jtjqrvd9s2x0201cztzwx9hq)



Installing `uv` is recommended to install using a shell script downloaded from their servers. [](https://read.readwise.io/read/01jtjqscbkcm67rj5ztndr1z7g)



Overview
 Anywhere I used to use `pipx` I’m now using `uv tool`. Other than that replacement, the syntax is the same. [](https://read.readwise.io/read/01jtjqsqp0stp1h17sa3zphk17)



oxdf@hacky$ uv tool install impacket Resolved 21 packages in 537ms Prepared 1 package in 522ms Installed 21 packages in 20ms + blinker==1.9.0 + cffi==1.17.1 [](https://read.readwise.io/read/01jtjqsyqzwpcfjp6xxy52v062)



Install from GitHub:
 $ uv tool install git+https://github.com/Pennyw0rth/NetExec [](https://read.readwise.io/read/01jtjqt8rtg2j81zrt6msw1ds2)



Install a specific branch from a GitHub repo:
 $ uv tool install git+https://github.com/dirkjanm/BloodHound.py.git@bloodhound-c [](https://read.readwise.io/read/01jtjqtn9b5n4c84pxj9487njz)



Install from a local directory (where it is set up as a package with the legacy `setup.py` or `setup.cfg`, or the modern `pyproject.toml` file):
 $ uv tool install . [](https://read.readwise.io/read/01jtjqv0dw00j2yhxm67t4fykh)



Updating
 `uv tool list` will show what packages are installed *and* what binaries are now runnable through those packages:
 oxdf@hacky$ uv tool list
 black v25.1.0
 - black
 - blackd
 bloodhound-ce v1.8.0
 - bloodhound-ce-python [](https://read.readwise.io/read/01jtjqvcaz2psad7pvxh6x08x0)



`uv tool upgrade --all` or `uv tool upgrade <package>` will update to later version of these packages. [](https://read.readwise.io/read/01jtjqvhqvz4qxeakv4cxq928z)



ScriptsRunningOverview
 A more interesting challenge is running Python scripts that aren’t set up with a package, but may just have a `requirements.txt` file listing their dependencies outside the Python standard library. [](https://read.readwise.io/read/01jtjqweqavmwnpwnb018f3dcj)



With uv, I’ll add the dependencies to the script with in-line metadata, and then run it. [](https://read.readwise.io/read/01jtjqwv674xgfcrqtsa99y54g)



The first step is to add the requirements to the script using [PEP 723 - inline script metadata](https://peps.python.org/pep-0723/).
 `uv add --script <target python script> <package name>` will add a single dependency:
 oxdf@hacky$ uv add --script targetedKerberoast.py impacket
 Updated `targetedKerberoast.py`
 oxdf@hacky$ head targetedKerberoast.py 
 #!/usr/bin/env python3
 # /// script
 # requires-python = ">=3.12"
 # dependencies = [
 # "impacket",
 # ]
 # ///
 # -*- coding: utf-8 -*-
 # File name : targetedKerberoast.py [](https://read.readwise.io/read/01jtjqx62pp3665mm9bc2gemkr)



`-r <requirements.txt file>` will add an entire file:
 oxdf@hacky$ uv add --script targetedKerberoast.py -r requirements.txt 
 Updated `targetedKerberoast.py`
 oxdf@hacky$ head targetedKerberoast.py 
 #!/usr/bin/env python3
 # /// script
 # requires-python = ">=3.12"
 # dependencies = [
 # "impacket",
 # "ldap3",
 # "pyasn1",
 # "pycryptodome",
 # "rich",
 # ] [](https://read.readwise.io/read/01jtjqxg8sh5q7978sk814gy15)



Run
 Now run the script with `uv run <script>`:
 oxdf@hacky$ uv run targetedKerberoast.py 
 Installed 26 packages in 21ms
 usage: targetedKerberoast.py [-h] [-v] [-q] [-D TARGET_DOMAIN] [-U USERS_FILE] [--request-user username] [-o OUTPUT_FILE] [-f {hashcat,john}] [--use-ldaps] [--only-abuse]
 [--no-abuse] [--dc-host DC_HOST] [--dc-ip ip address] [-d DOMAIN] [-u USER] [-k] [--no-pass | -p PASSWORD | -H [LMHASH:]NTHASH | --aes-key hex key]
 targetedKerberoast.py: error: need to set credentials [](https://read.readwise.io/read/01jtjqxnjkw9h4cjy8cvm0r6fh)



Shebang
 I can modify the [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) so that the script will just run with `uv run` if called directly:
 oxdf@hacky$ head targetedKerberoast.py 
 #!/usr/bin/env -S uv run --script
 # /// script
 # requires-python = ">=3.12"
 # dependencies = [
 # "impacket",
 # "ldap3",
 # "pyasn1",
 # "pycryptodome",
 # "rich",
 # ]
 Now it runs like a binary:
 oxdf@hacky$ chmod +x targetedKerberoast.py 
 oxdf@hacky$ ./targetedKerberoast.py 
 usage: targetedKerberoast.py [-h] [-v] [-q] [-D TARGET_DOMAIN] [-U USERS_FILE] [--request-user username] [-o OUTPUT_FILE] [-f {hashcat,john}] [--use-ldaps] [--only-abuse]
 [--no-abuse] [--dc-host DC_HOST] [--dc-ip ip address] [-d DOMAIN] [-u USER] [-k] [--no-pass | -p PASSWORD | -H [LMHASH:]NTHASH | --aes-key hex key]
 targetedKerberoast.py: error: need to set credentials [](https://read.readwise.io/read/01jtjqyx96g3r2pq8yv7vh9wvt)

