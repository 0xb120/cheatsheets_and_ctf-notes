---
Description: Discover vulnerabilities across a codebase with _CodeQL_, our industry-leading semantic code analysis engine. _CodeQL_ lets you query code as though it were data.
URL: https://github.com/github/codeql
url: https://frycos.github.io/vulns4free/2022/12/02/rce-in-20-minutes.html
---

>[!abstract] Documentation
>- General CodeQL: https://codeql.github.com/docs/
>- CodeQL CLI: https://docs.github.com/en/code-security/codeql-cli/using-the-codeql-cli
>- [CodeQL tutorial](https://appsec.guide/docs/static-analysis/codeql/) by Trail of Bits; appsec.guide

## CodeQL CLI

1. Download the CodeQL binary from the [official github repo](https://github.com/github/codeql-cli-binaries/releases).
2. Build the CodeQL database starting from the actual application to test
```bash
./codeql database create /opt/codeql_db -j 4 --language=python --source-root=/opt/app --overwrite
```
1. Analyze the application using some basic python queries:
```bash
./codeql database analyze --download --format=CSV --output=/opt/results/codeql_results.csv /opt/codeql_db codeql/python-queries
```

## CodeQL from [vscode](vscode.md)

1. **Install CodeQL CLI**
	1. Grab the latest `codeql-linux64.zip` (or your OS equivalent) from the [CodeQL GitHub releases](https://github.com/github/codeql-cli-binaries/releases).
	2. Place it in a dedicated directory, e.g., `$HOME/codeql-home/codeql-cli`
	3. Add the binary to your system PATH:  `export PATH=$PATH:$HOME/codeql-home/codeql-cli`
	4. Run `codeql --version` to ensure the binary is executable.
2. **Clone the CodeQL Standard Libraries**
```sh
cd $HOME/codeql-home
git clone https://github.com/github/codeql.git codeql-repo
```
3. **Extension configuration
	1. Open VSCode and go to the **Extensions** view (`Ctrl+Shift+X`).
	2. Search for and install **CodeQL**
	3. Search for `codeql.cli.executablePath` and set it to the absolute path of your binary: `/home/user/codeql-home/codeql-cli/codeql`.
4. **Create the CodeQL DB**:
```sh
codeql database create ./my-db --language=python --source-root=./my-source-code
```
5. **Open your CodeQL query packs folder** inside the same project and run them
## CodeQL queries and Packs

### Queries

CodeQL treats code like data. When you "build" a database, CodeQL transforms the source code into a relational database representing the **Abstract Syntax Tree (AST)**, Control Flow Graph (CFG), and Data Flow Graph.

A `.ql` file uses a declarative syntax similar to SQL but optimized for tree structures.

```c
/** * Metadata: Tells the engine how to display results
 * @name Hardcoded Secret
 * @kind problem
 * @id cs/hardcoded-secret
 */

import csharp // 1. Import the language library

from StringLiteral s // 2. "From" defines the type of data (Source)
where s.getValue().regexpMatch(".*[Pp]assword.*") // 3. "Where" defines the logic
select s, "This string looks like a hardcoded password." // 4. "Select" outputs the finding
```

Key Concepts:
- **Predicates:** Think of these as "reusable mini-functions" that return true or false for a given piece of code.
- **Classes:** You can define custom classes to represent specific code patterns (e.g., "All methods that handle HTTP requests").
- **Data Flow:** This is the most powerful part. You can track "tainted" input (e.g., a URL parameter) as it flows through the application into a "sink" (e.g., a SQL query).

### Packs

A **Query Pack** is essentially a package manager for CodeQL (similar to `npm` for Node or `NuGet` for C#). It tells the CodeQL engine where to find the libraries (like `import csharp`) and how to handle dependencies.

Every pack must have this file at its root. It defines the pack's identity and its requirements.
```yaml title=qlpack.yaml
name: my-redteam-queries/csharp-exploits
version: 1.0.0
dependencies:
  codeql/csharp-all: "*"  # Links to the official C# logic libraries
  codeql/csharp-queries: "*" # Links to the official C# default queries
```

Then, you must install the dependencies: `codeql pack install`

Types of Packs:
1. **Query Packs:** Contain the actual `.ql` files you run to find bugs.
2. **Library Packs:** Contain `.qll` files (reusable logic) that other queries import, but they don't produce results on their own.

### Develop and run custom queries

1. **Initialize a Custom Query Pack**: CodeQL requires a `qlpack.yml` file to manage dependencies. Without this, it won't know how to resolve `import csharp`.
	1. **Create a directory** for your research
	2. Create the `qlpack.yml`
	3. **Install the dependencies**: `codeql pack install`
2. **Write the Query (`.ql`)**
3. **Run the query**
 ```sh
 codeql query run FindSensitiveLogging.ql \
    --database=../my-db \
    --output=./results.bqrs
 ```


Standard libraries, queries, and packs are stored at https://github.com/github/codeql
- The Standard Libraries (`.qll` files)
- The Standard Queries (`.ql` files)
- CodeQL Query Packs (`qlpack.yml`)

You can also develop tailored custom queries:
```cardlink
url: https://spaceraccoon.dev/comparing-rule-syntax-codeql-semgrep/
title: "Rule Writing for CodeQL and Semgrep"
description: "One common perception is that it is easier to write rules for Semgrep than CodeQL. Having worked extensively with both of these static code analysis tools for about a year, I have some thoughts."
host: spaceraccoon.dev
image: https://spaceraccoon.dev/images/26/codeql-vs-code-ast.png
```

# Other resources
- [CodeQL-Community-Packs](https://github.com/GitHubSecurityLab/CodeQL-Community-Packs/) [^1] - Collection of community-driven CodeQL query, library and extension packs
- [Pre-Auth RCE With CodeQL in Under 20 Minutes](../../Readwise/Articles/Frycos%20Security%20Diary%20-%20Pre-Auth%20RCE%20With%20CodeQL%20in%20Under%2020%20Minutes.md), frycos.github.io
- [Jorge Rosillo - Security Research Without Ever Leaving GitHub From Code Scanning to CVE via Codespaces and Private Vulnerability Reporting](../../Readwise/Articles/Jorge%20Rosillo%20-%20Security%20Research%20Without%20Ever%20Leaving%20GitHub%20From%20Code%20Scanning%20to%20CVE%20via%20Codespaces%20and%20Private%20Vulnerability%20Reporting.md)


[^1]: [Erik - Last Week in Security (LWiS) - 2025-01-06](../../Readwise/Articles/Erik%20-%20Last%20Week%20in%20Security%20(LWiS)%20-%202025-01-06.md#^994fd4)