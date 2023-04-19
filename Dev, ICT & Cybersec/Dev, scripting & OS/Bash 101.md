# Environment variables

Every bash process has its own env vars. Some of them are

```bash
kali@kali:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
kali@kali:~$ echo $USER
kali
kali@kali:~$ echo $PWD
/home/kali
kali@kali:~$ echo $HOME
/home/kali
```

`export` command can be used to **set a custom var** which will be valid within any other shell's sub-process:

```bash
kali@kali:~$ export b=10.11.1.220
kali@kali:~$ ping -c 2 $b
PING 10.11.1.220 (10.11.1.220) 56(84) bytes of data.
64 bytes from 10.11.1.220: icmp_seq=1 ttl=62 time=2.23 ms
64 bytes from 10.11.1.220: icmp_seq=2 ttl=62 time=1.56 ms
```

We can use the `env` command to view every environment var:

```bash
kali@kali:~$ env
SHELL=/bin/bash
...
PWD=/home/kali
XDG_SESSION_DESKTOP=lightdm-xsession
LOGNAME=kali
XDG_SESSION_TYPE=x11
XAUTHORITY=/home/kali/.Xauthority
XDG_GREETER_DATA_DIR=/var/lib/lightdm/data/kali
HOME=/home/kali
...
TERM=xterm-256color
USER=kali
...
```

# history

We can use `history` command to **list all the recent submitted commands**. Each one of them will have a number assigned, that can be used to **re-run the same command** though the `$` sign:

```bash
kali@kali:~$ history
1 cat /etc/lsb-release
2 clear
3 history
kali@kali:~$ !1
cat /etc/lsb-release
DISTRIB_ID=Kali
DISTRIB_RELEASE=kali-rolling
DISTRIB_CODENAME=kali-rolling
DISTRIB_DESCRIPTION="Kali GNU/Linux Rolling"
```

Another useful command is `!!` which **execute the latest performed command**.

`CTRL + R` short-cut can also be used to **perform a reverse search within the history**.

# Customizing the Bash Environment

### Bash History Customization

`HISTCONTROL` variables defines if the history has to consider duplicates command etc.

`HISTIGNORE` variables creates a blacklist of command to filter out history

`HISTTIMEFORMAT` controls date and/or time stamps in the output of the history

## Alias

An alias is a string we can define that replaces a command name

## Persistent Bash Customization (.bashrc)

In order to persist customization, settings must added within the `.bashrc` file of the specific user

---

# Bash scripting

| Variable name | Description |
| --- | --- |
| $0 | The name of the bash script |
| $1 - $9 | The first 9 args passed to the bash script |
| $# | Number of args passed to the script |
| $@ | All arguments passed to the script |
| $? | Exit status of the most recent run process |
| \$\$ | PID of the current process |
| $USER | Username of the user running the script |
| $HOSTNAME | Host name of the machine |
| $RANDOM | Random number |
| $LINENO | Current line number in the script |

## Test statements

![](zzz_res/attachments/Bash-test-operators.png)

```bash
#!/bin/bash
read -p "What is your age: " age
if [ $age -lt 16 ]
then
	echo "You might need parental permission to take this course!"
elif [ $age -gt 60 ]
then
	echo "Hats off to you, respect!"
else
	echo "Welcome to the course!"
fi
```

## Boolean Logical Operators

In bash boolean logical operator (`&&` and `||`) determine the execution of some commands based on the results of the previous one.

```bash
kali@kali:~$ user2=kali
kali@kali:~$ grep $user2 /etc/passwd && echo "$user2 found!"
kali:x:1000:1000:,,,:/home/kali:/bin/bash
kali found!
kali@kali:~$ user2=bob
kali@kali:~$ grep $user2 /etc/passwd && echo "$user2 found!" || echo "$user2 not found!"
bob not found!
```

## Loops

One-liner (for):

```bash
kali@kali:~$ for ip in $(seq 1 10); do echo 10.11.1.$ip; done
10.11.1.1
10.11.1.2
10.11.1.3
10.11.1.4
10.11.1.5
10.11.1.6
10.11.1.7
10.11.1.8
10.11.1.9
10.11.1.10
```

Scripted (while):

```bash
#!/bin/bash
# while loop example  
counter=1  
while [ $counter -le 10 ]  
do  
	echo "10.11.1.$counter"  
	((counter++))
done
```

## Functions

```bash
#!/bin/bash
# function return value example
return_me() {
	echo "Oh hello there, I'm returning a random value!"
	return $RANDOM
}
return_me
echo "The previous function returned a value of $?"
```

## Math in-line eval

```bash
┌──(kali㉿kali)-[~/…/lab/exercises/binExpl/VulnApp1]
└─$ echo $(( 16#A00 ))
2560
```

## Piping and redirection

![Piping and Redirection](Dev,%20ICT%20&%20Cybersec/Dev,%20scripting%20&%20OS/Linux%20command%20cheatsheet.md#Piping%20and%20Redirection)

---

# Bash port scanner

```bash
#!/bin/bash
host=10.5.5.11
for port in {1..65535}; do
	timeout .1 bash -c "echo >/dev/tcp/$host/$port" && echo "port $port is open"
done
echo "Done"
```