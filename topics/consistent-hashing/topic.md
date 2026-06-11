---
id: consistent-hashing
title: "Consistent Hashing & Distributed Hashing"
category: "Databases & Storage"
difficulty: Advanced
readTime: 9
tags:
  - consistent-hashing
  - routing
  - distributed-systems
  - sharding
---

Consistent hashing is a specialized hashing technique used to distribute load across a dynamic set of servers. Unlike standard modulo hashing (hash(key) % N) where changing the server count (N) invalidates almost all cached keys, consistent hashing minimizes key migrations by mapping both keys and servers to a circular space (the hash ring). Changing server nodes only requires moving a small fraction of keys.

## Key Takeaways
* Traditional modulo hashing (hash(key) % N) requires re-hashing all keys when servers are added/removed, causing cache stampedes.
* Consistent hashing maps both keys and servers to a circular 360-degree 'hash ring'. Keys are routed to the first server encountered clockwise.
* Virtual Nodes (vnodes) map a single physical server to multiple locations on the ring, ensuring uniform load distribution.
* It enables elastic scaling, allowing cache clusters and databases to scale out or in dynamically with minimal data movement.
* Adding a node only takes keys from its immediate counter-clockwise neighbor; removing a node only transfers its keys to the clockwise neighbor.

## Core Concepts
### Hash Ring
A logical ring where integer hash values (e.g., from 0 to 2^32 - 1) wrap around. Servers and keys are assigned positions on this ring using cryptographic hash functions (e.g., MD5, SHA-1).

### Virtual Nodes (Vnodes)
Representations of physical nodes at multiple distinct hash values across the ring. If physical server A has 100 vnodes, its load is split into 100 separate segments, preventing 'hot spots' on the ring.

### Key Migration
The process of moving keys from one server to another when the ring topology changes. With N servers, adding a node only requires migrating 1/N of the keys.

## Architectural Trade-offs
Consistent hashing reduces key movement but increases the complexity of client-side routing libraries. Without virtual nodes, hashing distributions are highly skewed, which requires maintaining hundreds of virtual nodes per physical machine, inflating lookup metadata storage.

## Technology & Tools
* **Apache Cassandra**: Uses consistent hashing to distribute partitions across its masterless cluster nodes based on token values.
* **Amazon Dynamo**: The original paper popularized consistent hashing with virtual nodes to meet strict SLA guarantees.
* **libmemcached**: Client-side library implementing consistent hashing to locate Memcached instances without backend coordination.

## Real-Life Examples & FAANG Case Studies
### Discord Guild (Server) Routing
Discord routes real-time gateway connections for active guilds (servers) to specific backend workers. They use consistent hashing to map guild IDs to guild gateway processes.

#### Bottlenecks
* Re-hashing Latency: When gateway nodes crash or restart, guilds are remapped. Recalculating the ring and moving hundreds of thousands of active WebSocket sessions simultaneously causes major traffic spikes.
* Unbalanced Massive Servers: A guild like the Official Fortnite server has millions of concurrent users. Mapping this guild to a single backend worker saturates its resources. Discord had to split giant guilds into shard sub-processes.

#### Corner Cases
* Simultaneous Multi-node Failures: If multiple adjacent nodes on the ring fail at once, the immediate clockwise survivor is suddenly hit with a massive wave of migrated traffic, triggering a cascading crash.
* Consistent Ring Synchronization: Different routing proxies must have an identical view of the hash ring. A network split leading to mismatched rings causes routing splits, where different users in the same guild are sent to different backend processors.

### Memcached Client-Side Hashing
In large-scale Memcached deployments, Memcached servers do not communicate with each other. The client library uses consistent hashing locally to determine which server to write to or read from.

#### Bottlenecks
* Configuration Drift: If some client servers receive updated lists of Memcached IPs while others lag, they construct different hash rings. This leads to duplicate key writes and cache misses.

#### Corner Cases
* Temporary Node Flapping: A node that drops offline and comes back repeatedly (flapping) triggers constant key migrations back and forth, degrading performance. Clients use hashing libraries with dead-node cooldowns to mitigate this.

## Interview Cheat Sheet & Tips
* Explain the drawback of standard modulo hashing first: adding/removing a server invalidates almost the entire cache.
* Explain the concept of Virtual Nodes clearly; they are essential for achieving load balancing and resolving server capacity disparities (more powerful servers get more virtual nodes).
* Describe how data replication works in a consistent hashing system (e.g., replicate the key to the first N physical nodes clockwise on the ring).

