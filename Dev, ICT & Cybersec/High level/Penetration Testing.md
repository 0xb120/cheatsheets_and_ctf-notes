---
aliases:
  - Pentesting
---
# What is penetration testing

A penetration test, colloquially known as a pentest, is an authorized simulated cyberattack on a computer system, performed to evaluate the security of the system; this is not to be confused with a vulnerability assessment. The test is performed to identify weaknesses (or vulnerabilities), including the potential for unauthorized parties to gain access to the system's features and data, as well as strengths, enabling a full risk assessment to be completed.

## Gray Vs Black Vs White Box Pentesting

Gray Box Penetration Testing combines elements of both Black Box and White Box testing. Testers have partial knowledge of the system, enabling them to focus on potential weak points while still simulating realistic attack scenarios. This approach strikes a balance between external and internal perspectives, leading to a more comprehensive security assessment.

1. **Gray Box Penetration Testing:** Gray box penetration testing is a balanced approach where testers have partial knowledge of the internal workings of the system. This method combines the strengths of both black box and white box testing. It provides a realistic simulation of an attack scenario with some insider knowledge, making the process efficient and effective by focusing on high-risk areas.
2. **Black Box Penetration Testing:** Black box penetration testing involves testers with no prior knowledge of the system’s internal structure. This method simulates an external attack, providing an unbiased assessment of the system’s vulnerabilities from an outsider’s perspective. While it closely mimics real-world attack scenarios, it can be time-consuming and might miss internal vulnerabilities.
3. **White Box Penetration Testing:** White box penetration testing gives testers full access to the system’s internal information, including source code and architecture. This method allows for a comprehensive and thorough analysis, uncovering deep-seated vulnerabilities. It may lack realism in simulating external attacks and can introduce bias due to the extensive knowledge testers have.

**Detailed Comparison Table**

| **Aspect**        | **Black Box Testing**                                     | **Gray Box Testing**                                       | **White Box Testing**                                    |
| ----------------- | --------------------------------------------------------- | ---------------------------------------------------------- | -------------------------------------------------------- |
| Knowledge Level   | No prior knowledge of the system                          | Partial knowledge of the system                            | Full knowledge of the system                             |
| Perspective       | External (outsider)                                       | Hybrid (partial insider and outsider)                      | Internal (insider)                                       |
| Approach          | Simulates an external attack                              | Combines internal focus with external                      | Thorough internal analysis                               |
| Scope of Testing  | Limited to external attack vectors                        | Focused on critical areas informed by partial knowledge    | Comprehensive coverage, including internal workings      |
| Efficiency        | Can be time-consuming due to lack of information          | More efficient due to targeted testing                     | Efficient, but may be biased                             |
| Realism           | High, mimicking real-world external attacks               | Balanced, simulating partial insider attacks               | Low, less representative of external attacks             |
| Thoroughness      | May miss internal vulnerabilities                         | Uncovers both internal and external vulnerabilities        | Very thorough, covering all aspects                      |
| Typical Use Cases | External vulnerability assessments, regulatory compliance | Complex systems, sensitive data, potential insider threats | Internal audits, code reviews, development stage testing |
| Advantages        | Realistic external perspective, unbiased                  | Balanced view, efficient, comprehensive                    | Detailed, thorough, uncovering hidden flaws              |
|Disadvantages|May overlook internal issues, time-consuming|Dependent on level of partial knowledge|Potential bias, less realistic external scenario|

# How much does cost a Penetration Test activity

Generally, most application penetration tests average around 5 days, although they can vary from two days to six weeks for a single application [^1]. Hourly rates vary, $200 to $300 per hour is generally what most penetration testing firms charge.  

Given an average project of 40 hours that is $250 per hour, that multiplies out to roughly a $10,000 price tag. I would estimate that 90% of application assessments are performed for between $4,000 and $30,000 [^pt-cost] . 

[^pt-cost]: [Preparing for an Application Penetration Test](../../Readwise/Articles/Alex%20Lauerman%20-%20Preparing%20for%20an%20Application%20Penetration%20Test.md); Alex Lauerman



# External Resources

- [A Brief Guide to Black Box Penetration Testing](https://blog.securelayer7.net/black-box-penetration-testing/) 
- [Gray Box Penetration Testing: The Essential Guide](https://blog.securelayer7.net/gray-box-penetration-testing/)
- [All You Need to Know about White Box Penetration Testing](https://blog.securelayer7.net/white-box-penetration-testing/)

[^1]: [How Would You Scope WordPress?](../../Readwise/Articles/Tib3rius%20-%20How%20Would%20You%20Scope%20WordPress.md); Tib3rius
