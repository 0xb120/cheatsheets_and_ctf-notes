---
Description: A tool for de-obfuscating android package into Classes.dex which can be use Dex2jar and [JD-GUI](JD-GUI.md) to extract contents of dex file.
URL: https://github.com/CalebFenton/simplify
---

>[!summary]
>Simplify virtually executes an app to understand its behavior and then tries to optimize the code so that it behaves identically but is easier for a human to understand. Each optimization type is simple and generic, so it doesn't matter what the specific type of obfuscation is used.

## De-obfuscating decompiled smali code

```bash
$ simplify.jar -i "input smali files or folder" -o <output dex file>
```

![](../../zzz_res/attachments/simplify1.png)

![](../../zzz_res/attachments/simplify2.png)