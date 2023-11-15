Server-Side Includes are used to create dynamic content on HTML pages before loading or during the rendering process by evaluating SSI directives. SSI injection vulnerabilities are very similar to [Server Side Template Injection (SSTI)](Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Server%20Side%20Template%20Injection%20(SSTI).md).

SSI injection vulnerabilities can typically be exploited to inject arbitrary content, including JavaScript, into the application's response, with the same impact as cross-site scripting. Depending on the server configuration, it may also be possible to read protected files, or perform arbitrary code execution on the server, with the same impact as OS command injection.

Common extensions implementing SSI are:
- `.shtml`
- `.shtm`
- `.stm`

Some SSI examples are:
```html
// Date
<!--#echo var="DATE_LOCAL" -->

// Modification date of a file
<!--#flastmod file="index.html" -->

// CGI Program results
<!--#include virtual="/cgi-bin/counter.pl" -->

// Including a footer
<!--#include virtual="/footer.html" -->

// Executing commands
<!--#exec cmd="ls" -->

// Setting variables
<!--#set var="name" value="Rich" -->

// Including virtual files (same directory)
<!--#include virtual="file_to_include.html" -->

// Including files (same directory)
<!--#include file="file_to_include.html" -->

// Print all variables
<!--#printenv -->
```

More can be found on [HackTricks](https://book.hacktricks.xyz/pentesting-web/server-side-inclusion-edge-side-inclusion-injection#check)