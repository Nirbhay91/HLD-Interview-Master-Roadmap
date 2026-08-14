# 11. ACID vs BASE

## ACID

ACID describes transaction properties commonly associated with relational databases.

### Atomicity

A transaction is all-or-nothing.

### Consistency

A successful transaction moves the database from one valid state to another according to defined constraints.

### Isolation

Concurrent transactions should behave according to the database's isolation guarantees.

### Durability

Once a transaction is committed, its data should survive failures according to the database's durability guarantees.

## BASE

BASE is commonly used to describe distributed systems that prioritize availability and eventual consistency rather than strong transactional consistency for every operation.

- **Basically Available:** system attempts to provide a response despite failures/partitioning.
- **Soft State:** state may change over time as replicas converge.
- **Eventual Consistency:** if updates stop, replicas eventually converge to a consistent value, subject to the system's guarantees.

## ACID vs BASE

| ACID | BASE |
|---|---|
| Strong transactional guarantees are central | Availability and eventual convergence are often emphasized |
| Common in relational transaction processing | Common in distributed/high-scale systems depending on use case |
| Useful for financial/order invariants | Useful where temporary inconsistency is acceptable |

## Important Nuance

BASE is not simply "no consistency." Distributed systems can provide different consistency models, and eventual consistency is one specific model.

Likewise, ACID and BASE are not strict opposites for every architecture; modern databases can support combinations of transactional and distributed capabilities.

## Example

### Banking transfer

Moving money between accounts needs strong transactional correctness, so ACID-style transactional guarantees are important.

### Social-media like count

A temporarily stale like count may be acceptable if the system eventually converges. Availability and scalability may be prioritized.

## Interview-Ready Answer

> ACID focuses on transaction guarantees—atomicity, consistency, isolation and durability. BASE is a distributed-systems approach that emphasizes availability and eventual consistency where temporary divergence is acceptable. The right choice depends on business invariants and consistency requirements rather than simply choosing one model globally.
