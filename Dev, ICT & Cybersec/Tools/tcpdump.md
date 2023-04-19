---
Description: Is a text-based network sniffer, it can both analyze network traffic from a local dump or run-time in the network.
URL: https://www.tcpdump.org/manpages/tcpdump.1.html
---

## Import from a file

```bash
kali@kali:~$ sudo tcpdump -r password_cracking_filtered.pcap
reading from file password_cracking_filtered.pcap, link-type EN10MB (Ethernet)
08:51:20.800917 IP 208.68.234.99.60509 > 172.16.40.10.81: Flags [S], seq 1855084074,
win 14600, options [mss 1460,sackOK,TS val 25538253 ecr 0,nop,wscale 7], length 0
08:51:20.800953 IP 172.16.40.10.81 > 208.68.234.99.60509: Flags [S.], seq 4166855389,
ack 1855084075, win 14480, options [mss 1460,sackOK,TS val 71430591 ecr
25538253,nop,wscale 4], length 0
08:51:20.801023 IP 208.68.234.99.60509 > 172.16.40.10.81: Flags [S], seq 1855084074,
win 14600, options [mss 1460,sackOK,TS val 25538253 ecr 0,nop,wscale 7], length 0
08:51:20.801030 IP 172.16.40.10.81 > 208.68.234.99.60509: Flags [S.], seq 4166855389,
ack 1855084075, win 14480, options [mss 1460,sackOK,TS val 71430591 ecr
25538253,nop,wscale 4], length 0
08:51:20.801048 IP 208.68.234.99.60509 > 172.16.40.10.81: Flags [S], seq 1855084074,
win 14600, options [mss 1460,sackOK,TS val 25538253 ecr 0,nop,wscale 7], length 0
08:51:20.801051 IP 172.16.40.10.81 > 208.68.234.99.60509: Flags [S.], seq 4166855389,
ack 1855084075, win 14480, options [mss 1460,sackOK,TS val 71430591 ecr
25538253,nop,wscale 4], length 0
```

## Filtering:

```bash
sudo tcpdump -i tun0	# specifies which interface must be used
sudo tcpdump icmp		  # filter only icmp packets
sudo tcpdump -n src host 172.16.40.10  
sudo tcpdump -n dst host 172.16.40.10  
sudo tcpdump -n port 81  
sudo tcpdump -nX		  # print the packet in both HEX and ASCII
```

## Advanced filtering

Based on the TCP header flag's structure, it is possible to perform some advanced filtering based on those bits:

![](../../zzz_res/attachments/tcpdump-filters.png)

00011000 = 24 in decimal

```bash
kali@kali:~$ echo "$((2#00011000))"
24

kali@kali:~$ sudo tcpdump -A -n 'tcp[13] = 24'  # Filter only ACK

```

## Read from a file

```bash
tcpdump -r 0.pcap -n port 21
```