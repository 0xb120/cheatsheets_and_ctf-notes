---
Description: A command line search tool for Exploit-DB that also allows you to take a copy of Exploit Database with you, everywhere you go.
---

>[!info]
>Contents are placed within two main directories: `/usr/share/exploitdb/exploits` and `/usr/share/exploitdb/shellcodes`.

```bash
┌──(kali㉿kali)-[~]
└─$ searchsploit
  Usage: searchsploit [options] term1 [term2] ... [termN]

==========
 Examples 
==========
  searchsploit afd windows local
  searchsploit -t oracle windows
  searchsploit -p 39446
  searchsploit linux kernel 3.2 --exclude="(PoC)|/dos/"
  searchsploit -s Apache Struts 2.0.0
  searchsploit linux reverse password
  searchsploit -j 55555 | json_pp

  For more examples, see the manual: https://www.exploit-db.com/searchsploit

=========
 Options 
=========
## Search Terms
   -c, --case     [Term]      Perform a case-sensitive search (Default is inSEnsITiVe)
   -e, --exact    [Term]      Perform an EXACT & order match on exploit title (Default is an AND match on each term) [Implies "-t"]
                                e.g. "WordPress 4.1" would not be detect "WordPress Core 4.1")
   -s, --strict               Perform a strict search, so input values must exist, disabling fuzzy search for version range
                                e.g. "1.1" would not be detected in "1.0 < 1.3")
   -t, --title    [Term]      Search JUST the exploit title (Default is title AND the file\'s path)
       --exclude="term"       Remove values from results. By using "|" to separate, you can chain multiple values
                                e.g. --exclude="term1|term2|term3"

## Output
   -j, --json     [Term]      Show result in JSON format
   -o, --overflow [Term]      Exploit titles are allowed to overflow their columns
   -p, --path     [expDB-ID]    Show the full path to an exploit (and also copies the path to the clipboard if possible)
   -v, --verbose              Display more information in output
   -w, --www      [Term]      Show URLs to Exploit-DB.com rather than the local path
       --id                   Display the expDB-ID value rather than local path
       --colour               Disable colour highlighting in search results

## Non-Searching
   -m, --mirror   [expDB-ID]    Mirror (aka copies) an exploit to the current working directory
   -x, --examine  [expDB-ID]    Examine (aka opens) the exploit using $PAGER

## Non-Searching
   -h, --help                 Show this help screen
   -u, --update               Check for and install any exploitdb package updates (brew, deb & git)

## Automation
       --nmap     [file.xml]  Checks all results in Nmap's XML output with service version e.g.: nmap [host] -sV -oX file.xml
```

>[!note]
>- You can use any number of search terms.
>- Search terms are **not case-sensitive **(by default), and ordering is irrelevant.
>- Use `c` if you wish to reduce results by **case-sensitive** searching.
>- And/Or `e` if you wish to filter results by using an **exact match**.
>- Use `t` to **exclude the file's path** to filter the search results.
>- Remove false positives (especially when searching using numbers - i.e. versions).
>- When updating or displaying help, search terms will be ignored.