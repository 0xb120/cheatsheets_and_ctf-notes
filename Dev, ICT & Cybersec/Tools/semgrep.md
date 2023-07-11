---
Description: Static code analysis tool with table support for C#, Go, Java, JavaScript, JSON, Python, PHP, Ruby, and Scala. Experimental support for nineteen other languages
URL: https://github.com/returntocorp/semgrep
---


# Expansion tool for/using semgrep

## route-detect [^1]

[^1]: https://github.com/mschwager/route-detect

>[!summary]
>Find authentication (authn) and authorization (authz) security bugs in web application routes and frameworks.

Supported web frameworks (`route-detect` IDs in parentheses):
- Python: Django (`django`, `django-rest-framework`), Flask (`flask`), Sanic (`sanic`)
- PHP: Laravel (`laravel`), Symfony (`symfony`), CakePHP (`cakephp`)
- Ruby: Rails* (`rails`), Grape (`grape`)
- Java: JAX-RS (`jax-rs`), Spring (`spring`)
- Go: Gorilla (`gorilla`), Gin (`gin`), Chi (`chi`)
- JavaScript/TypeScript: Express (`express`), React (`react`), Angular (`angular`)

Use the `which` subcommand to point `semgrep` at the correct web application rules:

```
$ semgrep --config $(routes which django) path/to/django/code
```

Use the `viz` subcommand to visualize route information in your browser:

```
$ semgrep --json --config $(routes which django) --output routes.json path/to/django/code
$ routes viz --browser routes.json
```

If you're not sure which framework to look for, you can use the special `all` ID to check everything:

```
$ semgrep --json --config $(routes which all) --output routes.json path/to/code
```

If you have custom authn or authz logic, you can copy `route-detect`'s rules:

```
$ cp $(routes which django) my-django.yml
```

Then you can modify the rule as necessary and run it like above:

```
$ semgrep --json --config my-django.yml --output routes.json path/to/django/code
$ routes viz --browser routes.json
```