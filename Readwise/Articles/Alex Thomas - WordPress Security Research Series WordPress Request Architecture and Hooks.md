---
author: "Alex Thomas"
aliases: "WordPress Security Research Series: WordPress Request Architecture and Hooks"
tags: RW_inbox, readwise/articles
url: https://www.wordfence.com/blog/2024/07/wordpress-security-research-series-wordpress-request-architecture-and-hooks/
date: 2025-01-23
---
# WordPress Security Research Series: WordPress Request Architecture and Hooks

![rw-book-cover](https://www.wordfence.com/wp-content/uploads/2024/06/FeaturedImage_Wordfence_114.03.png)

## Highlights


The Request Lifecycle
 1. **Initial Request:** Everything starts when the server receives an HTTP request. This could be a simple GET request for a page or a POST request submitting form data.
 2. **WordPress Core Loading:** WordPress initializes its core, loading the necessary components to handle the request, including plugin and theme files.
 3. **Routing:** WordPress determines what the request is trying to access—be it a post, a page, an admin panel, or an AJAX call—and routes the request accordingly.
 4. **Hooks and Execution:** This stage of the WordPress request lifecycle is where the core, along with themes and plugins, actively engage with the incoming request. WordPress core and themes predominantly use hooks — actions and filters — to alter outputs and execute operations. These hooks are predefined points in the WordPress code where plugins and themes can intervene to modify behavior or add new functionality.
 5. **Response:** Finally, WordPress sends a response back to the client, which could be an HTML page, a JSON payload, or a simple status code.
 ![The WordPress Request Lifecycle](https://www.wordfence.com/wp-content/uploads/2024/06/wordpress_request_lifecycle.png)
[View Highlight](https://read.readwise.io/read/01jj9zw5wwwznbz469b8n0h7yd)



WordPress Core Request Handling
 When most requests hit a WordPress site, they are first intercepted by the `[index.php](https://github.com/WordPress/WordPress/blob/master/index.php)` file in the webroot directory. This file is the controller for all front-of-site WordPress requests. It loads the `[wp-blog-header.php](https://github.com/WordPress/WordPress/blob/master/wp-blog-header.php)` file, which in turn loads the `[wp-load.php](https://github.com/WordPress/WordPress/blob/master/wp-load.php)` file to initialize the WordPress environment and template. All WordPress requests result in `[wp-load.php](https://github.com/WordPress/WordPress/blob/master/wp-load.php)` being executed in some way or another.
 [![WordPress Request Flow](https://www.wordfence.com/wp-content/uploads/2024/06/wordpress_request_flow.png)](https://www.wordfence.com/wp-content/uploads/2024/06/wordpress_request_flow.png)
 WordPress Request Flow
[View Highlight](https://read.readwise.io/read/01jj9zx1e40xcnj1xd6fy9f0bf)



Key Routes and Their Hooks
 There are a number of routes that are interesting to WordPress vulnerability researchers because they trigger certain actions that plugin and theme authors often hook into with custom functions. As we discovered in previous sections, these functions may display, modify, or remove data, and they may not implement appropriate protections to prevent unauthorized users from accessing or manipulating them.
 Route
 Triggers These Hooks
 With this Authentication/Authorization
 / (Frontend)
 template_redirect, wp_head, wp_footer
 None (public)
 /wp-admin/ (Admin Dashboard)
 admin_init, admin_menu, admin_footer, admin_action_{action}
 Generally subscriber-level or higher role. 
 EXCEPTION: Sending an unauthenticated request to /wp-admin/admin-ajax.php or /wp-admin/admin-post.php will trigger the admin_init action.
 /wp-admin/admin-ajax.php (AJAX Endpoint)
 wp_ajax_{action}, wp_ajax_nopriv_{action}
 wp_ajax_{action} requires login and can be triggered by users with a subscriber-level or higher role with access to /wp-admin. 
 wp_ajax_nopriv_{action} can be accessed by unauthenticated users.
 /wp-admin/admin-post.php (Form Submission)
 admin_post_{action}, admin_post_nopriv_{action}
 None (public) for nopriv otherwise subscriber-level or higher role.
 User Profile/Update (wp-admin/profile.php)
 profile_update, wp_update_user, personal_options_update, edit_user_profile_update
 Generally subscriber-level or higher role
 Options Update
 update_option
 Authentication/Authorization depends, because the update_option action is tied to when the update_option() function is called by the plugin/theme developer. If, for example, update_option() is called in a function hooked to a wp_ajax_nopriv_{action}, then the required Authentication/Authorization would be “None (public)”.
 Any Page Load
 init
 None (public)
 /wp-json/ (REST API)
 rest_api_init
 None (public) by default. Permissions can be controlled by permission_callback.
 /wp-login.php (Login Page)
 wp_login, login_form
 None (public)
 /wp-signup.php (Multisite Registration)
 signup_header
 None (public)
 /wp-cron.php (WP-Cron)
 wp_scheduled_delete, wp_scheduled_auto_draft_delete
 None (public)
 /xmlrpc.php (XML-RPC)
 xmlrpc_call
 Depends on method
[View Highlight](https://read.readwise.io/read/01jja01jv7j8qbtn6rrxe8fn88)



Now that you’re familiar with routes and which hooks they trigger, let’s review the most important actions and filters for vulnerability research:
 **Actions**
 • `init:` Fires after WordPress has finished loading but before any headers are sent. This is an early stage in the WordPress loading process, making it a critical point for initializing plugin or theme functionalities.
 • `admin_init:` Triggers before any other hook when a user accesses the admin area (/wp-admin). Despite the name, this action can be triggered by unauthenticated users.
 • `wp_ajax_{action}:` A core part of the admin-side AJAX functionality. It’s used to handle authenticated AJAX requests.
 • `wp_ajax_nopriv_{action}:` Similar to wp_ajax_{action}, but specifically for handling unauthenticated AJAX requests. This hook is crucial for AJAX functionality available to public users.
 • `admin_post and admin_post_nopriv:` These hooks are used for handling form submissions in the WordPress admin area for authenticated (admin_post) and unauthenticated (admin_post_nopriv) users.
 • `admin_action_{action}:` Triggered in response to admin actions. It’s a versatile hook for custom admin functionalities and can be accessed via both GET and POST requests.
 • `profile_update:` Fires when a user’s profile is updated. It’s an important hook for executing actions post-user update.
 • `wp_update_user:` Similar to profile_update, this action occurs when a user’s data is updated. It is crucial for managing user information.
 • `personal_options_update:` This hook is triggered when a user updates their own profile in the WordPress admin area. It allows developers to execute custom code when a user updates their personal settings.
 • `edit_user_profile_update:` This hook is triggered when an admin or another user with the appropriate permissions updates another user’s profile. It allows developers to execute custom code during the profile update process for other users. It’s almost always used with personal_options_update.
 **Filters**
 • `the_content:` Filters the post content before it’s sent to the browser. If improperly sanitized, it can lead to XSS attacks.
 • `the_title:` Similar to the_content, this filter can be a vector for XSS if the title output is not properly escaped.
 • `user_has_cap:` Filters a user’s capabilities and can be used to alter permissions, potentially leading to privilege escalation.
 • `authenticate:` Filters the authentication process. If overridden or improperly extended, it can lead to authentication bypass vulnerabilities.
[View Highlight](https://read.readwise.io/read/01jja05v8tsncjj68t85c3b920)



Plugin Loading
 All plugins have a main PHP file and it can be identified by a [header comment](https://developer.wordpress.org/plugins/plugin-basics/header-requirements/)
[View Highlight](https://read.readwise.io/read/01jja0bw80348cbp1shgsx5n1m)



its main PHP file gets loaded on every subsequent request to the WordPress site. This behavior is crucial to understand because any code placed directly in the main plugin file executes during each page load, regardless of the context or the user’s intention.
[View Highlight](https://read.readwise.io/read/01jja0cz769rh7sx4pdypsawv9)

