---
author: "Doyensec's Blog"
aliases: ["Common OAuth Vulnerabilities"]
tags: [RW_inbox, readwise/articles]
url: https://blog.doyensec.com/2025/01/30/oauth-common-vulnerabilities.html
date: 2025-02-28
---
# Common OAuth Vulnerabilities

![rw-book-cover](https://blog.doyensec.com/public/images/apple-icon-57x57.png)

## Highlights


Common OAuth Vulnerabilities
[View Highlight](https://read.readwise.io/read/01jn62s27ztafp1fqsza9gapyv)



comprehensive guide on known attacks against OAuth implementations. Additionally, we have created a comprehensive checklist.
[View Highlight](https://read.readwise.io/read/01jn62tg18z5z5rkw2p448c1mz)



**Download the OAuth Security Cheat Sheet Now!** [Doyensec_OAuth_CheatSheet.pdf](https://doyensec.com/resources/Doyensec_OAuth_CheatSheet.pdf).
[View Highlight](https://read.readwise.io/read/01jn62tphapawpm0bgejzrxjn8)



OAuth Common Flows
 Attacks against OAuth rely on challenging various assumptions the authorization flows are built upon. It is therefore crucial to understand the flows to efficiently attack and defend OAuth implementations.
[View Highlight](https://read.readwise.io/read/01jn62yckhrqfx2f6wcs6gwmem)



Implicit Flow
 The Implicit Flow was originally designed for native or single-page apps that cannot securely store Client Credentials. However, its use is now discouraged and is not included in the OAuth 2.1 specification. Despite this, it is still a viable authentication solution within Open ID Connect (OIDC) to retrieve `id_tokens`.
[View Highlight](https://read.readwise.io/read/01jn62zq659rb71x6n5rghwgs5)



In this flow, the User Agent is redirected to the Authorization Server. After performing authentication and consent, the Authorization Server directly returns the Access Token, making it accessible to the Resource Owner. This approach exposes the Access Token to the User Agent, which could be compromised through vulnerabilities like XSS or a flawed `redirect_uri` validation. The implicit flow transports the Access Token as part of the URL if the `response_mode` is not set to `form_post`.
 ![OAuth Implicit Flow](https://blog.doyensec.com/2025/01/30/oauth-common-vulnerabilities.html/../../../public/images/oauth-implicit-flow.png)
[View Highlight](https://read.readwise.io/read/01jn631740jpq8t3xtgajcfgt0)



Authorization Code Flow
 The Authorization Code Flow is one of the most widely used OAuth flows in web applications. Unlike the Implicit Flow
[View Highlight](https://read.readwise.io/read/01jn632cbcfnxy8v5fg1g4z4eh)



In this process, the User Agent first retrieves an Authorization Code, which the application then exchanges, along with the Client Credentials, for an Access Token. This additional step ensures that only the Client Application has access to the Access Token, preventing the User Agent from ever seeing it.
[View Highlight](https://read.readwise.io/read/01jn6330v7q3j9rvjxdhebnrzg)



![OAuth Authorization Code Flow](https://blog.doyensec.com/2025/01/30/oauth-common-vulnerabilities.html/../../../public/images/oauth-authorization-code-flow.png)
[View Highlight](https://read.readwise.io/read/01jn6347x0h1cpn24cq4bk3p1q)



Authorization Code Flow with PKCE
[View Highlight](https://read.readwise.io/read/01jn6357dqt1z8nznjgxn5d7pn)



OAuth 2.0 provides a version of the Authorization Code Flow which makes use of a Proof Key for Code Exchange (PKCE).
[View Highlight](https://read.readwise.io/read/01jn636awgryp1knyqzm41ktfm)



it has become the main recommendation in the [OAuth 2.1 specification](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-01).
[View Highlight](https://read.readwise.io/read/01jn636ds6bckst808vty4yqnn)



Two new parameters are added to the default Authorization Code Flow, a random generated value called `code_verifier` and its transformed version, the `code_challenge`.
[View Highlight](https://read.readwise.io/read/01jn636rk7dm6gmetywv9j0v9r)



• First, the Client creates and records a secret `code_verifier` and derives a transformed version `t(code_verifier)`, referred to as the `code_challenge`, which is sent in the Authorization Request along with the transformation method `t_m` used.
 • The Client then sends the Authorization Code in the Access Token Request with the `code_verifier` secret.
 • Finally, the Authorization Server transforms `code_verifier` and compares it to `t(code_verifier)`
[View Highlight](https://read.readwise.io/read/01jn63808wqas12k8jfb2ez3br)



The available transformation methods (`t_m`) are the following:
 • plain `code_challenge = code_verifier`
 • S256 `code_challenge = BASE64URL-ENCODE(SHA256(ASCII(code_verifier)))`
[View Highlight](https://read.readwise.io/read/01jn6385rmvj6cgpyk09g26xbn)



Note that using the default Authorization Code flow with a custom `redirect_uri` scheme like `example.app://` can allow a malicious app to register itself as a handler for this custom scheme alongside the legitimate OAuth 2.0 app. If this happens, the malicious app can intercept the authorization code and exchange it for an Access Token.
[View Highlight](https://read.readwise.io/read/01jn639es3fz4y06404bzbfv5t)



refer to OAuth Redirect Scheme Hijacking.
[View Highlight](https://read.readwise.io/read/01jn639n3fsjycrgw622w9gbkj)



The diagram below illustrates the Authorization Code flow with PKCE:
 ![OAuth Authorization Code Flow with PKCE](https://blog.doyensec.com/2025/01/30/oauth-common-vulnerabilities.html/../../../public/images/oauth-authorization-code-pkce-flow.png)
[View Highlight](https://read.readwise.io/read/01jn63a74z93q9a5zxxwym116v)



Client Credentials Flow
[View Highlight](https://read.readwise.io/read/01jn63abvcg9g6capnvngdwjy8)



The Client Credentials Flow is designed for Machine-to-Machine (M2M) applications, such as daemons or backend services.
[View Highlight](https://read.readwise.io/read/01jn63atk100swrkr5rzdrdn5y)



The diagram below illustrates the Client Credentials Flow:
 ![OAuth Client Credentials Flow](https://blog.doyensec.com/2025/01/30/oauth-common-vulnerabilities.html/../../../public/images/oauth-client-credentials-flow.png)
[View Highlight](https://read.readwise.io/read/01jn63ba6bfq323q6t570n5tac)



Device Authorization Flow
[View Highlight](https://read.readwise.io/read/01jn63bztdvkqm6nkfyhvk44fg)



The Device Authorization Flow is designed for Internet-connected devices that either lack a browser for user-agent-based authorization or are too input-constrained to make text-based authentication practical
[View Highlight](https://read.readwise.io/read/01jn63cnz6w3stk900npds9gae)



This flow allows OAuth Clients on devices such as smart TVs, media consoles, digital picture frames or printer to obtain user authorization to access protected resources using a User Agent on a separate device.
[View Highlight](https://read.readwise.io/read/01jn63d49089r1nebedf4hqce0)



In this flow, first the Client application retrieves a User Code and Verification URL from the Authorization Server. Then, it instructs the User Agent to Authenticate and Consent with a different device using the provided User Code and Verification URL.
[View Highlight](https://read.readwise.io/read/01jn63drbqnpthdekbv8j51j44)



The following image illustrates the Device Authorization Code Flow:
 ![OAuth Device Authorization Flow](https://blog.doyensec.com/2025/01/30/oauth-common-vulnerabilities.html/../../../public/images/oauth-device-authorization-flow.png)
[View Highlight](https://read.readwise.io/read/01jn63dye56849x3cbrtpvn3e0)



Resource Owner Password Credentials Flow
[View Highlight](https://read.readwise.io/read/01jn63efwy5mhvwcj2ezjsnz4s)



This flow requires the Resource Owner to fully trust the Client with their credentials to the Authorization Server.
[View Highlight](https://read.readwise.io/read/01jn63f61f9vjc990sbf8dg7ab)



it has been removed in the recent [OAuth 2.1 RFC](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-01#name-differences-from-oauth-20) specification and its use is not recommended.
[View Highlight](https://read.readwise.io/read/01jn63fbbh7yn31yah38ymbgw4)



Instead of redirecting the Resource Owner to the Authorization Server, the user credentials are sent to the Client application, which then forwards them to the Authorization Server.
[View Highlight](https://read.readwise.io/read/01jn63fsbk0851k9ehxv6zamnz)



The following image illustrates the Resource Owner Password Credentials Flow:
 ![OAuth Resource Owner Password Credentials Flow](https://blog.doyensec.com/2025/01/30/oauth-common-vulnerabilities.html/../../../public/images/oauth-resource-owner-flow.png)
[View Highlight](https://read.readwise.io/read/01jn63fwtxyg6psg0p3e6sfp0a)



Attacks
 In this section we’ll present common attacks against OAuth with basic remediation strategies.
[View Highlight](https://read.readwise.io/read/01jn63g9cjx3cnzdx31rwbk87j)



CSRF
 OAuth CSRF is an attack against OAuth flows, where the browser consuming the authorization code is different than the one that has initiated the flow. It can be used by an attacker to coerce the victim to consume their Authorization Code, causing the victim to connect with attacker’s authorization context.
[View Highlight](https://read.readwise.io/read/01jn63h7d0tcp18j24r7z9zgwm)



![OAuth CSRF Attack](https://blog.doyensec.com/2025/01/30/oauth-common-vulnerabilities.html/../../../public/images/oauth-csrf-attack.svg)
[View Highlight](https://read.readwise.io/read/01jn63hf5bevvtscm4v7c5mzty)



Mitigation
 OAuth specification recommends to utilize the `state` parameter to prevent CSRF attacks.
[View Highlight](https://read.readwise.io/read/01jn63j4bzz1f38mqjn1z75d6k)



Redirect Attacks
 Well implemented Authorization Servers validate the `redirect_uri` parameter before redirecting the User Agent back to the Client. The allowlist of `redirect_uri` values should be configured per-client.
[View Highlight](https://read.readwise.io/read/01jn63k53t39m66ewxhwkcpwb3)



if the Authorization Server neglects or misimplements this verification, a malicious actor can manipulate a victim to complete a flow that will disclose their Authorization Code to an untrusted party.
[View Highlight](https://read.readwise.io/read/01jn63kqmbkdp6sbbcms3y37ek)



when `redirect_uri` validation is missing altogether, exploitation can be illustrated with the following flow:
 ![OAuth Redirect Attack](https://blog.doyensec.com/2025/01/30/oauth-common-vulnerabilities.html/../../../public/images/oauth-redirect-attack.svg)
[View Highlight](https://read.readwise.io/read/01jn63n0tabzpg4z9570w20k4p)



The only proper way is validation by comparing the **exact** `redirect_uri` including both the origin (scheme, hostname, port) and the path.
[View Highlight](https://read.readwise.io/read/01jn63nkq857qk6gkw74ndh0kh)



Mutable Claims Attack
[View Highlight](https://read.readwise.io/read/01jn63q67erfavsdh27fk8dr9j)



According to the OAuth specification, users are uniquely identified by the `sub` field. However there is no standard format of this field. As a result, many different formats are used, depending on the Authorization Server.
[View Highlight](https://read.readwise.io/read/01jn63rgghbhsbkr8kjwd1p3kg)



Some of the Client applications, in an effort to craft a uniform way of identifying users across multiple Authorization Servers, fall back to user handles, or emails. However this approach may be dangerous
[View Highlight](https://read.readwise.io/read/01jn63t0y8bw47098etj74pm7g)



In such cases account takeovers might be possible.
[View Highlight](https://read.readwise.io/read/01jn63tehkecr8375yr45kew49)



One of such cases emerges, when the feature “Login with Microsoft” is implemented to use the `email` field to identify users.. In such cases, an attacker might create their own AD organization (`doyensectestorg` in this case) on Azure, which can be used then to to perform “Login with Microsoft”. While the `Object ID` field, which is placed in `sub`, is immutable for a given user and cannot be spoofed, the `email` field is purely user-controlled and does not require any verification.
[View Highlight](https://read.readwise.io/read/01jn63vkscrvnjjkygg649nf7b)



Client Confusion Attack
[View Highlight](https://read.readwise.io/read/01jn63w62zh2arssjycry4sv2f)



When applications implement OAuth Implicit Flow for authentication they should verify that the final provided token was generated for that specific Client ID. If this check is not performed, it would be possible for an attacker to use an Access Token that had been generated for a different Client ID.
[View Highlight](https://read.readwise.io/read/01jn63x31x55az18rn6qry9s3d)



Imagine the attacker creates a public website which allows users to log in with Google’s OAuth Implicit flow. Assuming thousands of people connect to the hosted website, the attacker would then have access to their Google’s OAuth Access Tokens generated for the attacker website.
 If any of these users already had an account on a vulnerable website that does not verify the Access Token, the attacker would be able to provide the victim’s Access Token generated for a different Client ID and will be able to take over the account of the victim.
[View Highlight](https://read.readwise.io/read/01jn63y5mtdhdj1f9f7b1ahme1)



![OAuth Secure Implicit Flow](https://blog.doyensec.com/2025/01/30/oauth-common-vulnerabilities.html/../../../public/images/oauth-implicit-secure.svg)
 If steps 8 to 10 are not performed and the token’s Client ID is not validated, it would be possible to perform the following attack:
 ![OAuth Client Confusion Attack](https://blog.doyensec.com/2025/01/30/oauth-common-vulnerabilities.html/../../../public/images/oauth-client-confusion.svg)
[View Highlight](https://read.readwise.io/read/01jn63ytz0jm38e6n1e2bvej9n)



Scope Upgrade Attack
[View Highlight](https://read.readwise.io/read/01jn63zbs47t2ktzc2fdwwspg6)



With the Authorization Code Grant type, the user’s data is requested and sent via secure server-to-server communication.
 If the Authorization Server accepts and implicitly trusts a `scope` parameter sent in the Access Token Request (Note this parameter is not specified in the RFC for the Access Token Request in the Authorization Code Flow), a malicious application could try to upgrade the scope of Authorization Codes retrieved from user callbacks by sending a higher privileged scope in the Access Token Request.
[View Highlight](https://read.readwise.io/read/01jn640y2bwpb27h1pe93np5xw)



Once the Access Token is generated, the Resource Server must verify the Access Token for every request.
[View Highlight](https://read.readwise.io/read/01jn641r64w4zrg0j9qkg41npr)



verification depends on the Access Token format, the commonly used ones are the following:
 • **JWT Access Token**: With this kind of access token, the Resource Server only needs to check the JWT signature and then retrieve the data included in the JWT (`client_id`, `scope`, etc.)
 • **Random String Access Token**: Since this kind of token does not include any additional information in them, the Resource Server needs to retrieve the token information from the Authorization Server.
 ![OAuth Scope Upgrade](https://blog.doyensec.com/2025/01/30/oauth-common-vulnerabilities.html/../../../public/images/oauth-scope-upgrade.svg)
[View Highlight](https://read.readwise.io/read/01jn641yvy9twg9ex6yb19p5cb)



Redirect Scheme Hijacking
[View Highlight](https://read.readwise.io/read/01jn642ahqne13ks74d2m52c31)



When the need to use OAuth on mobile arises, the mobile application takes the role of OAuth User Agents. In order for them to be able to receive the redirect with Authorization Code developers often rely on the mechanism of custom schemes. However, multiple applications can register given scheme on a given device. This breaks OAuth’s assumption that the Client is the only one to control the configured `redirect_uri` and may lead to Authorization Code takeover in case a malicious app is installed in victim’s devices.
[View Highlight](https://read.readwise.io/read/01jn643f7s57v0gf8wvrpg38s6)
> #android 


for instance the following URI `com.example.app://oauth` depicts an Intent with `scheme=com.example.app` and `host=oauth`. In order to receive these Intents an Android application would need to export an Activity similar to the following:
 <intent-filter>
 <action android:name="android.intent.action.VIEW"/>
 <category android:name="android.intent.category.DEFAULT"/>
 <category android:name="android.intent.category.BROWSABLE"/>
 <data android:host="oauth" android:scheme="=com.example.app"/>
 </intent-filter>
[View Highlight](https://read.readwise.io/read/01jn645pqngt37anmwsgbxsk59)



the more specific original developers were, the easier it is to craft a bypass and take over the redirect without user interaction.
[View Highlight](https://read.readwise.io/read/01jn646tr01z4mt25hthm25sae)



[Ostorlab](https://ostorlab.co/) has created the following flowchart to quickly assess whether it is possible:
 ![OAuth Scheme Hijacking](https://blog.doyensec.com/2025/01/30/oauth-common-vulnerabilities.html/../../../public/images/oauth-scheme-hijacking.svg)
[View Highlight](https://read.readwise.io/read/01jn64760f17x49mew253yhwwj)

