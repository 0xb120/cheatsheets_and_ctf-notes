>[!abstract]
>**Execution After Redirect** (EAR) [^1] is an attack where an attacker **ignores redirects and retrieves sensitive content intended for authenticated users**. A successful EAR exploit can lead to complete compromise of the application.

[^1]: https://cwe.mitre.org/data/definitions/698.html

Example:

```php
<?php if (!$loggedin) {
     print "<script>window.location = '/login';</script>\n\n"; 
} ?>
<h1>Admin</h1>
<a href=/mu>Manage Users</a><br />
<a href=/ud>Update Database Settings</a>
```

A non-authenticated attacker visiting the page above would discover the existence of `/mu` and `/ud` because the site does not force the redirection after the `if (!$loggedin)` but instead it return the full page containing a redirection within the source code (executed by the client). In this scenario an attacker is able to see the full page source code before being redirected to the login page.

A secure implementation should look like the following one:

```php
<?php

    if (!$loggedin) {
        header('location: login.php');
        die(); // this makes sure nothing more is sent
    }

    echo "This should only be shown to logged in users.";

?>
```
