---
author: Alex Thomas
aliases:
  - "WordPress Security Research Series: WordPress Security Architecture"
tags:
  - RW_inbox
  - readwise/articles
url: https://www.wordfence.com/blog/2025/03/wordpress-security-research-series-wordpress-security-architecture/?__readwiseLocation=
date: 2025-04-01
---
# WordPress Security Research Series: WordPress Security Architecture

![rw-book-cover](https://www.wordfence.com/wp-content/uploads/2025/03/FeaturedImage_Wordfence_210.05.02.png)

## Highlights


Unlike typical **MVC (Model-View-Controller)** frameworks with a clear separation of concerns and predictable routing, WordPress is built around a hook-based system. Code executes procedurally, with actions and filters running at various points during a request.
 There are no controllers handling specific routes or directories dedicated to application logic. Instead, plugins, themes, and core all share the same global environment and hook into the same execution flow. [](https://read.readwise.io/read/01jpys0ksn8ag756sfgdm7301x)



**vulnerability researcher mindset** when analyzing WordPress code:
 • Does this WordPress application (plugin/theme) consume user-supplied data, how is it handled, and for what purpose is it used?
 • Does this WordPress application use WordPress functions and is it using them properly? Does it attempt to roll its own validation, sanitization, escaping, or authorization functionality instead of using the functions provided by WordPress?
 • Does this WordPress application incorporate third-party dependencies that are out-of-date?
 • Are WordPress escaping functions being used throughout? Are they being used correctly?
 • Are assumptions made on how a user will supply input?
 • Are denylists used instead of allowlists? [](https://read.readwise.io/read/01jpys3n3cdh3hp7tcs6rr4w2x)



**Core Security Features in WordPress**
 WordPress provides the [Security API](https://developer.wordpress.org/apis/security/) to developers, which includes the suite of functions that WordPress core uses to secure its own code. [](https://read.readwise.io/read/01jpys51j6p4g811h6k3gcydcb)



As vulnerability researchers, it’s important to understand what these functions are, their specific purposes, and how developers are expected to use them. By gaining an in-depth knowledge of these functions, we can identify their usage during **static analysis**. [](https://read.readwise.io/read/01jpys5cqyfgpe8nwarcrd3v87)



Common Sanitization Functions
 The following are some commonly used [sanitization functions available in the WordPress Security API](https://developer.wordpress.org/apis/security/sanitizing/#sanitization-functions). [](https://read.readwise.io/read/01jpyseehr4xdshxx8yrvtdzq8)



**Common Escaping Functions**
 The following are some commonly used [escaping functions available in the WordPress Security API](https://developer.wordpress.org/apis/security/escaping/#escaping-functions). [](https://read.readwise.io/read/01jpysg804n9snm0x94wj21wtc)



**Pro-Tip: Dynamically Evaluating Code with WP Shell**
 Using `[wp shell](https://developer.wordpress.org/cli/commands/shell/)`, a part of WP-CLI, you can interactively evaluate PHP code from **within your WordPress environment**. This is a great way for vulnerability researchers to familiarize themselves with WordPress’s core security features. [](https://read.readwise.io/read/01jpyshsmmedkrr3s0jmy3m07r)



[![Using wp shell and var_dump() to visualize how sanitize_file_name() handles a path traversal sequence.](https://www.wordfence.com/wp-content/uploads/2025/03/wp-shell-bg.png)](https://www.wordfence.com/wp-content/uploads/2025/03/wp-shell-bg.png) 
 Using `wp shell` and `var_dump()` to visualize how `sanitize_file_name()` handles a path traversal sequence. [](https://read.readwise.io/read/01jpyskdkchztqv3hwdwsks1wg)



**Database Interactions and SQL**
 The WordPress API makes a large number of database functions available to use so developers can avoid passing untrusted user-supplied input directly into database queries. These are usually either prefixed with `add_`, `update_`, `delete_`, or `get_` (e.g., `add_option()`, `update_user_meta()`, `delete_comment_meta()`, or `get_comment_meta()`) or with a `wp_insert`, `wp_update`, or `wp_delete` (e.g., `wp_insert_post()`, `wp_update_comment()`, `wp_delete_user()`). **However, developers often fail to use these functions or use them correctly**. [](https://read.readwise.io/read/01jpysmc23e9dj33x4pwmmat2q)



The `$wpdb` class is a database abstraction layer that simplifies database interactions and ensures developers follow best practices from a security perspective, but they are often used in a way that makes them vulnerable to SQL injection. [](https://read.readwise.io/read/01jpysp44cqs6dhftg9n341ptd)



**SQL Injection – What to Look For**
 SQL Injections can be hard to find as there may exist multiple layers of abstraction from the point user input is consumed to where it is used in a SQL query. [](https://read.readwise.io/read/01jpyssctt5drsb2cnjrwwbb1q)



In addition, WordPress utilizes [wp_magic_quotes()](https://developer.wordpress.org/reference/functions/wp_magic_quotes/) which automatically adds single quotes (`‘`) to input from superglobals [](https://read.readwise.io/read/01jq8qbyfghxjr2wdqpmvjr2y0)



a SQL query like the one below may initially appear vulnerable to SQL Injection [](https://read.readwise.io/read/01jq8qc9myyry6tyqqn2j3g8zg)



it would not be [](https://read.readwise.io/read/01jq8qce920bk6c0qwevk36664)



`SELECT * FROM wp_posts WHERE ID = ''1''`.
 1
 2
 3
 4
 5
 `// Vulnerable concatenation with user-supplied input`
 `$post_id` `=` `$_GET``[``'post_id'``];`
 `...`
 `$query` `=` `"SELECT * FROM wp_posts WHERE ID = ' "` `.` `$post_id` `.` `"'"``;`
 `$wpdb``->get_results(` `$query` `);` [](https://read.readwise.io/read/01jq8qcrrbb7aw6kd2p3k2p9xn)



**What is esc_sql()?**
 WordPress provides the `esc_sql()` function to escape characters in strings that would be used in SQL queries. [](https://read.readwise.io/read/01jq8qdpf9g3ehawp06hqfwjgm)



This function is often used incorrectly by developers to “sanitize” user-supplied values before being used in SQL queries instead of `$wpdb->prepare()`. While it may mitigate some attacks by escaping these characters, it’s not intended to thwart SQL injection attacks like `$wpdb->prepare()`. [](https://read.readwise.io/read/01jq8qeyxmf93tczfcp5c04hvc)



If a user-supplied value is not wrapped in single quotes in a query, `esc_sql()` as the only form of protection could easily be bypassed considering there are no single quotes needed to escape out of. [](https://read.readwise.io/read/01jq8qgak5zq5q0n6rrf0v7skn)



**Authentication, User Roles, Capabilities, and Authorization****WordPress Authentication in a Nutshell** [](https://read.readwise.io/read/01jq8qgx0pgwgbn0damkvne2bd)



The authentication process starts with a user submitting a username and password to `wp-login.php`. This form submission gets passed to the `wp_signon()` function, which then calls the `wp_authenticate()` function and checks the submitted username and password against stored user data in the database. If authentication is successful, the `wp_set_auth_cookie()` function is called to set the user’s cookie and store the user’s session details. The cookie and session data is then used to authenticate subsequent requests via the `wp_validate_auth_cookie()` function. [](https://read.readwise.io/read/01jq8qjc0eejsw8c98v1z7y9jw)



Each unique user session is managed through the `wp_usermeta` database table. Cookies are cleared and sessions are destroyed via the `wp_clear_auth_cookie()` function when a user logs out.
 Passwords are stored in the database using the `wp_hash_password()` function. [](https://read.readwise.io/read/01jq8qkkwnr0a25d2k55vrdza0)



**User Roles** [](https://read.readwise.io/read/01jq8qkv7bzqtmme7t0pft2cqh)



**Administrator:** Full access to all administrative features and settings. 
 **Editor:** Can manage and publish posts, including those of other users. 
 **Author:** Can write, edit, and publish their own posts. 
 **Contributor:** Can write and edit their own posts but cannot publish them. 
 **Subscriber:** Can manage their own profile. [](https://read.readwise.io/read/01jq8qm0j1tya1wfvkk18y3a8y)



**Custom Roles**
 Custom roles can be defined by themes or plugins by invoking the `add_role()` function. Custom roles are useful when a set of capabilities not defined by a default role is needed. [](https://read.readwise.io/read/01jq8qn0wwve36s3ek803fv5rd)



Capabilities for a given role can be modified using the `add_cap()` or `remove_cap()` functions. [](https://read.readwise.io/read/01jq8qn7cvx2pc2dxxc6jpffp4)



**Custom Capabilities**
 Custom capabilities in WordPress allow developers to define specific permissions that are not covered by the default roles and capabilities. [](https://read.readwise.io/read/01jq8qnjrhpkp72h3b2egj0jvz)



**Authorization**
 In WordPress, [Authorization](https://www.wordfence.com/blog/2025/03/wordpress-security-research-series-wordpress-security-architecture/#definitions) is done by checking capabilities against the request that is being made by the user. [](https://read.readwise.io/read/01jq8qpyemj83v0d0qfk8nm6hw)



WordPress uses contextual authorization. This means that plugin and theme developers must implement authorization checks within the specific areas of code (e.g., within the function) that need authorization checks before an action is performed by that function. [](https://read.readwise.io/read/01jq8qsz10ezkqm9mf6p99acz8)



**Nonces**
 [*Nonce*](https://developer.wordpress.org/apis/security/nonces/) stands for “number used once”. It is a [pseudo-random](https://www.wordfence.com/blog/2025/03/wordpress-security-research-series-wordpress-security-architecture/#definitions) alphanumeric string that is generated using one of WordPress’s nonce generation functions and **embedded within a page or a link from which a request will originate**. It can be viewed within the page’s source HTML. [](https://read.readwise.io/read/01jq8qx5ddkfeheg39j5fj11gy)



Including a nonce in a request and verifying it at the receiving function ensures that the request is intentional. If implemented and validated correctly, nonces can prevent [Cross-Site Request Forgery](https://owasp.org/www-community/attacks/csrf). However, developers often misuse this feature (or don’t use it at all). [](https://read.readwise.io/read/01jq8qxxy1sdyc88kjaemwkvhe)



Nonces in WordPress aren’t actually a “number used once” and instead last for up to 24 hours, meaning they can be reused within that time period. [](https://read.readwise.io/read/01jq8qy9watb8nfawqkzszavpr)



**Nonce Functions and Parameters**
 WordPress provides developers with a number of nonce generation and verification functions. To prevent Cross-Site Request Forgery, nonces should be generated and placed into requests that perform actions on the WordPress instance (e.g., changing settings or deleting posts) and they should be verified within the function that performs the action before it is performed. [](https://read.readwise.io/read/01jq8qzkcrqeppv236b2b06gc0)



Vulnerability researchers can search for these functions within plugins and themes to see if they are used in conjunction with an authorization mechanism. If they are not, the researcher can search the code to see if the nonce generation function is used within a page that is accessible by a lower-level user. [](https://read.readwise.io/read/01jq8r0ghe21d6yrfc5x1gjvbw)



This nonce might be generated and printed on a subscriber-level user’s profile page. This means that a subscriber-level user could modify plugin settings by passing a valid nonce that they have access to. [](https://read.readwise.io/read/01jq8r1435ywgg81p246waqcgs)



**Nonce Generation Functions**
 **`wp_create_nonce()`**: This function generates a nonce token and returns it as a string. It takes one optional parameter, `$action`, which is a string that identifies the action being performed.
 **`wp_nonce_field()`**: This function generates a hidden input field with a nonce token and returns it as a string. It takes two parameters, `$action` and `$name`. `$action` is a string that identifies the action being performed, while `$name` is the name of the input field.
 **`wp_nonce_url()`**: This function generates a URL with a nonce added as a query string parameter. It takes two parameters, `$actionurl` and `$action`. `$actionurl` is the URL to which the user will be directed, while `$action` is a string that identifies the action being performed. [](https://read.readwise.io/read/01jq8r1gfdevy4sj3qjgpgb6gr)



**Nonce Verification Functions**
 **`wp_verify_nonce()`**: This function checks the referer for a nonce value. It takes two parameters: `$nonce` and `$action`. `$nonce` is the nonce value, while `$action` is a string that identifies the action being performed.
 **`check_admin_referer()`**: This function checks the referer for a WordPress admin screen against the nonce value. It takes two parameters, `$action` and `$query_arg`. `$action` is a string that identifies the action being performed, while `$query_arg` is the name of the URL parameter that contains the nonce value.
 **`check_ajax_referer()`**: This function checks the referer for an AJAX request against the nonce value. It takes two parameters, `$action` and `$query_arg`. $action is a string that identifies the action being performed, while `$query_arg` is the name of the URL parameter that contains the nonce value. [](https://read.readwise.io/read/01jq8r1mmj8mdnrw4gavgnks5y)



**REST API Authentication and Authorization**
 The [WordPress REST API](https://developer.wordpress.org/rest-api/) handles authentication a bit differently from its standard login-based authentication. It supports not only **cookie-based authentication**, but also **application passwords**, and **OAuth**. Custom REST API endpoints are registered via the `register_rest_route()` function. [](https://read.readwise.io/read/01jq8r37pb1g8qxdestfjqpghp)



**Cookie-Based Authentication:** When a user is logged in, REST API requests can be authenticated via the same cookies used for normal WordPress operations. This means a registered rest route (e.g., `http://example-wordpress-site.com/wp-json/{rest_route}`) is accessible in the same way as any other route. [](https://read.readwise.io/read/01jq8r56hp574xwkpr4dcygdxy)



However, if a valid `X-WP-Nonce` for that request is not provided, then WordPress will consider the request unauthenticated despite supplying valid authentication cookies in the request. This is because the WordPress REST API implementation has built-in Cross-Site Request Forgery (CSRF) protection. The automated checking of `X-WP-Nonce` is only relevant for cookie-based authentication. [](https://read.readwise.io/read/01jq8r6b5wqc74snvwkk946zmb)



**Application Passwords**: Introduced in WordPress 5.6, application passwords allow users to create unique passwords for external applications accessing the site via REST API. [](https://read.readwise.io/read/01jq8r7ct9ys3r891pzj69svzg)



**OAuth**: OAuth can be implemented to provide secure, token-based authentication for certain applications or third-party integrations. [](https://read.readwise.io/read/01jq8r7kkvwn4h726r28ed1kfg)



Vulnerabilities in REST API callback functions usually stem from a lack of authorization (i.e. the route’s permission callback returns true and the callback function itself does not perform an authorization check either) [](https://read.readwise.io/read/01jq8r8pgr7575c22xtnqk2pwv)



It is also possible for developers to implement their own custom authorization mechanism for REST API routes that should be closely examined for potential bypasses. [Here is an example](https://www.wordfence.com/blog/2021/10/1000000-sites-affected-by-optinmonster-vulnerabilities/) of a poorly implemented authorization check that was easily bypassed. [](https://read.readwise.io/read/01jq8r9hv143m440mp35f6hy5y)



**REST API Authentication and Authorization Flow Logic**
 `REST API Request`
 `│`
 `├── Using Application Password, OAuth, or Basic Auth?`
 `│       └── Yes → Authenticated User (based on credentials)`
 `│`
 `├── Using Cookies?`
 `│       └── Yes → Is X-WP-Nonce valid?`
 `│               ├── No → User ID 0 (unauthenticated)`
 `│               └── Yes → Authenticated User (from cookie)`
 `│`
 `└── None of the above? → User ID 0 (unauthenticated)`
 `│`
 `Check permission_callback`
 `│`
 `├── Does permission_callback exist?`
 `│       ├── No → ❌ Public endpoint (allow everyone)`
 `│       │`
 `│       └── Yes → Execute permission_callback()`
 `│               │`
 `│               ├── Does it perform a capability check?`
 `│               │       ├── Yes (e.g., current_user_can())`
 `│               │       │       ├── Pass → ✅ Allow access`
 `│               │       │       └── Fail → ❌ Deny access (401/403)`
 `│               │`
 `│               │       └── No (returns true unconditionally)`
 `│               │               → ✅ Public endpoint (allow everyone)`
 `│`
 `└── Proceed to callback function if allowed` [](https://read.readwise.io/read/01jq8r9vv3ngzcn79zr94prbcb)



Deserialization and JSON Methods [](https://read.readwise.io/read/01jqe0kn059tyhskxbsw058m54)



`maybe_serialize()` and `maybe_unserialize()` functions, which are simply wrappers around the PHP `serialize()` and `unserialize()` functions with an additional check for serialized data before the serialization or deserialization process occurs. [](https://read.readwise.io/read/01jqe0m3qfevxhj110dpxpjpe6)



**Unserialized allowed_classes Option**
 PHP’s `unserialize()` has an allowed classes option. Developers who are aware of PHP Object Injection vulnerabilities will often set the `allowed_classes` option to `false` to prevent the instantiation of objects. However, this option does not always mitigate PHP Object Injection. [](https://read.readwise.io/read/01jqe0nsgdrgn6kvza6bp6gyfv)



If a serialized payload is passed to `unserialize()` with the `array(‘allowed_classes' => false)` option, the resulting object will be an instance of `__PHP_Incomplete_Class`, which is a placeholder object. This object contains the original malicious property values but isn’t unserialized. If this instance of `__PHP_Incomplete_Class` is passed to some function that serializes data (like the WordPress `update_option()` function, which uses `maybe_serialize()`) it will re-serialize `__PHP_Incomplete_Class` effectively generating the original payload, and in the `update_option()` case, the serialized payload will be stored in the database. If that value is later unserialized, you have PHP Object Injection. [](https://read.readwise.io/read/01jqe17ny036mgmqg8dsqeng3q)



**Deserialization Flow**
 `User Input (malicious serialized payload)`
 `│`
 `▼`
 `Stored in database (options, post meta, user meta, etc.)`
 `│`
 `▼`
 `Retrieved from database`
 `│`
 `▼`
 `maybe_unserialize()`
 `│`
 `▼`
 `unserialize()`
 `│`
 `├── With allowed_classes => false`
 `│       │`
 `│       ▼`
 `│   __PHP_Incomplete_Class object created`
 `│       │`
 `│       ▼`
 `│   Stored again (e.g., via update_option())`
 `│       │`
 `│       ▼`
 `│   Retrieved and unserialized (without allowed_classes)`
 `│       │`
 `│       ▼`
 `│   🚨 POP Chain executes → Remote Code Execution (or other impact)`
 `│`
 `└── Without allowed_classes`
 `│`
 `▼`
 `POP Chain executes:`
 `├── __destruct()`
 `├── __wakeup()`
 `├── __call()`
 `▼`
 `🚨 Remote Code Execution (or other impact)` [](https://read.readwise.io/read/01jqe17t4h8fb5bpkj9gyp5a6m)



**Uploading Files**
 Allowing the upload of arbitrary files can result in PHP code execution or Cross-Site Scripting (e.g. via SVG files). [](https://read.readwise.io/read/01jqe18be4csj9am5fmwa4mc6e)



WordPress provides the `wp_check_filetype()` and `wp_check_filetype_and_ext()` functions to help WordPress plugin and theme developers secure file uploads. [](https://read.readwise.io/read/01jqe18w440wjsvnpx6ag368bz)



developers can leverage the `upload_mimes` filter to modify the WordPress global array of allowed MIME types [](https://read.readwise.io/read/01jqe196qy31c7nyck5z55b97r)



Finally, the `wp_handle_upload()` function handles the full file upload process, including sanitizing file names, checking for MIME type [](https://read.readwise.io/read/01jqe19m460ybv5vg4xe8gm5ce)



**Finding File Upload Vulnerabilities**
 The omission of these functions in the presence of file upload functionality is a good indicator that an arbitrary file upload vulnerability might be present. [](https://read.readwise.io/read/01jqe1a62rk642ygvx8khmyk3q)



Path Traversals and File Inclusions [](https://read.readwise.io/read/01jqeda1tt4nvqpsv3e61z3fyp)



WordPress does not provide specialized functions solely for preventing path traversal or file inclusion vulnerabilities. [](https://read.readwise.io/read/01jqedaa1s5e2b7b6n6zc6yfd1)



The most important function in this context is `sanitize_file_name()`, which ensures that user-provided filenames are stripped of dangerous characters and path traversal sequences. [](https://read.readwise.io/read/01jqedaj3v73xe9zj2rk8qnwwk)



`basename()` function can be used to strip directory paths from a filename. [](https://read.readwise.io/read/01jqedaqf6xn6n3mjmk13evebs)



**Server-Side Request Forgery (SSRF)**
 WordPress provides a set of HTTP request functions via its HTTP API. [](https://read.readwise.io/read/01jqedb6vmgx9hwdy411ewavvy)



Developers using these functions must implement their own validation.
 • `wp_remote_get()`
 • `wp_remote_post()`
 • `wp_remote_request()`
 • `wp_remote_head()` [](https://read.readwise.io/read/01jqedc00dxd7ab7a6zamcs0t9)



These functions prevent requests to localhost, private IP ranges (e.g., `192.168.x.x`, `10.x.x.x`, `172.16.x.x`), loopback addresses (e.g., `127.0.0.1`), IPv6 private and local ranges, and non-http:// or https:// URL schemes.
 • `wp_safe_remote_get()`
 • `wp_safe_remote_post()`
 • `wp_safe_remote_request()`
 • `wp_safe_remote_head()` [](https://read.readwise.io/read/01jqedce28pkhg9pm78h3as9e6)


Finding Server-Side Request Forgery (SSRF) Vulnerabilities [](https://read.readwise.io/read/01jserym10nhb58t4955njjsq7)



Researchers should look for the use of `wp_remote_*()` functions to find SSRF vulnerabilities. [](https://read.readwise.io/read/01jseryxf16zrfzzbdsdv6vbqv)



**Conclusion**
 In this post, we reviewed the **security architecture of WordPress from a vulnerability research perspective**. [](https://read.readwise.io/read/01jses04wxcm929q78nhqmckmb)



Combining this knowledge with [***WordPress Request Architecture and Hooks***](https://www.wordfence.com/blog/2024/07/wordpress-security-research-series-wordpress-request-architecture-and-hooks/), we understand that WordPress doesn’t follow the same design patterns as typical MVC web applications. [](https://read.readwise.io/read/01jses0rvd98hxd1rtxd6v759j)



Instead, it relies on a hook-based system where code executes procedurally across plugins, themes, and core. [](https://read.readwise.io/read/01jses0zsnwrtebdramb755cs9)

