---
id: chat-messaging-systems
title: "Chat & Instant Messaging Systems"
category: "Application Architectures"
difficulty: Advanced
readTime: 11
tags:
  - chat
  - websocket
  - whatsapp
  - slack
  - presence
---

Instant messaging systems (like WhatsApp, Slack, and Messenger) require real-time, low-latency, bidirectional communication. Traditional HTTP request-response is inefficient for this. Instead, chat apps establish persistent connections (WebSockets) to route messages from sender to receiver immediately. They must handle status synchronization, offline queues, and push notifications.

## Key Takeaways
* WebSockets provide a persistent, bidirectional TCP connection, eliminating HTTP header overhead for active message streams.
* Connection Managers manage the state of active user sockets, routing incoming messages to the correct server holding the recipient's socket.
* Presence Servers track user online/offline status, publishing changes to the user's active contact list.
* For offline users, messages are stored temporarily in a database and sent via mobile Push Notifications (APNS/FCM).
* Group Chat scalability requires message fan-out optimization to prevent server crashes during large group broadcasts.

## Core Concepts
### WebSocket Protocol
Upgrades a standard HTTP connection to a persistent, full-duplex TCP channel. Essential for real-time events.

### Presence Status Sync
User online status is updated using heartbeat messages. If no heartbeat is received within e.g. 30 seconds, the user is marked offline. Status changes are broadcast to friends/subscribed users.

### Message Delivery States
Sent (received by server), Delivered (received by recipient device), and Read (opened by recipient). Managed using unique message IDs and acknowledgement payloads.

### Write Path vs Read Path in Chat
Write path sends the message from client to websocket gate, writes to a message store, and queues for delivery. Read path pulls unread messages from database when the client reconnects.

## Architectural Trade-offs
Maintaining millions of active WebSocket connections consumes massive server memory (RAM), requiring large connection manager fleets. In-memory presence tracking is extremely fast but can suffer from network overhead when broadcasting status updates to millions of friends.

## Technology & Tools
* **Socket.io**: Popular Node.js library for WebSockets, offering automatic reconnection, fallbacks to HTTP polling, and room-based routing.
* **Apache Pulsar / Kafka**: Used internally as high-throughput message buses to distribute chat payloads across chat server nodes.
* **HBase / Cassandra**: Wide-column stores optimized for high-volume, key-value sequential writes, making them perfect for chat histories.

## Real-Life Examples & FAANG Case Studies
### WhatsApp Chat Delivery
WhatsApp maintains hundreds of millions of concurrent WebSocket connections, using a custom Erlang-based server architecture to route messages with minimal overhead.

#### Bottlenecks
* Memory Consumption per Connection: Maintaining millions of idle TCP sockets on a single server exhausts memory buffer pools and file descriptors quickly, requiring fine-tuning of the OS TCP stack.

#### Corner Cases
* Offline Queue Overflow: If a user goes offline for weeks, thousands of unread messages accumulate. Upon reconnecting, sending all messages at once can choke the client socket. WhatsApp chunks and paginates historical sync payloads.
* Group Message Fan-out: In a group of 500 users, one sent message must be duplicated and routed to 499 sockets. If many group members are offline, this triggers 499 database writes and push notifications, requiring async background workers.

### Slack Real-Time Presence (RTM)
Slack manages real-time status (active, away, typing indicator) across workspaces, where users are grouped into shared organizational channels.

#### Bottlenecks
* Status Fan-out Storms: When a user changes status in a workspace with 10,000 users, sending updates to all active sessions creates O(N^2) traffic, causing server and client bandwidth exhaustion.

#### Corner Cases
* Frequent Network Reconnects: Mobile users switching between Wi-Fi and Cellular cause sockets to drop and reconnect constantly. Slack uses session resume tokens to skip full handshake and message sync during brief reconnects.
* Split-brain Presence: A user logged in on both phone and laptop can have conflicting activity states. The system must aggregate heartbeat signals to show a single unified presence state.

## Interview Cheat Sheet & Tips
* Explain the protocol choices: WebSockets for active chat session, HTTP for message attachments upload/download (to offload socket servers), and push notifications (APNS/FCM) for offline users.
* Address the presence fan-out problem: for small chat lists, use publish-subscribe; for large groups, do not push status updates, instead query presence 'on-demand' when user opens the chat window.
* Highlight message ordering: use client-generated sequence numbers combined with server-assigned monotonic IDs to display messages chronologically despite network jitter.

