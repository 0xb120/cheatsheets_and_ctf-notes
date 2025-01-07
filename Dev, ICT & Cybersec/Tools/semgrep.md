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

# Custom rules

- [semgrep-rules](https://github.com/semgrep/semgrep-rules/tree/develop) official repo
- [Android rules](https://github.com/mindedsecurity/semgrep-rules-android-security/tree/main/rules) by mindedsecurity
- https://github.com/akabe1/akabe1-semgrep-rulesiOS (swift) rules by akabe1

## Developing custom rules

Four main elements can be used inside custom rules [^pattern-syntax] :

[^pattern-syntax]: https://semgrep.dev/docs/writing-rules/pattern-syntax/

|Element|Description|
|---|---|
|`...`|Match a sequence of zero or more items such as arguments, statements, parameters, fields, characters.|
|`"..."`|Match any single hardcoded string.|
|`$A`|Match variables, functions, arguments, classes, object methods, imports, exceptions, and more.|
|`<...` e `...>`|Match an expression ("e") that could be deeply nested within another expression.|

Semgrep also supports taint [^taint], join [^join] and extract [^extract] modes. 

[^taint]: https://semgrep.dev/docs/writing-rules/data-flow/taint-mode/
[^join]: https://semgrep.dev/docs/writing-rules/experiments/join-mode/overview/
[^extract]: https://semgrep.dev/docs/writing-rules/experiments/extract-mode/

Example:
```yml title:"tainted-file-inclusion-for-opencart.yml"
rules:
- id: tainted-file-inclusion
  mode: taint
  pattern-sources:
  - patterns:
    - pattern-either:
      - pattern: $OBJ->request->get['...']
      - pattern: $OBJ->request->post['...']
      - pattern: $OBJ->request->cookie['...']
      - pattern: $OBJ->request->server['...']
  pattern-sanitizers:
  - patterns:
    - pattern-either:
      - pattern-inside: basename($PATH, ...)
      - pattern-inside: linkinfo($PATH, ...)
      - pattern-inside: readlink($PATH, ...)
      - pattern-inside: realpath($PATH, ...)
      - pattern-inside: include_safe(...)
  pattern-sinks:
  - patterns:
    - pattern-inside: $FUNC(...);
    - pattern: $VAR
    - metavariable-regex:
        metavariable: $FUNC
        regex: \b(include|include_once|require|require_once)\b
  severity: WARNING
  languages: [php]
  message: >-
    Detected non-constant file inclusion. This can lead to Local File Inclusion (LFI) or Remote File Inclusion
    (RFI) if user input reaches this statement. LFI and RFI could lead to sensitive files being obtained
    by attackers. Instead, explicitly specify what to include. If that is not a viable solution, validate
    user input thoroughly.
  metadata:
    category: security
    subcategory:
    - vuln
    technology:
    - php
    confidence: LOW
    likelihood: HIGH
    impact: MEDIUM
    owasp:
    - A03:2021 - Injection
    cwe:
    - "CWE-98: Improper Control of Filename for Include/Require Statement in PHP Program ('PHP Remote\
      \ File Inclusion')"
    references:
    - https://www.php.net/manual/en/function.include.php
    - https://github.com/FloeDesignTechnologies/phpcs-security-audit/blob/master/Security/Sniffs/BadFunctions/EasyRFISniff.php
    - https://en.wikipedia.org/wiki/File_inclusion_vulnerability#Types_of_Inclusion
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

```sh
$ semgrep --config $(routes which django) path/to/django/code
```

Use the `viz` subcommand to visualize route information in your browser:

```sh
$ semgrep --json --config $(routes which django) --output routes.json path/to/django/code
$ routes viz --browser routes.json
```

If you're not sure which framework to look for, you can use the special `all` ID to check everything:

```sh
$ semgrep --json --config $(routes which all) --output routes.json path/to/code
```

If you have custom authn or authz logic, you can copy `route-detect`'s rules:

```sh
$ cp $(routes which django) my-django.yml
```

Then you can modify the rule as necessary and run it like above:

```sh
$ semgrep --json --config my-django.yml --output routes.json path/to/django/code
$ routes viz --browser routes.json
```