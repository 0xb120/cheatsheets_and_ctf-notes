---
Description: Directory/File, DNS and VHost busting tool written in Go
URL: https://github.com/OJ/gobuster
---

```bash
┌──(kali㉿kali)-[/opt/post-expl/windows/SysinternalsSuite]
└─$ gobuster
Usage:
  gobuster [command]

Available Commands:
  dir         Uses directory/file brutceforcing mode
  dns         Uses DNS subdomain bruteforcing mode
  help        Help about any command
  vhost       Uses VHOST bruteforcing mode

Flags:
  -h, --help              help for gobuster
  -z, --noprogress        Don't display progress
  -o, --output string     Output file to write results to (defaults to stdout)
  -q, --quiet             Don't print the banner and other noise
  -t, --threads int       Number of concurrent threads (default 10)
  -v, --verbose           Verbose output (errors)
  -w, --wordlist string   Path to the wordlist

Use "gobuster [command] --help" for more information about a command.
```

### dir examples

```bash
# Common directory enumeration
gobuster dir -u http://10.11.1.73:8080/php/ -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php,txt -t 20 -f -o phpDir.txt
```

### dns examples

```bash
# Common sub-domain enumeration
gobuster dns -d delivery.htb -w /usr/share/seclists/Discovery/Web-Content/common.txt -t 20 -o sub-domain.txt
```

### vhost examples

```bash
# Common virtual hosts enumeration
gobuster vhost -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -u http://dms-pit.htb/ -t 25 -o vhosts.txt
```