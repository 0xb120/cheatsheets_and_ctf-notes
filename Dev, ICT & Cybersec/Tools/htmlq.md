---
URL: https://github.com/mgdm/htmlq
Description: Like jq, but for HTML.
---
>[!Summary]
>Like [jq](jq.md), but for HTML.

### Usage

```bash
$ htmlq -h
htmlq 0.4.0
Michael Maclean <michael@mgdm.net>
Runs CSS selectors on HTML

USAGE:
    htmlq [FLAGS] [OPTIONS] [--] [selector]...

FLAGS:
    -B, --detect-base          Try to detect the base URL from the <base> tag in the document. If not found, default to
                               the value of --base, if supplied
    -h, --help                 Prints help information
    -w, --ignore-whitespace    When printing text nodes, ignore those that consist entirely of whitespace
    -p, --pretty               Pretty-print the serialised output
    -t, --text                 Output only the contents of text nodes inside selected elements
    -V, --version              Prints version information

OPTIONS:
    -a, --attribute <attribute>         Only return this attribute (if present) from selected elements
    -b, --base <base>                   Use this URL as the base for links
    -f, --filename <FILE>               The input file. Defaults to stdin
    -o, --output <FILE>                 The output file. Defaults to stdout
    -r, --remove-nodes <SELECTOR>...    Remove nodes matching this expression before output. May be specified multiple
                                        times

ARGS:
    <selector>...    The CSS expression to select [default: html]
```

#### Extractions

```bash
% cat 0xbro.html| htmlq --attribute href a | sort -u
/
/about/
/achievements/
/cheatsheets/File%20Transfer/
/cheatsheets/Mobile%20hacking%20cheatsheets/
/cheatsheets/My%20Notes/
...
```