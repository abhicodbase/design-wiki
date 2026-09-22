---
id: dns
title: "DNS (Domain Name System)"
category: "Networking & Core Concepts"
difficulty: Easy
readTime: 5
tags:
  - networking
  - dns
  - routing
---

The Domain Name System (DNS) is often referred to as the phonebook of the Internet. It translates human-readable domain names (like `www.google.com`) into machine-readable IP addresses (like `192.0.2.1`). This is crucial because routers and networking equipment use IP addresses to route traffic.

## Core Concepts

### DNS Hierarchy
The DNS namespace is organized in a tree-like hierarchy:
- **Root Level**: Represented by a dot (`.`), there are 13 logical root server clusters worldwide.
- **Top-Level Domain (TLD) Level**: Such as `.com`, `.org`, `.net`, `.io`. Managed by organizations like ICANN.
- **Second-Level Domain Level**: The actual names like `google` or `netflix`.

### DNS Record Types
- **A Record**: Maps a hostname to an IPv4 address.
- **AAAA Record**: Maps a hostname to an IPv6 address.
- **CNAME (Canonical Name)**: Maps an alias name to a true or canonical domain name.
- **MX (Mail Exchange)**: Specifies the mail server responsible for accepting email messages on behalf of a recipient's domain.
- **NS (Name Server)**: Indicates which DNS server is authoritative for that domain.
- **TXT**: Allows an administrator to insert arbitrary text into a DNS record (often used for verification or security like SPF/DKIM).

### DNS Resolution Process
1. **Local Cache Check**: The browser and operating system check their local cache.
2. **DNS Resolver**: If not found, the OS queries a DNS Resolver (usually provided by your ISP or a public one like Google `8.8.8.8` or Cloudflare `1.1.1.1`).
3. **Root Server**: The resolver queries a root nameserver to find the TLD nameserver.
4. **TLD Server**: The resolver queries the TLD nameserver (e.g., for `.com`) to find the Authoritative nameserver.
5. **Authoritative Server**: The resolver queries the Authoritative nameserver which holds the actual `A` or `AAAA` record.
6. **Return & Cache**: The IP address is returned to the client and cached along the way based on the TTL (Time to Live).

## Architectural Trade-offs
- **TTL (Time To Live)**: A high TTL means faster lookups and less load on DNS servers but slower propagation of changes (e.g., if you change a server IP, clients might cache the old one). A low TTL ensures quick updates but increases DNS query traffic.

## Real-Life Examples & FAANG Case Studies
### Route 53 (AWS)
Route 53 is a highly available and scalable cloud Domain Name System (DNS) web service. It goes beyond simple translation and includes features like health checks and traffic policies (Geo DNS, Latency Based Routing).

#### Bottlenecks
- DNS Propagation delays when migrating databases or switching primary/secondary active clusters.
- Potential DNS cache poisoning or DDoS attacks against authoritative name servers.

## Interview Cheat Sheet & Tips
- Understand the difference between an Iterative Query and a Recursive Query.
- Know the common default port for DNS: **UDP Port 53** (falls back to TCP for large responses).
- In System Design interviews, DNS is often used for **Global Load Balancing** (e.g., Geo-routing users to the nearest data center).
