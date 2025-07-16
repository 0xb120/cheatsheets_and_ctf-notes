---
Description: Take URLs or filenames for HTML documents on stdin and extract tag contents, attribute values, or comments.
URL: https://github.com/tomnomnom/hacks/tree/master/html-tool
---
>[!summary]
>Take URLs or filenames for HTML documents on stdin and extract tag contents, attribute values, or comments.

### Usage

```bash
▶ html-tool 
Accept URLs or filenames for HTML documents on stdin and extract parts of them.

Usage: html-tool <mode> [<args>]

Modes:
	tags <tag-names>        Extract text contained in tags
	attribs <attrib-names>  Extract attribute values
	comments                Extract comments

Examples:
	cat urls.txt | html-tool tags title a strong
	find . -type f -name "*.html" | html-tool attribs src href
	cat urls.txt | html-tool comments
```

#### Extract specific tags from download files

```bash
▶ echo 0xbro.html | html-tool tags title
Home | 0xbro
Link
Menu
Expand
(external link)
Document
Search
Copy
Copied

▶ echo 0xbro.html | html-tool tags script
{"@context":"https://schema.org","@type":"WebSite","description":"0xbro personal site and blog related to everything that surrounds ethical hacking, penetration testing, AppSec, CTFs, and other various cybersecurity stuff.","headline":"Home","name":"0xbro","publisher":{"@type":"Organization","logo":{"@type":"ImageObject","url":"https://0xbro.red/assets/images/0xbro_minimal1.png"}},"url":"https://0xbro.red/"}
import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11.4.1/dist/mermaid.esm.min.mjs'; var config = {} ; mermaid.initialize(config); mermaid.run({ querySelector: '.language-mermaid', });
```

#### Extract specific attributes from download files

```bash
▶ echo 0xbro.html | html-tool attribs src
/assets/js/vendor/lunr.min.js
/assets/js/just-the-docs.js
https://i.creativecommons.org/l/by-sa/4.0/88x31.png

▶ echo 0xbro.html | html-tool attribs href
/assets/css/just-the-docs-default.css
/assets/css/just-the-docs-head-nav.css
/favicon.ico
https://0xbro.red/
#main-content
/
...
```

#### Extract comments from download files

```bash
▶ echo 0xbro.html | html-tool comments    
Begin Jekyll SEO tag v2.8.0
End Jekyll SEO tag
Feather. MIT License: https://github.com/feathericons/feather/blob/master/LICENSE
Bootstrap Icons. MIT License: https://github.com/twbs/icons/blob/main/LICENSE.md
Custom icons from https://icons.getbootstrap.com/icons/
```