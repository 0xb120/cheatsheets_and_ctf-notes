# OpenID Connect basics

>[!question] What is OpenID connect?
OpenID Connect extends the OAuth protocol to provide a dedicated identity and authentication layer that sits on top of the basic [OAuth 2.0](OAuth%202.0.md) implementation. 

OpenID Connect solves a lot of the authentication problems in OAuth by adding standardized, identity-related features to make authentication via OAuth work in a more reliable and uniform way.

Compared to [OAuth 2.0 components](OAuth%202.0.md#OAuth%202.0%20components), the key difference are:
- an additional, standardized set of [scopes](OAuth%202.0.md#OAuth%202.0%20scopes) that are the same for all providers (`openid`)
- an extra response type: `id_token`.

As with basic OAuth, it's also a good idea to take a look at the OAuth provider's documentation to see if there's any useful information about their OpenID Connect support. You may also be able to access the configuration file from the standard endpoint:
- `/.well-known/openid-configuration`


## OpenID Connect roles (aka components)

Roles for OpenID Connect are essentially the same as for standard OAuth. The main difference is that the specification uses slightly different terminology:
- **Relying party** = the application that is requesting authentication of a user (aka *client application*)
- **End user** = user who is being authenticated (aka *resource owner*)
- **OpenID provider** = OAuth service that is configured to support OpenID Connect

## OpenID Connect claims and scopes

The term "claims" refers to the `key:value` pairs that represent information about the user on the resource server. 
>[!example]
>One example of a claim could be `"family_name":"Montoya"`.

Unlike basic OAuth, whose [scopes are unique to each provider](https://portswigger.net/web-security/oauth/grant-types#oauth-scopes), all OpenID Connect services use an identical set of scopes. In order to use OpenID Connect, the client application **must specify the scope `openid` in the authorization request**. They can then include one or more of the other standard scopes:
- `profile`
- `email`
- `address`
- `phone`

Each of these scopes corresponds to read access for a subset of claims about the user that are defined in the OpenID specification. 
>[!example]
>Requesting the scope `openid profile` will grant the client application read access to a series of claims related to the user's identity, such as `family_name`, `given_name`, `birth_date`, etc.
>
>`https://oauth-0a49007104101d0f80a9b039028b0021.oauth-server.net/auth?client_id=fp4j0o3a0nlxoftdr1xqy&redirect_uri=https://0a0b00e904761d5b8030b27f00e5005a.web-security-academy.net/oauth-callback&response_type=code&scope=openid%20profile%20email`

## ID token

The other main addition provided by OpenID Connect is the `id_token` response type. This returns a JSON web token ([JWT](JWT%20Vulnerabilities.md#Structure)) signed with a JSON web signature (JWS).
The JWT payload contains:
- a list of claims based on the scope that was initially requested
- information about how and when the user was last authenticated by the OAuth service

Rather than simply relying on a trusted channel, as happens in basic OAuth, the integrity of the data transmitted in an ID token is based on a JWT cryptographic signature. For this reason, the use of ID tokens may help protect against some man-in-the-middle attacks. However, given that the cryptographic keys for signature verification are transmitted over the same network channel (normally exposed on `/.well-known/jwks.json`), some attacks are still possible.

>[!note]
>Multiple response types (aka grant types) are supported by OAuth, so it's perfectly acceptable for a client application to send an authorization request with both a basic OAuth response type and OpenID Connect's `id_token` response type. Eg. `response_type=id_token code`

# OpenID Connect vulnerabilities

As it is just a layer that sits on top of OAuth, the client application or OAuth service may still be vulnerable to some of the OAuth-based common attacks, but it can also has some specific vulnerabilities:

- [OAuth 2.0 attacks](OAuth%202.0%20attacks.md)
- [Unprotected dynamic client registration](OpenID%20Connect%20attacks.md#Unprotected%20dynamic%20client%20registration)
- [Allowing authorization requests by reference](OpenID%20Connect%20attacks.md#Allowing%20authorization%20requests%20by%20reference)