---
Description: The whois utility looks up records in the databases maintained by several Network Information Centers (NICs).
---

## Classic lookup

```bash
kali@kali:~$ whois megacorpone.com
	Domain Name: MEGACORPONE.COM
	Registry Domain ID: 1775445745_DOMAIN_COM-VRSN
	Registrar WHOIS Server: whois.gandi.net
	Registrar URL: http://www.gandi.net
	Updated Date: 2019-01-01T09:45:03Z
	Creation Date: 2013-01-22T23:01:00Z
	Registry Expiry Date: 2023-01-22T23:01:00Z
	...
	Registry Registrant ID:
	Registrant Name: Alan Grofield
	Registrant Organization: MegaCorpOne
	Registrant Street: 2 Old Mill St
	Registrant City: Rachel
	Registrant State/Province: Nevada
	Registrant Postal Code: 89001
	Registrant Country: US
	Registrant Phone: +1.9038836342
	...
	Registry Admin ID:
	Admin Name: Alan Grofield
	Admin Organization: MegaCorpOne
	Admin Street: 2 Old Mill St
	Admin City: Rachel
	Admin State/Province: Nevada
	Admin Postal Code: 89001
	Admin Country: US
	Admin Phone: +1.9038836342
	...
	Registry Tech ID:
	Tech Name: Alan Grofield
	Tech Organization: MegaCorpOne
	Tech Street: 2 Old Mill St
	Tech City: Rachel
	Tech State/Province: Nevada
	Tech Postal Code: 89001
	Tech Country: US
	Tech Phone: +1.9038836342
	...
	Name Server: NS1.MEGACORPONE.COM
	Name Server: NS2.MEGACORPONE.COM
	Name Server: NS3.MEGACORPONE.COM
...
```

## Reverse lookup

```bash
kali@kali:~$ whois 38.100.193.70
	...
	NetRange: 38.0.0.0 - 38.255.255.255
	CIDR: 38.0.0.0/8
	NetName: COGENT-A
	...
	OrgName: PSINet, Inc.
	OrgId: PSI
	Address: 2450 N Street NW
	City: Washington
	StateProv: DC
	PostalCode: 20037
	Country: US
	RegDate:
	Updated: 2015-06-04
	...
```