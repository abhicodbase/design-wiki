---
id: rate-limiting
title: "Rate Limiting & Traffic Shaping"
category: "Security & Reliability"
difficulty: Intermediate
readTime: 9
tags:
  - rate-limiter
  - redis
  - token-bucket
  - security
  - scalability
---

A rate limiter controls the rate of traffic sent by a client or service. It blocks requests exceeding a defined threshold (returning HTTP 429 Too Many Requests), protecting APIs from abuse, DDoS attacks, resource starvation, and billing spikes. Traffic shaping smooths out spikes by queueing requests instead of immediately rejecting them.

## Key Takeaways
* Rate limiting is implemented using algorithms like Token Bucket, Leaky Bucket, Fixed Window Counter, and Sliding Window Log/Counter.
* Token Bucket allows short bursts of traffic up to the bucket capacity, making it popular for APIs.
* Leaky Bucket processes requests at a constant, smooth rate, queueing requests in a FIFO buffer, useful for batch processing.
* Distributed rate limiting requires shared storage (e.g., Redis) to track API counts across multiple web servers.
* To minimize latency, local in-memory caching of limits is combined with periodic background sync to a global datastore.

## Core Concepts
### Token Bucket Algorithm
Tokens are added to a bucket at a constant rate. Each request consumes one token. If the bucket is empty, requests are dropped. Allows bursts of traffic while enforcing an overall average rate limit.

### Leaky Bucket Algorithm
Requests flow into a queue and leak out at a constant, fixed rate. If the queue fills up, new requests overflow and are discarded. Ensures smooth traffic shaping.

### Sliding Window Counter
Combines fixed window efficiency with sliding window accuracy. Calculates current window requests plus a weighted percentage of the previous window, preventing limit-bypass hacks at window boundaries.

### Distributed Race Conditions
Occur when two concurrent API requests read and increment Redis counters at the same time. Resolved using Redis Lua scripts or locks.

## Architectural Trade-offs
Adding a global rate limiter protects backend services but introduces a network hop and latency overhead for every incoming API request. If the rate-limiting datastore (like Redis) fails, the system must decide whether to 'fail open' (allowing all requests, risking backend crashes) or 'fail closed' (blocking all users, ruining availability).

## Technology & Tools
* **Redis Lua Scripts**: Enables atomic execution of read-and-write rate-limiting logic on Redis, avoiding lock overhead.
* **Envoy Proxy**: A high-performance service proxy with built-in, highly configurable rate-limiting filters for gRPC and HTTP traffic.
* **Resilience4j**: Lightweight Java library providing rate limiting, circuit breaking, and bulkhead patterns at the application layer.

## Real-Life Examples & FAANG Case Studies
### Stripe API Rate Limiter
Stripe uses a distributed rate limiter running on Redis to protect its API endpoints from card testing attacks, accidental loops, and massive traffic spikes.

#### Bottlenecks
* Redis Network and CPU Saturation: Checking Redis on every single API request creates severe scaling bottlenecks. A single Redis instance can become a performance hotspot under high transaction volumes.

#### Corner Cases
* Fail-Open Decision: During Redis cluster partitions, Stripe's rate limiters fall back to local in-memory estimates and eventually fail open to ensure legitimate merchants are not blocked from charging customers.
* Race Conditions in Counter Updates: Concurrent requests updating the same token counter can read stale values. Stripe uses atomic Lua scripts in Redis to read, check, and decrement tokens in a single execution block.

### GitHub API Rate Limiter
GitHub enforces per-user and per-IP rate limits (e.g., 5000 requests/hour for authenticated users) tracked across their globally distributed load balancers.

#### Bottlenecks
* Token Rotation and Secondary Limits: Managing multiple limit counters (core API, search API, GraphQL API) simultaneously for a single user request increases lookup complexity.

#### Corner Cases
* Webhook/CI Loop Exhaustion: Automated systems like GitHub Actions can trigger infinite loops of API requests. GitHub uses sliding window checks to instantly detect short-term bursts and temporarily ban offending tokens.

## Interview Cheat Sheet & Tips
* Be prepared to compare Token Bucket (handles bursts) and Leaky Bucket (smooth constant flow).
* Discuss scaling the rate limiter: using local memory filters (e.g., Guava) to check limits instantly, only querying Redis when local limits are breached or syncing periodically.
* Specify how to return rate limit metadata in HTTP response headers (e.g., X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset).

