---
id: network-protocols
title: "Network Protocols (TCP/UDP)"
category: "Networking & Core Concepts"
difficulty: Intermediate
readTime: 6
tags:
  - networking
  - tcp
  - udp
---

Network protocols govern how data is transmitted over a network. The two most prominent transport layer protocols in the Internet protocol suite are TCP and UDP.

## Core Concepts

### TCP (Transmission Control Protocol)
TCP is a connection-oriented, reliable protocol. It guarantees that data sent from one end of the connection will arrive at the other end in the same order.
- **Three-Way Handshake**: SYN, SYN-ACK, ACK. Establishes a connection before sending data.
- **Reliability**: Uses acknowledgments and retransmissions for lost packets.
- **Congestion Control**: Slows down transmission when network congestion is detected.
- **Use Cases**: HTTP/HTTPS (Web browsing), File Transfer (FTP), Email (SMTP/IMAP).

### UDP (User Datagram Protocol)
UDP is a connectionless, unreliable protocol. It sends packets (datagrams) without verifying they were received.
- **Speed**: No handshake, no overhead of ensuring delivery, meaning it's faster than TCP.
- **Unreliable**: Packets can be dropped, duplicated, or arrive out of order.
- **Use Cases**: Streaming video/audio, VoIP (Voice over IP), Online gaming, DNS queries.

## Architectural Trade-offs
- **Overhead vs Speed**: TCP has a larger header (20 bytes) and higher latency due to connection setup and acknowledgment. UDP has a tiny header (8 bytes) and lower latency, but requires the application layer to handle error checking if needed.
- **Head-of-Line Blocking**: In TCP, if a packet is lost, subsequent packets must wait in the buffer until the lost packet is retransmitted. UDP avoids this, which is why HTTP/3 uses QUIC (built over UDP) to fix TCP's head-of-line blocking problem.

## Real-Life Examples
### Video Conferencing (Zoom / Google Meet)
Video calls use UDP for media streams. It's better to drop a few frames (causing a brief glitch) than to delay the entire stream waiting for a retransmission.

### HTTP/3 and QUIC
Modern browsers use QUIC (built on top of UDP) to establish faster connections while implementing their own reliability and congestion control mechanisms in user space, bypassing TCP's limitations.

## Interview Cheat Sheet & Tips
- When asked to design a fast-paced multiplayer game or live video stream, default to mentioning UDP for the data plane and TCP for the control plane (chat/matchmaking).
- Know that TCP is stream-oriented (no message boundaries) while UDP is datagram-oriented (preserves message boundaries).
