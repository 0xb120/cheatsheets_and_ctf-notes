---
title: "TheR1D/shell_gpt: A command-line productivity tool powered by AI large language models like GPT-4, will help you accomplish your tasks faster and more efficiently."
source: "https://github.com/TheR1D/shell_gpt"
author:
  - "TheR1D"
published:
created: 2025-08-19
description: "A command-line productivity tool powered by AI large language models like GPT-4, will help you accomplish your tasks faster and more efficiently. - TheR1D/shell_gpt"
tags:
  - "clippings/articles"
  - "_inbox"
  - "tools"
  - "AI"
  - "LLM"
---
# TheR1D/shell_gpt: A command-line productivity tool powered by AI large language models like GPT-4, will help you accomplish your tasks faster and more efficiently.

![](https://opengraph.githubassets.com/fc1590d01c6bedecf7c4a410cd45d904ff780326000e013af5c27cc11d1e2aaf/TheR1D/shell_gpt)

> [!summary]
> > ShellGPT is a command-line productivity tool powered by AI large language models (LLM) like GPT-4.
> It helps generate shell commands, code snippets, and documentation, compatible with Linux, macOS, Windows, and various shells.
> 
> Key features include:
> - **Installation:** `pip install shell-gpt`, requiring an OpenAI API key by default (or local models like Ollama).
> - **Usage:** Analyze and retrieve information via direct prompts or piped input (e.g., `git diff | sgpt`).
> - **Shell Commands:** Generates OS/shell-aware commands (`-s` option), with interactive execution or `--no-interaction` for piping.
> - **Shell Integration:** Hotkeys (Ctrl+l by default) to directly insert generated commands into the terminal buffer for editing.
> - **Code Generation:** Provides pure code output (`-c` option), which can be redirected to files.
> - **Chat Mode:** Preserves conversational context across requests (`--chat <session_name>`).
> - **REPL Mode:** Interactive chat session (`--repl <session_name>`) supporting shell and code generation within the loop.
> - **Function Calling:** Allows LLM to execute defined Python functions (`--install-functions`), enabling complex tasks like installing missing tools or playing music.
> - **Roles:** Custom roles can be created to influence LLM behavior (e.g., `json_generator`).
> - **Request Cache:** Caches responses for quicker retrieval of identical queries (`--cache`).
> - **Runtime Configuration:** Customizable settings via `~/.config/shell_gpt/.sgptrc`.
> - **Docker Support:** Can be run as a Docker container, with options for Ollama integration.
> 
> The tool aims to reduce the need for external searches by bringing AI assistance directly to the command line.

A command-line productivity tool powered by AI large language models like GPT-4, will help you accomplish your tasks faster and more efficiently.

A command-line productivity tool powered by AI large language models (LLM). This command-line tool offers streamlined generation of **shell commands, code snippets, documentation**, eliminating the need for external resources (like Google search). Supports Linux, macOS, Windows and compatible with all major Shells like PowerShell, CMD, Bash, Zsh, etc.

## Installation

pip install shell-gpt

By default, ShellGPT uses OpenAI's API and GPT-4 model. You'll need an API key, you can generate one [here](https://beta.openai.com/account/api-keys). You will be prompted for your key which will then be stored in `~/.config/shell_gpt/.sgptrc`. OpenAI API is not free of charge, please refer to the [OpenAI pricing](https://openai.com/pricing) for more information.

Alternatively, you can use locally hosted open source models which are available for free. To use local models, you will need to run your own LLM backend server such as [Ollama](https://github.com/ollama/ollama). To set up ShellGPT with Ollama, please follow this comprehensive [guide](https://github.com/TheR1D/shell_gpt/wiki/Ollama).

## Usage

sgpt "What is the fibonacci sequence"
# -> The Fibonacci sequence is a series of numbers where each number ...

git diff | sgpt "Generate git commit message, for my changes"
# -> Added main feature details into README.md

docker logs -n 20 my\_app | sgpt "check logs, find errors, provide possible solutions"

### Shell commands

With `--shell` or shortcut `-s` option, you can quickly generate and execute the commands you need right in the terminal.

sgpt --shell "find all json files in current folder"
# -> find . -type f -name "\*.json"
# -> \[E\]xecute, \[D\]escribe, \[A\]bort: e

### Generating code

By using the `--code` or `-c` parameter, you can specifically request pure code output, for instance:

sgpt --code "solve fizz buzz problem using python"

### Chat Mode

Often it is important to preserve and recall a conversation. `sgpt` creates conversational dialogue with each LLM completion requested. The dialogue can develop one-by-one (chat mode) or interactively, in a REPL loop (REPL mode). Both ways rely on the same underlying object, called a chat session. The session is located at the [configurable](https://github.com/TheR1D/#runtime-configuration-file) `CHAT_CACHE_PATH`.

To start a conversation, use the `--chat` option followed by a unique session name and a prompt.

sgpt --chat conversation\_1 "please remember my favorite number: 4"
# -> I will remember that your favorite number is 4.
sgpt --chat conversation\_1 "what would be my favorite number + 4?"
# -> Your favorite number is 4, so if we add 4 to it, the result would be 8.

### REPL Mode

There is very handy REPL (read–eval–print loop) mode, which allows you to interactively chat with GPT models. To start a chat session in REPL mode, use the `--repl` option followed by a unique session name. You can also use "temp" as a session name to start a temporary REPL session. Note that `--chat` and `--repl` are using same underlying object, so you can use `--chat` to start a chat session and then pick it up with `--repl` to continue the conversation in REPL mode.

```
sgpt --repl temp
Entering REPL mode, press Ctrl+C to exit.
>>> What is REPL?
REPL stands for Read-Eval-Print Loop. It is a programming environment ...
>>> How can I use Python with REPL?
To use Python with REPL, you can simply open a terminal or command prompt ...
```