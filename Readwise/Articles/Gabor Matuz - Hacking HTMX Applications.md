---
author: Gabor Matuz
aliases:
  - Hacking HTMX Applications
tags:
  - readwise/articles
url: https://infosecwriteups.com/hacking-htmx-applications-f8d29665faf
created: 2024-08-20
---
# Hacking HTMX Applications

![rw-book-cover](https://miro.medium.com/v2/resize:fit:1200/1*S7jhK8lvqhQaF1X_Iu8mnw.png)

## Highlights


> The only way htmx will be different from other web applications is how HTML is assembled and interacted with
> [View Highlight](https://read.readwise.io/read/01hcmjqffy2c2ee5zd4d7me6be)



> In htmx all attributes have 2 versions, hx-<attribute_name> and data-hx-<attribute_name> they are functionally identical
> [View Highlight](https://read.readwise.io/read/01hcmjs65cq95hgysjpcwzvf5g)



> a tag with a hx-<verb> (e.g. hx-get) attribute can trigger a (ajax)request and the response is an HTML that gets swapped in based on css selectors defining the [target](https://htmx.org/docs/#targets) of the swap
> [View Highlight](https://read.readwise.io/read/01hcmjt1rscqjgza99a303r61k)



> In htmx escaping is expected to be done on server side by design and not by the framework itself
> [View Highlight](https://read.readwise.io/read/01hcmjtw0q9qmp3fys62kg7t2a)



> The library uses xhr to make requests for fragments, which does follow redirects meaning an open redirect in htmx is effectively an XSS
> [View Highlight](https://read.readwise.io/read/01hcmjw3v38nm3k1a4s9sq352y)



> Htmx introduces a number of attributes that work as javascript sinks
> [View Highlight](https://read.readwise.io/read/01hcmjy5mqd1er75j0vhfqx674)



> hx-on (for htmx event handlers) and hx-vars (way to change request behavior) are straight up evaluated
> [View Highlight](https://read.readwise.io/read/01hcmjyjz1a9hkg8p56ga4av23)



> hx-headers, hx-vals, hx-request are evaluated IF the string starts with js: or javascript:
> [View Highlight](https://read.readwise.io/read/01hcmjyt5d8dc6wxezm4r0che8)



> hx-trigger can include arbitrary javascript e.g. in the following format hx-trigger=load[<arbitrary js>]
> [View Highlight](https://read.readwise.io/read/01hcmjz1m8c27b9rv2w00dv99r)



> include-vals extension is present, values of the include-vals attribute gets into the following context: eval(“({“ + includeVals + “})”).
> [View Highlight](https://read.readwise.io/read/01hcmjza1aawxsksgsrt23f66v)



> Changing response handling behavior can be done through [response headers](https://htmx.org/docs/#response-headers)
> [View Highlight](https://read.readwise.io/read/01hcmk6f3e1xba6b3xkp4rftd9)



> Controlling HX-Location allows you to force htmx to make a request to the location and swap the response into the top level htmx element effectively resulting in an XSS
> [View Highlight](https://read.readwise.io/read/01hcmk6y8e7zc5n7gxgj8j1cbc)



> HX-Reselect will allow you to select a subset of the elements from the *response* to swap in while HX-Reswap will allow you to define *where* to swap them in.
> [View Highlight](https://read.readwise.io/read/01hcmk7cnpqyh24yysgn8dksem)



> HX-Trigger would allow you to trigger event handlers
> [View Highlight](https://read.readwise.io/read/01hcmk7j569ned4xrryqneechm)



> HX-Redirect will redirect
> [View Highlight](https://read.readwise.io/read/01hcmk7s7vr03y88y2ynq1zz8b)

