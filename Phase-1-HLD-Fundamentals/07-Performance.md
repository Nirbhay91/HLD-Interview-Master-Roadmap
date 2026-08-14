# 07. Performance

## Definition

Performance describes how efficiently a system responds to workload. In HLD, common dimensions are latency, throughput, resource utilization and efficiency.

## Main Levers

- Efficient algorithms
- Database indexing
- Caching
- Connection pooling
- Batching
- Asynchronous processing
- Compression where useful
- Query optimization
- Horizontal scaling
- Reducing unnecessary network calls

## Performance Is Not Just CPU

A service can have low CPU usage and still be slow because it is waiting on:

- Database
- Network
- External API
- Lock contention
- Disk I/O
- Queue backlog

## Tail Latency

Average latency can hide bad user experiences. Percentiles such as p95, p99 and p99.9 show the slow end of the distribution.

Example: p99 = 500 ms means 99% of requests are at or below 500 ms and the slowest 1% are above it.

## Performance Optimization Process

1. Define the target.
2. Measure current behavior.
3. Identify the bottleneck.
4. Optimize the bottleneck.
5. Measure again.
6. Validate that the optimization did not harm correctness or reliability.

## Interview-Ready Answer

> Performance is the efficiency with which a system handles workload, usually discussed through latency, throughput and resource utilization. I would avoid premature optimization: first define measurable targets, identify the bottleneck using metrics/traces, then optimize using techniques such as caching, indexing, batching, asynchronous processing or horizontal scaling.
