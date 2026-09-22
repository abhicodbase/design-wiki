---
id: message-queues
title: "Message Queues"
category: "Architecture"
difficulty: Intermediate
readTime: 9
tags:
  - messaging
  - async
  - kafka
  - rabbitmq
---

Message queues enable asynchronous, decoupled communication between services. A producer sends a message to a queue; a consumer processes it independently — they don't need to be running at the same time. This decoupling improves resilience (messages are durably stored), enables back-pressure handling, and allows services to scale independently based on queue depth.

## Key Takeaways
* Decouples producers from consumers — they don't need to be available simultaneously
* Provides durability — messages are persisted until successfully consumed
* Enables natural back-pressure and rate limiting via queue depth
* Supports fan-out (one message to many consumers) and point-to-point delivery
* At-least-once, at-most-once, or exactly-once delivery semantics are configurable
* Dead Letter Queues (DLQ) handle messages that repeatedly fail processing

## Core Concepts
### Point-to-Point (Queue)
One producer, one consumer. Each message is processed by exactly one consumer. Used for task distribution like email sending, image processing jobs.

### Publish-Subscribe (Topic)
One producer, many consumers. Each subscriber receives a copy of the message. Used for event broadcast — e.g., order placed event consumed by inventory, billing, and email services.

### At-Least-Once Delivery
The system guarantees each message is delivered at least once, but may deliver duplicates. Consumer must be idempotent to handle duplicate processing safely.

### Dead Letter Queue (DLQ)
Messages that fail processing repeatedly are moved to a DLQ for inspection and replay. Essential for production reliability and debugging.

### Consumer Groups
Multiple consumers in a group process messages in parallel from the same topic, each getting a subset of partitions. Enables horizontal scaling of consumers.

## Architectural Trade-offs
Message queues add resilience and enable async processing but introduce eventual consistency. The producer can't get an immediate response from the consumer — use request-reply patterns (correlation IDs) when synchronous confirmation is needed. Message ordering is guaranteed per partition in Kafka but not globally. Exactly-once semantics are complex and have performance overhead.

## Technology & Tools
* **Apache Kafka**: High-throughput distributed event streaming platform, log-based storage, ideal for event sourcing and analytics pipelines
* **RabbitMQ**: Feature-rich AMQP broker with complex routing, dead letter exchanges, and priority queues
* **AWS SQS / SNS**: Managed queue (SQS) and notification (SNS) services, zero infrastructure management
* **Google Pub/Sub**: Globally distributed messaging with at-least-once delivery, auto-scaling, and push/pull delivery

## Real-Life Examples & FAANG Case Studies
### Apache Kafka at Uber (Telemetry & Ingestion)
Uber uses Kafka to ingest trillions of daily messages, including real-time driver/rider GPS telemetry, ride requests, and app analytics. This data feeds live mapping, matching, and dynamic pricing engines.

#### Bottlenecks
* Partition Hotspots: If trip telemetry partitions by driverID and a driver's app goes wild sending excessive requests, that partition and broker experience load spikes.
* Consumer Group Rebalancing Overhead: Under heavy load, if a partition consumer misses its heartbeat, the entire consumer group rebalances, pausing message consumption.

#### Corner Cases
* Replica Lagging: During traffic surges, follower replicas can lag behind the leader partition. If the leader crashes, messages might be lost unless acks=all is used.
* Out-of-Order Telemetry Processing: GPS packets arriving late due to tunnel disconnects must be ignored or sorted appropriately to prevent ride matching errors.

### RabbitMQ at Robinhood (Order Executions)
Robinhood uses RabbitMQ for order execution notifications. Once an order is executed, RabbitMQ routes confirmation messages to push notification, email, and audit systems.

#### Bottlenecks
* Queue Depth Spikes: High volatility periods (e.g., market open) generate sudden surges of order fills. High queue depths delay notifications, hurting user experience.

#### Corner Cases
* Broker Network Partition: If nodes in a RabbitMQ cluster split, publishers might receive errors or confirmations might fail, risking missing notifications unless publisher confirms and queue mirroring are set.
* Double-processed Fills: If connection drops before consumer acknowledges message, RabbitMQ will redeliver it. The push notification engine must guarantee idempotency to avoid sending duplicate alerts.

## Interview Cheat Sheet & Tips
* Kafka vs RabbitMQ: Kafka is better for high-throughput log streaming; RabbitMQ for complex routing and task queues
* Always address idempotency — consumers should handle duplicate messages gracefully
* Discuss ordering guarantees — Kafka guarantees per-partition order, not global order
* Message queues are a key tool for handling traffic spikes without losing requests

