>[!important] Authorization vs Authentication
>Authentication is the process of verifying that a user really is who they claim to be, whereas authorization involves verifying whether a user is allowed to do something.

# What is Access Control

Access control (or authorization) is the application of constraints (using [Access control security models](Access%20control%20security%20models.md)) on who (or what) can perform attempted actions or access resources that they have requested. Design and management of access controls is a complex and dynamic problem that applies business, organizational, and legal constraints to a technical implementation, and those decisions have to be made by humans, not technology, so the potential for errors is higher.

Access controls can be divided into 3 different categories:

- **Vertical access controls**: mechanisms that restrict access to sensitive functionality that is not available to other types of users. With vertical access controls, different types of users have access to different application functions.

- **Horizontal access controls**: mechanisms that restrict access to resources to the users who are specifically allowed to access those resources. With horizontal access controls, different users have access to a subset of resources of the same type.

- **Context-dependent access controls**: restrict access to functionality and resources based upon the state of the application or the user's interaction with it. Context-dependent access controls prevent a user performing actions in the wrong order.

# Broken Access Control examples

Broken access control vulnerabilities exist when a user can in fact access some resource or perform some action that they are not supposed to be able to access.

>[!warning]
>This attacks can lead to the complete takeover of the application or full account takeover of any user

## Vertical privilege escalation

If a user can gain access to functionality that they are not permitted to access then this is vertical privilege escalation.

Common attack vectors are:
- Unprotected pages or functionality
- Parameter-based access control (cookies, user fields, query string, guessed parameters, etc.)
>[!info]-
>Also UUID and similar non-predictable IDs can be a vector if they get [disclosed](Information%20Disclosure.md) or retrieved using [Execution After Redirect (EAR)](Execution%20After%20Redirect%20(EAR).md)
- [Insecure Direct Object Reference](Access%20control%20vulnerabilities.md#Insecure%20Direct%20Object%20Reference)
- Platform misconfigurations and custom headers (`X-Original-URL`, `X-Rewrite-URL`, switching GET/POST and parameter positions, [HTTP Verb Tampering](HTTP%20Verb%20Tampering.md), etc.)

*Custom headers*:
```HTTP
GET /?username=carlos HTTP/1.1
Host: 0a8900f70456c693c4cfdc7f00db00ee.web-security-academy.netUpgrade-Insecure-Requests: 1
X-Original-URL: /admin/delete
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Connection: close
```

*Switching HTTP methods*:
```http
POST /admin-roles HTTP/1.1
Host: 0a64002703929c3fc08c45b800460050.web-security-academy.net
Cookie: session=LJAwaBhBMAQQ2ZcIXVQs0fpEltQPacYt
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36
Connection: close
Content-Length: 30
Content-Type: application/x-www-form-urlencoded

username=wiener&action=upgrade
```
```http
GET /admin-roles?username=wiener&action=upgrade HTTP/1.1
Host: 0a64002703929c3fc08c45b800460050.web-security-academy.net
Cookie: session=LJAwaBhBMAQQ2ZcIXVQs0fpEltQPacYt
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36
Connection: close
```

## Horizontal privilege escalation

Horizontal privilege escalation arises when a user is able to gain access to resources belonging to another user, instead of their own resources of that type. 
Attack vectors are similar to [Vertical privilege escalation](Access%20control%20vulnerabilities.md#Vertical%20privilege%20escalation), but they differs in which data types are retrieved and the associated user roles:

- Parameter-based access control (cookies, user fields, query string, guessed parameters, etc.)
>[!info]-
>Also UUID and similar non-predictable IDs can be a vector if they get [disclosed](Information%20Disclosure.md) or retrieved using [Execution After Redirect (EAR)](Execution%20After%20Redirect%20(EAR).md)
- [Insecure Direct Object Reference](Access%20control%20vulnerabilities.md#Insecure%20Direct%20Object%20Reference)
- Platform misconfigurations and custom headers (`X-Original-URL`, `X-Rewrite-URL`, switching GET/POST and parameter positions, [HTTP Verb Tampering](HTTP%20Verb%20Tampering.md), etc.)

### Horizontal to vertical

Often, a horizontal privilege escalation attack can be turned into a vertical privilege escalation, by compromising a more privileged user. For example, a horizontal escalation might allow an attacker to reset or capture the password belonging to another user. If the attacker targets an administrative user and compromises their account, then they can gain administrative access and so perform vertical privilege escalation.

## Insecure Direct Object Reference

>[!summary]
>Insecure direct object references (IDOR) arises when an application uses user-supplied input to access files/objects directly.

Examples:
- IDOR vulnerability with direct reference to database objects: `https://insecure-website.com/customer_account?customer_number=132355`
- IDOR vulnerability with direct reference to static files: `https://insecure-website.com/static/12144.txt`

## Access control vulnerabilities in multi-step processes

Many web sites implement important functions over a series of steps. This is often done when a variety of inputs or options need to be captured, or when the user needs to review and confirm details before the action is performed. Sometimes, a web site will implement rigorous access controls over some of these steps, but ignore others.

>[!example]
>Suppose access controls are correctly applied to the first and second steps, but not to the third step. Effectively, the web site assumes that a user will only reach step 3 if they have already completed the first steps, which are properly controlled. 
>
>Here, an attacker can gain unauthorized access to the function by skipping the first two steps and directly submitting the request for the third step with the required parameters.

## Referer-based access control

Some websites base access controls on the Referer header submitted in the HTTP request. The Referer header is generally added to requests by browsers to indicate the page from which a request was initiated.

>[!example]
>Suppose an application robustly enforces access control over the main administrative page at `/admin`, but for sub-pages such as `/admin/deleteUser` only inspects the Referer header. If the Referer header contains the main `/admin` URL, then the request is allowed.

In this situation, since the Referer header can be fully controlled by an attacker, they can forge direct requests to sensitive sub-pages, supplying the required Referer header, and so gain unauthorized access.

## Location-based access control

Some web sites enforce access controls over resources based on the user's geographical location. This can apply, for example, to banking applications or media services where state legislation or business restrictions apply. These access controls can often be circumvented by the use of web proxies, VPNs, or manipulation of client-side geolocation mechanisms.

## Request Smuggling

In some applications, the front-end web server is used to implement some security controls, deciding whether to allow individual requests to be processed. Allowed requests are forwarded to the back-end server, where they are deemed to have passed through the front-end controls.

Suppose an application uses the front-end server to implement access control restrictions, only forwarding requests if the user is authorized to access the requested URL. The back-end server then honors every request without further checking. In this situation, an [HTTP Request Smuggling](HTTP%20Request%20Smuggling.md) vulnerability can be used to [Bypass front-end security controls](Exploiting%20HTTP%20Request%20Smuggling.md#Bypass%20front-end%20security%20controls), by smuggling a request to a restricted URL.