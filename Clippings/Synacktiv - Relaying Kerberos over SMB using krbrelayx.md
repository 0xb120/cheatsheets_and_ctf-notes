---
title: "Relaying Kerberos over SMB using krbrelayx"
source: "https://www.synacktiv.com/publications/relaying-kerberos-over-smb-using-krbrelayx"
author:
  - "Synacktiv"
published:
created: 2025-08-13
description:
tags: ["clippings/articles", "_inbox"]
---
# Relaying Kerberos over SMB using krbrelayx

![](https://www.synacktiv.com/sites/default/files/styles/blog_grid_view/public/2024-11/9azn60_copy_660x330.jpg)

> [!summary]
> The article details a technique for relaying Kerberos authentication over SMB, building on James Forshaw's 2021 research and implementing it with Synacktiv's modified krbrelayx tool. It explains how the `CredMarshalTargetInfo` trick allows a client to request a Kerberos ticket for a short SPN while secretly targeting a different service by embedding marshaled target information within the SPN itself, bypassing the need for DNS/DHCPv6 poisoning. The authors demonstrate registering a malicious DNS record, coercing a target to authenticate, and relaying the Kerberos AP_REQ via SMB to an ADCS HTTP endpoint to obtain a PFX certificate, usable for acquiring a TGT for a domain controller.
>This method proved effective in real-world scenarios where NTLM was refused, and a key mitigation involves monitoring DNS records for the static magic value used in the marshaled string.

Relaying Kerberos over SMB using krbrelayx

## Kerberos relaying 101

Kerberos relaying is theoretically straightforward. The goal is to relay an `AP_REQ` message, initiated by a client for one service, to another one. There is however one crucial prerequisite: the targeted service and client must not enforce encryption or signing, as we do not possess the secret (the session key) required to perform these operations, similarly to an NTLM relay attack.

There is also an additional challenge: an `AP_REQ` message cannot be relayed to a different service running under a different identity from the one initially requested by the client. To make the attack successful, we therefore need to force the client to generate an `AP_REQ` for the targeted service and send it to us. Here is a visual representation of what we want to achieve:

In 2021, James Forshaw published a [long blogpost](https://googleprojectzero.blogspot.com/2021/10/using-kerberos-for-authentication-relay.html) explaining how this could be achieved using various protocols

Then in 2022, Dirk-jan Mollema published [another blogpost](https://dirkjanm.io/relaying-kerberos-over-dns-with-krbrelayx-and-mitm6/) detailing how this could be accomplished using DNS.

Dirk-jan demonstrated that, using his tools `mitm6`/`Krbrelayx` and SOA DNS messages, it is possible to poison a client and force it to send an `AP_REQ` message for an arbitrary service, which can then be relayed. An interesting service that meets all the requirements is the ADCS HTTP endpoint, which is by default vulnerable to relay attacks since it does not enforce signing with HTTP.

While this attack is effective, it still requires the ability to poison the client with DHCPv6 messages to establish a man-in-the-middle position by advertising oneself as a DNS server. Consequently, it is not possible to poison arbitrary clients.

## CredMarshalTargetInfo

In his blogpost, James Forshaw showed that when an SMB client builds the SPN from the service class and name, the [SecMakeSPNEx2](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/ntifs/nf-ntifs-secmakespnex2) method is called. For the hostname `fileserver` and service class `cifs`, the returned SPN will look like the following:

`cifs/fileserver1UWhRCAAAAAAAAAAUAAAAAAAAAAAAAAAAAAAAAfileserversBAAAA`

The `SecMakeSPNEx2` function makes a call to the API function [`CredMarshalTargetInfo`](https://learn.microsoft.com/en-us/windows/win32/api/ntsecpkg/nf-ntsecpkg-credmarshaltargetinfo). This API takes a list of target information in a [`CREDENTIAL_TARGET_INFORMATION`](https://learn.microsoft.com/en-us/windows/win32/api/wincred/ns-wincred-credential_target_informationw) structure, marshals it using Base64 encoding and appends it to the end of the real SPN.

He also showed that if we register the DNS record `fileserver1UWhRCAAAAAAAAAAUAAAAAAAAAAAAAAAAAAAAAfileserversBAAAA`, the client would ask a Kerberos ticket for `cifs/fileserver` but would connect to `fileserver1UWhRCAAAAAAAAAAUAAAAAAAAAAAAAAAAAAAAAfileserversBAAAA`.

Under the hood, the client will call `CredUnmarshalTargetInfo` to parse the marshaled target information. However, the client does not consider the unmarshaled results. Instead, it simply determines the length of the marshaled data and removes that portion from the end of the target SPN string. Consequently, when the Kerberos package receives the target name, it has already been restored to the original SPN.