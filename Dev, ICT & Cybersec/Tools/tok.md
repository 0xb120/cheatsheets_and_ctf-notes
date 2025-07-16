---
URL: https://github.com/tomnomnom/hacks/tree/master/tok
Description:
---
>[!summary]
>Like [strings](../Dev,%20scripting%20&%20OS/Linux%20command%20cheatsheet.md#strings) but more powerful

### Usage

```bash
% tok -h             
Usage of tok:
  -alpha-num-only
    	return only strings containing at least one letter and one number
  -delim-exceptions string
    	don't use the characters provided as delimiters
  -max int
    	max length of string to be output (default 25)
  -min int
    	min length of string to be output (default 1)
```

#### Extractions

```bash
 % cat www.sateltrack.com/osint.txt | tok
http
sateltrack
com
80
css
cssMainIE
...

% cat www.sateltrack.com/osint.txt | tok | sort | uniq -c | sort -nr | head
    869 80
    840 sateltrack
    840 com
    823 http
```