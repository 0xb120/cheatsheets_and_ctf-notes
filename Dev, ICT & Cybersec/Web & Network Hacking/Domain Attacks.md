>[!info]
>Domain attacks are all those attacks aimed at stealing an existing domain, spoofing real domains or registering expired sub-domains contained in their main sites.

Domain attacks allows for many different kind of attacks, including [Supply Chain vulnerabilities](../Dev,%20scripting%20&%20OS/Supply%20Chain.md#Supply%20Chain%20vulnerabilities)
## Domain Hijacking

The practice of taking possession of a domain by registering it with other data and then exterminating the legitimate owner.
When talking about domain hijacking, it is essential to know that the change of header can only be carried out if it expires or if ownership is transferred.

**Grace period**: Period of time (30 + 45 days) at the end of which the user can decide to renew the domain lease or let it expire.
It is useful to know that there are bots that search for domains that are in this period in order to keep an eye on them.

### Transferring a Domain

It can occur mainly due to 3 causes:

- Registrar breach: the attacker breaches the services of the Registrar (the company that leases the domains) by taking possession of all the domains
- Account violation: the attacker violates the account associated with the Registrar's services
- Phishing attack: the attacker tricks the user into providing the access data of the account under which the domain is administered

The transfer takes place via a code: the AUTHINFO, provided by the previous registrar and used by the next registrar.

---

## Typosquatting

It is a common practice to buy domains that are similar to the original by exploiting barely visible errors.
Let's take the domain [example.com](http://example.com/) as a reference
We can categorize 7 different types of typosquatting based on the exploitation of errors:

- Poor language skills ([exemple.com](http://exemple.com/))
- Typos ([examlpe.com](http://examlpe.com/))
- Plural variants ([examples.com](http://examples.com/))
- Different TLD ([example.net](http://example.net/))
- Varying characters ([examplle.com](http://examplle.com/))
- Using a sub-domain ([ex.ample.com](http://ex.ample.com/))
- Different TLD with character omission ([example.co](http://example.co/) or [example.cm](http://example.cm/))

In order to find common typosquatting domains it is possible to use [dnstwist.py](https://github.com/elceef/dnstwist) in order to create complex lists of typo domains.

---

## Subdomain takeover

Enumeration of expired sub-domains in order to register them and obtain authority to attack the main domain.
A useful tool in order to find expired sub-domains is [takeover.py](https://github.com/m4ll0k/takeover)

One of the technique used to achieve subdomain takeover is [Dangling DNS](../../Readwise/Articles/Chris%20Hosking%20-%20Re-Assessing%20Risk%20Subdomain%20Takeovers%20as%20Supply%20Chain%20Attacks.md#Dangling%20DNS). 

![Dangling DNS](../../Readwise/Articles/Chris%20Hosking%20-%20Re-Assessing%20Risk%20Subdomain%20Takeovers%20as%20Supply%20Chain%20Attacks.md#Dangling%20DNS)