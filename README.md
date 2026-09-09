# Network Traffic Analysis - Wireshark / TCP-IP

A small hands-on network analysis project built around a sanitized packet capture. The goal is to practice reading packets in Wireshark and connecting what appears on screen to the TCP/IP model.

## What is included

```text
network-traffic-analysis/
├── capture/
│   └── sample_traffic.pcap
├── report/
│   ├── technical_report.md
│   ├── technical_report.pdf
│   └── packet_summary.csv
├── README.md
├── wireshark_filters.txt
└── .gitignore
```

The capture is synthetic and uses reserved lab addresses/domain names, so it is safe to publish in a portfolio. It contains 15 packets showing:

- ARP address resolution
- DNS name resolution
- a TCP three-way handshake
- an unencrypted HTTP GET request and `200 OK` response
- normal TCP connection teardown
- ICMP echo request/reply

## Lab topology

| Role | Address |
|---|---|
| Client | `192.168.56.10` |
| Gateway | `192.168.56.1` |
| DNS server | `192.168.56.53` |
| Web server | `203.0.113.80` |
| Hostname | `training.example` |

`203.0.113.0/24` is a documentation network and `example` is reserved for examples. No real user traffic is included.

## How to use it

1. Install Wireshark from the official Wireshark website.
2. Open `capture/sample_traffic.pcap`.
3. Use `wireshark_filters.txt` to isolate protocols.
4. Compare the packet list with `report/packet_summary.csv`.
5. Read `report/technical_report.pdf` or the Markdown version for the completed analysis.

## What to inspect in Wireshark

### ARP
Filter:

```text
arp
```

Packets 1-2 show the client asking for the MAC address associated with its gateway and receiving an answer.

### DNS
Filter:

```text
dns
```

Packets 3-4 show the client asking for an A record for `training.example`. The DNS response maps the hostname to `203.0.113.80`.

### TCP handshake
Filter:

```text
tcp.port == 80
```

Packets 5-7 are the three-way handshake:

1. client -> server: SYN
2. server -> client: SYN/ACK
3. client -> server: ACK

This establishes the TCP session before application data is exchanged.

### HTTP
Filter:

```text
http
```

Packet 8 contains a plaintext request:

```text
GET /lab/index.html HTTP/1.1
Host: training.example
```

Packet 9 contains `HTTP/1.1 200 OK`. Because this is HTTP rather than HTTPS, the request headers and response body are visible to the packet analyzer.

### Connection close
Packets 10-13 show acknowledgments and FIN/ACK packets used to close the TCP connection cleanly.

### ICMP
Filter:

```text
icmp
```

Packets 14-15 show an echo request and echo reply, the basic exchange used by `ping`.

## Skills demonstrated

- packet capture review with Wireshark
- OSI/TCP-IP protocol identification
- IPv4 addressing and ports
- ARP and DNS analysis
- TCP flags, sequence/acknowledgment behavior, handshake and teardown
- HTTP request/response inspection
- ICMP analysis
- display-filter usage
- technical documentation

## Important limitation

Seeing a packet is not the same as proving malicious activity. This capture contains normal network behavior. A security analyst needs context, baselines, timing, endpoint information, and often many more events before calling something suspicious or malicious.
