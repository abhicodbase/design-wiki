---
id: database-scaling
title: "Database Scaling & Partitioning"
category: "Databases & Storage"
difficulty: Advanced
readTime: 12
tags:
  - database
  - sharding
  - replication
  - nosql
  - sql
---

As application traffic and data volume grow, databases must scale. While read scale is achieved via replica pools, write scale requires partitioning data across multiple database instances (sharding). Relational databases (SQL) focus on consistency and ACID guarantees, whereas distributed NoSQL databases sacrifice strict consistency to achieve horizontal scaling and high availability.

## Key Takeaways
* Vertical scaling (adding CPU/RAM) has physical limits; horizontal scaling (adding nodes) is the standard for web scale.
* Read replication handles high read traffic by designating one master node for writes and multiple replica nodes for reads.
* Sharding partitions database rows across multiple machines based on a shard key (e.g., hash-based or range-based).
* CAP Theorem states that a distributed data store can simultaneously provide at most two of: Consistency, Availability, and Partition Tolerance.
* NoSQL databases relax ACID requirements in favor of BASE (Basically Available, Soft state, Eventual consistency).

## Core Concepts
### Horizontal Partitioning (Sharding)
Splitting a single dataset across multiple database engines. E.g., user profiles starting with A-M on Shard 1, N-Z on Shard 2. Selecting a good shard key is crucial to avoid hot partitions.

### Replication Lag
The delay between writes on the master database and their replication to read replicas. Can lead to read-after-write inconsistencies where a user submits a post and doesn't see it immediately on refresh.

### CAP Theorem (Consistency vs Availability)
During a network partition (P), a database must choose between Consistency (C) (rejecting writes/reads until resolved) or Availability (A) (allowing local reads/writes, yielding temporary inconsistency).

### ACID vs BASE
ACID: Atomicity, Consistency, Isolation, Durability (standard SQL transactions). BASE: Basically Available, Soft-state, Eventual consistency (typical distributed NoSQL stores).

## Architectural Trade-offs
Relational databases provide powerful transactional queries and joins but are notoriously hard to scale horizontally. Sharding solves the scale issue but disables cross-shard joins, breaks foreign key constraints, and makes transactional integrity extremely difficult and slow to coordinate (2-Phase Commit).

## Technology & Tools
* **PostgreSQL / MySQL**: Mature relational databases supporting transactional operations. Can be scaled via read replicas and manual/middleware-based sharding.
* **Apache Cassandra**: Distributed NoSQL wide-column store with a masterless peer-to-peer ring structure. Optimized for high write throughput and high availability.
* **MongoDB**: Document-oriented NoSQL database that simplifies horizontal scaling with built-in sharding and replica sets.
* **CockroachDB**: NewSQL database providing global horizontal scaling with strict ACID consistency, built on consensus protocols (Raft).

## Real-Life Examples & FAANG Case Studies
### Pinterest SQL Sharding (Scaling MySQL)
Pinterest scaled its MySQL databases to support billions of pins by partitioning data across logical database shards hosted on physical machines, avoiding single-point-of-failure constraints.

#### Bottlenecks
* Cross-Shard Joins: Pinterest had to perform joins at the application layer, fetching data from multiple shards in parallel and stitching them together, creating network and latency bottlenecks.
* Unbalanced Sharding Key: If an influencer with millions of pins is placed on a single shard, that shard experiences extreme read/write traffic compared to others.

#### Corner Cases
* Shard Migration and Split: When a shard runs out of storage, Pinterest must split the logical shard onto two separate physical machines. Coping with live writes during database transfers requires careful master-master replication setups or dual-writing strategies.
* Auto-Increment Key Collisions: Standard database auto-increment fails when sharding. Pinterest resolved this by generating unique 64-bit IDs containing the shard ID inside the generated number.

### Amazon DynamoDB (Shopping Cart Architecture)
Amazon designed Dynamo, a highly available key-value store, specifically to ensure that customer shopping carts are always writable, even if multiple servers or entire datacenters fail.

#### Bottlenecks
* Write Conflicts (Split-Brain): If a network partition occurs and a customer writes to two different datacenters, the system receives conflicting updates to the shopping cart. Dynamo resolves this using Vector Clocks and application-level resolution (merging items).

#### Corner Cases
* Sloppy Quorum and Hinted Handoff: If primary nodes are unreachable, writes are saved on temporary neighboring nodes. When the primary recovers, the neighbor hands off the writes. If the neighbor crashes before handoff, data can temporarily disappear.
* Eventual Consistency Read Latency: A customer might remove an item from their cart, but a subsequent read hits a replica that hasn't received the write yet, showing the deleted item again.

## Interview Cheat Sheet & Tips
* Choose a database based on access patterns: SQL for complex relational queries and financial ledger transactions; NoSQL for simple key-value reads/writes at scale.
* When sharding, discuss key selection criteria: aim for uniform data distribution and minimal cross-shard queries.
* Address replication lag: mention read-after-write consistency solutions (e.g., routing user's own reads to the master node temporarily after they perform a write).

