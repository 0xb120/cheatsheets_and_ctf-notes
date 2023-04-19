---
Ports: 119, 433, 563
Description: The Network News Transfer Protocol, allows clients to retrieve (read) and post (write) news articles to the NNTP server.
---

>[!info]
> The Network News Transfer Protocol, allows clients to retrieve (read) and post (write) news articles to the NNTP server. Since it relates to reading and writing, we may be able to find helpful information here.


# Basic Usage

- [NNTP Commands](http://www.networksorcery.com/enp/protocol/nntp.htm)

>[!tip]
>NNTP commands responses always end with a period (.) on a line by itself.

```bash
CAPABILITIES            List server capabilities.
HELP                    Show available commands.
MODE READER             Use Reader mode. Reader mode uses a lot of commands, use HELP.
LIST                    List groups.
SELECT <group>          Select group.
LISTGROUP <group>       List article in a group.
HEAD <article_id>       Retrieve article header.
BODY <article_id>       Retrieve article body.
ARTICLE <article_id>    Retrieve article.
POST                    Post article.
```


---

# Enumeration

- [netcat](../Tools/netcat.md)

```bash
root@kali:~$ nc -nvC 10.11.1.72 119
(UNKNOWN) [10.11.1.72] 119 (nntp) open
200 beta NNTP Service Ready, posting permitted
HELP
100 Help text follows
.
LIST
215 list of newsgroups follows
org.apache.avalon.dev 0 0 y
org.apache.avalon.user 0 0 y
org.apache.james.user 0 0 y
org.apache.james.dev 0 0 y
.
^C
root@kali:~#
```

- [NSE (Nmap Scripting Engine)](../Tools/nmap.md#NSE%20(Nmap%20Scripting%20Engine))

```bash
nmap -p 119,433,563 --script nntp-ntlm-info <target>
```