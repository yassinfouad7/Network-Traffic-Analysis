# Network Traffic Analysis

A beginner Wireshark lab I used to practice following normal network traffic from packet to packet.

The capture includes ARP, DNS, TCP, HTTP, and ICMP traffic. My main goal was to get comfortable with Wireshark filters and understand what actually happens when a client resolves a name, opens a connection, sends an HTTP request, and closes the connection.

## Files

- `capture/sample_traffic.pcap` - packet capture used for the lab
- `report/technical_report.md` - my notes and findings
- `report/technical_report.pdf` - PDF copy of the report
- `report/packet_summary.csv` - short packet-by-packet summary
- `wireshark_filters.txt` - filters I used while checking the capture

## What I found

The capture has 15 packets.

- Packets 1-2: ARP request and reply
- Packets 3-4: DNS query and response
- Packets 5-7: TCP three-way handshake
- Packets 8-9: HTTP GET request and `200 OK` response
- Packets 10-13: TCP connection close
- Packets 14-15: ICMP echo request and reply

One thing that stood out to me was how easy it is to read HTTP traffic when it is not encrypted. In packet 8, Wireshark can show the requested path and Host header directly.

## Opening the capture

1. Open Wireshark.
2. Open `capture/sample_traffic.pcap`.
3. Try the filters in `wireshark_filters.txt`.
4. Compare the packets with `report/packet_summary.csv`.

## Main filters I used

```text
arp
dns
tcp.port == 80
http
icmp
```

## What I practiced

- following a TCP conversation
- reading source and destination IP addresses and ports
- recognizing SYN, ACK, and FIN flags
- checking DNS queries and responses
- inspecting an unencrypted HTTP request
- using Wireshark display filters

This is a small practice capture, so it only shows normal traffic and does not represent a real incident.
