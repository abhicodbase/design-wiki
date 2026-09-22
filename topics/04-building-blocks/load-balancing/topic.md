---
id: load-balancing
title: "Load Balancing & API Gateways"
category: "Networking & Routing"
difficulty: Intermediate
readTime: 10
tags:
  - load-balancer
  - routing
  - api-gateway
  - nginx
  - proxy
---

Load balancers distribute incoming network traffic across multiple backend servers to prevent overload, ensure high availability, and maximize resource utilization. They operate at Layer 4 (Transport layer, TCP/UDP routing) or Layer 7 (Application layer, HTTP/HTTPS parsing). API Gateways extend Layer 7 load balancing by offering centralized features like authentication, rate limiting, logging, and routing rules.

## Key Takeaways
* Layer 4 load balancing routes traffic based on IP address and TCP/UDP port, with lower latency.
* Layer 7 load balancing routes traffic by parsing HTTP headers, cookies, and URLs, allowing smart content-based routing.
* Routing algorithms include Round Robin, Weighted Round Robin, Least Connections, IP Hash, and Consistent Hashing.
* SSL/TLS Termination at the load balancer offloads expensive cryptographic handshake calculations from backend application servers.
* API Gateways serve as a single entry point, orchestrating requests, handling rate limiting, authentication, and service discovery.

## Core Concepts
### Layer 4 (L4) Load Balancing
Performs packet-level routing without inspecting HTTP content. High performance, low CPU usage, cannot route based on request headers/URL paths.

### Layer 7 (L7) Load Balancing
Performs request-level routing by parsing HTTP protocol. Can perform header manipulation, path-based routing (e.g., /api/v1/users to User Service), and cookie-based sticky sessions.

### SSL Termination vs SSL Passthrough
Termination decrypts traffic at the LB and passes it unencrypted to backend servers (fast but less secure). Passthrough passes encrypted packets straight to backend servers (secure but backend does crypto heavy lifting).

### Anycast Routing
Allows multiple servers across different geographical locations to share the same IP address. Routers send packets to the closest physical server.

## Architectural Trade-offs
Layer 7 load balancing allows for smarter routing and centralized authentication but requires significantly more CPU power and memory to parse application-layer content. Centralized API gateways simplify backend microservices but create a single point of failure and add network hops to request paths.

## Technology & Tools
* **NGINX**: High-performance reverse proxy and L7 load balancer, widely used as an ingress controller and API gateway.
* **HAProxy**: Extremely fast L4 and L7 software load balancer optimized for speed and high concurrency.
* **AWS ALB / NLB**: Managed cloud balancers. Application Load Balancer (ALB) handles L7 routing. Network Load Balancer (NLB) handles high-throughput L4 routing.
* **Kong**: Cloud-native API gateway built on NGINX/OpenResty, providing extensible plugin support for rate limiting, auth, and telemetry.

## Real-Life Examples & FAANG Case Studies
### Netflix Zuul (L7 API Gateway)
Netflix uses Zuul to route all incoming traffic from millions of devices to appropriate backend microservices, handling dynamic routing, monitoring, and security.

#### Bottlenecks
* CPU Exhaustion: Large surges of TLS handshakes and dynamic filter executions on Zuul nodes can quickly saturate CPU cores, delaying routing.
* Downstream Cascade Failures: If a downstream service slows down, Zuul's thread-per-request model (in Zuul 1) can exhaust the Tomcat thread pool, causing outages across unrelated paths.

#### Corner Cases
* Thundering Herd on Outage Recovery: When a backend service recovers, Zuul might instantly dump a massive queue of retried requests on it, crashing it again. (Mitigated by backoff and circuit breakers).
* Dynamic Filter Failures: A buggy dynamic filter script pushed to Zuul in runtime can error out all incoming requests, demanding robust sandboxing and canary deploys.

### Google Front End (GFE) & Maglev (L4 Load Balancer)
Google routes global user traffic to GFE servers, which decrypt SSL and pass requests internally. Maglev is Google's custom software load balancer that sits in front of GFEs to handle initial L4 packet distribution.

#### Bottlenecks
* Packet Processing Rate: Maglev must process millions of packets per second. Standard OS networking stacks cannot keep up, requiring kernel bypass techniques (e.g., DPDK).

#### Corner Cases
* Consistent Hashing Ring Changes: When a Maglev node fails or is added, client connections must not break. Maglev uses a specialized consistent hashing lookup table to ensure connection affinity is preserved for ongoing TCP streams.
* SYN Flood Attacks: L4 load balancers are primary targets for DDoS. GFE uses SYN cookies to avoid allocating memory for connection states until the three-way handshake is verified.

## Interview Cheat Sheet & Tips
* Understand the distinction between L4 (packet-level routing) and L7 (request-level parsing) and when to use each.
* Discuss SSL termination and its performance benefits for backend servers.
* Address the Single Point of Failure (SPOF) risk of load balancers: explain how to use Keepalived or DNS round-robin with multiple active-passive LB nodes.

