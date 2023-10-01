---
Description: Static code analysis tool with table support for C#, Go, Java, JavaScript, JSON, Python, PHP, Ruby, and Scala. Experimental support for nineteen other languages
URL: https://github.com/returntocorp/semgrep
---

# Basic usage

## Automatic scan

```bash
$ semgrep scan --config=auto . -o output.txt

Scanning across multiple languages:
    <multilang> |  76 rules × 5178 files
            php |  33 rules ×  931 files
             js | 179 rules ×  372 files
           html |   1 rule  ×   55 files
           json |   4 rules ×   15 files
           yaml |  28 rules ×    1 file
           bash |   4 rules ×    1 file
...
Ran 1071 rules on 2589 files: 184 findings.
```

## Custom rule scan 

>[!tldr] Rule list
>Official rules can be found on https://semgrep.dev/explore

```bash
# Run a specific rulset against a file
semgrep --config="RULESET-ID" PATH/TO/SRC

# Run a specific custom rule
semgrep --config=PATH/TO/MYRULE.YAML PATH/TO/SRC

# Run a language specific ruleset against a full directory
semgrep --config p/python .
```


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