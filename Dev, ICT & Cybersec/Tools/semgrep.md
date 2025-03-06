---
Description: Static code analysis tool with table support for C#, Go, Java, JavaScript, JSON, Python, PHP, Ruby, and Scala. Experimental support for nineteen other languages
URL: https://github.com/returntocorp/semgrep
---

# Basic usage

>[!abstract] Documentation
>- [Semgrep tutorial](https://appsec.guide/docs/static-analysis/semgrep/) by Trail of Bits; appsec.guide

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

Other useful flags:
- `--dataflow-traces`: Explain how non-local values reach the location of a finding (only affects text and SARIF output).
- `--exclude=PATTERN`: Skip  any  file  or  directory  whose  path  that  matches `PATTERN`. Eg. `--exclude=*.py`
- `--include=PATTERN`: Specify file  or  directory that should be scanned, excluding other files. Eg. `--include=*.py`
- `--experimental`: enable experimental features
- `--files-with-matches`: Output  only  the  names  of  files  containing  matches. Requires `--experimental`
- `--force-color` / `--no-force-color`
- `--historical-secrets`: Scans git history using Secrets rules
- `-j`: Number of subprocess to use to run checks
- `--matching-explanations`: Add debugging information in the JSON output to trace how different parts of a rule are matched
- `--severity=VAL`: Report  findings  only  from  rules  matching the supplied severity level.
- `--json-output=file.json` / `--text-output=file.txt`: output to specific file
- `--no-git-ignore`: Do not take into account `.git` repo and `.gitignore` files

## Predefined rules scan 

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

List of already existing custom rules:
- [semgrep-rules](https://github.com/semgrep/semgrep-rules/tree/develop) official GitHub repo
- [Android rules](https://github.com/mindedsecurity/semgrep-rules-android-security/tree/main/rules) by mindedsecurity
- https://github.com/akabe1/akabe1-semgrep-rulesiOS (swift) rules by akabe1
- [Marco Ivaldi - A Collection of Weggli Patterns for CC++ Vulnerability Research](../../Readwise/Articles/Marco%20Ivaldi%20-%20A%20Collection%20of%20Weggli%20Patterns%20for%20CC++%20Vulnerability%20Research.md)

## Developing custom rules

Official documentation:
- https://semgrep.dev/docs/writing-rules/overview
- https://semgrep.dev/playground/new

Other useful resources and articles:
- https://spaceraccoon.dev/comparing-rule-syntax-codeql-semgrep/

### Pattern syntax

The main elements that can be used inside patterns: [^pattern-syntax] are:

[^pattern-syntax]: https://semgrep.dev/docs/writing-rules/pattern-syntax/

| Element                                                                                     | Description                                                                                                                                                                                           | Examples                                                                                                                                                                                                                                                              |
| ------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `...` ([ellipsis](https://semgrep.dev/docs/writing-rules/pattern-syntax#ellipsis-operator)) | Match a sequence of zero or more items such as arguments, statements, parameters, fields, characters.                                                                                                 | `insecure_function(...)`<br>`func(1, ...)`<br>`func(..., 1)`<br>`$OBJECT.extractall(...)`<br>`$Obj.foo(). ... .bar()`<br>`pattern: \|  class CLASS(InsecureBaseClass): ... `<br>`$foo = /.../;`<br>`crypto.set_secret_key("...")` (match any single hardcoded string) |
| `$A` ([metavariables](https://semgrep.dev/docs/writing-rules/pattern-syntax#metavariables)) | Match variables, functions, arguments, classes, object methods, imports, exceptions, and more.                                                                                                        | `$LOGGER.log(...)`<br>`(java.util.logging.Logger $LOGGER).log(...)` (typed metavariable to put a type constraint)<br>`$LOGGER.log(java.util.logging.LogRecord $RECORD)` (typed argument)<br>`def function($_, $_, $_)`  (always have 3 arguments)                     |
| `<... [your_pattern] ...>` (deep expression operator)                                       | Match an expression that could be deeply nested within another expression. The deep expression operator matches your pattern in the current expression context and recursively in any subexpressions. | `if <... $USER.is_admin() ...>: ...`<br>matches<br>`if user.authenticated() and user.is_admin() and user.has_group(gid): [ CONDITIONAL BODY ]`                                                                                                                        |
**metavariable unification**
For search mode rules, metavariables with the same name are treated as the same metavariable within the patterns operator. This is called metavariable unification.

For taint mode rules, patterns defined within pattern-sinks and pattern-sources still unify. However, metavariable unification between pattern-sinks and pattern-sources is not enabled by default.

**metavariable in messages**
Metavariables can also be referenced inside output messages:

```yaml
- pattern: $MODEL.set_password(...)
- message: Setting a password on $MODEL
```

```java
user.set_password(new_password)
```

In the message of a taint-mode rule, you can refer to any metavariable bound by `pattern-sinks`, as well as to any metavariable bound by `pattern-sources` that does not conflict with a metavariable bound by `pattern-sinks`.

The resulting message is:
>Setting a password on user

**deep expression operator**
The deep expression operator works in:

- `if` statements: `if <... $X ...>:`
- nested calls: `sql.query(<... $X ...>)`
- operands of a binary expression: `"..." + <... $X ...>`
- any other expression context

### Rule syntax

Required fields are the one below:

```yaml
rules:
  - id: untitled_rule # Unique, descriptive identifier, for example: no-unused-variable
    languages: # the languages for which the rule has been created and the files that will be scanned
      - php
    severity: ERROR # One of the following values: INFO (Low severity), WARNING (Medium severity), or ERROR (High severity). The severity key specifies how critical are the issues that a rule potentially detects.
    message: | # Message that includes why Semgrep matched this pattern and how to remediate it
      Semgrep found a match
    pattern:
    patterns:
    pattern-either:
    pattern-regex:
```

>[!info]
> Only one of the following is required: `pattern`, `patterns`, `pattern-either`, `pattern-regex`

Other optional fields are:
```yaml
options: # Options object to enable/disable certain matching features. https://semgrep.dev/docs/writing-rules/rule-syntax#options
fix: # Simple search-and-replace autofix functionality
metadata: # Arbitrary user-provided data
paths: # Paths to include or exclude when running this rule
	exclude:  
		- "**/*.jinja2"  
		- "*_test.go"
	include:
		- "*_test.go"  
		- "project/server"
# only inside patterns or pattern-either
pattern-inside: # Keep findings that lie inside this pattern

# Only inside patterns
metavariable-regex: # Search metavariables for Python re compatible expressions; regex matching is left anchored
metavariable-pattern: # Matches metavariables with a pattern formula
metavariable-comparison: # Compare metavariables against basic Python expressions
pattern-not: # Logical NOT, remove findings matching this expression
pattern-not-inside: # Keep findings that do not lie inside this pattern
pattern-not-regex: # Filter results using a PCRE2-compatible pattern in multiline mode
```

Patterns operators:
- `pattern`: looks for code matching its expression
- `pattern-not`: is the opposite of the `pattern` operator. It finds code that does not match its expression. This is useful for eliminating common false positives.
- `patterns`: performs a logical AND operation on one or more **child patterns**
- `pattern-either`: performs a logical OR operation on one or more **child patterns**
- `pattern-regex`: searches files for substrings matching the given PCRE2 pattern
- `pattern-inside`: keeps matched findings that reside within its expression. This is useful for finding code inside other pieces of code like functions or if blocks.
- `pattern-not-inside`: operator keeps matched findings that do not reside within its expression. It is the opposite of `pattern-inside`. This is useful for finding code that’s missing a corresponding cleanup action like disconnect, close, or shutdown. It’s also useful for finding problematic code that isn't inside code that mitigates the issue.
- `pattern-not-regex`: filters results using a PCRE2 regular expression in multiline mode

Metavariable operators:
- `focus-metavariable`: puts the focus on the code region matched by a single metavariable or a list of metavariables
	- In short, `focus-metavariable: $X` is not a pattern in itself, it does not perform any matching, it only focuses the matching on the code already bound to $X by other patterns.
- `metavariable-regex`: searches metavariables for a PCRE2 regular expression. This is useful for filtering results based on a metavariable’s value
	- It requires the `metavariable` and `regex` keys
- `metavariable-pattern`: matches metavariables with a pattern formula. This is useful for filtering results based on a metavariable’s value
	- It requires the `metavariable` key and exactly one key of `pattern`, `patterns`, `pattern-either`, or `pattern-regex`. 
- `metavariable-comparison` : compares metavariables against a basic Python comparison expression. This is useful for filtering results based on a metavariable's numeric value

`patterns` operator evaluation strategy:
1. Semgrep evaluates all _positive_ patterns
2. Semgrep evaluates all _negative_ patterns
3. Semgrep evaluates all _conditionals
4. Semgrep applies all focus-metavariables_

### Taint Mode

Semgrep also supports taint [^taint] mode featuring constant propagation and taint-tracking [^data-flow]. Taint mode can be activated with the special option `mode: taint`, which enables new operators:

```yaml
mode: taint
options:
  # Report findings on the sources
  taint_focus_on: source/sink
  # Minimize false positives
  taint_assume_safe_booleans: true/false
  taint_assume_safe_numbers: true/false
  taint_assume_safe_indexes: true/false
  taint_assume_safe_functions: true/false
  # Inter file analyses
  interfile: true/false


pattern-sources: # required
	- pattern:
	- patterns:
		- pattern: make_tainted($X)
		- focus-metavariable: $X # Note that the use of `focus-metavariable: $X` is very important, and using `pattern: $X` is not equivalent. With `focus-metavariable: $X`, Semgrep matches the formal parameter exactly.
		by-side-effect: true/only
	- patter-either:
	- patter-regex:

pattern-sinks: # required
	- pattern: sink(...)
		exact: false/true
	- pattern-either:
	  - pattern: return ...
	  - pattern: $F(...)
	  at-exit: true # At-exit sinks are meant to facilitate writing leak-detection rules using taint mode. you can restrict a sink specification to only match at "exit" statements, that is statements after which the control-flow will exit the function being analyzed.

pattern-propagators: # By default, tainted data automatically propagates through assignments, operators, and function calls (from inputs to output). However, there are other ways in which taint can propagate, which can require language or library-specific knowledge that Semgrep does not have built-in.
	- pattern: strcpy($DST, $SRC)
	  from: $SRC
	  to: $DST

pattern-sanitizers:
  - patterns:
      - pattern: check_if_safe($X)
      - focus-metavariable: $X
    by-side-effect: true # If the sanitizer does not set by-side-efect, then only the very occurrence of x in check_if_safe(x) will be sanitized, but not the occurrence of x in sink(x). The sanitizer specification matches only the first occurrence and, without by-side-effect: true, Semgrep does not know that check_if_safe is updating/sanitizing the variable x by side-effect. Thus, a taint rule using such specification does produce a finding for sink(x) in the example above.
```

Those operators acts as `pattern-either` operators. They take a list of patterns that specify what is considered a source, a propagator, a sanitizer, or a sink. Note that you can use **any** pattern operator and you have the same expressive power as in a `mode: search` rule.


[^taint]: https://semgrep.dev/docs/writing-rules/data-flow/taint-mode/
[^data-flow]: https://semgrep.dev/docs/writing-rules/data-flow/data-flow-overview

## Testing rules

Semgrep provides a convenient testing mechanism for your rules. You can simply write code and provide a few annotations to let Semgrep know where you are or aren't expecting findings.

- `ruleid: <rule-id>`, for protecting against false negatives
- `ok: <rule-id>` for protecting against false positives
- `todoruleid: <rule-id>` for future "positive" rule improvements
- `todook: <rule-id>` for future "negative" rule improvements

Other than annotations there are three things to remember when creating tests:

1. The `--test` flag tells Semgrep to run tests in the specified directory.
2. Annotations are specified as a comment above the offending line.
3. Semgrep looks for tests based on the rule filename and the languages specified in the rule. In other words, `path/to/rule.yaml` searches for `path/to/rule.py`, `path/to/rule.js` and similar, based on the languages specified in the rule.

```py
from lib import get_user_input, safe_get_user_input, secure_eval

user_input = get_user_input()
# ruleid: insecure-eval-use
eval(user_input)

# ok: insecure-eval-use
eval('print("Hardcoded eval")')

totally_safe_eval = eval
# todoruleid: insecure-eval-use
totally_safe_eval(user_input)

# todook: insecure-eval-use
eval(safe_get_user_input())
```

```yaml
rules:
- id: insecure-eval-use
  patterns:
  - pattern: eval($VAR)
  - pattern-not: eval("...")
  fix: secure_eval($VAR)
  message: Calling 'eval' with user input
  languages: [python]
  severity: WARNING
```

```sh
$ semgrep --validate --config [filename]
$ semgrep --test rules/
1/1: ✓ All tests passed
No tests for fixes found.
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