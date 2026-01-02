---
title: "TheR1D/shell_gpt: A command-line productivity tool powered by AI large language models like GPT-4, will help you accomplish your tasks faster and more efficiently."
source: https://github.com/TheR1D/shell_gpt
author:
  - TheR1D
published:
created: 2025-08-19
description: A command-line productivity tool powered by AI large language models like GPT-4, will help you accomplish your tasks faster and more efficiently. - TheR1D/shell_gpt
tags:
  - clippings/articles
  - tools
  - AI
  - LLM
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

```sh
pip install shell-gpt
```

By default, ShellGPT uses OpenAI's API and GPT-4 model. You'll need an API key, you can generate one [here](https://beta.openai.com/account/api-keys). You will be prompted for your key which will then be stored in `~/.config/shell_gpt/.sgptrc`. 

Alternatively, you can use locally hosted open source models which are available for free. To use local models, you will need to run your own LLM backend server such as [Ollama](https://github.com/ollama/ollama). To set up ShellGPT with Ollama, please follow this comprehensive [guide](https://github.com/TheR1D/shell_gpt/wiki/Ollama).

## Usage

```sh
sgpt "What is the fibonacci sequence"
# -> The Fibonacci sequence is a series of numbers where each number ...

git diff | sgpt "Generate git commit message, for my changes"
# -> Added main feature details into README.md

docker logs -n 20 my\_app | sgpt "check logs, find errors, provide possible solutions"
Error Detected: Connection timeout at line 7.
Possible Solution: Check network connectivity and firewall settings.
Error Detected: Memory allocation failed at line 12.
Possible Solution: Consider increasing memory allocation or optimizing application memory usage.
```

### Shell commands

With `--shell` or shortcut `-s` option, you can quickly generate and execute the commands you need right in the terminal.

```sh
sgpt --shell "find all json files in current folder"
# -> find . -type f -name "*.json"
# -> [E]xecute, [D]escribe, [A]bort: e
```

### Generating code

By using the `--code` or `-c` parameter, you can specifically request pure code output, for instance:

```sh
sgpt --code "solve fizz buzz problem using python"
for i in range(1, 101):
    if i % 3 == 0 and i % 5 == 0:
        print("FizzBuzz")
    elif i % 3 == 0:
        print("Fizz")
    elif i % 5 == 0:
        print("Buzz")
    else:
        print(i)
        
sgpt --code "solve classic fizz buzz problem using Python" > fizz_buzz.py
python fizz_buzz.py

cat fizz_buzz.py | sgpt --code "Generate comments for each line of my code"
```

### Chat Mode

Often it is important to preserve and recall a conversation. `sgpt` creates conversational dialogue with each LLM completion requested. The dialogue can develop one-by-one (chat mode) or interactively, in a REPL loop (REPL mode). Both ways rely on the same underlying object, called a chat session. The session is located at the [configurable](https://github.com/TheR1D/#runtime-configuration-file) `CHAT_CACHE_PATH`.

To start a conversation, use the `--chat` option followed by a unique session name and a prompt.

```sh
sgpt --chat conversation_1 "please remember my favorite number: 4"
# -> I will remember that your favorite number is 4.
sgpt --chat conversation_1 "what would be my favorite number + 4?"
# -> Your favorite number is 4, so if we add 4 to it, the result would be 8.

sgpt --show-chat conversation_1
# user: please remember my favorite number: 4
# assistant: I will remember that your favorite number is 4.
# user: what would be my favorite number + 4?
# assistant: Your favorite number is 4, so if we add 4 to it, the result would be 8.

sgpt --chat conversation_2 --code "make a request to localhost using python"
sgpt --chat conversation_2 --code "add caching"

sgpt --chat conversation_3 --shell "what is in current folder"
# -> ls
sgpt --chat conversation_3 "Sort by name"
# -> ls | sort
sgpt --chat conversation_3 "Concatenate them using FFMPEG"
# -> ffmpeg -i "concat:$(ls | sort | tr '\n' '|')" -codec copy output.mp4
sgpt --chat conversation_3 "Convert the resulting file into an MP3"
# -> ffmpeg -i output.mp4 -vn -acodec libmp3lame -ac 2 -ab 160k -ar 48000 final_output.mp3
```

### REPL Mode

There is very handy REPL (read–eval–print loop) mode, which allows you to **interactively chat with GPT models**. 

To start a chat session in REPL mode, use the `--repl` option followed by a unique session name. You can also use "temp" as a session name to start a temporary REPL session. 

>[!note]
>Note that `--chat` and `--repl` are using same underlying object, so you can use `--chat` to start a chat session and then pick it up with `--repl` to continue the conversation in REPL mode.

```sh
sgpt --repl temp
Entering REPL mode, press Ctrl+C to exit.
>>> What is REPL?
REPL stands for Read-Eval-Print Loop. It is a programming environment ...
>>> How can I use Python with REPL?
To use Python with REPL, you can simply open a terminal or command prompt ...


sgpt --repl temp --shell
Entering shell REPL mode, type [e] to execute commands or press Ctrl+C to exit.
>>> What is in current folder?
ls
>>> Show file sizes
ls -lh
>>> Sort them by file sizes
ls -lhS
>>> e (enter just e to execute commands, or d to describe them)
```

### Function calling

[Function calls](https://platform.openai.com/docs/guides/function-calling) is a powerful feature OpenAI provides. It allows LLM to execute functions in your system, which can be used to accomplish a variety of tasks. To install [default functions](https://github.com/TheR1D/shell_gpt/tree/main/sgpt/llm_functions/) run:

```sh
sgpt --install-functions
```

ShellGPT has a convenient way to define functions and use them. In order to create your custom function, navigate to `~/.config/shell_gpt/functions` and create a new .py file with the function name. Inside this file, you can define your function.

The docstring comment inside the class will be passed to OpenAI API as a description for the function, along with the `title` attribute and parameters descriptions. The `execute` function will be called if LLM decides to use your function.

```sh
sgpt "What are the files in /tmp folder?"
# -> @FunctionCall execute_shell_command(shell_command="ls /tmp")
# -> The /tmp folder contains the following files and directories:
# -> test.txt
# -> test.json
```

This is just a simple example of how you can use function calls. It is truly a powerful feature that can be used to accomplish a variety of complex tasks. We have dedicated [category](https://github.com/TheR1D/shell_gpt/discussions/categories/functions) in GitHub Discussions for sharing and discussing functions. LLM might execute destructive commands, so please use it at your own risk❗️

## Full list of arguments

You can setup some parameters in runtime configuration file `~/.config/shell_gpt/.sgptrc`:

```
╭─ Arguments ──────────────────────────────────────────────────────────────────────────────────────────────╮
│   prompt      [PROMPT]  The prompt to generate completions for.                                          │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────╯
╭─ Options ────────────────────────────────────────────────────────────────────────────────────────────────╮
│ --model            TEXT                       Large language model to use. [default: gpt-4o]             │
│ --temperature      FLOAT RANGE [0.0<=x<=2.0]  Randomness of generated output. [default: 0.0]             │
│ --top-p            FLOAT RANGE [0.0<=x<=1.0]  Limits highest probable tokens (words). [default: 1.0]     │
│ --md             --no-md                      Prettify markdown output. [default: md]                    │
│ --editor                                      Open $EDITOR to provide a prompt. [default: no-editor]     │
│ --cache                                       Cache completion results. [default: cache]                 │
│ --version                                     Show version.                                              │
│ --help                                        Show this message and exit.                                │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────╯
╭─ Assistance Options ─────────────────────────────────────────────────────────────────────────────────────╮
│ --shell           -s                      Generate and execute shell commands.                           │
│ --interaction         --no-interaction    Interactive mode for --shell option. [default: interaction]    │
│ --describe-shell  -d                      Describe a shell command.                                      │
│ --code            -c                      Generate only code.                                            │
│ --functions           --no-functions      Allow function calls. [default: functions]                     │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────╯
╭─ Chat Options ───────────────────────────────────────────────────────────────────────────────────────────╮
│ --chat                 TEXT  Follow conversation with id, use "temp" for quick session. [default: None]  │
│ --repl                 TEXT  Start a REPL (Read–eval–print loop) session. [default: None]                │
│ --show-chat            TEXT  Show all messages from provided chat id. [default: None]                    │
│ --list-chats  -lc            List all existing chat ids.                                                 │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────╯
╭─ Role Options ───────────────────────────────────────────────────────────────────────────────────────────╮
│ --role                  TEXT  System role for GPT model. [default: None]                                 │
│ --create-role           TEXT  Create role. [default: None]                                               │
│ --show-role             TEXT  Show role. [default: None]                                                 │
│ --list-roles   -lr            List roles.                                                                │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────╯
```