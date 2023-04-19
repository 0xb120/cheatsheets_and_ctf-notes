---
Ports: 137, 138 (UDP), 139
Description: Name service for name registration and resolution (ports 137/udp and 137/tcp). Datagram distribution service for connectionless communication (port 138/udp). Session service for connection-oriented communication (port 139/tcp).
---

# Basic Usage

## Name Service (137 UDP)

Every machine should have a name inside the NetBios network. To request a name, a machine should send a "Name Query" packet in broadcast and if anyone answer that it is already using that name, the machine can use that name. If there is a Name Service server, the computer could ask the Name Service server if someone is using the name that it wants to use.

To discover the IP address of a Name, a PC has to send a "Name Query" packet and wait if anyone answers. If there is a Name Service server, the PC can ask it for the IP of the name.

- [nmblookup](../Tools/nmblookup.md)
- [nbtscan](../Tools/nbtscan.md)
- [NSE (Nmap Scripting Engine)](../Tools/nmap.md#NSE%20(Nmap%20Scripting%20Engine))

```bash
nmblookup -A <IP>
nbtscan <IP>/30
sudo nmap -sU -sV -T4 --script nbstat.nse -p137 -Pn -n <IP>
```

## Datagram Distribution Service (138 UDP)

NetBIOS datagrams are sent over UDP. A datagram is sent with a "Direct Unique" or "Direct Group" packet if it's being sent to a particular NetBIOS name, or a "Broadcast" packet if it's being sent to all NetBIOS names on the network.

## Session Service (139 TCP)

Session mode lets two computers establish a connection for a "conversation", allows larger messages to be handled, and provides error detection and recovery.

Sessions are established by exchanging packets. The computer establishing the session attempts to make a [TCP](https://en.wikipedia.org/wiki/Transmission_Control_Protocol) connection to port 139 on the computer with which the session is to be established. If the connection is made, the computer establishing the session then sends over the connection a "Session Request" packet with the NetBIOS names of the application establishing the session and the NetBIOS name to which the session is to be established. The computer with which the session is to be established will respond with a "Positive Session Response" indicating that a session can be established or a "Negative Session Response" indicating that no session can be established (either because that computer isn't listening for sessions being established to that name or because no resources are available to establish a session to that name).

Data is transmitted during an established session by Session Message packets.

TCP handles flow control and re-transmission of all session service packets, and the dividing of the data stream over which the packets are transmitted into [IP](https://en.wikipedia.org/wiki/Internet_Protocol) datagrams small enough to fit in link-layer packets.

Sessions are closed by closing the TCP connection.
