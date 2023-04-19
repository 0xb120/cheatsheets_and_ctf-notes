---
Ports: 1098, 1099
Description: Is a Java API that performs remote method invocation, the object-oriented equivalent of remote procedure calls (RPC), with support for direct transfer of serialized Java classes and distributed garbage-collection.
---

>[!info]
> The Java Remote Method Invocation, or Java RMI, is a mechanism that allows an object that exists in one Java virtual machine to access and call methods that are contained in another Java virtual machine; This is basically the same thing as a [remote procedure call](https://null-byte.wonderhowto.com/how-to/hack-like-pro-exploit-and-gain-remote-access-pcs-running-windows-xp-0134709/), but in an object-oriented paradigm instead of a procedural one, which allows for communication between Java programs that are not in the same address space.

One of the major advantages of RMI is the ability for remote objects to load new classes that aren't explicitly defined already, extending the behavior and functionality of an application [^1].

[^1]: https://null-byte.wonderhowto.com/how-to/exploit-java-remote-method-invocation-get-root-0187685/

# Enumeration

The default configuration of `rmiregistry`allows loading classes from remote URLs, which can lead to remote code execution.
**Basically this service could allow you to execute code.**

- [msfconsole (auxiliary modules)](../Tools/msfconsole.md#Auxiliary%20Modules)
- [NSE (Nmap Scripting Engine)](../Tools/nmap.md#NSE%20(Nmap%20Scripting%20Engine))

```bash
msf> use auxiliary/scanner/misc/java_rmi_server
msf> use auxiliary/gather/java_rmi_registry
nmap -sV --script "rmi-dumpregistry or rmi-vuln-classloader" -p <PORT> <IP>
```

## RMI methods enumeration

- [RMIscout](https://github.com/BishopFox/rmiscout)
- [BaRMIe](https://github.com/NickstaDB/BaRMIe)

---

# Exploitation

## Reverse Shell

- [msfconsole](../Tools/msfconsole.md)

```bash
msf> use exploit/multi/browser/java_rmi_connection_impl
```