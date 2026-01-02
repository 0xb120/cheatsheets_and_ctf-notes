---
author: Scott Sutherland
aliases: [Hijacking SQL Server Credentials Using Agent Jobs for Domain Privilege Escalation]
tags: [readwise/articles]
url: https://www.netspi.com/blog/technical-blog/network-pentesting/hijacking-sql-server-credentials-with-agent-jobs-for-domain-privilege-escalation/
date: 2024-10-09
---
# Hijacking SQL Server Credentials Using Agent Jobs for Domain Privilege Escalation

![rw-book-cover](https://www.netspi.com/wp-content/uploads/2024/09/091024_TECH_Hijacking-SQL-Server-Credentials_Social.webp)

## Highlights


## What is a Credential Object in SQL Server?
 Credentials are objects in SQL Server that store information, such as usernames and passwords, which can be used to authenticate to external resources like other SQL Servers, file shares, or web services, and execute processes/tasks in the context of another user. Credential types include SQL Server logins, local Windows users, and Active Directory domain users.
> [View Highlight](https://read.readwise.io/read/01j9s7724x2afhv9gyrxtcdyta)



There are many legitimate use cases for credential objects in SQL Server, but like all stored authentication tokens, they can be targeted and abused by threat actors.
> [View Highlight](https://read.readwise.io/read/01j9s787vmw8g1c1t7ehxmszpz)



## So how do we recover them from the SQL Server credential objects? 
The big hurdle is encryption. The information stored in credential objects is encrypted through the process described [here](https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/sql-server-encryption?view=sql-server-ver16).
> [View Highlight](https://read.readwise.io/read/01j9s790pwzwwv69g2r9ktf0qr)



Antti Rantasaari developed a PowerShell script in 2014 that decrypts the credentials stored in SQL Server objects. He also provided a detailed blog [post](https://www.netspi.com/blog/technical-blog/adversary-simulation/decrypting-mssql-credential-passwords/) outlining the decryption process. This script has since been ported to the [Get-DecryptedObject](https://github.com/dataplat/dbatools/blob/7ad0415c2f8a58d3472c1e85ee431c70f1bb8ae4/private/functions/Get-DecryptedObject.ps1#L7) function within the DBATools module
> [View Highlight](https://read.readwise.io/read/01j9s7a4hxvks3ee4qhnhaxz95)



`Get-MSSQLCredentialPasswords`
> [View Highlight](https://read.readwise.io/read/01j9s7acm07sdz174hxk35t4km)



Antti Rantasaari’s technique is highly effective, but it requires that we already have local administrative privileges on the Windows system hosting the SQL Server instance. Without these administrative privileges, the technique cannot be applied.
> [View Highlight](https://read.readwise.io/read/01j9s7c4hppnwfvxrse18yhesg)



## How can I Abuse SQL Server Credential Objects without Local Administrator Access?
> [View Highlight](https://read.readwise.io/read/01j9s7c9n1pbhp6jnfzvjnj9tj)



we do not need to recover the cleartext usernames and passwords stored in credential objects to run code in another user’s context—we can leverage the functionality as it was designed.
> [View Highlight](https://read.readwise.io/read/01j9s7d2m8nd0hx6hjpaw8jygp)



Below is a process that can be used to “hijack” an existing credential object configured on the SQL Server instance, allowing you to execute code in the provided user’s context using SQL Server Agent jobs. No password or local OS administrator privileges required. 🙂
> [View Highlight](https://read.readwise.io/read/01j9s7ds56k4brxkkgdv20psae)

