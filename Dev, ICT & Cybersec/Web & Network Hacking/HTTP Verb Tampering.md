HTTP Verb Tampering is an attack that exploits vulnerabilities in HTTP verb (also known as HTTP method) authentication and access control mechanisms. Many authentication mechanisms only limit access to the most common HTTP methods, thus allowing [Authentication Attacks](Authentication%20Attacks.md) and [Access control vulnerabilities](Access%20control%20vulnerabilities.md).

```http
GET /admin HTTP/1.1
Host: vuln.com

403 Forbidden



HEAD /admin HTTP/1.1
Host: vuln.com

403 Forbidden



FOO /admin HTTP/1.1
Host: vuln.com

200 OK
Hi admin!
```

A vulnerable Java configuration:
```xml
<security-constraint>  
<web-resource-collection>  
<url-pattern>/admin/*</url-pattern>  
<http-method>GET</http-method>  
<http-method>POST</http-method>  
</web-resource-collection>  
<auth-constraint>  
<role-name>admin</role-name>  
</auth-constraint>  
</security-constraint>
```
These security rules ensure that GET or POST requests to _admin/_ directories from non admin users will be blocked. However, HTTP requests to _admin/_ directories other than GET or POST will not be blocked.


External references:
- https://www.imperva.com/learn/application-security/http-verb-tampering/
- https://www.apisec.ai/blog/web-attacks-intro-to-http-verb-tampering
