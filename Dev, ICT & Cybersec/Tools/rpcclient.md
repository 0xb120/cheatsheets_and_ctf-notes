---
Description: It’s a tool for executing client side MS-RPC functions
---

>[!info]
>`rpcclient` is a utility initially developed to test MS-RPC functionality in Samba itself.  It has undergone several stages of development and stability. Many system administrators have now written scripts around it to manage Windows NT clients from their UNIX workstation.

```bash
┌──(kali㉿kali)-[~/Documents/lab]
└─$ rpcclient
Usage: rpcclient [OPTION...] <server>
Options:
  -c, --command=COMMANDS                 Execute semicolon separated cmds
  -I, --dest-ip=IP                       Specify destination IP address
  -p, --port=PORT                        Specify port number

Help options:
  -?, --help                             Show this help message
      --usage                            Display brief usage message

Common samba options:
  -d, --debuglevel=DEBUGLEVEL            Set debug level
  -s, --configfile=CONFIGFILE            Use alternate configuration file
  -l, --log-basename=LOGFILEBASE         Base name for log files
  -V, --version                          Print version
      --option=name=value                Set smb.conf option from command line

Connection options:
  -O, --socket-options=SOCKETOPTIONS     socket options to use
  -n, --netbiosname=NETBIOSNAME          Primary netbios name
  -W, --workgroup=WORKGROUP              Set the workgroup name
  -i, --scope=SCOPE                      Use this Netbios scope

Authentication options:
  -U, --user=USERNAME                    Set the network username
  -N, --no-pass                          Don't ask for a password
  -k, --kerberos                         Use kerberos (active directory) authentication
  -A, --authentication-file=FILE         Get the credentials from a file
  -S, --signing=on|off|required          Set the client signing state
  -P, --machine-pass                     Use stored machine account password
  -e, --encrypt                          Encrypt SMB transport
  -C, --use-ccache                       Use the winbind ccache for authentication
      --pw-nt-hash                       The supplied password is the NT hash
```

### RPC Null Sessions

```bash
root@kali:~# rpcclient -U "" 192.168.1.99
Enter's password:                           # void password 
rpcclient $> srvinfo                        # info sul server
rpcclient $> enumdomusers                   # enum utenti
rpcclient $> getdompwinfo                   # info sulla password
```

### Obtain information on the target

```bash
# Connect to the rpc
rpcclient -U "" -N <IP> #No creds
rpcclient //machine.htb -U domain.local/USERNAME%754d87d42adabcca32bdb34a876cbffb  --pw-nt-hash
# You can use querydispinfo and enumdomusers to query user information
```