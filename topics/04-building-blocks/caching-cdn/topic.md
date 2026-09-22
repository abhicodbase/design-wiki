---
id: caching-cdn
title: "Caching & Content Delivery Networks (CDN)"
category: "Networking & Routing"
difficulty: Intermediate
readTime: 10
tags:
  - cache
  - cdn
  - redis
  - memcached
  - latency
---

Caching involves storing frequently accessed data in high-speed, temporary storage (like RAM) to serve subsequent requests faster. A Content Delivery Network (CDN) is a geographically distributed network of proxy servers that cache static assets (like video, images, JS/CSS) close to end-users, dramatically reducing latency and saving origin server bandwidth.

## Key Takeaways
* Caching strategies dictate how data is written and read: Cache-Aside (Lazy Loading), Write-Through, and Write-Around.
* CDNs use Edge Servers situated in Point of Presence (PoP) locations to serve cached static assets via Anycast routing.
* Cache eviction policies like Least Recently Used (LRU) and Least Frequently Used (LFU) discard old data when memory is full.
* Cache Invalidation is a hard problem; TTLs (Time to Live), versioning, and explicit purge requests are used to clear stale cache.
* Cache Stampede occurs when many concurrent requests query a missing key at once, causing massive load on the database.

## Core Concepts
### Cache-Aside (Lazy Loading)
Application queries the cache first. If a cache miss occurs, it loads the data from the database, stores it in the cache, and returns it. Easy to implement, but cache misses incur a latency penalty.

### Write-Through vs Write-Around
Write-Through updates both the cache and the database simultaneously (ensures consistency, slow writes). Write-Around writes directly to the database, bypassing the cache (prevents cache clutter for infrequently read writes).

### Cache stampede / Cache Penetration
Penetration occurs when requests query keys that never exist in the DB (mitigated by caching nulls or Bloom Filters). Stampede occurs when a hot cache key expires and a wave of concurrent requests fall through to the DB.

### Edge Caching
Caching content physically closer to the user at ISP networks or internet exchange points (IXPs), reducing round-trip time (RTT).

## Architectural Trade-offs
Caching dramatically improves read speed and lowers database load, but introduces consistency challenges (stale data). Over-reliance on caches can lead to cascading failures: if the cache cluster crashes under heavy load, the database will immediately fail under the raw traffic load.

## Technology & Tools
* **Redis**: In-memory data structure store supporting advanced data structures, persistence, replication, clustering, and pub/sub.
* **Memcached**: Simple, high-performance, multithreaded in-memory key-value store optimized for caching small string/object blocks.
* **Cloudflare / Akamai**: Global CDNs offering edge logic, Web Application Firewall (WAF), DDoS protection, and intelligent routing.

## Real-Life Examples & FAANG Case Studies
### Netflix Open Connect CDN
Netflix deploys its own custom CDN hardware appliances (Open Connect Appliances) directly inside internet service provider (ISP) facilities worldwide, localizing video stream delivery.

#### Bottlenecks
* Hardware Storage Throughput: Streaming 4K video requires massive disk read speed. Netflix optimizes OS kernel and disk layouts (using SSDs and HDDs combined) to saturate network cards without CPU bottlenecks.

#### Corner Cases
* Flash Crowd Stampede: The release of a globally anticipated show can trigger a storm of requests for a newly created video asset. Netflix pre-positions/pre-warms content onto regional edge servers during low-traffic hours to prevent origin server overload.
* Edge Cache Miss Routing: If a request misses the local ISP cache, it must seamlessly redirect to a nearby regional exchange point rather than immediately routing to US origin servers, avoiding long buffering times.

### Facebook Memcached Architecture
Facebook uses a giant, distributed Memcached cluster as a look-aside buffer to scale reads for its social graph database, reducing load on MySQL databases.

#### Bottlenecks
* Stale Data Reads: High concurrent write rates mean cache invalidations must propagate instantly. Invalidation lags result in users seeing old feed items or incorrect profile states.

#### Corner Cases
* Thundering Herd on Deletion: When a highly active page updates, the cache key is deleted. Facebook uses lease-based caching where Memcached issues a token to only one thread to recalculate the DB value while others wait, preventing DB exhaustion.
* Gutter Servers: During memcached pool failures, traffic shifts to a secondary cluster of servers called 'Gutter' servers to absorb the temporary load, preventing MySQL database crashes.

## Interview Cheat Sheet & Tips
* Always discuss cache eviction policies (e.g., LRU) and TTL settings based on write vs read patterns.
* Be ready to explain how to prevent Cache Stampede (using locks/single-flight, pre-warming cache, or background updates).
* Describe how CDN utilizes geo-proximity and Anycast routing to accelerate static file delivery.

