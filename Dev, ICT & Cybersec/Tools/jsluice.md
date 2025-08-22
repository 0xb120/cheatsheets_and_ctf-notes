---
Description: Extract URLs, paths, secrets, and other interesting bits from JavaScript
URL: https://github.com/BishopFox/jsluice
---
>[!summary]
`jsluice` is a Go package and [command-line tool](https://github.com/BishopFox/jsluice/blob/main/cmd/jsluice) for extracting URLs, paths, secrets, and other interesting data from JavaScript source code.

### Usage

```sh
jsluice - Extract URLs, paths, and secrets from JavaScript files

Usage:
  jsluice <mode> [options] [file...]

Modes:
  urls      Extract URLs and paths
  secrets   Extract secrets and other interesting bits
  tree      Print syntax trees for input files
  query     Run tree-sitter a query against input files
  format    Format JavaScript source using jsbeautifier-go
```

### Examples

```sh
# Local and Remote JS analyses 
jsluice urls -C 'auth=true; user=admin;' -H 'Specific-Header-One: true' -H 'Specific-Header-Two: false' local_file.js https://remote.host/example.js

# Local JS query
jsluice query -q '(object) @m' one.js two.js

# Pattern extraction from a list of files
find . -name '*.js' | jsluice secrets -c 5 --patterns=apikeys.json
```

