# 09. CAP Theorem

## What CAP Says

For a distributed data system, during a **network partition**, you cannot simultaneously guarantee both:

- **Consistency (C):** reads reflect a single, up-to-date view according to the system's consistency model.
- **Availability (A):** every request to a non-failing node receives a response.
- **Partition Tolerance (P):** the system continues operating despite communication failures between nodes.

The key interview phrase is:

> CAP is fundamentally about the trade-off exposed when a partition occurs.

## Why Partition Tolerance Matters

Network failures are possible in distributed systems. Therefore, a practical distributed system generally has to tolerate partitions and make a choice about behavior during that partition.

## CP

Prefer consistency over availability during a partition.

A request may be rejected or delayed rather than returning potentially stale/conflicting data.

Useful when correctness is more important than serving every request.

## AP

Prefer availability during a partition, accepting that responses may temporarily diverge or be stale depending on the system's consistency model.

Useful for systems where availability and eventual convergence are acceptable.

## Important Nuance

CAP does **not** mean a system simply chooses any two of C, A and P all the time. The trade-off is specifically visible when a network partition happens.

Also, CAP consistency is not the same thing as ACID transaction consistency.

## Interview-Ready Answer

> CAP states that when a distributed system experiences a network partition, it cannot guarantee both strong consistency and availability at the same time. Since partitions must be considered in distributed systems, we choose whether the system should favor consistent responses or continued availability during that failure scenario.
