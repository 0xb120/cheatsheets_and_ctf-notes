---
URL: https://github.com/tomnomnom/qsreplace
Description: Accept URLs on stdin, replace all query string values with a user-supplied value
---
>[!summary]
>Accept URLs on stdin, replace all query string values with a user-supplied value, only output each combination of query string parameters once per host and path.

### Usage

```bash
▶ cat urls.txt 
https://example.com/path?one=1&two=2
https://example.com/path?two=2&one=1
https://example.com/pathtwo?two=2&one=1
https://example.net/a/path?two=2&one=1

▶ cat urls.txt | qsreplace newval
https://example.com/path?one=newval&two=newval
https://example.com/pathtwo?one=newval&two=newval
https://example.net/a/path?one=newval&two=newval

▶ cat urls.txt | qsreplace -a newval
https://example.com/path?one=1newval&two=2newval
https://example.com/pathtwo?one=1newval&two=2newval
https://example.net/a/path?one=1newval&two=2newval
```

You can combine it with other tools to perform automated vulnerability detection!
- [qsreplace + ffuf](../../Readwise/Articles/ProjectDiscovery%20-%20Building%20a%20Fast%20One-Shot%20Recon%20Script%20for%20Bug%20Bounty.md#qsreplace%20+%20ffuf)