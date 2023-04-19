>[!summary]
>Directory traversal allow attackers to gain unauthorized access to files within an application or files normally not accessible through a web interface, such as those outside the application’s web root directory. This vulnerability occurs when input is poorly validated, subsequently granting an attacker the ability to manipulate file paths with “../” or “..\” characters.

Examples:
- `http://victim/vuln/vulnerabilities/fi/?page=../../../../../../etc/passw`
- `http://victim/vuln/vulnerabilities/fi/?page=c:\windows\system32\drivers\etc\hosts`

## Evading Path Restrictions
 
- [Evading Restrictions](Evading%20Restrictions.md)
- [Basic LFI and bypasses](File%20Inclusion%20(LFI%20&%20RFI).md#Basic%20LFI%20and%20bypasses)
- [Zip slip](Insecure%20File%20Upload.md#Zip%20slip)


