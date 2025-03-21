---
author: "Piotr Bazydlo (@chudyPB)"
aliases: "By Executive Order, We Are Banning Blacklists - Domain-Level RCE in Veeam Backup & Replication"
tags: RW_inbox, readwise/articles, dotnet
url: ?__readwiseLocation=
date: 2025-03-21
summary: A new vulnerability in Veeam Backup & Replication allows remote code execution due to flaws in its blacklist-based security mechanism for deserialization. Researchers found that the blacklist was incomplete, missing known deserialization gadgets that could be exploited. The post argues that using blacklists for security is ineffective, suggesting that whitelists are a better approach.
---
# By Executive Order, We Are Banning Blacklists - Domain-Level RCE in Veeam Backup & Replication

![rw-book-cover](https://labs.watchtowr.com/content/images/size/w1200/2025/03/veeam-1.png)

## Highlights


today we'll explore the painful world of blacklist-based security mechanisms. You can treat this post as a natural continuation of our [CVE-2024-40711 writeup](https://labs.watchtowr.com/veeam-backup-response-rce-with-auth-but-mostly-without-auth-cve-2024-40711-2/) [](https://read.readwise.io/read/01jpw83jrcrz8z02413kdjvs8k)



Our previous watchTowr Labs post provided a detailed walk-through of a Remote Code Execution vulnerability RCE issue in Veeam Backup & Replication [](https://read.readwise.io/read/01jpw842ezdcyskf2w3ebgmsv4)



You all probably know that the uncontrolled deserialization always leads to bad things. This is why one should always implement strict control of classes that are being deserialized. Ideally, it should be a whitelist, which allows the deserialization of selected classes only. [](https://read.readwise.io/read/01jpw84h3qpmtwc90f3t99y5ge)



Blacklists (also known as block-lists or deny-lists) are based on a very optimistic (and provably flawed) idea that we can just make a list of all the bad classes, and we just keep a record of everything bad that can be done and update our list as and when new bad is introduced. [](https://read.readwise.io/read/01jpw859rxt3kzg97aytdted4p)



There are many technical posts and papers about abusing the deserialization blacklists. I can shamelessly point you to my [whitepaper](https://github.com/thezdi/presentations/blob/main/2023_Hexacon/whitepaper-net-deser.pdf?ref=labs.watchtowr.com) and [Hexacon talk](https://www.youtube.com/watch?v=_CJmUh0_uOM&ref=labs.watchtowr.com), where I’ve been pwning blacklists through new deserialization gadgets discovered in .NET Framework, targeted product codebase and 3rd party libraries. [](https://read.readwise.io/read/01jpw85tpzed110v56395gfnn3)



In this blog post, we will show you 2 Remote Code Execution vulnerabilities in the Veeam Backup & Response solution, which are based on very similar deserialization gadgets existing in the Veeam codebase. [](https://read.readwise.io/read/01jpw879b9yvb37ejrs2z0cknj)



CVE-2024-40711 Recap
 CVE-2024-40711 is fully detailed in [our previous blog post](https://labs.watchtowr.com/veeam-backup-response-rce-with-auth-but-mostly-without-auth-cve-2024-40711-2/). [](https://read.readwise.io/read/01jpw9gav1v3xwfjww4cwjhdtn)



Veeam Backup & Replication exposes the .NET Remoting Channel, which as you may know allows you to reach some internal deserialization capabilities based on `BinaryFormatter`. [](https://read.readwise.io/read/01jpw9ghkjbtdrqb3kqjsmqmbw)



the code defines the `RestrictedSerializationBinder` for the deserialization process. This binder is responsible for verifying the target deserialization type. Unfortunately, ofcourse, it is based on a blacklist. It means that we can:
 1. Deserialize the **whitelisted** `CDbCryptoKeyInfo` class.
 2. Reach the internal deserialization mechanism, which is **controlled by a blacklist.** [](https://read.readwise.io/read/01jpw9j7dry8ygama0xfbmdv9c)



we still need to verify the blacklist and see if it misses some commonly known deserialization gadgets. This list is defined in the `Veeam.Backup.Common.Sources.System.IO.BinaryFormatter.blacklist.txt` file, which is attached to the `Veeam.Backup.Common.dll` as a resource. [](https://read.readwise.io/read/01jpw9jkp02dz0qz8vbfd6hxkv)



This list originally missed the `System.Runtime.Remoting.ObjRef` gadget, which is publicly known and can be used to achieve the RCE. [](https://read.readwise.io/read/01jpw9js3pp3dvr9aj7epec992)



`ObjRef` is a relatively new gadget discovered [Markus Wulftange](https://x.com/mwulftange?ref=labs.watchtowr.com) and there is a relatively huge chance that this gadget is younger than the Veeam deserialization blacklist. [](https://read.readwise.io/read/01jpw9k4t92nea3mxdrr2nwt0c)



If you're familiar with .NET `BinaryFormatter` deserialization, you may quickly notice that this list still misses some known gadgets. [](https://read.readwise.io/read/01jpw9mafp2pxfa8zpa7x9ayhx)



Veeam extended the blacklist again:
 System.CodeDom.Compiler.TempFileCollection
 System.IO.DirectoryInfo
 This list is getting better and better, and soon it will be extra perfect!
 But, can we actually find something more there? [](https://read.readwise.io/read/01jpw9mwgqxb46c210b2pgdzkb)



WT-2025-0014 RCE: xmlFrameworkDs gadget
 Once we've determined how we can reach the deserialization sink based on the blacklist, the game becomes quite simple. [](https://read.readwise.io/read/01jpw9patxb6bqqkhc7fx10b1v)



you only need to find a deserialization gadget which is not blacklisted and leads to some potentially malicious impact.
 As we have already stated, abusable classes can be found not only in the .NET Framework, but also typically in the target product's codebase. [](https://read.readwise.io/read/01jpw9pfthp1c7dht63zg78dj6)



because we are targeting `BinaryFormatter`, it strongly narrows our gadget-searching capabilities. This is because `BinaryFormatter` can only deserialize classes that fulfil specific conditions, like:
 • `Serializable` attribute set for a class.
 • Magic deserialization methods need to be defined for a class (like `SerializationInfo` based constructor). [](https://read.readwise.io/read/01jpw9q94n85gbtyxa4tjvsp7m)



namespace Veeam.Backup.EsxManager
 {
 [DesignerCategory("code")]
 [ToolboxItem(true)]
 [XmlSchemaProvider("GetTypedDataSetSchema")]
 [XmlRoot("xmlFrameworkDs")]
 [HelpKeyword("vs.data.DataSet")]
 [Serializable]
 public class xmlFrameworkDs : DataSet // [1]
 If you have ever read .NET-based deserialization research or used [ysoserial.net](http://ysoserial.net/?ref=labs.watchtowr.com), you will notice the red flag right away.
 At `[1]`, you can see that `xmlFrameworkDs` extends `DataSet`. This is a very common and popular RCE gadget. When you're able to deserialize `DataSet`, you get insta-RCE capabilities. [](https://read.readwise.io/read/01jpw9qsrw504rbp8t98ztg085)



Now, let’s have a quick look at `xmlFrameworkDs` magic constructor:
 protected xmlFrameworkDs(SerializationInfo info, StreamingContext context) : base(info, context, false) // [1]
 {
 if (base.IsBinarySerialized(info, context))
 {
 this.InitVars(false);
 CollectionChangeEventHandler value = new CollectionChangeEventHandler(this.SchemaChanged);
 this.Tables.CollectionChanged += value;
 this.Relations.CollectionChanged += value;
 return;
 }
 //...
 } [](https://read.readwise.io/read/01jpw9r9ye79kcjp8nd0rf5fds)



Here, we can see that `xmlFrameworkDs` **will call its parent's constructor at `[1]`.** As its parent is `DataSet`, we get an easy RCE here. We just need to modify the `DataSet` gadget and accordingly modify the type/assembly names. [](https://read.readwise.io/read/01jpw9rjnxehbnsyrh8zxemn4e)



In order to exploit this vulnerability, inspired readers and keyboard bashes can likely implement some modifications to Sina’s [CVE-2024-40711](https://github.com/watchtowrlabs/CVE-2024-40711?ref=labs.watchtowr.com) PoC. [](https://read.readwise.io/read/01jpw9s2cwzbbkxxds1qhw5a9k)



WT-2025-0015 RCE: BackupSummary gadget
 The second deserialization gadget is `Veeam.Backup.Core.BackupSummary`. [](https://read.readwise.io/read/01jpw9sqej74rywhg3w8nqtcnc)



namespace Veeam.Backup.Core { [DesignerCategory("code")] [ToolboxItem(true)] [XmlSchemaProvider("GetTypedDataSetSchema")] [XmlRoot("BackupSummary")] [HelpKeyword("vs.data.DataSet")] [Serializable] public class BackupSummary : DataSet // [1] [](https://read.readwise.io/read/01jpw9svpdrdy3k09k45fhz2kh)



There is a second class in the Veeam codebase, which also extends the `DataSet`. [](https://read.readwise.io/read/01jpw9t26bmk71m8vkaqstb03r)

