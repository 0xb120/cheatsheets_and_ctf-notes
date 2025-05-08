---
alias: Client Side Path Traversal (CSPT)
---

Original article: [Fetch Diversion, acut3](https://acut3.github.io/bug-bounty/2023/01/03/fetch-diversion.html#diverting-fetch-requests)

# Diverting fetch requests

Fetch diversion consists on tricking the victim into clicking a malicious link - created ad-hoc - that will fetch the supposed request to a different API, eventually causing [DOM-based XSS](DOM-based%20vulnerabilities.md#DOM-based%20XSS), leaking secrets or making authenticated API call.

The vulnerability exploits [Path Traversal](Path%20Traversal.md) on the client-side [^CSPT] in order to hijack legit API call to arbitrary APIs.

[^CSPT]: [Bypassing WAFs to Exploit CSPT Using Encoding Levels](https://matanber.com/blog/cspt-levels), matanber.com

>[!example]
>Browser URL `https://app.target.com/#/users/123456/profile` fetch requests to the API endpoint `https://api.target.com/v2/users/123456/profile`

If there are no enough control, we can create a link like  
```
https://app.target.com/#/users/..%2Fmalicious%2Fpath%23/profile
``` 
that once clicked fetch the following request:
```
https://api.target.com/v2/users/../malicious/path#/profile
```
which is translated into 
```
https://api.target.com/v2/malicious/path
```

# Attacks

## DOM XSS with uploaded file

If the application allows [Insecure File Upload](Insecure%20File%20Upload.md), and if the uploaded file can be retrieved on a endpoint that can be reached with a Fetch Diversion, then we can control the response to any request we are able to divert. It can result in a [DOM-based XSS](DOM-based%20vulnerabilities.md#DOM-based%20XSS) if a property from the response is inserted into the DOM in an insecure way.

>[!tip]
>The `Content-Type` used to serve the uploaded file doesn’t matter. Since the application is making a simple fetch, it will happily treat the response as whatever it expects (usually `application/json`), irrespective of its stated `Content-Type`.

Requirements:
1. We need to be able to upload a file with arbitrary content, which will be served unmodified. If the back-end checks the content of the file or tries to process it in any way (image transcoding for example), it probably won’t be exploitable.
    
2. The uploaded file must be accessible on the host toward which requests can be diverted. If, for example, the file is served directly from the CDN it’s uploaded to, we probably won’t be able to exploit it.
    
3. The uploaded file must be accessible by someone else, or else we would just end up with a self-XSS
    
4. There need to be a DOM XSS using one of the attributes returned by one of the requests we can divert

>[!tip] Finding exploitable API calls
The first issue was that most of them could not be be exploited for a DOM XSS. When properties were inserted into the DOM, it was done in a safe way. I used Burp’s “Match and Replace” to inject an XSS payload in all the json values returned by those API calls, and finally detected a few properties that were inserted in an insecure way. The Match and Replace was simple but effective:
>- Match in response body: `"([^"]*)":"`
>- Replace: `"$1":"<img src onerror=\\"console.log('XSS on \${origin} using $1')\\">`

>[!bug] Example #1: XSS in translation file

The application had an integrated web editor. This web editor used angular-translate for i18n, and the locale could be set through the `locale` query parameter. The translation file was loaded from
```
https://app.target.com/i18n/locale-<locale>.json
```
where `<locale>` was the value of the `locale` query parameter.

All you had to do was upload a malicious json document that would add an XSS payload to the translation of the appropriate message, share it with your team, and make one of your team members visit:
```
https://app.target.com/path/to/web_editor?lang=en-/../../path/to/preview/uuid?
```

It would make the application load its messages from:
```
https://app.target.com/i18n/locale-en-/../../path/to/preview/uuid?.json
```

which would normalize to the malicious preview file:
```
https://app.target.com/path/to/preview/uuid?.json
```

While the file was being loaded, the web editor would conveniently display status messages using the very unsafe `innerHTML`, resulting in a DOM XSS.



## Forcing authenticated requests (CSRF)

See also [Maxence Schmitt - Exploiting Client-Side Path Traversal to Perform Cross-Site Request Forgery - Introducing CSPT2CSRF](../../Readwise/Articles/Maxence%20Schmitt%20-%20Exploiting%20Client-Side%20Path%20Traversal%20to%20Perform%20Cross-Site%20Request%20Forgery%20-%20Introducing%20CSPT2CSRF.md)

Applications that use a custom header (like `Authorization` or `X-CSRF-Token`) or require `Content-Type: application/json` are normally immune to [Cross-Site Request Forgery (CSRF)](Cross-Site%20Request%20Forgery%20(CSRF).md) (barring CORS misconfiguration). But since we’re diverting a legitimate call issued by the application itself, we’re gaining the ability to make calls with our victim’s headers.

Keep in mind though that only the path and query parameters can be controlled. We will have to do with whatever method and body the diverted request happens to have.

Still, if we can find an API that can change data based on query parameters, then we might be able to exploit it. A great example is [GraphQL](GraphQL.md), which sometimes allows mutations through `GET` requests ([Bypassing GraphQL introspection defences](GraphQL%20vulnerabilities.md#Bypassing%20GraphQL%20introspection%20defences)).

```
https://app.target.com/users?id=../../graphql%3Fquery%3D{mutation ...}
```

Sometimes `POST` requests will take their parameters from the URL if those parameters cannot be found in the body. When this is the case and if we’re able to divert a `POST` request then it most likely can be exploited.

>[!bug] Diverting a POST request to bypass CSRF

One of the pages was using some custom code that was extracting the `id` query parameter from the URL, checking that it looked like a UUID, and then injecting it inside the path of an API call. But all values that _started_ like a UUID were accepted. As a result, visiting a URL such as:

```
https://app.target.com/vulnerable/page?id=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/../../../target?
```

would send a `POST` request to:

```
https://app.target.com/api/endpoint/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/../../../target?some/action
```

which would be normalized as:

```
https://app.target.com/target?some/action
```

Being generated by the application, the request of course contained the user’s anti-CSRF header (in addition to their authorization cookie).