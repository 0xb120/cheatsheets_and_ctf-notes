>[!tip] cloud_metadata.txt
>[https://gist.github.com/jhaddix/78cece26c91c6263653f31ba453e273b](https://gist.github.com/jhaddix/78cece26c91c6263653f31ba453e273b)

# AWS 101

## Cognito

>[!question] What is Cognito?
>AWS Cognito manages user authentication and authorization for client applications, usually mobile or web. 

It is made up by three different entities:

- **Client**: is the app the user registers for and logs in to
- **User Pool**: is a user directory that manages user registration and login
- **Identity Pool**: It gives temporary AWS credentials to end users, allowing direct interaction with the AWS account.

### Common security risks

Enumeration and footholds [^enum] :
- Cognito “client” and “user pool” IDs are shown in HTTP responses, allowing for enumeration.  
- Cognito user registration is left open, allowing an attacker to register using the IDs from step 1, even if no login portal exists in the web or mobile app.  
- The “identity pool” ID is shown in HTTP responses, allowing an attacker to obtain IAM credentials

Privilege escalation [^privesc] :
- Custom attributes that are user-modifiable are used for role-based access control (RBAC). This lets an attacker gain privileges by updating attributes.  
- Standard user-modifiable attributes such as “email” are _case-sensitive_, accept changes immediately, and perform RBAC. This lets an attacker take over an account by mimicking its “email” if the back end _is not case-sensitive_. [^report]
- The user’s “id” token lists extra IAM roles that can be assumed by the user via the API, allowing an attacker to obtain more permissions and pivot through the AWS account.

[^enum]: [David Kutz-Marks - Attacking AWS Cognito With Pacu](../../Readwise/Articles/David%20Kutz-Marks%20-%20Attacking%20AWS%20Cognito%20With%20Pacu.md#^2fe305), id608778543
[^privesc]: [David Kutz-Marks - Attacking AWS Cognito With Pacu](../../Readwise/Articles/David%20Kutz-Marks%20-%20Attacking%20AWS%20Cognito%20With%20Pacu.md#^fae561), id608778569
[^report]: [Flickr Account Takeover using AWS Cognito API](https://hackerone.com/reports/1342088), hackerone.com
# External tool & References

- [AWS penetration testing: a step-by-step guide](https://www.hackthebox.com/blog/aws-pentesting-guide)
- [Query AWS IAM permission policies with ease](https://rhinosecuritylabs.com/aws/iamactionhunter-aws-iam-permissions/)
- [Pacu](https://github.com/RhinoSecurityLabs/pacu) [^pacu] [^pacu-2]

[^pacu]: [Attacking AWS Cognito With Pacu (p1)](https://rhinosecuritylabs.com/aws/attacking-aws-cognito-with-pacu-p1/)
[^pacu-2]: [Attacking AWS Cognito with Pacu (p2)](https://rhinosecuritylabs.com/aws/attacking-aws-cognito-with-pacu-p2/)