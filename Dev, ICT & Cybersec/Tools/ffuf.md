---
Description: Fast web fuzzer written in Go
URL: https://github.com/ffuf/ffuf
---

## Examples

```bash
#Fuzz file paths from wordlist.txt, match all responses but filter out those with content-size 42. Colored, verbose output.
$ ffuf -w wordlist.txt -u https://example.org/FUZZ -mc all -fs 42 -c -v

# Fuzz Host-header, match HTTP 200 responses.
$ ffuf -w hosts.txt -u https://example.org/ -H "Host: FUZZ" -mc 200

# Fuzz POST JSON data. Match all responses not containing text "error".
$ ffuf -w entries.txt -u https://example.org/ -X POST -H "Content-Type: application/json" \
  -d '{"name": "FUZZ", "anotherkey": "anothervalue"}' -fr "error"

# Pitchfork mode scan
$ ffuf -mode pitchfork -u https://streamio.htb/login.php -d "username=UFUZZ&PASSWORD=PFUZZ" -w loot/users.txt:UFUZZ -w loot/passwords.txt:PFUZZ

# Fuzz every endpoint with multiple HTTP methods
$ ffuf -u https://api.example.com/PATH -X METHOD -w /path/to/wordlist:PATH -w /path/to/http_methods:METHOD
```

## Useful scans

- [Virtual host (VHost) enumeration](../../Readwise/Articles/blackbird-eu%20-%207%20Overlooked%20Recon%20Techniques%20to%20Find%20More%20Vulnerabilities.md#Virtual%20host%20(VHost)%20enumeration)
- 