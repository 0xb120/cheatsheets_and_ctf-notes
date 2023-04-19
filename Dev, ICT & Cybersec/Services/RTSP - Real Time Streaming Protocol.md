---
Ports: 554, 8554
Description: Is an application-level network protocol designed for multiplexing and packetizing multimedia transport streams (such as interactive media, video and audio) over a suitable transport protocol.
---

>[!info]
> The Real Time Streaming Protocol (RTSP) is a network control protocol designed for use in entertainment and communications systems to control streaming media servers. The protocol is used for establishing and controlling media sessions between end points. Clients of media servers issue VHS-style commands, such as play, record and pause, to facilitate real-time control of the media streaming from the server to a client (Video On Demand) or from a client to the server (Voice Recording).

# Basic Usage

The transmission of streaming data itself is not a task of RTSP. Most RTSP servers use the Real-time Transport Protocol (RTP) in conjunction with Real-time Control Protocol (RTCP) for media stream delivery. However, some vendors implement proprietary transport protocols. The RTSP server software from RealNetworks, for example, also used RealNetworks' proprietary Real Data Transport (RDT).

## Detailed Information

First and foremost RTSP is an HTTP like protocol. It has different structure and control commands but is textual in its format and once you learn the basics of the commands and how they interact, fairly easy to use. The specification for RTSP is pretty straightforward [^1].

[^1]: https://tools.ietf.org/html/rfc2326

RTSP can be accessed unauthenticated (common in off-the-shelf devices) or authenticated. Authenticated access mirrors HTTP in that you have Basic and Digest authentication, both nearly identical to HTTP. To find out whether your device is authenticated or unauthenticated, simply send a “DESCRIBE” request. A simple DESCRIBE request looks like:
`DESCRIBE rtsp://<ip>:<port> RTSP/1.0\r\nCSeq: 2\r\n\r\n`

This can be sent down a raw socket. Just like HTTP, a successful response indicating unauthenticated access is available will contain a “200 OK”. In this case with DESCRIBE, it will also contain all of the operational parameters of the video feed.

If the device requires authentication, the the response back will contain “401 Unauthorized”. The response will also indicate what authentication mechanisms are available. If Basic authentication is available the response string will contain an information line that has “WWW-Authenticate: Basic”. The rest of the information provide with Basic authentication is largely irrelevant to actually conduct basic authentication.

If Digest authentication is required, then the “401 Unauthorized” response will have an information line containing “WWW-Authenticate: Digest”. The information with the Digest specification IS very important if you are going to do Digest authentication, so don’t ignore it.

Basic authentication is the way to go, hopefully the response received indicates that it is available. If not there are three different methods to assemble a Digest authentication element, so Digest can become troublesome, especially blind (unauthenticated). The rest of this article will stick with Basic authentication. I may write a follow-up article later once I decipher the secret sauce to doing Digest authentication blind.

To formulate a Basic authentication element, one simple has to base 64 encode `<username>` “:” `<password>` and add it to the request. So a new request would look like:
`DESCRIBE rtsp://<ip>:<port> RTSP/1.0\r\nCSeq: 2\r\nAuthorization: Basic YWRtaW46MTIzNA==\r\n\r\n`

The value `YWRtaW46MTIzNA==` is the base 64 encoded username and password concatenated with “:”. In this case I have used “admin”/”1234”. Some simple python scripting to try this out looks like:

```python
import socket
req = "DESCRIBE rtsp://<ip>:<port> RTSP/1.0\r\nCSeq: 2\r\nAuthorization: Basic YWRtaW46MTIzNA==\r\n\r\n"
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("192.168.1.1", 554))
s.sendall(req)
data = s.recv
print data
```


---

# Enumeration

- [NSE (Nmap Scripting Engine)](../Tools/nmap.md#NSE%20(Nmap%20Scripting%20Engine))

```bash
nmap -sV --scripts "rtsp-*" -p <PORT> <IP>
```

---

# Exploitation

## Brute Force

- [hydra](../Tools/hydra.md#RTSP%20Brute-Force)

## Other tools

- [rtsp_authgrinder](https://github.com/Tek-Security-Group/rtsp_authgrinder)
- [cameradar](https://github.com/Ullaakut/cameradar)
