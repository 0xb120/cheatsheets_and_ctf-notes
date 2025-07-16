---
URL: https://github.com/tomnomnom/unfurl
Description: Pull out bits of URLs provided on stdin
---
>[!summary]
>Pull out bits of URLs provided on stdin

### Usage

```bash
% unfurl
unknown mode: 
Format URLs provided on stdin

Usage:
  unfurl [OPTIONS] [MODE] [FORMATSTRING]

Options:
  -u, --unique   Only output unique values
  -v, --verbose  Verbose mode (output URL parse errors)

Modes:
  keys     Keys from the query string (one per line)
  values   Values from the query string (one per line)
  keypairs Key=value pairs from the query string (one per line)
  domains  The hostname (e.g. sub.example.com)
  paths    The request path (e.g. /users)
  apexes   The apex domain (e.g. example.com from sub.example.com)
  json     JSON encoded url/format objects
  format   Specify a custom format (see below)
```

#### Extractions

Extract domains:
```bash
$ cat www.sateltrack.com/osint.txt | unfurl domains | sort -u
sateltrack.com
www.sateltrack.com
```

Extract URL paths:
```bash
$ cat www.sateltrack.com/osint.txt | unfurl paths | sort -u
/
/Ciprian.Suta@cefin.hu
/css/cssGeneric.css
/css/cssMainIE.css
/css/cssStaticSite.css
/download/sateltrack_features_en.pdf
/download/sateltrack_features_it.pdf
/favicon.ico
...
```

Extract parameter fields:
```bash
$ cat www.sateltrack.com/osint.txt | unfurl keys | sort -u
amp
call
days
email
h
idref
idVeh
ioa
lang
load
mode
page
pageref
partnerid
password
res
scale
SESSIONID
showdetails
trlang
ved
```