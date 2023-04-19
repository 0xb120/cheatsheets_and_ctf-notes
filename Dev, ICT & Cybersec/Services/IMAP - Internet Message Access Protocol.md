---
Ports: 143, 993
Description: Is an Internet standard protocol used by email clients to retrieve email messages from a mail server over a TCP/IP connection.
---

>[!info]
> As its name implies, IMAP allows you to **access your email messages wherever you are**; much of the time, it is accessed via the Internet. Basically, email **messages are stored on servers**. Whenever you check your inbox, your email client contacts the server to connect you with your messages. When you read an email message using IMAP, **you aren't actually downloading** or storing it on your computer; instead, you are **reading it off of the server**. As a result, it's possible to check your email from **several different devices** without missing a thing.


# Enumeration


## Banner grabbing

- [netcat](../Tools/netcat.md)
- openssl

```bash
nc -nv <IP> 143
openssl s_client -connect <IP>:993 -quiet
```

## NTLM Auth information disclosure

```bash
root@kali: telnet example.com 143 
* OK The Microsoft Exchange IMAP4 service is ready. 
>> a1 AUTHENTICATE NTLM 
+ 
>> TlRMTVNTUAABAAAAB4IIAAAAAAAAAAAAAAAAAAAAAAA= 
+ TlRMTVNTUAACAAAACgAKADgAAAAFgooCBqqVKFrKPCMAAAAAAAAAAEgASABCAAAABgOAJQAAAA9JAEkAUwAwADEAAgAKAEkASQBTADAAMQABAAoASQBJAFMAMAAxAAQACgBJAEkAUwAwADEAAwAKAEkASQBTADAAMQAHAAgAHwMI0VPy1QEAAAAA
```

## Automatic Enumeration

- [NSE (Nmap Scripting Engine)](../Tools/nmap.md#NSE%20(Nmap%20Scripting%20Engine)): `imap-ntlm-info.nse`

---

# Exploitation

## Brute Force

- [hydra](../Tools/hydra.md#IMAP%20Brute-Force)
- [NSE (Nmap Scripting Engine)](../Tools/nmap.md#NSE%20(Nmap%20Scripting%20Engine))

```bash
nmap -sV --script imap-brute -p <PORT> <IP>
```

---

# Misc

## Synatx

From [here](https://donsutherland.org/crib/imap)

```bash
# Login
    A1 LOGIN username password
# Values can be quoted to enclose spaces and special characters. A " must then be escape with a \
    A1 LOGIN "username" "password"

# List Folders/Mailboxes
    A1 LIST "" *
    A1 LIST INBOX *
    A1 LIST "Archive" *

# Create new Folder/Mailbox
    A1 CREATE INBOX.Archive.2012
    A1 CREATE "To Read"

# Delete Folder/Mailbox
    A1 DELETE INBOX.Archive.2012
    A1 DELETE "To Read"

# Rename Folder/Mailbox
    A1 RENAME "INBOX.One" "INBOX.Two"

# List Subscribed Mailboxes
    A1 LSUB "" *

# Status of Mailbox (There are more flags than the ones listed)
    A1 STATUS INBOX (MESSAGES UNSEEN RECENT)

# Select a mailbox
    A1 SELECT INBOX

# List messages
    A1 FETCH 1:* (FLAGS)
    A1 UID FETCH 1:* (FLAGS)

# Retrieve Message Content
    A1 FETCH 2 body[text]
    A1 FETCH 2 all
    A1 UID FETCH 102 (UID RFC822.SIZE BODY.PEEK[])

# Close Mailbox
    A1 CLOSE

# Logout
    A1 LOGOUT
```

## CURL interaction

Basic navigation is possible with [CURL](https://ec.haxx.se/usingcurl/usingcurl-reademail#imap), but the documentation is light on details so checking the [source](https://github.com/curl/curl/blob/master/lib/imap.c) is recommended for precise details.

1. Listing mailboxes (imap command `LIST "" "*"`)

```bash
$ curl -k 'imaps://1.2.3.4/' --user user:pass
```

1. Listing messages in a mailbox (imap command `SELECT INBOX` and then `SEARCH ALL`)

```bash
$ curl -k 'imaps://1.2.3.4/INBOX?ALL' --user user:pass
$ curl -k 'imaps://1.2.3.4/Drafts?TEXT password' --user user:pass
```

1. Downloading a message (imap command `SELECT Drafts` and then `FETCH 1 BODY[]`)

```bash
 $ curl -k 'imaps://1.2.3.4/Drafts;MAILINDEX=1' --user user:pass
```

It is also possible to use `UID` (unique id) to access messages, however it is less convenient as the search command needs to be manually formatted. E.g.

```bash
$ curl -k 'imaps://1.2.3.4/INBOX' -X 'UID SEARCH ALL' --user user:pass
$ curl -k 'imaps://1.2.3.4/INBOX;UID=1' --user user:pass
```

Also, possible to download just parts of a message, e.g. subject and sender of first 5 messages (the `-v` is required to see the subject and sender):

```bash
$ curl -k 'imaps://1.2.3.4/INBOX' -X 'FETCH 1:5 BODY[HEADER.FIELDS (SUBJECT FROM)]' --user user:pass -v 2>&1 | grep '^<'
```

Although, its probably cleaner to just write a little for loop:

```bash
for m in {1..5}; do
  echo $m
  curl "imap://1.2.3.4/INBOX;MAILINDEX=$m;SECTION=HEADER.FIELDS%20(SUBJECT%20FROM)" --user user:pass
done
```