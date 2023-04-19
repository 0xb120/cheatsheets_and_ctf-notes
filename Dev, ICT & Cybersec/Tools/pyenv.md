---
Description: Simple Python version management
URL: https://github.com/pyenv/pyenv
---

>[!abstract]
>Use `pyenv` to manage multiple isolated version of Python

```bash
$ pyenv
pyenv 1.2.23
Usage: pyenv <command> [<args>]

Some useful pyenv commands are:
   activate    Activate virtual environment
   commands    List all available pyenv commands
   deactivate   Deactivate virtual environment
   doctor      Verify pyenv installation and development tools to build pythons.
   exec        Run an executable with the selected Python version
   global      Set or show the global Python version(s)
   help        Display help for a command
   hooks       List hook scripts for a given pyenv command
   init        Configure the shell environment for pyenv
   install     Install a Python version using python-build
   local       Set or show the local application-specific Python version(s)
   prefix      Display prefix for a Python version
   rehash      Rehash pyenv shims (run this after installing executables)
   root        Display the root directory where versions and shims are kept
   shell       Set or show the shell-specific Python version
   shims       List existing pyenv shims
   uninstall   Uninstall a specific Python version
   --version   Display the version of pyenv
   version     Show the current Python version(s) and its origin
   version-file   Detect the file that sets the current pyenv version
   version-name   Show the current Python version
   version-origin   Explain how the current Python version is set
   versions    List all Python versions available to pyenv
   virtualenv   Create a Python virtualenv using the pyenv-virtualenv plugin
   virtualenv-delete   Uninstall a specific Python virtualenv
   virtualenv-init   Configure the shell environment for pyenv-virtualenv
   virtualenv-prefix   Display real_prefix for a Python virtualenv version
   virtualenvs   List all Python virtualenvs found in `$PYENV_ROOT/versions/*'.
   whence      List all Python versions that contain the given executable
   which       Display the full path to an executable
```

## Install

```bash
$ sudo apt install -y build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev python3-openssl git
$ curl https://pyenv.run | bash

# Kali ZSH specific settings
$ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
$ echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
$ echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.zshrc
```

## Setting up the basic environment

- Install a specific version of python:

```bash
$ pyenv install --list
Available versions:
  2.1.3
  2.2.3
  2.3.7
  ...
$ pyenv install -v 3.7.2
```

- Remove a specific version of python:

```bash
$ rm -rf ~/.pyenv/versions/2.7.15
# or
$ pyenv uninstall 2.7.15
```

- Switch through the various versions of python:

```bash
$ pyenv versions
  system
* 2.7.17 (set by /home/kali/.pyenv/version)
  2.7.18
$ pyenv global 2.7.18
```

## Customizing local environments

- Set an application-specific python version for the **current shell**:

```bash
$ pyenv shell 2.7.15
$ pyenv shell --unset
```

- Set an application-specific python version for the **current folder**:

```
$ pyenv local 2.7.15
```

![https://files.realpython.com/media/pyenv-pyramid.d2f35a19ded9.png](https://files.realpython.com/media/pyenv-pyramid.d2f35a19ded9.png)

## Creating Virtual Environment

- Create and activate a virtual environment

```bash
$ pyenv virtualenv 3.6.8 project_name

$ pyenv local project_name
# or
$ pyenv activate <environment_name>
$ pyenv deactivate
```