---
Description: Active Directory information dumper via LDAP
URL: https://github.com/dirkjanm/ldapdomaindump
---

>[!info]
>Install: `pip3 install ldapdomaindump`

## Usage

```bash
┌──(kali㉿kali)-[~/CTFs/HTB/box/Active]
└─$ ldapdomaindump
usage: ldapdomaindump [-h] [-u USERNAME] [-p PASSWORD] [-at {NTLM,SIMPLE}]
                      [-o DIRECTORY] [--no-html] [--no-json] [--no-grep]
                      [--grouped-json] [-d DELIMITER] [-r] [-n DNS_SERVER]
                      [-m]
                      HOSTNAME
```

### Dump the entire AD

```bash
ldapdomaindump <IP> [-r <IP>] -u '<domain>\<username>' -p '<password>' [--authtype SIMPLE] --no-json --no-grep [-o /path/dir]
$ ldapdomaindump 10.10.10.100 -u 'active.htb\SVC_TGS' -p 'GPPstillStandingStrong2k18' --no-json --no-grep -o LDAP

$ ls LDAP
domain_computers_by_os.html  domain_computers.html  domain_groups.html  domain_policy.html  domain_trusts.html  domain_users_by_group.html  domain_users.html
```