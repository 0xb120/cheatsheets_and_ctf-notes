---
Description: Discover vulnerabilities across a codebase with _CodeQL_, our industry-leading semantic code analysis engine. _CodeQL_ lets you query code as though it were data.
URL: https://github.com/github/codeql
---

>[!abstract] Official documentation
>- General CodeQL: https://codeql.github.com/docs/
>- CodeQL CLI: https://docs.github.com/en/code-security/codeql-cli/using-the-codeql-cli

## Start a project using CodeQL CLI

1. Download the CodeQL binary from the [official github repo](https://github.com/github/codeql-cli-binaries/releases).
2. Build the CodeQL database starting from the actual application to test
   ```bash
   ./codeql database create /opt/codeql_db -j 4 --language=python --source-root=/opt/app --overwrite
   ```
3. Analyze the application using some basic python queries:
   ```bash
   ./codeql database analyze --download --format=CSV --output=/opt/results/codeql_results.csv /opt/codeql_db codeql/python-queries
   ```


## Custom queries

```cardlink
url: https://spaceraccoon.dev/comparing-rule-syntax-codeql-semgrep/
title: "Rule Writing for CodeQL and Semgrep"
description: "One common perception is that it is easier to write rules for Semgrep than CodeQL. Having worked extensively with both of these static code analysis tools for about a year, I have some thoughts."
host: spaceraccoon.dev
image: https://spaceraccoon.dev/images/26/codeql-vs-code-ast.png
```


```bash
# Initializate a codeql pack
./codeql pack install ~/Projects/codeql-prj/queries/
```

```yml
# basic codeql pack for python
name: foo
groups: 
  - python
dependencies:
  codeql/python-all: "*"
```

```bash
# Install dependencies for the codeql pack
./codeql pack install ~/Projects/codeql-prj/queries/

# Run the custom query
./codeql query run ~/Projects/codeql-prj/queries/query2.ql -d ~/Projects/codeql-prj/db
```

