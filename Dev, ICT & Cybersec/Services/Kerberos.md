---
Ports: 88
Description: Kerberos is an authentication protocol, not authorization. In other words, it allows to identify each user, who provides a secret password, however, it does not validates to which resources or services can this user access. Kerberos is used in Active Directory.
---

>[!info]
> Firstly, Kerberos is an **authentication protocol**, not authorization. In other words, it allows to identify each user, who provides a secret password, however, it does not validates to which resources or services can this user access. **Kerberos is used in Active Directory**. In this platform, Kerberos provides information about the privileges of each user, but it is responsibility of each service to determine if the user has access to its resources.

---

# Enumeration

```bash
$ nmap -p 88 --script=krb5-enum-users --script-args krb5-enum-users.realm="{Domain_Name}",userdb={Big_Userlist} {IP}
```

```bash
Protocol_Name: Kerberos    #Protocol Abbreviation if there is one.
Port_Number:  88   #Comma separated if there is more than one.
Protocol_Description: AD Domain Authentication         #Protocol Abbreviation Spelled out

Entry_1:
  Name: Notes
  Description: Notes for Kerberos
  Note: |
    Firstly, Kerberos is an authentication protocol, not authorization. In other words, it allows to identify each user, who provides a secret password, however, it does not validates to which resources or services can this user access.
    Kerberos is used in Active Directory. In this platform, Kerberos provides information about the privileges of each user, but it is responsability of each service to determine if the user has access to its resources.

    https://book.hacktricks.xyz/pentesting/pentesting-kerberos-88

Entry_2:
  Name: Pre-Creds
  Description: Brute Force to get Usernames
  Command: nmap -p 88 --script=krb5-enum-users --script-args krb5-enum-users.realm="{Domain_Name}",userdb={Big_Userlist} {IP}

Entry_3:
  Name: With Usernames
  Description: Brute Force with Usernames and Passwords
  Note: consider git clonehttps://github.com/ropnop/kerbrute.git ./kerbrute -h

Entry_4:
  Name: With Creds
  Description: Attempt to get a list of user service principal names
  Command: GetUserSPNs.py -request -dc-ip {IP} active.htb/svc_tgs
```