---
URL: https://github.com/tomnomnom/gf
Description: A wrapper around grep, to help you grep for things
---
>[!summary]
>A wrapper around grep to avoid typing common patterns.

### Usage

```bash
▶ gf php-sources

# where the php-sources configuration is
▶ cat ~/.gf/php-sources.json
{
    "flags": "-HnrE",
    "pattern": "(\\$_(POST|GET|COOKIE|REQUEST|SERVER|FILES)|php://(input|stdin))"
}

# Save new config
▶ gf -save php-serialized -HnrE '(a:[0-9]+:{|O:[0-9]+:"|s:[0-9]+:")'
```