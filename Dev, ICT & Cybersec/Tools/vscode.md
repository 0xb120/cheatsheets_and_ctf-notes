---
aliases: [Visual Studio Code]
---
# Useful shortcuts

https://code.visualstudio.com/docs/editor/codebasics

Folding:
- **Toggle Fold** (`Ctrl+K Ctrl+L`): folds or unfolds the region at the cursor.
- **Fold All** (`Ctrl+K Ctrl+0`): folds all regions in the editor.
- **Unfold All** (`Ctrl+K Ctrl+J`): unfolds all regions in the editor.
- **Fold Level X** (`Ctrl+K Ctrl+2` for level 2): folds all regions of level X, except the region at the current cursor position.
- **Fold All Block Comments** (`Ctrl+K Ctrl+/`): folds all regions that start with a block comment token.
# vscode 101

`.vscode/launch.json`: is what contains the various definitions used for launching and/or debugging the project

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "request": "launch",
            "name": "Launch Program",
            "program": "${file}"
        }
    ]
}
```

# Debug different languages
- [Remote Debugging NodeJS](../Dev,%20scripting%20&%20OS/JavaScript%20&%20NodeJS.md#Remote%20Debugging%20NodeJS)
- [Remote debug python application](../Dev,%20scripting%20&%20OS/Python.md#Remote%20debug%20python%20application)
- [Enable remote debug in PHP with Xdebug](../Dev,%20scripting%20&%20OS/PHP.md#Enable%20remote%20debug%20in%20PHP%20with%20Xdebug)


# Useful extension

## Compare Folders

[Compare Folders](https://marketplace.visualstudio.com/items?itemName=moshfeu.compare-folders): useful for #patch-diffing  and investigating newer/older/modified files 
  ![](attachments/vscode-compare-folders.png)
  ^397c19

## CodeQL for VSCode

[CodeQL VSCode extention](../../Readwise/Articles/Frycos%20Security%20Diary%20-%20Pre-Auth%20RCE%20With%20CodeQL%20in%20Under%2020%20Minutes.md#CodeQL%20VSCode%20extention): integration to the [CodeQL](CodeQL.md) engine
![codeql-ext-vscode](../../Readwise/Articles/attachments/codeql-ext-vscode.png)
![codeql-run-query](../../Readwise/Articles/attachments/codeql-run-query.png)

## SARIF Explorer

[SARIF Explorer](https://marketplace.visualstudio.com/items?itemName=trailofbits.sarif-explorer): SARIF Explorer [^1] is a VSCode extension that enables you to review static analysis results effectively and enjoyably. Use it with [semgrep](semgrep.md) for fantastic results. ^b54027

[^1]: [Marco Ivaldi - Aiding Reverse Engineering With Rust and a Local LLM](../../Readwise/Articles/Marco%20Ivaldi%20-%20Aiding%20Reverse%20Engineering%20With%20Rust%20and%20a%20Local%20LLM.md)
