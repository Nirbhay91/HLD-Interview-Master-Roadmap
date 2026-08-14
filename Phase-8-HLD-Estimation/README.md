# Phase 8 — HLD Interview Calculations

This section is especially important for HLD interviews. Before choosing an architecture, understand the expected scale and workload.

## 28. Back-of-the-Envelope Estimation

- Users
- DAU / MAU
- Requests per second (RPS)
- Peak QPS
- Storage
- Bandwidth
- Cache size
- Replication overhead

## Example

Assume:

- 10M DAU
- 1 user = 20 requests/day

Total requests:

```text
10M × 20 = 200M requests/day
```

Average QPS:

```text
200M / 86,400 ≈ 2,315 QPS
```

Then apply an appropriate peak multiplier to estimate peak QPS and use that scale to drive architecture decisions.

## Interview Goal

The interviewer wants to see that you understand the system's scale before selecting components such as databases, caches, queues, load balancers, storage, replication and service capacity.

> Detailed estimation techniques, formulas, worked examples and interview questions will be added later.
