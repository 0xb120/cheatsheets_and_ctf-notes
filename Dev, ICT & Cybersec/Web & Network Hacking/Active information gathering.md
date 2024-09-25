>[!abstract]
>Every information gathered **directly from** the target.

# Website recon

Gathering information simply **browsing the site**. Important sections are:

- Corporate **Emails** and **Members**
- "**Contact us**" pages
- "**Social Media**" or "Follow us" pages and sections

---

# Manual IP Extraction
- **IP extraction by email**If the web app uses some **web server's functions to send emails**, it will **expose the IP address** of the real server.Once we get the email from the web server (registration, newsletter, etc.) we will have to extract and examine the various **headers** and finally find the IP.

- **IP Extraction by Upload**If the web app allows to **upload images and retrieve the URL from external resources**, it is possible use some [IP-logger](https://iplogger.org/) to get the real IP. Once the script has been uploaded in the form of an image, an HTTP request will be sent to our resource, which will immediately grab the information from the headers, including the IP.

---

# Network Sweeping & Port Scanning [^1]

[^1]: [Type of scans (TCP & UDP)](Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Type%20of%20scans%20(TCP%20&%20UDP).md)

**Tools:**
- [netcat (port scanning script)](../Tools/netcat.md#Port%20scanning) (tool)
- [Bash port scanner](../Dev,%20scripting%20&%20OS/Bash%20101.md#Bash%20port%20scanner) (script)
- [nmap](../Tools/nmap.md) (tool)
- [masscan](../Tools/masscan.md) (tool)
- [Ping Sweep](../Dev,%20scripting%20&%20OS/Powershell%20for%20pentesters.md#Ping%20Sweep) (script)
- [fping](https://github.com/schweikert/fping) (tool)
- [scapy](https://scapy.net/) (tool)
- [AutoRecon](../Tools/AutoRecon.md) (script)
- [nmapAutomator sh](../Tools/nmapAutomator%20sh.md) (script)

---

# Client information gathering

- [Social Engineering](https://en.wikipedia.org/wiki/Social_engineering_(security))
- Client Fingerprinting
    - [fingerprintjs2](https://github.com/Valve/fingerprintjs2)
- [Online UserAgent databses](http://developers.whatismybrowser.com/)

```html
<!doctype html>
<html>
<head>
<title>Blank Page</title>
</head>
<body>
<h1>You have been given the finger!</h1>
<script src="fingerprint2.js"></script>
<script>
var d1 = new Date();
var options = {};
Fingerprint2.get(options, function (components) {
var values = components.map(function (component) { return component.value })
var murmur = Fingerprint2.x64hash128(values.join(''), 31)
var clientfp = "Client browser fingerprint: " + murmur + "\n\n";
var d2 = new Date();
var timeString = "Time to calculate fingerprint: " + (d2 - d1) + "ms\n\n";
var details = "Detailed information: \n";
if(typeof window.console !== "undefined") {
for (var index in components) {
var obj = components[index];
var value = obj.value;
if (value !== null) {
var line = obj.key + " = " + value.toString().substr(0, 150);
details += line + "\n";
}
}
}
var xmlhttp = new XMLHttpRequest();
xmlhttp.open("POST", "/fp/js.php");
xmlhttp.setRequestHeader("Content-Type", "application/txt");
xmlhttp.send(clientfp + timeString + details);
});
</script>
</body>
</html>
```

```php
<?php
$data = "Client IP Address: " . $_SERVER['REMOTE_ADDR'] . "\n";
$data .= file_get_contents('php://input');
$data .= "---------------------------------\n\n";
file_put_contents('/var/www/html/fp/fingerprint.txt', print_r($data, true),
FILE_APPEND | LOCK_EX);
?>
```
