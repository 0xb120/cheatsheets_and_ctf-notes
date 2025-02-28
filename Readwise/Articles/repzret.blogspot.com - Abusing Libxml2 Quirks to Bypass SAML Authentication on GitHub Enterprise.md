---
author: "repzret.blogspot.com"
aliases: "Abusing Libxml2 Quirks to Bypass SAML Authentication on GitHub Enterprise"
tags: RW_inbox, readwise/articles
url: https://repzret.blogspot.com/2025/02/abusing-libxml2-quirks-to-bypass-saml.html?m=1
date: 2025-02-28
---
# Abusing Libxml2 Quirks to Bypass SAML Authentication on GitHub Enterprise

![rw-book-cover](https://blogger.googleusercontent.com/img/a/AVvXsEj5dptO7B4ki4UcpTz8gnARufauW_tUyU2W1AhDhGp5SWTJk6iyqPfWL-BOhQgHei6U7Ak0e_muf3trtzQtWLUVi3chBYTf0EVzq_BjLAvs4_Q2-8Ts6qhKwjPhbmQbwjRM33uXgkAOzsuDgeFeFQVMwfjvDrdnJgApgQ_DSJU9HiE1E0wZukaH3C0vtJQ=w1200-h630-p-k-no-nu)

## Highlights


SAML primer
 SAML works pretty much like oauth2/OpenID, a user authenticates to an IdP, which returns an access token to the service provider, which is then used to access the user identity via some protect IdP endpoint, SAML on the other hand does not return an access token but rather a Response object with the user attributes (email, name, ..).
[View Highlight](https://read.readwise.io/read/01jn68de14xggjkgkz64a19qyk)



the **Response** contains an **Assertion** which contains the authenticated user attributes, the **NameID** element is what is usually used to identify the authenticated user (email, user id, ..). the response is protected against tampering using the **Signature** element, which contains the hash of the element it refers to (**Response**/**Assertion**) in the **DigestValue** element, the **SignedInfo** element is then signed by the IdP and the signature is stored in the **SignatureValue**, so any tampering to the protected element would make **DigestValue** incorrect, and any tampering to **DigestValue** would make the **SignatureValue** incorrect.
[View Highlight](https://read.readwise.io/read/01jn68em0r41wc137j5mvpp9hk)

