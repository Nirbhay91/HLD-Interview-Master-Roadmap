# 10. PACELC

## Definition

PACELC extends the CAP discussion.

It says:

- **If there is a Partition (P), choose between Availability (A) and Consistency (C).**
- **Else (E), during normal operation, choose between Latency (L) and Consistency (C).**

So:

`P → A/C`

`Else → L/C`

## Why PACELC Matters

CAP focuses on partition behavior. Real systems also make trade-offs during normal operation.

For example, stronger consistency may require coordination between replicas, increasing latency.

## Example

Suppose a distributed database replicates data across regions.

Normal operation:

- Cross-region coordination may improve consistency.
- But coordination adds network latency.

During partition:

- Continue serving locally for availability.
- Or reject/delay operations to preserve stronger consistency.

## CAP vs PACELC

| CAP | PACELC |
|---|---|
| Focuses on partition scenario | Covers partition and normal operation |
| P → A/C | P → A/C, else L/C |
| Highlights partition trade-off | Also highlights latency-consistency trade-off |

## Interview-Ready Answer

> PACELC extends CAP by explaining that even when there is no partition, distributed systems often trade latency against consistency. During a partition we choose between availability and consistency; otherwise we may choose between lower latency and stronger consistency.
