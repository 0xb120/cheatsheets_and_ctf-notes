[Cross-Origin Read Blocking](https://chromium.googlesource.com/chromium/src/+/main/services/network/cross_origin_read_blocking_explainer.md) (CORB) is a **security mechanism** implemented by web browsers to **prevent sensitive data leakage** when websites attempt to load resources from different origins. 

The core problem CORB addresses is that while the same-origin policy generally restricts cross-origin reads, historical exceptions and features like `<img>` or `<script>` tags could inadvertently expose data if they attempted to load certain content types like **JSON, HTML, or XML**.

CORB mitigates attacks such as Cross-Site Script Inclusion (XSSI) and speculative side-channel attacks (e.g., Spectre) by **blocking the response body and headers** of these protected resource types before they reach the web page's execution environment. The algorithm intelligently determines if a response is CORB-protected by evaluating its **Content-Type header** and **sniffing the response body** to confirm the content type, especially when the `X-Content-Type-Options: nosniff` header [^1] is present.

This ensures that only safe and intended cross-origin resource loads are permitted, thereby enhancing web security with minimal impact on existing web compatibility.

[^1]: [X-Content-Type-Options](X-Content-Type-Options.md)
