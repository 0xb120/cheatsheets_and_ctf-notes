>[!summary]
>Remote Code Execution or execution, also known as Arbitrary Code Execution, **is a concept** (and not a specific type of vulnerability) that describes a form of cyberattack in which the attacker can solely command the operation of another person’s computing device or computer.

# Introduction to RCE and how to find them

Remote Code Execution is not a specific vulnerability but it’s a behavior achieved exploiting and eventually chaining other types of vulnerabilities, like:

- [File Inclusion (LFI & RFI)](File%20Inclusion%20(LFI%20&%20RFI).md)
- [Insecure File Upload](Insecure%20File%20Upload.md)
- [XML External Entity Injection (XXE Injection)](XML%20External%20Entity%20Injection%20(XXE%20Injection).md)
- [Command Injection](Command%20Injection.md)
- [Insecure Deserialization & Object Injection](Insecure%20Deserialization%20&%20Object%20Injection.md)
- [Cross-Site Scripting (XSS)](Cross-Site%20Scripting%20(XSS).md)
- [SQL Injection](SQL%20Injection.md)
- [Remote code execution via server-side prototype pollution](Prototype%20Pollution%20server-side.md#Remote%20code%20execution%20via%20server-side%20prototype%20pollution)
- etc.

## Finding RCE vulnerabilities

- In a white-box approach, **trace-back all the sinks and sources of dangerous function** and check if user-controllable data is passed to them.

- In a grey/black-box approach, fuzzing, experimenting and relying on existing PoC is the only way to find command execution vulnerabilities.