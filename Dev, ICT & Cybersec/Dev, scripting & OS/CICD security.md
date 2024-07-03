>[!question] What is CI/CD?
>In [software engineering](https://en.wikipedia.org/wiki/Software_engineering "Software engineering"), **CI/CD** or **CICD** is the combined practices of [continuous integration](https://en.wikipedia.org/wiki/Continuous_integration "Continuous integration") (CI) and [continuous delivery](https://en.wikipedia.org/wiki/Continuous_delivery "Continuous delivery") (CD) or, less often, [continuous deployment](https://en.wikipedia.org/wiki/Continuous_deployment "Continuous deployment").

# CI/CD Security Risk

CI/CD environments, processes and systems are the beating heart of any modern software organization. They **deliver code from an engineer’s workstation to production**. However, they have also reshaped the attack surface with a multitude of new avenues and opportunities for attackers. Adversaries are shifting their attention to CI/CD, realizing **CI/CD services provide an efficient path to reaching an organization's crown jewels**:
- The compromise of the SolarWinds build system, used to spread malware through to 18,000 customers.
- The PHP breach, resulting in publication of a malicious version of PHP containing a backdoor [^php-dev] .

[^php-dev]: [Knife Foothold](../../Play%20ground/CTFs/Knife.md#Foothold)

Currently, an OWASP Top 10 for CI/CD is still under construction, however a raw version [^owasp-cicd] has already been though.  

[^owasp-cicd]: [OWASP Top 10 CI/CD Security Risks by Cider Security](https://owasp.org/www-project-top-10-ci-cd-security-risks/)

## CICD-SEC-1: Insufficient Flow Control Mechanisms

>[!summary]
>Insufficient flow control mechanisms refer to the **ability of an attacker** that has obtained permissions to a system within the CI/CD process (SCM, CI, Artifact repository, etc.) **to single handedly push malicious code or artifacts down the pipeline, due to a lack in mechanisms that enforce additional approval or review**.

An attacker with access to the SCM, CI, or systems further down the pipeline, can abuse insufficient flow control mechanisms to deploy malicious artifacts. Once created, the artifacts are shipped through the pipeline - potentially all the way to production - without any approval or review.  [^cicd1]

[^cicd1]: [CICD-SEC-1: Insufficient Flow Control Mechanisms](https://owasp.org/www-project-top-10-ci-cd-security-risks/CICD-SEC-01-Insufficient-Flow-Control-Mechanisms), owasp.org

## CICD-SEC-2: Inadequate Identity and Access Management

>[!summary]
>Inadequate Identity and Access Management risks stem from the difficulties in managing the vast amount of identities spread across the different systems in the engineering ecosystem, from source control to deployment.

Each system provides multiple methods of access and integration. The different types of accounts and method of access can potentially have their own unique provisioning method, set of security policies and authorization model. This complexity creates challenges in managing the different identities throughout the entire identity lifecycle and ensuring their permissions are aligned with the principle of least privilege. 

Some of the major concerns and challenges around identity and access management within the CI/CD ecosystem include:
- **Overly permissive identities**
- **Stale identities**
- **Local identities**
- **External identities**
- **Self-registered identities**
- **Shared identities**

The existence of hundreds (or sometimes thousands) of identities - both human and programmatic - across the CI/CD ecosystem, paired with a lack of strong identity and access management practices and common usage of overly permissive accounts, leads to a state where compromising nearly any user account on any system, could grant powerful capabilities to the environment, and could serve as a segue into the production environment. [^cicd2]

[^cicd2]: [CICD-SEC-2: Inadequate Identity and Access Management](https://owasp.org/www-project-top-10-ci-cd-security-risks/CICD-SEC-02-Inadequate-Identity-And-Access-Management), owasp.org
## CICD-SEC-3: Dependency Chain Abuse

>[!summary]
>Dependency chain abuse risks refer to an attacker’s ability to abuse flaws relating to how engineering workstations and build environments fetch code dependencies. Dependency chain abuse results in a malicious package inadvertently being fetched and executed locally when pulled.

in the context of using dependencies, there is an equally important set of controls required to secure the dependency ecosystem - involving securing the process defining how dependencies are pulled. Inadequate configurations may cause an unsuspecting engineer, or worse - the build system, to download a malicious package instead of the package that was intended to be pulled. In many cases, the package is not only downloaded, but also immediately executed after download, due to pre-install scripts and similar processes which are designed to run a package’s code immediately after the package is pulled.

The main attack vectors in this context are:
- **Dependency confusion**
- **Dependency hijacking**
- **Typosquatting**
- **Brandjacking**

The objective of adversaries which upload packages to public package repositories using one of the aforementioned techniques is to execute malicious code on a host pulling the package. This could either be a developer’s workstation, or a build server pulling the package. [^cicd3]

[^cicd3]: [CICD-SEC-3: Dependency Chain Abuse](https://owasp.org/www-project-top-10-ci-cd-security-risks/CICD-SEC-03-Dependency-Chain-Abuse), owasp.org
## CICD-SEC-4: Poisoned Pipeline Execution

>[!summary]
>Poisoned Pipeline Execution (PPE) risks refer to the ability of an attacker with access to source control systems - and without access to the build environment, to manipulate the build process by injecting malicious code/commands into the build pipeline configuration, essentially ‘poisoning’ the pipeline and running malicious code as part of the build process.

The PPE vector abuses permissions against an SCM repository, in a way that causes a CI pipeline to execute malicious commands. Users that have permissions to manipulate the CI configuration files, or other files which the CI pipeline job relies on, can modify them to contain malicious commands, ultimately “poisoning” the CI pipeline executing these commands. 

There are three types of PPE:
- **Direct PPE (D-PPE)**: the attacker modifies the CI config file in a repository they have access to, either by pushing the change directly to an unprotected remote branch on the repo, or by submitting a PR with the change from a branch or a fork.
  >[!example]
  >[White Rabbit 100](../../Play%20ground/CTFs/White%20Rabbit%20100.md)
- **Indirect PPE (I-PPE):** when the CI configuration file is protected, the attacker can still poison the pipeline by injecting malicious code into files referenced by the pipeline configuration file (eg. the `Makefile` if the script uses `make`, scripts referenced by the CI and stored in the compromised SCM, etc.)
  >[!example]
  >[Mad Hatter 100](../../Play%20ground/CTFs/Mad%20Hatter%20100.md)
- **Public-PPE (3PE):** in some cases poisoning CI pipelines is available to anonymous attackers on the internet: Public repositories oftentimes allow any user to contribute. If the CI pipeline of a public repository runs unreviewed code suggested by anonymous users, it is susceptible to a Public PPE

In a successful PPE attack, attackers execute malicious unreviewed code in the CI. This provides the attacker with the same abilities and level of access as the build job. [^cicd4]

[^cicd4]: [CICD-SEC-4: Poisoned Pipeline Execution (PPE)](https://owasp.org/www-project-top-10-ci-cd-security-risks/CICD-SEC-04-Poisoned-Pipeline-Execution), owasp.org

## CICD-SEC-5: Insufficient PBAC (Pipeline-Based Access Control)

>[!summary]
>Pipeline execution nodes have access to numerous resources and systems within and outside the execution environment. When running malicious code within a pipeline, adversaries leverage insufficient Pipeline-Based Access Controls risks to abuse the permission granted to the pipeline for moving laterally within or outside the CI/CD system.

PBAC is a term which refers to the context in which each pipeline - and each step within that pipeline - is running. Given the highly sensitive and critical nature of each pipeline, it is imperative to limit each pipeline to the exact set of data and resources it needs access to.

PBAC includes controls relating to numerous elements having to do with the pipeline execution environment:
- Access within the pipeline execution environment: to code, secrets, environment variables, and other pipelines.
- Permissions to the underlying host and other pipeline nodes.
- Ingress and egress filters to the internet.

A piece of malicious code that is able to run in the context of the pipeline execution node has the full permissions of the pipeline stage it runs in. It can access secrets, access the underlying host and connect to any of the systems the pipeline in question has access to. This can lead to exposure of confidential data, lateral movement within the CI environment - potentially accessing servers and systems outside the CI environment, and deployment of malicious artifacts down the pipeline, including to production. [^cicd5]

[^cicd5]: [CICD-SEC-5: Insufficient PBAC (Pipeline-Based Access Controls)](https://owasp.org/www-project-top-10-ci-cd-security-risks/CICD-SEC-05-Insufficient-PBAC), owasp.org
## CICD-SEC-6: Insufficient Credential Hygiene

>[!summary]
>Insufficient credential hygiene risks deal with an attacker’s ability to obtain and use various secrets and tokens spread throughout the pipeline due to flaws having to do with access controls around the credentials, insecure secret management and overly permissive credentials.

CI/CD environments are built of multiple systems communicating and authenticating against each other, creating great challenges around protecting credentials due to the large variety of contexts in which credentials can exist. This variety of contexts, paired with the large amount of methods and techniques for storing and using them, creates a large potential for insecure usage of credentials. 

Some major flaws that affect credential hygiene: [^cicd6]
- **Code containing credentials being pushed to one of the branches of an SCM repository**
- **Credentials used insecurely inside the build and deployment processes**
- **Credentials in container image layers**
- **Credentials printed to console output**
- **Unrotated credentials**

[^cicd6]: [CICD-SEC-6: Insufficient Credential Hygiene](https://owasp.org/www-project-top-10-ci-cd-security-risks/CICD-SEC-06-Insufficient-Credential-Hygiene), owasp.org
## CICD-SEC-7: Insecure System Configuration

>[!summary]
>Insecure system configuration risks stem from flaws in the security settings, configuration and hardening of the different systems across the pipeline

In a similar way to other systems storing and processing data, CI/CD systems involve various security settings and configurations on all levels. These settings have a major influence on the security posture of the CI/CD environments and the susceptibility to a potential compromise. [^cicd7]

Examples of potential hardening flaws:
- A self-managed system and/or **component using an outdated version** or** lacking important security patches**.
- A system having **overly permissive network access controls**.
- A self-hosted **system that has administrative permissions on the underlying OS**.
- A system with **insecure system configurations** (authorization, access controls, logging and more).
- A system with **inadequate credential hygiene** (default credentials which are not disabled, overly permissive programmatic tokens, and more).

[^cicd7]: [CICD-SEC-7: Insecure System Configuration](https://owasp.org/www-project-top-10-ci-cd-security-risks/CICD-SEC-07-Insecure-System-Configuration), owasp.org
## CICD-SEC-8: Ungoverned Usage of 3rd Party Services

>[!summary]
>The CI/CD attack surface consists of an organization’s organic assets, such as the SCM or CI, and the 3rd party services which are granted access to those organic assets. Risks having to do with ungoverned usage of 3rd party services rely on the extreme ease with which a 3rd party service can be granted access to resources in CI/CD systems, effectively expanding the attack surface of the organization.

Their ease of implementation, combined with their immediate value, has made 3rd parties an integral part of the engineering day-to-day. Taking a common SCM - GitHub SAAS - as en example, 3rd party applications can be connected through one or more of these 5 methods:
- GitHub Application
- OAuth application
- Provisioning of an access token provided to the 3rd party application
- Provisioning of an SSH key provided to the 3rd party application.
- Configuring webhook events to be sent to the 3rd party.

Lack of governance and visibility around 3rd party implementations prevents organizations from maintaining RBAC within their CI/CD systems. Given how permissive 3rd parties tend to be, organizations are only as secure as the 3rd parties they implement. Insufficient implementation of RBAC and least privilege around 3rd parties, coupled with minimal governance and diligence around the process of 3rd party implementations create a significant increase of the organization’s attack surface. [^cicd8]

[^cicd8]: [CICD-SEC-8: Ungoverned Usage of 3rd Party Services](https://owasp.org/www-project-top-10-ci-cd-security-risks/CICD-SEC-08-Ungoverned-Usage-of-3rd-Party-Services), owasp.org

## CICD-SEC-9: Improper Artifact Integrity Validation

>[!summary]
>Improper artifact integrity validation risks allow an attacker with access to one of the systems in the CI/CD process to push malicious (although seemingly benign) code or artifacts down the pipeline, due to insufficient mechanisms for ensuring the validation of code and artifacts.

If a tampered resource was able to successfully infiltrate the delivery process, without raising any suspicion or encountering any security gates - it will most likely continue flowing through the pipeline - all the way to production - in the guise of a legitimate resource.

Improper artifact integrity validation can be abused by an adversary with a foothold within the software delivery process to ship a malicious artifact through the pipeline, ultimately resulting in the execution of malicious code - either on systems within the CI/CD process or worse - in production. [^cicd9]

[^cicd9]: [CICD-SEC-9: Improper Artifact Integrity Validation](https://owasp.org/www-project-top-10-ci-cd-security-risks/CICD-SEC-09-Improper-Artifact-Integrity-Validation), owasp.org

## CICD-SEC-10: Insufficient Logging and Visibility

>[!summary]
>Insufficient logging and visibility risks allow an adversary to carry out malicious activities within the CI/CD environment without being detected during any phase of the attack kill chain, including identifying the attacker’s TTPs (Techniques, Tactics and Procedures) as part of any post-incident investigation.

Given the amount of potential attack vectors leveraging engineering environments and processes it is imperative that security teams build the appropriate capabilities to detect these attacks as soon as they happen. As many of these vectors involve leveraging programmatic access against the different systems, a key aspect of facing this challenge is to create strong levels of visibility around both human and programmatic access. [^cicd10]

[^cicd10]: [CICD-SEC-10: Insufficient Logging and Visibility](https://owasp.org/www-project-top-10-ci-cd-security-risks/CICD-SEC-10-Insufficient-Logging-And-Visibility), owasp.org
