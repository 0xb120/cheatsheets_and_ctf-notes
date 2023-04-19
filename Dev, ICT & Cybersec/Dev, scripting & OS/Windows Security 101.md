# Understanding Windows Privileges and Integrity Levels

[Privileges](https://docs.microsoft.com/en-gb/windows/win32/secauthz/privileges?redirectedfrom=MSDN) refer to the **permissions of a specific account to perform system-related local operations**.
When a user logs into the system, Windows assign him an [Access Token](https://docs.microsoft.com/en-us/windows/win32/secauthz/access-tokens), which contains various information, included privileges. Finally, these tokens need to be **uniquely identifiable** given the information they contain. This is accomplished **using** a security identifier or **SID**, which is a unique value that is assigned to each object (including tokens), such as a user or group account.

These SIDs are generated and maintained by the Windows Local Security Authority.

In addition to privileges, Windows also implements what is known as an [integrity mechanism](https://docs.microsoft.com/en-us/previous-versions/dotnet/articles/bb625957(v=msdn.10)?redirectedfrom=MSDN).
This is a core component of the Windows security architecture and works by assigning integrity levels to application processes and securable objects. Simply put, this **describes the level of trust the operating system has in running applications or securable objects**.
As an example, the configured integrity level dictates what actions an application can perform, including the ability to read from or write to the local file system. APIs can also be blocked from specific integrity levels.

From Windows Vista onward, processes run on **four integrity levels**:

- `System integrity process`: SYSTEM rights
- `High integrity process`: administrative rights
- `Medium integrity process`: standard user rights
- `Low integrity process`: very restricted rights often used in sandboxed processes

# Introduction to User Account Control (UAC)

[UAC](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-overview) has been introduced with Vista and WinServer 2008.
It forces applications and tasks to **run in the context of a non-administrative account** until an administrator authorizes elevated access. It will block installers and unauthorized applications from running without the permissions of an administrative account and also blocks changes to system settings.

Two different modes:
- **credential prompt**: user have to provide admin credentials to gain the elevated access
- **consent prompt**: user have to consent in order to confirm their elevated access

Even while logged in as an administrative user, the account will have two security tokens, one running at a medium integrity level and the other at high integrity level. UAC acts as the separation mechanism between those two integrity levels.

```powershell
c:\Users\admin>whoami /groups
GROUP INFORMATION
-----------------
Group Name Type SID Attributes
======================================================== ============ ================
Everyone Well-known group S-1-1-0 Mandatory group,
NT AUTHORITY\Local account and member Well-known group S-1-5-114 Group used for d
BUILTIN\Administrators Alias S-1-5-32-544 Group used for d
...
Mandatory Label\Medium Mandatory Level Label S-1-16-8192

C:\Users\admin>powershell.exe Start-Process cmd.exe -Verb runAs
C:\Windows\system32> whoami /groups
GROUP INFORMATION
-----------------
Group Name Type SID Attributes
======================================================== ============ ================
Everyone Well-known group S-1-1-0 Mandatory group,
NT AUTHORITY\Local account and member Well-known group S-1-5-114 Mandatory group,
BUILTIN\Administrators Alias S-1-5-32-544 Mandatory group,
BUILTIN\Users Alias S-1-5-32-545 Mandatory group,
...
Mandatory Label\High Mandatory Level Label S-1-16-12288
```

# Access Control List (ACL)

Permissions to access a certain resource in Windows are controlled by the **access control list** (ACL) for that resource. Each ACL is made up of zero or more **access control entries** (ACEs). Each ACE defines the relationship between a principal (user, group, etc.) and a certain access right to a specific service, file o registry entry.

# Windows Registry

The Windows Registry is a [hierarchical database](https://docs.microsoft.com/en-us/windows/win32/sysinfo/structure-of-the-registry) that stores critical information for the operating system and for applications that choose to use it. The registry stores settings, options, and other miscellaneous information in a hierarchical [tree structure](https://docs.microsoft.com/en-us/windows/win32/sysinfo/structure-of-the-registry) of hives, keys, sub-keys, and values.

---

# Windows Hash

Hashed password are stored inside the **Security Account Manager** ([SAM](https://en.wikipedia.org/wiki/Security_Account_Manager)).
To deter offline SAM database password attacks, Microsoft introduced the SYSKEY feature (Windows NT 4.0 SP3), which partially encrypts the SAM file.

SAM database cannot be copied while the operating system is running because the Windows kernel keeps an exclusive file system lock on the file.

## Supported hash

Windows NT-based operating systems **<= Windows 2003** store two different password hashed:

- **LAN Manager** (LM): based on DES, it is known to be very weak since **passwords longer than seven characters** are **split into two strings** and each piece is hashed separately. Each password string is also **converted to upper-case** before being hashed and, moreover, the LM hashing system **does not include salts**.
- **NT LAN Manger** (NTLM): uses MD4, is case sensitive, supports all Unicode characters, and does not split the hash into smaller, weaker parts. However, NTLM hashes stored in the SAM database are still not salted.

**From Windows Vista**, LM has been disabled by default and only NTLM is used.