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
	- `hx-on` and `hx-vars` [^sink1]
	- `hx-headers`, `hx-vals` and `hx-request` [^sink2]
	- `hx-trigger` [^sink3]
	- `include-vals` [^sink4]
- Changing response handling behavior can be done through custom response headers [^headers]
	- `HX-Location` [^header1]
	- `HX-Reselect` [^header2]
	- `HX-Trigger` [^header3]
	- `X-Redirect` [^header4]

[^headers]: https://htmx.org/docs/#response-headers
[^sink1]: [Hacking HTMX Applications](../../Readwise/Articles/Gabor%20Matuz%20-%20Hacking%20HTMX%20Applications.md#^ec04a4), `hx-on` and `hx-vars`
[^sink2]: [Hacking HTMX Applications](../../Readwise/Articles/Gabor%20Matuz%20-%20Hacking%20HTMX%20Applications.md#^fee07b), `hx-headers`, `hx-vals` and `hx-request`
[^sink3]: [Hacking HTMX Applications](../../Readwise/Articles/Gabor%20Matuz%20-%20Hacking%20HTMX%20Applications.md#^c223db), `hx-trigger`
[^sink4]: [Hacking HTMX Applications](../../Readwise/Articles/Gabor%20Matuz%20-%20Hacking%20HTMX%20Applications.md#^234450), `include-vals`
[^header1]: [Hacking HTMX Applications](../../Readwise/Articles/Gabor%20Matuz%20-%20Hacking%20HTMX%20Applications.md#^771ab2), `HX-Location`
[^header2]: [Hacking HTMX Applications](../../Readwise/Articles/Gabor%20Matuz%20-%20Hacking%20HTMX%20Applications.md#^427f69), `HX-Reselect`
[^header3]: [Hacking HTMX Applications](../../Readwise/Articles/Gabor%20Matuz%20-%20Hacking%20HTMX%20Applications.md#^8a0362), `HX-Trigger`
[^header4]: [Hacking HTMX Applications](../../Readwise/Articles/Gabor%20Matuz%20-%20Hacking%20HTMX%20Applications.md#^85b913), `HX-Redirect`