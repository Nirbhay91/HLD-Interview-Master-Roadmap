# HLD Question — Standard Solution Framework

## Har HLD Question ko Same Framework se Solve Karenge

Interview mein random diagram nahi banana hai. Har system-design problem ke liye ek fixed, repeatable structure follow karna hai.

## Step 1 — Requirements

Identify and clarify:

- Functional Requirements
- Non-Functional Requirements

## Step 2 — Scale Estimation

Estimate:

- DAU / MAU
- Requests per second (QPS)
- Peak QPS
- Storage
- Bandwidth
- Cache size where applicable

## Step 3 — API Design

Define the core APIs required by the system.

Example:

```text
POST /messages
GET /messages/{conversationId}
```

## Step 4 — High-Level Architecture

Start with the major components and their interaction:

```text
Client
  ↓
Load Balancer
  ↓
API Gateway
  ↓
Microservices
  ↓
Cache / DB / Kafka
```

Then explain the request/data flow.

## Step 5 — Database Design

Discuss:

- Tables / Collections
- Primary Keys
- Indexes
- Partitioning
- Replication
- Data access patterns

## Step 6 — Scaling

Explain how the system will scale:

- Horizontal Scaling
- Caching
- Sharding
- CDN
- Async Processing

## Step 7 — Reliability

Handle failures using:

- Retry
- Timeout
- Circuit Breaker
- Idempotency
- Failover

## Step 8 — Deep Dive

The interviewer may select any component and ask for deeper design.

Examples:

- Why Kafka?
- Why Redis?
- Why SQL vs NoSQL?
- How does sharding work?
- How do you handle duplicate messages?
- What happens when a service goes down?

Deep-dive according to the interviewer’s direction instead of trying to explain every component at once.

## Step 9 — Bottlenecks & Trade-offs

Identify possible bottlenecks and explicitly explain design trade-offs.

Example:

> "I am choosing eventual consistency here because availability and scalability are more important than immediate consistency."

Always explain **why** a technology or architecture decision was made instead of only naming the technology.

## Step 10 — Final 2-Minute Explanation

End the interview with a concise summary of the complete design.

The goal is to explain the complete system in approximately **2–3 minutes**:

```text
Requirements
    ↓
Scale
    ↓
APIs
    ↓
Architecture
    ↓
Database / Cache
    ↓
Scaling
    ↓
Reliability
    ↓
Trade-offs
```

# Actual Preparation Sequence

We will not solve all system-design problems at once.

First build the foundation:

```text
HLD Fundamentals
        ↓
Distributed Systems
        ↓
Database
        ↓
Cache
        ↓
Kafka / Messaging
        ↓
Reliability
        ↓
Security
        ↓
HLD Estimation
```

After the fundamentals are strong:

```text
Problem 1
   ↓
Complete Interview Solution
   ↓
Problem 2
   ↓
Complete Interview Solution
   ↓
...
```

# What Each System Design Solution Will Contain

For every system, the preparation material will include:

1. Requirements
2. Scale Estimation
3. APIs
4. High-Level Architecture
5. DB Schema / Data Model
6. Scaling Strategy
7. Kafka / Queue / Cache where required
8. Failure Handling
9. Reliability
10. Security
11. Bottlenecks
12. Trade-offs
13. Common Interview Questions
14. Follow-up Questions
15. 2-minute Interview Answer
16. Architecture Diagram

## Goal

The objective is **not to memorize diagrams**.

The objective is to understand the building blocks and trade-offs deeply enough to **derive a new HLD design from requirements and scale during an interview**.
