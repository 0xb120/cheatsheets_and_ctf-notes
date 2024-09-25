---
author: David Kutz-Marks
aliases:
  - Attacking AWS Cognito With Pacu
tags:
  - readwise/articles
url: https://rhinosecuritylabs.com/aws/attacking-aws-cognito-with-pacu-p1/
date: 2023-10-11
---
# Attacking AWS Cognito With Pacu

![rw-book-cover](https://rhinosecuritylabs.com/wp-content/themes/rhino-v1.2/img/favicons/favicon-16x16.png)

## Highlights

### id608777811

> [Pacu](https://github.com/RhinoSecurityLabs/pacu), our open-source AWS exploitation framework
> [View Highlight](https://read.readwise.io/read/01hcf3b4fcrfa8nqtqbmf7ezmc)
> #tools 

### id608777957

> AWS Cognito manages user authentication and authorization for client applications, usually mobile or web
> [View Highlight](https://read.readwise.io/read/01hcf3ec5n1wkc9v2bmc04yaeq)


### id608778223

> The “client,” “user pool” and “identity pool” have their own associated “ID” that can be used for direct API calls
> [View Highlight](https://read.readwise.io/read/01hcf3mv1q2ftddmg9ybkz5awv)


### id608778543

^2fe305

> • Cognito “client” and “user pool” IDs (e.g. “59f6tuhfmt5lq2mqrnvl8l00sx” and “us_east_2_0b0RfnML2”) are shown in HTTP responses, allowing for enumeration.
> • Cognito user registration is left open (the default setting), allowing an attacker to register using the IDs from step 1, even if no login portal exists in the web or mobile app.
> • The “identity pool” ID (e.g. “us-east-2:a4531178-c987-4abc-9a22-9dm4dlmc2ee5”) is shown in HTTP responses, allowing an attacker to obtain IAM credentials
> [View Highlight](https://read.readwise.io/read/01hcf3pn13mmz8aekjtkfet5xx)


### id608778569

^fae561

> • Custom attributes that are user-modifiable (the default setting) are used for role-based access control (RBAC). This lets an attacker gain privileges by updating attributes.
> • Standard user-modifiable attributes such as “email” are *case-sensitive*, accept changes immediately, and perform RBAC. This lets an attacker take over an account by mimicking its “email” if the back end *is not case-sensitive*. See this report: [https://hackerone.com/reports/1342088](https://hackerone.com/reports/1342088)
> • The user’s “id” token lists extra IAM roles that can be assumed by the user via the API, allowing an attacker to obtain more permissions and pivot through the AWS account.
> [View Highlight](https://read.readwise.io/read/01hcf3qtma9sfe4a222nv09m7d)

