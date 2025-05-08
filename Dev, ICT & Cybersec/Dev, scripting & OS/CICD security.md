>[!question] What is CI/CD?
>In [software engineering](https://en.wikipedia.org/wiki/Software_engineering "Software engineering"), **CI/CD** or **CICD** is the combined practices of [continuous integration](https://en.wikipedia.org/wiki/Continuous_integration "Continuous integration") (CI) and [continuous delivery](https://en.wikipedia.org/wiki/Continuous_delivery "Continuous delivery") (CD) or, less often, [continuous deployment](https://en.wikipedia.org/wiki/Continuous_deployment "Continuous deployment").

# CI/CD Security Risk

CI/CD environments, processes and systems are the beating heart of any modern software organization. They **deliver code from an engineer’s workstation to production**. However, they have also reshaped the attack surface with a multitude of new avenues and opportunities for attackers. Adversaries are shifting their attention to CI/CD, realizing **CI/CD services provide an efficient path to reaching an organization's crown jewels**:
- The compromise of the SolarWinds build system, used to spread malware through to 18,000 customers.
- The PHP breach, resulting in publication of a malicious version of PHP containing a backdoor [^php-dev] .
- Dependency Confusions
- [Supply Chain vulnerabilities](Supply%20Chain.md#Supply%20Chain%20vulnerabilities)

[^php-dev]: [Knife Foothold](../../Play%20ground/CTFs/Knife.md#Foothold)

Currently, an OWASP Top 10 for CI/CD is still under construction, however a raw version [^owasp-cicd] has already been though.  

[^owasp-cicd]: [OWASP Top 10 CI/CD Security Risks by Cider Security](https://owasp.org/www-project-top-10-ci-cd-security-risks/)


# Tools

- [dastardly](https://portswigger.net/burp/dastardly): A free DAST web application scanner for your CI/CD pipeline