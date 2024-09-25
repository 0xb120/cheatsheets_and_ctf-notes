---
author: PortSwigger Research
aliases:
  - Finding That One Weird Endpoint
  - With Bambdas
tags:
  - readwise/articles
url: https://portswigger.net/research/finding-that-one-weird-endpoint-with-bambdas
date: 2024-08-20
---
# Finding That One Weird Endpoint, With Bambdas

![rw-book-cover](https://portswigger.net/cms/images/0a/f5/7287-twittercard-twitter.png)

## Highlights


## Large redirect responses

>  This Bambda will flag redirect responses with a body over 1000 bytes - this can indicate sites that forgot to terminate script execution when the user fails authentication, typically leading to information disclosure:
>  `return requestResponse.hasResponse() && requestResponse.response().statusCode() <= 399 && requestResponse.response().statusCode() >= 300 && requestResponse.response().body().length() > 1000;`
> [View Highlight](https://read.readwise.io/read/01hhg0n1bn713g425f6bvcg8vt)



## Responses with multiple `</html>` tags

>  What if a page fails to exit a script at the right point, but isn't serving a redirect response? In some cases this will result in the response containing multiple closing HTML tags. Our initial attempt to find these got a bunch of false positives from JavaScript files so we filtered those out by only showing responses with a HTML content-type. This approach revealed a page that we're pretty sure is meant to be behind authentication, and a completely unexpected source code leak.
>  `return requestResponse.response().statedMimeType() == MimeType.HTML && utilities().byteUtils().countMatches( requestResponse.response().body().getBytes(), "</html>".getBytes()) > 1;`
> [View Highlight](https://read.readwise.io/read/01hhg0n6mczz51nf1cs15x3frb)



## Incorrect content-length

>  I love to exploit sketchy HTTP middleware and one thing some of the worst middleware does is inject extra content into responses but fail to correct the Content-Length. This one is super easy to detect:
>  `int realContentLength = requestResponse.response().body().length(); int declaredContentLength = Integer.parseInt(      requestResponse.response().headerValue("Content-Length")); return declaredContentLength != realContentLength;`
> [View Highlight](https://read.readwise.io/read/01hhg0n9a1t203raqgjybz6mwc)



## Malformed HTTP header

>  Finally, I decided to look for responses containing a space in the header name. I wasn't really looking for anything in particular, and it yielded a bunch of servers running SMTP on port 443!
>  `return requestResponse.response().headers().stream().anyMatch(      e -> e.name().contains(" "));`
> [View Highlight](https://read.readwise.io/read/01hhg0nctkhetd2zjj619g4351)



## Find all JSON endpoints with no or `text/html` mime type

>  I absolutely love Bambdas and as James mentioned they provide a quick way to easily test your proxy history and find interesting nuggets that have been missed by standard filtering. When writing a Bambda it's useful to have a question in mind. One of those questions was "What sites are still using an invalid content-type for JSON responses?". Browsers nowadays are pretty strict when it comes to content sniffing however, if a site declares a text/html mime type with JSON data HTML will be rendered of course! I wrote a couple of lines of code and in no time I was finding stuff that I didn't know existed in my massive project file.
>  `if(!requestResponse.hasResponse()) { return false; } var response = requestResponse.response(); if (response.hasHeader("Content-Type")) { if (!response.headerValue("Content-Type").contains("text/html")) { return false; } } String body = response.bodyToString().trim(); boolean looksLikeJson = body.startsWith("{") || body.startsWith("["); if(!looksLikeJson) { return false; } return true;`
> [View Highlight](https://read.readwise.io/read/01hhg0nqymwh8vacqdqt40t9cz)



## Find all GraphQL endpoints

>  Next I need to find a lot of GraphQL endpoints for some testing I was doing. Using traditional filtering you can find common endpoints that for example contain /graphql, but what happens when you want to find endpoints that are not at a common location? This is where Bambdas come in, you can use a couple lines of Java to find parameters named "query" and the value contains a new line. Wham and there are a load of non-standard endpoints for testing!
>  `var req = requestResponse.request(); if(!req.hasParameters()) { return false; } var types = new HttpParameterType[]{ HttpParameterType.JSON, HttpParameterType.BODY, HttpParameterType.URL }; for(HttpParameterType type: types) { if(req.hasParameter("query", type)) { var value = req.parameterValue("query", type); if(type == HttpParameterType.JSON) { if(value.contains("\\n")) { return true; } } else { if(value.toLowerCase().contains("%0a")) { return true; } } } } return false;`
> [View Highlight](https://read.readwise.io/read/01hhg0nv0m7y5ex1fyr424zmvg)



## Find JSONP for CSP bypass

>  Let's say you've got XSS but the site is protected by CSP, what you need to do is find scripts on the site that you can control because the CSP allows "same site" script resources. You can easily do this with a Bambda! The next Bambda looks for JSONP endpoints. It first looks for a parameter that looks like a callback with 4 or more characters. Then it searches the response to see if it's reflected with an opening parenthesis. This was surprisingly effective and found lots of JSONP for me!
>  `var req = requestResponse.request(); var res = requestResponse.response(); var paramRegex = Pattern.compile("^[a-zA-Z][.\\w]{4,}$"); if(res == null || res.body().length() == 0) return false; if(!req.hasParameters()) return false; var body = res.bodyToString().trim(); var params = req.parameters(); for(var param: params) { var value = param.value(); if(param.type() != HttpParameterType.URL)continue; if(paramRegex.matcher(value).find()) { var start = "(?:^|[^\\w'\".])"; var end = "\\s*[(]"; var callbackRegex = Pattern.compile(start+Pattern.quote(value)+end); if(callbackRegex.matcher(body).find())return true; } } return false;`
> [View Highlight](https://read.readwise.io/read/01hhg0p2b6ztnnf299895nmhfs)



> curated repo of the best at [https://github.com/PortSwigger/bambdas](https://github.com/PortSwigger/bambdas)
> [View Highlight](https://read.readwise.io/read/01hhg0p9x9nxnrdqhavsjrppdy)

