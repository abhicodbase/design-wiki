---
id: distributed-id-generator
title: "Distributed ID Generation"
category: "Security & Reliability"
difficulty: Easy
readTime: 8
tags:
  - id-generator
  - snowflake
  - uuid
  - distributed-systems
  - database
---

In a distributed architecture, generating unique identifiers for entities (like users, tweets, orders) across multiple databases is a core requirement. Traditional database auto-increment fails because databases are partitioned and cannot easily coordinate. Distributed ID generators must produce unique, scale-appropriate, and ideally sortable IDs with ultra-low latency.

## Key Takeaways
* Requirements: 64-bit size (storage efficiency), globally unique, and roughly sortable by time (helps index clustering).
* UUIDs (128-bit) are globally unique and require zero coordination, but their large size and lack of sortability hurt index performance.
* Ticket Servers (Flickr style) use dedicated central database auto-increments with offset/increments. Easy but creates a Single Point of Failure (SPOF).
* Twitter Snowflake generates 64-bit sortable IDs using Timestamp (41 bits), Machine ID (10 bits), and Sequence number (12 bits).
* Clock drift is a major challenge; if a server's clock synchronizes backwards (via NTP), it can generate duplicate IDs.

## Core Concepts
### Twitter Snowflake Format
A 64-bit ID split into: 1 sign bit (unused), 41 bits timestamp (millisec), 10 bits machine/node ID, and 12 bits sequence (increments when multiple IDs are requested in the same millisecond on the same machine).

### UUID (Universally Unique Identifier)
128-bit values (e.g., UUID v4 based on random bits). Generated locally with negligible collision probability. Disadvantage: huge indexing overhead on database B-Trees due to random write patterns.

### Roughly Time-Sortable
IDs where the timestamp occupies the most significant bits. This ensures IDs generated later are numerically larger than older IDs, which maintains database insertion order and preserves cache locality.

## Architectural Trade-offs
Generating IDs locally (like UUID or Snowflake) offers massive scalability and zero database coordination overhead. However, Snowflake relies heavily on local server clocks. If clock drift occurs, ID generation can halt, whereas centralized generators are slower but clock-independent.

## Technology & Tools
* **Twitter Snowflake**: The open-source Scala service that popularized clock-based distributed 64-bit ID generation.
* **Zookeeper / Consul**: Coordination tools used to allocate unique machine/node IDs (0-1023) to Snowflake generator instances upon startup.
* **PostgreSQL Sequences**: Can generate partitioned IDs by assigning separate sequence offsets and steps to different database nodes.

## Real-Life Examples & FAANG Case Studies
### Twitter Snowflake ID Generation
Twitter created Snowflake to generate unique, 64-bit, time-sortable IDs for millions of tweets posted every second, removing the MySQL database auto-increment bottleneck.

#### Bottlenecks
* Coordination for Machine IDs: Multiple Snowflake instances must not share the same Machine ID. Twitter used Zookeeper to assign and track active machine IDs (0 to 1023) dynamically, introducing a coordination dependency.

#### Corner Cases
* NTP Clock Synchronization Drift: NTP can adjust a server's clock backward to correct drift. If Snowflake detects the system time is behind the last generated timestamp, it immediately raises an error and refuses to generate IDs until the clock catches up, avoiding duplicate ID creation.
* Sequence Overflow in a Millisecond: If more than 4096 IDs (12 bits) are requested on a single machine within one millisecond, the generator must pause and wait for the next millisecond to reset the sequence counter.

### Instagram ID Generator (Postgres Sequences)
Instagram built a custom ID generator using logical shards inside PostgreSQL tables, combining shard IDs and schema-specific sequences to generate unique 64-bit integers.

#### Bottlenecks
* Database Write Load: Running SQL sequence increments for every post create can saturate database write-ahead logs (WAL) if not optimized.

#### Corner Cases
* Shard Reorganization: If logical database shards are moved to new physical machines, the hardcoded shard IDs inside the SQL PL/pgSQL function must be updated correctly to avoid ID namespace overlapping.

## Interview Cheat Sheet & Tips
* Always start by listing ID design constraints: size (64 vs 128 bit), sortability, generation rate, and coordination needs.
* Break down the bit layout of Twitter Snowflake: explain how 41 bits of timestamp allow for ~69 years of operation.
* Emphasize the clock synchronization problem (NTP drift) and explain how the system should handle or wait out negative clock adjustments.

