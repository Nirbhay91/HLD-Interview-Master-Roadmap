# 03. Scalability

## Definition

Scalability is the ability of a system to handle increasing workload by adding resources while maintaining acceptable performance.

## Vertical Scaling

Increase resources of one machine:

- More CPU
- More RAM
- Faster storage

**Pros:** simple, fewer distributed-system problems.

**Cons:** hardware limits, expensive at the upper end, single-machine failure remains a concern.

## Horizontal Scaling

Add more machines/instances.

`Client → Load Balancer → Instance 1 / Instance 2 / Instance 3`

**Pros:** higher capacity, better fault isolation, elastic scaling.

**Cons:** requires distributed coordination, stateless design, load balancing and data partitioning strategies.

## Stateless Services

Horizontal scaling is easiest when application instances are stateless.

Avoid storing user session state only in local memory. Use shared/session infrastructure such as a distributed cache or token-based authentication where appropriate.

## Database Scalability

Application scaling alone is not enough. The database can become the bottleneck.

Common techniques:

- Read replicas
- Caching
- Indexing
- Partitioning/sharding
- Connection pooling
- Archival
- CQRS where justified

## Scaling Dimensions

Think about:

- Compute
- Database
- Cache
- Network
- Storage
- Message processing

## Interview-Ready Answer

> Scalability is the ability to handle increasing load without unacceptable degradation. Vertical scaling increases resources on an existing machine, while horizontal scaling adds more instances. In distributed systems I generally prefer horizontal scaling for large workloads, combined with stateless services, load balancing, caching and appropriate database scaling strategies.

## Common Follow-ups

- Why should services be stateless?
- What happens when the database cannot scale horizontally?
- How would you scale reads differently from writes?
- When is vertical scaling sufficient?
