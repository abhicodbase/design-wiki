# References & Resources — Database Scaling & Partitioning

## 📹 Free Videos

* [Database Sharding — ByteByteGo](https://www.youtube.com/watch?v=5faMjKuB9bc) — Horizontal partitioning, shard key selection, and hot shard mitigation.
* [CAP Theorem Simplified — ByteByteGo](https://www.youtube.com/watch?v=BHqjEjzAicA) — Visual explanation of Consistency, Availability, and Partition Tolerance.
* [SQL vs NoSQL — Academind](https://www.youtube.com/watch?v=ZS_kXvOeQ5Y) — Side-by-side comparison of data modeling, scalability, and use cases.
* [How Amazon DynamoDB Works — AWS re:Invent 2021](https://www.youtube.com/watch?v=yvBR71D0nAQ) — DynamoDB internals, partitioning, and consistent hashing explained by AWS engineers.
* [Designing Data-Intensive Applications Overview — Martin Kleppmann](https://www.youtube.com/watch?v=fU9hR3kiOK0) — Overview talk covering replication, sharding, and consistency models.

## 📖 Free Articles & Engineering Blogs

* [Scaling Pinterest — Pinterest Engineering](https://medium.com/pinterest-engineering/sharding-pinterest-how-we-scaled-our-mysql-fleet-3f341e96ca6f) — Real blog post on how Pinterest sharded their MySQL at massive scale.
* [Dynamo: Amazon's Highly Available Key-Value Store — Amazon Research](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf) — The original DynamoDB system design paper with vector clocks and consistent hashing.
* [How Cassandra Scales Writes — DataStax Blog](https://www.datastax.com/blog/how-apache-cassandra-balances-consistency-availability-and-performance) — Log-structured storage, compaction, and tuneable consistency levels.
* [Read Replicas vs Sharding — AWS Documentation](https://aws.amazon.com/rds/features/read-replicas/) — How RDS read replicas work with replication lag implications.
* [Database Internals — Alex Petrov (Free Chapters)](https://www.databass.dev/) — Deep dive into storage engine internals: B-Trees, LSM trees, WAL.

## 📄 Key Papers & Deep Dives

* [Dynamo Paper (Full PDF) — Werner Vogels et al.](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf) — SOSP'07 paper on eventual consistency, vector clocks, and sloppy quorums.
* [Spanner: Google's Globally Distributed Database — Google Research](https://research.google/pubs/pub39966/) — TrueTime-based global linearizable transactions at Google scale.
