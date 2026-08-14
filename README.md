# HLD Interview Master Roadmap

Interview-focused High-Level Design (HLD) preparation for experienced Java/backend engineers.

## Learning Approach

Each topic will be covered in this sequence:

1. Concept and intuition
2. Real-world examples
3. HLD interview perspective
4. Common interview questions
5. Architecture and data flow where applicable
6. Trade-offs
7. Interview-ready answer
8. Follow-up questions and edge cases

# Phase 1 — HLD Fundamentals

## 1. System Design Basics
- [01. HLD vs LLD](Phase-1-HLD-Fundamentals/01-HLD-vs-LLD.md)
- [02. Functional vs Non-Functional Requirements](Phase-1-HLD-Fundamentals/02-Functional-vs-Non-Functional-Requirements.md)
- [03. Scalability](Phase-1-HLD-Fundamentals/03-Scalability.md)
- [04. Availability](Phase-1-HLD-Fundamentals/04-Availability.md)
- [05. Reliability](Phase-1-HLD-Fundamentals/05-Reliability.md)
- [06. Maintainability](Phase-1-HLD-Fundamentals/06-Maintainability.md)
- [07. Performance](Phase-1-HLD-Fundamentals/07-Performance.md)
- [08. Latency vs Throughput](Phase-1-HLD-Fundamentals/08-Latency-vs-Throughput.md)
- [09. CAP Theorem](Phase-1-HLD-Fundamentals/09-CAP-Theorem.md)
- [10. PACELC](Phase-1-HLD-Fundamentals/10-PACELC.md)
- [11. ACID vs BASE](Phase-1-HLD-Fundamentals/11-ACID-vs-BASE.md)

## 2. Architecture
- [12. Monolith](Phase-1-HLD-Fundamentals/12-Monolith.md)
- [13. Modular Monolith](Phase-1-HLD-Fundamentals/13-Modular-Monolith.md)
- [14. Microservices](Phase-1-HLD-Fundamentals/14-Microservices.md)
- [15. SOA](Phase-1-HLD-Fundamentals/15-SOA.md)
- [16. Event-Driven Architecture](Phase-1-HLD-Fundamentals/16-Event-Driven-Architecture.md)
- [17. Client-Server Architecture](Phase-1-HLD-Fundamentals/17-Client-Server.md)
- [18. Layered Architecture](Phase-1-HLD-Fundamentals/18-Layered-Architecture.md)
- [19. Async Architecture](Phase-1-HLD-Fundamentals/19-Async-Architecture.md)
- [20. Sync vs Async Communication](Phase-1-HLD-Fundamentals/20-Sync-vs-Async-Communication.md)

## 3. Scalability
- [21. Vertical Scaling](Phase-1-HLD-Fundamentals/21-Vertical-Scaling.md)
- [22. Horizontal Scaling](Phase-1-HLD-Fundamentals/22-Horizontal-Scaling.md)
- [23. Stateless Services](Phase-1-HLD-Fundamentals/23-Stateless-Services.md)
- [24. Load Balancer](Phase-1-HLD-Fundamentals/24-Load-Balancer.md)
- [25. Reverse Proxy](Phase-1-HLD-Fundamentals/25-Reverse-Proxy.md)
- [26. Auto Scaling](Phase-1-HLD-Fundamentals/26-Auto-Scaling.md)
- [27. Horizontal Pod Scaling](Phase-1-HLD-Fundamentals/27-Horizontal-Pod-Scaling.md)
- [28. Bottleneck Identification](Phase-1-HLD-Fundamentals/28-Bottleneck-Identification.md)

## 4. Load Balancing
- [29. L4 vs L7](Phase-1-HLD-Fundamentals/29-L4-vs-L7.md)
- [30. Round Robin](Phase-1-HLD-Fundamentals/30-Round-Robin.md)
- [31. Weighted Round Robin](Phase-1-HLD-Fundamentals/31-Weighted-Round-Robin.md)
- [32. Least Connections](Phase-1-HLD-Fundamentals/32-Least-Connections.md)
- [33. IP Hash](Phase-1-HLD-Fundamentals/33-IP-Hash.md)
- [34. Consistent Hashing](Phase-1-HLD-Fundamentals/34-Consistent-Hashing.md)
- [35. Health Checks](Phase-1-HLD-Fundamentals/35-Health-Checks.md)
- [36. Failover](Phase-1-HLD-Fundamentals/36-Failover.md)

# Phase 2 — Database & Storage

## 5. Database Fundamentals
- [01. SQL vs NoSQL](Phase-2-Database-and-Storage/01-SQL-vs-NoSQL.md)
- [02. Relational DB](Phase-2-Database-and-Storage/02-Relational-DB.md)
- [03. Document DB](Phase-2-Database-and-Storage/03-Document-DB.md)
- [04. Key-Value DB](Phase-2-Database-and-Storage/04-Key-Value-DB.md)
- [05. Wide-Column DB](Phase-2-Database-and-Storage/05-Wide-Column-DB.md)
- [06. Graph DB](Phase-2-Database-and-Storage/06-Graph-DB.md)
- [07. When to Choose Which Database](Phase-2-Database-and-Storage/07-When-to-Choose-Which-Database.md)

## 6. Database Scaling
- [08. Read Replica](Phase-2-Database-and-Storage/08-Read-Replica.md)
- [09. Write Scaling](Phase-2-Database-and-Storage/09-Write-Scaling.md)
- [10. Partitioning](Phase-2-Database-and-Storage/10-Partitioning.md)
- [11. Sharding](Phase-2-Database-and-Storage/11-Sharding.md)
- [12. Horizontal vs Vertical Partitioning](Phase-2-Database-and-Storage/12-Horizontal-vs-Vertical-Partitioning.md)
- [13. Shard Key](Phase-2-Database-and-Storage/13-Shard-Key.md)
- [14. Hot Partition](Phase-2-Database-and-Storage/14-Hot-Partition.md)
- [15. Rebalancing](Phase-2-Database-and-Storage/15-Rebalancing.md)

## 7. Distributed Database Concepts
- [16. Replication](Phase-2-Database-and-Storage/16-Replication.md)
- [17. Leader-Follower](Phase-2-Database-and-Storage/17-Leader-Follower.md)
- [18. Multi-Leader](Phase-2-Database-and-Storage/18-Multi-Leader.md)
- [19. Leaderless](Phase-2-Database-and-Storage/19-Leaderless.md)
- [20. Quorum](Phase-2-Database-and-Storage/20-Quorum.md)
- [21. Consistency](Phase-2-Database-and-Storage/21-Consistency.md)
- [22. Strong vs Eventual Consistency](Phase-2-Database-and-Storage/22-Strong-vs-Eventual-Consistency.md)
- [23. Distributed Transactions](Phase-2-Database-and-Storage/23-Distributed-Transactions.md)

## 8. Caching
- [24. Why Caching](Phase-2-Database-and-Storage/24-Why-Caching.md)
- [25. Cache-Aside](Phase-2-Database-and-Storage/25-Cache-Aside.md)
- [26. Read-Through](Phase-2-Database-and-Storage/26-Read-Through.md)
- [27. Write-Through](Phase-2-Database-and-Storage/27-Write-Through.md)
- [28. Write-Back](Phase-2-Database-and-Storage/28-Write-Back.md)
- [29. Redis](Phase-2-Database-and-Storage/29-Redis.md)
- [30. Cache Invalidation](Phase-2-Database-and-Storage/30-Cache-Invalidation.md)
- [31. TTL](Phase-2-Database-and-Storage/31-TTL.md)
- [32. Eviction Policies](Phase-2-Database-and-Storage/32-Eviction-Policies.md)
- [33. Cache Stampede](Phase-2-Database-and-Storage/33-Cache-Stampede.md)
- [34. Cache Penetration](Phase-2-Database-and-Storage/34-Cache-Penetration.md)
- [35. Hot Keys](Phase-2-Database-and-Storage/35-Hot-Keys.md)

# Phase 3 — Core Distributed System Components

## 9. Message Queues
- [01. Queue](Phase-3-Distributed-System-Components/01-Queue.md)
- [02. Producer and Consumer](Phase-3-Distributed-System-Components/02-Producer-Consumer.md)
- [03. Kafka](Phase-3-Distributed-System-Components/03-Kafka.md)
- [04. RabbitMQ](Phase-3-Distributed-System-Components/04-RabbitMQ.md)
- [05. Pub/Sub](Phase-3-Distributed-System-Components/05-Pub-Sub.md)
- [06. Consumer Groups](Phase-3-Distributed-System-Components/06-Consumer-Groups.md)
- [07. Partition](Phase-3-Distributed-System-Components/07-Partition.md)
- [08. Offset](Phase-3-Distributed-System-Components/08-Offset.md)
- [09. Ordering](Phase-3-Distributed-System-Components/09-Ordering.md)
- [10. Retry](Phase-3-Distributed-System-Components/10-Retry.md)
- [11. Dead Letter Queue](Phase-3-Distributed-System-Components/11-Dead-Letter-Queue.md)

## 10. Event-Driven Architecture
- [12. Event](Phase-3-Distributed-System-Components/12-Event.md)
- [13. Command](Phase-3-Distributed-System-Components/13-Command.md)
- [14. Event Sourcing](Phase-3-Distributed-System-Components/14-Event-Sourcing.md)
- [15. CQRS](Phase-3-Distributed-System-Components/15-CQRS.md)
- [16. Async Processing](Phase-3-Distributed-System-Components/16-Async-Processing.md)
- [17. Eventual Consistency](Phase-3-Distributed-System-Components/17-Eventual-Consistency.md)

## 11. API Design
- [18. REST](Phase-3-Distributed-System-Components/18-REST.md)
- [19. gRPC](Phase-3-Distributed-System-Components/19-gRPC.md)
- [20. GraphQL](Phase-3-Distributed-System-Components/20-GraphQL.md)
- [21. WebSocket](Phase-3-Distributed-System-Components/21-WebSocket.md)
- [22. SSE](Phase-3-Distributed-System-Components/22-SSE.md)
- [23. Polling](Phase-3-Distributed-System-Components/23-Polling.md)
- [24. Long Polling](Phase-3-Distributed-System-Components/24-Long-Polling.md)
- [25. API Gateway](Phase-3-Distributed-System-Components/25-API-Gateway.md)
- [26. Rate Limiting](Phase-3-Distributed-System-Components/26-Rate-Limiting.md)

## 12. Rate Limiter
- [27. Token Bucket](Phase-3-Distributed-System-Components/27-Token-Bucket.md)
- [28. Leaky Bucket](Phase-3-Distributed-System-Components/28-Leaky-Bucket.md)
- [29. Fixed Window](Phase-3-Distributed-System-Components/29-Fixed-Window.md)
- [30. Sliding Window](Phase-3-Distributed-System-Components/30-Sliding-Window.md)
- [31. Distributed Rate Limiting](Phase-3-Distributed-System-Components/31-Distributed-Rate-Limiting.md)
- [32. Redis-Based Implementation](Phase-3-Distributed-System-Components/32-Redis-Based-Implementation.md)

# Phase 4 — Reliability & Resilience

## 13. Fault Tolerance
- [01. Retry](Phase-4-Reliability-and-Resilience/01-Retry.md)
- [02. Exponential Backoff](Phase-4-Reliability-and-Resilience/02-Exponential-Backoff.md)
- [03. Timeout](Phase-4-Reliability-and-Resilience/03-Timeout.md)
- [04. Circuit Breaker](Phase-4-Reliability-and-Resilience/04-Circuit-Breaker.md)
- [05. Bulkhead](Phase-4-Reliability-and-Resilience/05-Bulkhead.md)
- [06. Failover](Phase-4-Reliability-and-Resilience/06-Failover.md)
- [07. Graceful Degradation](Phase-4-Reliability-and-Resilience/07-Graceful-Degradation.md)

## 14. Distributed Systems Problems
- [08. Network Partition](Phase-4-Reliability-and-Resilience/08-Network-Partition.md)
- [09. Duplicate Messages](Phase-4-Reliability-and-Resilience/09-Duplicate-Messages.md)
- [10. Message Loss](Phase-4-Reliability-and-Resilience/10-Message-Loss.md)
- [11. Out-of-Order Events](Phase-4-Reliability-and-Resilience/11-Out-of-Order-Events.md)
- [12. Idempotency](Phase-4-Reliability-and-Resilience/12-Idempotency.md)
- [13. Exactly-Once vs At-Least-Once](Phase-4-Reliability-and-Resilience/13-Exactly-Once-vs-At-Least-Once.md)
- [14. Distributed Locking](Phase-4-Reliability-and-Resilience/14-Distributed-Locking.md)

## 15. Distributed Transactions
- [15. Two-Phase Commit (2PC)](Phase-4-Reliability-and-Resilience/15-2PC.md)
- [16. Saga Pattern](Phase-4-Reliability-and-Resilience/16-Saga-Pattern.md)
- [17. Choreography](Phase-4-Reliability-and-Resilience/17-Choreography.md)
- [18. Orchestration](Phase-4-Reliability-and-Resilience/18-Orchestration.md)
- [19. Compensating Transactions](Phase-4-Reliability-and-Resilience/19-Compensating-Transactions.md)

# Phase 5 — Communication and Infrastructure
- [01. Service Communication](Phase-5-Communication-and-Infrastructure/01-Service-Communication.md)
- [02. Service Discovery](Phase-5-Communication-and-Infrastructure/02-Service-Discovery.md)
- [03. API Gateway](Phase-5-Communication-and-Infrastructure/03-API-Gateway.md)
- [04. CDN](Phase-5-Communication-and-Infrastructure/04-CDN.md)
- [05. Object and File Storage](Phase-5-Communication-and-Infrastructure/05-Object-and-File-Storage.md)

# Phase 6 — Security
- [01. Authentication](Phase-6-Security/01-Authentication.md)
- [02. Authorization](Phase-6-Security/02-Authorization.md)
- [03. Security in Distributed Systems](Phase-6-Security/03-Security-in-Distributed-Systems.md)

# Phase 7 — Observability
- [01. Logging](Phase-7-Observability/01-Logging.md)
- [02. Metrics](Phase-7-Observability/02-Metrics.md)
- [03. Distributed Tracing](Phase-7-Observability/03-Distributed-Tracing.md)
- [04. Monitoring and Alerting](Phase-7-Observability/04-Monitoring-and-Alerting.md)

# Phase 8 — HLD Estimation
- [01. Back-of-the-Envelope Estimation](Phase-8-HLD-Estimation/01-Back-of-the-Envelope-Estimation.md)

# Phase 9 — System Design Patterns
- [01. Important HLD Patterns](Phase-9-System-Design-Patterns/01-Important-HLD-Patterns.md)

# Phase 10 — System Design Problems

## Tier 1 — Must Do
1. Ride Sharing App
2. Gmail
3. WhatsApp
4. Zoom
5. Google Docs
6. Dropbox
7. Spotify
8. Google Maps
9. Twitter/X
10. Video Streaming Service (Netflix/YouTube)
11. BookMyShow
12. Uber
13. Zomato
14. Swiggy
15. Flipkart

## Tier 2 — Very Important
16. AWS Lambda
17. Tinder
18. Event Calendar
19. Cryptocurrency Exchange
20. Codepair Platform
21. Chat System
22. BitTorrent
23. Distributed Search
24. Pinterest
25. Logistics System
26. Online Hotel Booking System
27. Udaan

## Tier 3 — Additional Practice
28. Lift/Elevator System
29. 2048 Game
30. Splitwise
31. Game Engine
32. Newsletter Service
33. Mentorship Platform
34. Music Recognition System (Shazam)
35. CricInfo/Cricbuzz
36. E-Commerce Review System
37. Online Book Reader System
38. Blockchain
39. Maps Navigator Client
40. Tic Tac Toe
41. Configuration Management System
42. Vending Machine
43. Food Delivery App
44. Ola

## System Design Solution Framework

For every problem we will follow:
1. Requirements — Functional + Non-Functional
2. Scale Estimation
3. API Design
4. High-Level Architecture
5. Database / Data Model
6. Cache and Storage
7. Communication and Events
8. Scaling Strategy
9. Reliability and Failure Handling
10. Security
11. Bottlenecks and Trade-offs
12. Interview Follow-up Questions
13. 2–3 minute Interview Summary

> The problem list is intentionally organized for interview preparation. Detailed solutions will be added one problem at a time after the fundamentals are studied.
