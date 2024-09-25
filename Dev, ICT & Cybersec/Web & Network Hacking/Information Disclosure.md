Information disclosure, also known as **information leakage**, is when a website unintentionally reveals sensitive information to its users.
Although some of this information will be of limited use, **it can potentially be a starting point for exposing an additional attack surface, which may contain other interesting vulnerabilities.**

>[!important]
>Information disclosure vulnerabilities are a gold cave for complex vulnerability chains!

Three main macro-categories:

-   Data about other users, such as usernames or financial information
-   Sensitive commercial or business data
-   Technical details about the website and its infrastructure
	- [0xdf hacks stuff - Default 404 Pages](../../Readwise/Articles/0xdf%20hacks%20stuff%20-%20Default%20404%20Pages.md)

## Basic information disclosure examples

- Revealing the names of hidden directories, their structure, and their contents via a `robots.txt` file or directory listing
- Providing access to source code files via temporary backups
- Explicitly mentioning database table or column names in error messages
- Unnecessarily exposing highly sensitive information, such as credit card details
- Hard-coding API keys, IP addresses, database credentials, and so on in the source code
- Hinting at the existence or absence of resources, usernames, and so on via subtle differences in application behavior
- Disclosing **secrets** used for sensitive functions (eg. CSRF-token generation, password-reset tokens, authorization checks, etc.) [^secret-fragments]

[^secret-fragments]: [Ambionics - Secret Fragments Remote Code Execution on Symfony Based Websites](../../Readwise/Articles/Ambionics%20-%20Secret%20Fragments%20Remote%20Code%20Execution%20on%20Symfony%20Based%20Websites.md)

### How can they arise?

- Failure to remove internal content from public content
- Insecure configuration of the website and related technologies
- Flawed design and behavior of the application
	- [File Inclusion (LFI & RFI)](File%20Inclusion%20(LFI%20&%20RFI).md)
	- [Access control vulnerabilities](Access%20control%20vulnerabilities.md)

## How to test for information disclosure vulnerabilities

>[!important]
> It is important not to develop "tunnel vision" during testing. Sensitive data can be leaked in all kinds of places, so it is important not to miss anything that could be useful later. Key skill: being able to recognize interesting information whenever and wherever you do come across it.

- [Fuzzing](../Reversing%20&%20Binary%20Exploitation/Fuzzing.md): try submitting unexpected data types and specially crafted fuzz strings to see what effect this has. Pay close attention to responses size, contents and time taken to process the request.
- Using scanners and other auditing tools
- Searching them actively using filters, grep, ecc.
- Engineering informative responses: submitting an invalid parameter value might lead to a stack trace or debug response that contains interesting details. You can sometimes cause error messages to disclose the value of your desired data in the response.

### Common sources of information disclosure

- Files for web crawlers (`/robots.txt`, `/sitemap.xml`, etc.)
- Directory listing → `intext: "Index of"`
- Developer comments
- Error messages → `intext: error filetype: log`
- Debugging data
- User account page (combined with [Business logic vulnerabilities](Business%20logic%20vulnerabilities.md) and [Access control vulnerabilities](Access%20control%20vulnerabilities.md))
- Source code disclosure (backup files, templates, temp files, etc.)
- Insecure configurations (TRACE method, third-parties configurations, etc.)
- Version control history (`.git`, etc.)
