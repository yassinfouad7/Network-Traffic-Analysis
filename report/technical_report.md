# Network Traffic Analysis - Technical Report

## 1. Objective

The purpose of this lab was to inspect a controlled packet capture in Wireshark and identify how common network protocols appear on the wire. The analysis focused on ARP, DNS, TCP, HTTP, and ICMP, with particular attention to address resolution, name resolution, TCP connection establishment, application-layer requests, and connection teardown.

## 2. Lab Environment

The capture is synthetic and contains only reserved lab addressing. The client is `192.168.56.10`, the DNS server is `192.168.56.53`, the gateway is `192.168.56.1`, and the web server is `203.0.113.80`. The hostname used in the DNS and HTTP exchange is `training.example`.

Because this is generated training data, it can be included in a public GitHub repository without exposing real users, credentials, or production systems.

## 3. Capture Overview

The capture contains 15 packets. The observed protocol sequence is:

1. ARP request and reply
2. DNS query and response
3. TCP three-way handshake
4. HTTP GET request and HTTP 200 response
5. TCP acknowledgment and orderly FIN-based teardown
6. ICMP echo request and reply

This sequence represents a simplified example of a host resolving network information, establishing a transport-layer session, exchanging application data, closing the connection, and performing a reachability test.

## 4. Findings

### 4.1 ARP - Packets 1-2

Packet 1 is an ARP request from `192.168.56.10` asking which MAC address owns `192.168.56.1`. The destination Ethernet address is broadcast because the client does not yet know the gateway's hardware address. Packet 2 is the gateway's ARP reply.

**Interpretation:** ARP maps an IPv4 address on the local network to a Layer 2 MAC address. This occurs before the client can send an Ethernet frame to the local next hop.

### 4.2 DNS - Packets 3-4

Packet 3 is a UDP DNS query from source port `53000` to destination port `53`. The client asks for the IPv4 A record associated with `training.example`. Packet 4 is the response, returning `203.0.113.80`.

**Interpretation:** The application wants to communicate using a hostname, while IP routing requires an IP address. DNS provides that mapping. The source port on the client is ephemeral; port 53 identifies the DNS service on the server.

### 4.3 TCP Three-Way Handshake - Packets 5-7

The client opens a TCP connection from ephemeral port `51514` to server port `80`.

- Packet 5: SYN - the client requests a new connection.
- Packet 6: SYN/ACK - the server acknowledges the client's request and supplies its own initial sequence state.
- Packet 7: ACK - the client acknowledges the server.

**Interpretation:** TCP establishes shared connection state before application data is transferred. Sequence and acknowledgment numbers allow each side to track reliable byte delivery.

### 4.4 HTTP Request - Packet 8

After TCP is established, the client sends:

```text
GET /lab/index.html HTTP/1.1
Host: training.example
```

The request is carried inside the TCP payload. Wireshark can decode the application data because the lab uses plaintext HTTP.

**Security observation:** Plain HTTP does not encrypt headers or content. Anyone with legitimate access to the capture point can potentially read the request and response. This is one reason HTTPS is used for real web traffic.

### 4.5 HTTP Response - Packet 9

The server responds with `HTTP/1.1 200 OK` and a small HTML body.

**Interpretation:** The `200` status code indicates that the request was processed successfully. At this point Wireshark can correlate the request and response as part of the same TCP conversation.

### 4.6 TCP Teardown - Packets 10-13

The client acknowledges the HTTP response and the endpoints exchange FIN/ACK packets.

**Interpretation:** A FIN indicates that one side has finished sending data. TCP connection termination is directional, so both sides close their sending direction and acknowledge the other's close.

### 4.7 ICMP - Packets 14-15

Packet 14 is an ICMP Echo Request from the client to the server. Packet 15 is an Echo Reply.

**Interpretation:** This is the core request/reply behavior used by the `ping` utility to test IP reachability and estimate round-trip delay. A successful reply confirms that these particular ICMP packets were able to traverse the path; it does not prove that every application service is available.

## 5. Wireshark Filters Used

Useful display filters for this capture include:

```text
arp
dns
tcp
tcp.port == 80
http
icmp
ip.addr == 192.168.56.10
ip.addr == 203.0.113.80
```

A more targeted SYN filter is:

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

## 6. TCP/IP Model Mapping

| Observed data | TCP/IP layer | Purpose |
|---|---|---|
| Ethernet / ARP | Link | Local frame delivery and address resolution |
| IPv4 / ICMP | Internet | Addressing, routing, diagnostics |
| TCP / UDP | Transport | End-to-end transport, ports, reliability or datagrams |
| DNS / HTTP | Application | Name resolution and web communication |

## 7. Security Takeaways

The capture shows why packet analysis is useful in cybersecurity. It can reveal which systems communicated, which services and ports were used, how a connection was established, and what plaintext application data crossed the network. However, packet evidence must be interpreted carefully. Normal protocol behavior alone is not evidence of an attack.

Encrypted protocols also change what an analyst can see. With HTTPS, IP addresses, ports, timing, packet sizes, and parts of the TLS handshake may still be observable, but the HTTP content itself is normally encrypted.

## 8. Conclusion

The lab demonstrated the full path from local address resolution through DNS, TCP connection setup, an HTTP request/response, connection termination, and ICMP reachability testing. The exercise reinforced how application-layer activity depends on lower-layer protocols and how Wireshark can be used to move from raw packets to a structured explanation of network behavior.

## 9. Limitations

This is a small controlled capture rather than production traffic. It does not contain TLS, IPv6, packet loss, retransmissions, malicious traffic, or high-volume flows. Those would be useful follow-up labs once the fundamentals are comfortable.
