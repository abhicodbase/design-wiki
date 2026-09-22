---
id: url-shortening
title: "Designing a URL Shortening Service (TinyURL)"
category: Application Architectures
difficulty: Easy
readTime: 14
tags:
  - url-shortener
  - hashing
  - key-generation
  - base64
  - tinyurl
  - caching
source: Grokking the System Design Interview
---

A URL shortening service creates shorter aliases for long URLs, redirecting users seamlessly. Services like TinyURL, bit.ly, and goo.gl serve billions of redirections daily. This is a classic "Easy" system design question asked at Google, Facebook, Amazon, and Microsoft interviews to test your understanding of hashing, capacity estimation, caching, data partitioning, and reliability.

## Key Takeaways
* URL shortening is fundamentally a **key-value mapping** problem: short_key → original_url.
* The system is **read-heavy** (100:1 read/write ratio); design accordingly with heavy caching.
* Short URLs must be **unique, non-guessable, and compact** (6-8 characters using base62/base64 encoding).
* A **Key Generation Service (KGS)** pre-generates unique keys to eliminate hash collision logic at request time.
* **Data partitioning** using consistent hashing prevents hot partition problems as scale grows.
* A **lazy cleanup** strategy for expired URLs avoids hammering the database with active expiry scans.

## Core Concepts

### Functional vs Non-Functional Requirements
Always clarify requirements first. **Functional**: Given a URL, generate a short alias; redirect users on short link access; support custom aliases; support link expiration. **Non-Functional**: Highly available (a downed service breaks all redirections); real-time low-latency redirections; short links must not be predictable/guessable.

### Capacity Estimation (Back-of-the-envelope)
Assume 500M new shortened URLs/month with a 100:1 read/write ratio:
- **Write QPS**: 500M / (30×24×3600) ≈ **200 URLs/sec**
- **Read QPS**: 100 × 200 = **20,000 redirections/sec**
- **Storage** (5-year retention, 500 bytes/object): 500M × 12 × 5 × 500 bytes ≈ **15 TB**
- **Cache memory** (20% of daily traffic, 500 bytes each): ~170 GB

### URL Encoding: Base62 vs MD5
Two approaches to generate the 6-character short key: **(1) MD5 Hash** — compute MD5 of the original URL, take the first 6 characters. Problem: collisions, encoding, and multi-user conflicts. **(2) Base62 Encoding** — encode a unique integer ID (from a counter or KGS) in base62 (a-z, A-Z, 0-9). 6 characters → 62⁶ = **56.8 billion unique keys**. Use base64 (adds `+` and `/`) for 64⁶ = **68.7 billion keys**.

### Key Generation Service (KGS)
A dedicated background service pre-generates all possible 6-character base64 keys and stores them in a **key database (key-DB)**. Application servers request keys from KGS on demand — no collision detection needed at runtime. KGS maintains two tables: **unused_keys** and **used_keys**. Concurrency is handled by KGS acquiring locks before handing out keys. A **standby replica** of KGS eliminates the single point of failure.

### Database Schema
Two tables: **(1) URL table** — `hash VARCHAR(6) PK, original_url TEXT, creation_date DATETIME, expiration_date DATETIME, user_id INT`. **(2) User table** — `user_id INT PK, name VARCHAR, email VARCHAR, created_date DATETIME`.

### Data Partitioning (Sharding)
**(a) Range-Based Partitioning**: Partition by first character of the hash key (A→Partition 1, B→Partition 2, …). Simple but causes hot partitions (e.g., all 'E' URLs on one shard). **(b) Hash-Based Partitioning**: Hash the key to determine the partition number uniformly. Can be combined with **Consistent Hashing** to avoid hotspots when adding/removing DB nodes.

### Caching Layer
Cache **frequently-accessed URLs** using Memcached/Redis sitting between app servers and the database. Start with **20% of daily traffic** as the cache size target (≈170 GB, fits in 1–2 modern servers with 256 GB RAM). Use **Least Recently Used (LRU)** eviction. On cache miss, fetch from DB and propagate the result to all cache replicas. Replicas should ignore entries they already hold.

### Load Balancing
Place load balancers at three points: **(1) Between clients and app servers** — distributes incoming requests. **(2) Between app servers and DB** — prevents any single DB node being overwhelmed. **(3) Between app servers and cache servers** — ensures cache reads are distributed. Start with Round Robin; upgrade to load-aware balancing that periodically queries server health metrics.

### URL Purging & Cleanup
Active expiry scanning is expensive. Use **lazy cleanup** instead:
- When a user accesses an expired link, delete it on-the-spot and return an HTTP 404.
- Run a **lightweight nightly cleanup job** (during low-traffic window) to sweep expired links.
- After deletion, **recycle keys** back to key-DB for reuse.
- Optionally, provide a default 2-year expiration for all links.

## Architectural Trade-offs
Choosing MD5 hashing over KGS is simpler to implement but introduces hash collision retry loops, multiple users shortening the same URL to the same key, and encoding issues. KGS is cleaner and collision-free but adds a service dependency and a single point of failure (mitigated by standby). Using synchronous DB writes guarantees durability but increases write latency — asynchronous writes with replication are faster but risk minor data loss on crash. Caching dramatically reduces DB read load but requires careful invalidation when URLs are deleted or expire.

## Technology & Tools
* **Redis**: Primary caching layer for hot URL lookups; also useful as the key-DB store.
* **MySQL / PostgreSQL**: Relational database for URL mappings and user data with sharding support.
* **Cassandra**: NoSQL alternative — write-optimized, horizontally scalable, good for petabyte-scale URL stores.
* **Nginx / AWS ALB**: Load balancers distributing requests across app server fleets.
* **ZooKeeper**: Coordination service for KGS standby election and distributed locking.

## Real-Life Examples & FAANG Case Studies

### TinyURL & bit.ly at Scale
TinyURL serves billions of redirections per day with sub-millisecond latency by aggressively caching hot URLs in memory and using geographic CDN routing to serve redirections from the closest datacenter.

#### Bottlenecks
* **Hot URL Problem**: Viral URLs (e.g., a tweet with a short link) create sudden massive spikes on a single cache key. Standard LRU caches become hot; solution is replication across cache shards.
* **KGS Lock Contention**: Under very high write loads, thousands of app servers request keys simultaneously from KGS, creating lock contention on the key table. Solution: app servers pre-load a batch of keys (e.g., 1000 at a time) from KGS to reduce round-trips.

#### Corner Cases
* **Custom Alias Collision**: Two users simultaneously submit the same custom alias for different URLs. Solution: DB unique constraint + optimistic locking; return error to the second requester.
* **Redirect Loop**: A shortened URL pointing to another shortened URL (possibly circular). Solution: limit redirect depth to 2 hops; detect cycles at creation time with a quick fetch-and-check.
* **URL Expiry During Redirect**: A URL expires between the cache check and the DB fetch. Solution: set cache TTL = remaining URL expiry TTL so cache entries auto-expire in sync with the DB record.

## Interview Cheat Sheet & Tips
* Start by clarifying scale: "Is this 500M shortens/month or 5B?" — this drives the entire architecture.
* Always quantify your capacity estimation explicitly: write QPS, read QPS, storage, and cache size.
* Compare MD5 hashing vs KGS before choosing — state the trade-off clearly, then justify KGS for production.
* When discussing partitioning, mention consistent hashing specifically to show awareness of hot partition risk.
* Mention the **3-tier LB placement** (clients↔app, app↔DB, app↔cache) — interviewers love systematic thoroughness.
* End with **purging strategy** — lazy cleanup is the elegant answer; never say "scan the whole DB for expired links."
