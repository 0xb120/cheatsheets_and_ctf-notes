# What is a WAF
A Web Application Firewall (WAF) is a security tool designed to protect web applications from various attacks. Usually, a WAF is placed in front of web servers or applications to analyze all incoming HTTP/HTTPS traffic, blocking requests that match predefined security rules. 

WAFs can be configured to allow only specific types of traffic based on criteria such as IP address, user agent, and other request attributes. They employ techniques like signature-based detection, anomaly detection, and behavioral analysis to identify and prevent attacks. Additionally, WAFs may offer extra security features such as encryption and authentication.[^WAF]

[^WAF]: [What is WAF (Web Application Firewall), And How Do You Bypass It?](https://blog.securelayer7.net/web-application-firewall/), securelayer7.net

WAFs can operate in three different modes:
- **Blocklist/ Blacklist**
- **Allowlist/ Whitelist**
- **Hybrid**

## Types of Web Application Firewall Implementation

### Network-based WAF

A network-based WAF involves placing the WAF between the client and the server to monitor all traffic. This type is generally hardware-based and installed locally, which minimizes latency and provides comprehensive protection against web application threats. However, network-based WAFs are the most expensive option and require the storage and maintenance of physical equipment. Additionally, they may introduce higher latency and necessitate a more complex network infrastructure.

### Hosted-based WAF

A host-based WAF involves installing the WAF software directly on the web server, where it operates as a module within the operating system or as an extension to the web server software. This approach offers more customizability and granular control over web traffic to the server and is less expensive than network-based WAFs. However, host-based WAFs consume local server resources, have limited scalability, and involve implementation complexity. They also require significant maintenance and engineering time.

### Cloud-based WAF

A cloud-based WAF is deployed as a service in the cloud, typically offered by third-party providers such as Amazon Web Services (AWS) or Microsoft Azure. Cloud-based WAFs provide high scalability, flexibility, and reduced maintenance. They are affordable with minimal upfront costs and easy to implement, often requiring just a DNS change. These WAFs are also regularly updated to protect against new threats without additional work or cost. The main drawbacks are higher ongoing costs, reliance on the third-party provider, and potentially limited visibility into the WAF’s operations, as it may function as a “black box.”

Choosing the right WAF implementation depends on the organization’s specific needs and security requirements, as well as factors such as budget, existing infrastructure, and staff resources. Each type of WAF offers unique advantages and trade-offs, so understanding these can help in selecting the most suitable solution.


# WAF limitations

## Documented WAF Limitations

| WAF Provider          | Maximum Request Body Inspection Size Limit             |
| --------------------- | ------------------------------------------------------ |
| Cloudflare            | 128 KB for ruleset engine, up to 500 MB for enterprise |
| AWS WAF               | 8 KB - 64 KB (configurable depending on service)       |
| Akamai                | 8 KB - 128 KB                                          |
| Azure WAF             | 128 KB                                                 |
| Fortiweb by Fortinet  | 100 MB                                                 |
| Barracuda WAF         | 64 KB                                                  |
| Sucuri                | 10 MB                                                  |
| Radware AppWall       | up to 1 GB for cloud WAF                               |
| F5 BIG-IP WAAP        | 20 MB (configurable)                                   |
| Palo Alto             | 10 MB                                                  |
| Cloud Armor by Google | 8 KB (can be increased to 128 KB)                      |
# Bypass WAFs

- [When WAFs Go Awry: Common Detection & Evasion Techniques for Web Application Firewalls](../../Readwise/Articles/Admin%20-%20When%20WAFs%20Go%20Awry%20Common%20Detection%20&%20Evasion%20Techniques%20for%20Web%20Application%20Firewalls.md)
- [Evading Restrictions](../Web%20&%20Network%20Hacking/Evading%20Restrictions.md)
- [PortSwigger Research - Bypassing WAFs With the Phantom $Version Cookie](../../Readwise/Articles/PortSwigger%20Research%20-%20Bypassing%20WAFs%20With%20the%20Phantom%20$Version%20Cookie.md)