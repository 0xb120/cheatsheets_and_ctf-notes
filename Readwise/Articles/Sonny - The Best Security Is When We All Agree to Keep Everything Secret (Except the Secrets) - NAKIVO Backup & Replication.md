---
author: Sonny
aliases:
  - The Best Security Is When We All Agree to Keep Everything Secret (Except the Secrets) - NAKIVO Backup & Replication
tags:
  - readwise/articles
  - java
url: https://labs.watchtowr.com/the-best-security-is-when-we-all-agree-to-keep-everything-secret-except-the-secrets-nakivo-backup-replication-cve-2024-48248/
date: 2025-02-28
---
# The Best Security Is When We All Agree to Keep Everything Secret (Except the Secrets) - NAKIVO Backup & Replication

![rw-book-cover](https://labs.watchtowr.com/content/images/2025/02/nakivo-1.png)


A quick glance shows us a #Tomcat folder and a bunch of `jar` files - fantastic news.

As always, our first aim is to understand what we’re looking at, and map functionality so that we can ultimately begin to understand where we should begin prodding. As with Tomcat applications deployed via `war` files, the `web.xml` defines the routes available to the application and the corresponding servlet that supports requests to defined endpoints.

For example, within this file:
```xml
<servlet>
	<servlet-name>dispatcher</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	<load-on-startup>2</load-on-startup>
</servlet>

<servlet-mapping>
	<servlet-name>dispatcher</servlet-name>
	<url-pattern>/c/*</url-pattern>
</servlet-mapping>
```

In the above example, we can see that any value that follows on from the `/c/` URI is mapped to the #Spring Framework class `org.springframework.web.servlet.DispatcherServlet` .

This is typically driven by its accompanying dispatcher servlet configuration file, which contains directives on how the servlet behaves.

For example, within this file (`dispatcher-servlet.xml`), we find the tag `<context:annotation-config/>`, which enables support for annotated controllers (`@Controller` or `@RestController`) and handler methods (`@RequestMapping`, `@GetMapping`, etc.) for jar files loaded within the classpath.

Looking outside the Tomcat folder, we find a large `jar` file named `backup_replication.jar` which contains usage of these annotations.

For example, we found the following annotation within `com/company/product/ui/actions/LoginController.java` , as can be seen below the `RequestMapping` maps to the URI `/login`.
```java
Controller
@RequestMapping({"/login"})
public class LoginController
extends AbstractController
{
@Autowired
@Qualifier("SettingsService")
private SettingsService settingsService;
@Autowired
private RegistrationService registrationService;
@Autowired
private ConfigurationInfoService configurationInfoService;
@Autowired
private WebApplicationContext applicationContext;
private static final Gson gson = SerializationUtils.createGsonSerializer().create();
@RequestMapping(method = {RequestMethod.GET})
public ModelAndView getIndex(Locale locale, HttpServletResponse response, HttpServletRequest request) {
CanTryResponse canTryResponse;
CanUseDefaultCredentialsResponse defaultCredentialsResponse;
addSecurityHeaders(response::addHeader);
```

By combining the prefix of the `url-pattern` in the `web.xml` with the `RequestMapping` above we arrive at a URI of `/c/login` to reach the login page. [](https://read.readwise.io/read/01jn64m7ym0srf4dd2mvn795eb)

