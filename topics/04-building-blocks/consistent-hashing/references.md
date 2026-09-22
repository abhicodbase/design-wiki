# References & Resources — Consistent Hashing

## 📹 Free Videos

* [Consistent Hashing — Gaurav Sen](https://www.youtube.com/watch?v=zaRkONvyGr8) — The classic, definitive visual explanation of the hash ring and virtual nodes.
* [Consistent Hashing Explained — ByteByteGo](https://www.youtube.com/watch?v=UF9Iqmg94tk) — Animation showing node addition/removal with key migration.
* [System Design: Consistent Hashing — Exponent](https://www.youtube.com/watch?v=pHW7lQcm0E0) — Interview-focused walkthrough with worked examples.
* [How Discord Handles Millions of Concurrent Users — Discord Engineering Talk](https://www.youtube.com/watch?v=gXsBLq3UHjw) — Discord's gateway scaling strategy (includes consistent routing discussion).
* [Amazon Dynamo's Consistent Hashing Implementation — MIT 6.824](https://www.youtube.com/watch?v=hMt1X8CRQEM) — MIT distributed systems lecture covering consistent hashing in Dynamo.

## 📖 Free Articles & Engineering Blogs

* [Consistent Hashing and Random Trees — Karger et al. (Original Paper)](https://www.cs.princeton.edu/courses/archive/fall09/cos518/papers/chash.pdf) — The seminal 1997 paper that introduced consistent hashing.
* [A Guide to Consistent Hashing — TopTal](https://www.toptal.com/big-data/consistent-hashing) — Developer-friendly walkthrough with clear diagrams and Java pseudocode.
* [How Cassandra Uses Consistent Hashing — DataStax](https://docs.datastax.com/en/cassandra-oss/3.x/cassandra/architecture/archDataDistributeHashing.html) — Cassandra token rings with virtual nodes explained step-by-step.
* [Rendezvous Hashing vs Consistent Hashing — Medium](https://medium.com/@dgryski/consistent-hashing-algorithmic-tradeoffs-ef6b8e2fcae8) — Comparison of consistent hashing variants and their trade-offs.
* [Libketama — Last.fm's Consistent Hashing Library](https://github.com/RJ/ketama) — The open-source Memcached client consistent hashing implementation.

## 📄 Key Papers & Deep Dives

* [Chord: A Scalable Peer-to-Peer Lookup Service — Stoica et al.](https://pdos.csail.mit.edu/papers/ton:chord/paper-ton.pdf) — P2P DHT built entirely on consistent hashing with O(log N) routing.
* [Dynamo: Consistent Hashing with Virtual Nodes — Amazon SOSP'07](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf) — Section 4 covers the virtual node ring strategy in production.
