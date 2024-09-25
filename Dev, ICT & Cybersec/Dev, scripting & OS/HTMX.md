# HTMX 101

The only way htmx [^htmx-doc] will be different from other web applications is how HTML is assembled and interacted with.

[^htmx-doc]: https://htmx.org/

In htmx all attributes have 2 versions: `hx-<attribute_name>` and `data-hx-<attribute_name>`, which are functionally identical.

A tag with a `hx-` attribute (e.g. `hx-get`) can trigger an ajax request, and the response is an HTML that gets swapped in based on CSS selectors defining the [target](https://htmx.org/docs/#targets) of the swap.

>[!warning]
>In htmx escaping is expected to be done on server side by design and not by the framework itself.

## Security implications

- The library uses [**XMLHttpRequest (XHR)**](JavaScript%20&%20NodeJS.md#**XMLHttpRequest%20(XHR)**) to make requests for fragments, which does follow redirects meaning an open redirect in htmx is effectively an XSS
- Htmx introduces a number of attributes that work as javascript sinks:
	- `hx-on` and `hx-vars`
	- `hx-headers`, `hx-vals` and `hx-request` 
	- `hx-trigger`
	- `include-vals` 
- Changing response handling behavior can be done through custom response headers 
	- `HX-Location` 
	- `HX-Reselect`
	- `HX-Trigger` 
	- `X-Redirect` 

Both `hx` attributes and headers can be misused and lead to vulnerabilities [^hacking-htmx] .

[^headers]: https://htmx.org/docs/#response-headers
[^hacking-htmx]: [Hacking HTMX Applications](../../Readwise/Articles/Gabor%20Matuz%20-%20Hacking%20HTMX%20Applications.md)