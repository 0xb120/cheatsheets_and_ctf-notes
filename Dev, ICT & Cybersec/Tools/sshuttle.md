---
Description: Transparent proxy server that works as a poor man's VPN. Forwards over ssh. Doesn't require admin. Works with Linux and MacOS. Supports DNS tunneling.
URL: https://github.com/sshuttle/sshuttle
---

>[!summary]
>**sshuttle** allows you to create a VPN connection from your machine to any remote server that you can connect to via ssh, as long as that server has python 3.6 or higher. To work, you must have root access on the local machine, but you can have a normal account on the server. It’s valid to run **sshuttle** more than once simultaneously on a single client machine, connecting to a different server every time, so you can be on more than one VPN at once.

If run on a router, **sshuttle** can forward traffic for your entire subnet to the VPN: `sshuttle -r user@a.b.c.d w.x.y.z/24`

```bash
kali@kali:~$ sshuttle -r user@192.168.1.123 10.11.1.0/24
```

>[!note]
>Some mileage may vary. I’ve never had success running nmap through sshuttle, and there are a lot of people out there posting similar complaints. But it is a very nice way to interact with a host over a tunnel.
