---
Description: Rpcinfo is a utility that can connect to an RPC server, and report back any information the server divulges. 
---

In the following command, we use the "-s" switch to provide compact results:

```bash
root@kali:~# rpcinfo -s 10.11.1.72
   program version(s) netid(s)                         service     owner
    100000  2,3,4     local,udp,tcp,udp6,tcp6          portmapper  superuser
    100024  1         tcp6,udp6,tcp,udp                status      115
    100003  4,3,2     udp6,tcp6,udp,tcp                nfs         superuser
    100227  3,2       udp6,tcp6,udp,tcp                -           superuser
    100021  4,3,1     tcp6,udp6,tcp,udp                nlockmgr    superuser
    100005  3,2,1     tcp6,udp6,tcp,udp                mountd      superuser
 root@kali:~# rpcinfo -p 10.11.1.72
   program vers proto   port  service
    100000    4   tcp    111  portmapper
    100000    3   tcp    111  portmapper
    100000    2   tcp    111  portmapper
    100000    4   udp    111  portmapper
    100000    3   udp    111  portmapper
    100000    2   udp    111  portmapper
    100024    1   udp  51962  status
    100024    1   tcp  52832  status
    100003    2   tcp   2049  nfs
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100227    2   tcp   2049
    100227    3   tcp   2049
    100003    2   udp   2049  nfs
    100003    3   udp   2049  nfs
    100003    4   udp   2049  nfs
    100227    2   udp   2049
    100227    3   udp   2049
    100021    1   udp  49912  nlockmgr
    100021    3   udp  49912  nlockmgr
    100021    4   udp  49912  nlockmgr
    100021    1   tcp  57680  nlockmgr
    100021    3   tcp  57680  nlockmgr
    100021    4   tcp  57680  nlockmgr
    100005    1   udp  58072  mountd
    100005    1   tcp  58889  mountd
    100005    2   udp  35393  mountd
    100005    2   tcp  38944  mountd
    100005    3   udp  59705  mountd
    100005    3   tcp  33139  mountd
```