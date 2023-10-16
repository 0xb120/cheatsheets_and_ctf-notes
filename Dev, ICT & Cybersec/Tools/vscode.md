---
aliases: Visual Studio Code
---

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