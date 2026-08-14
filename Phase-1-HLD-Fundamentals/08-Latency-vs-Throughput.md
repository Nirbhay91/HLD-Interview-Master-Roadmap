# 08. Latency vs Throughput

## Latency

Latency is the time taken to complete a request or operation.

Example:

`Request → 120 ms → Response`

Lower latency is generally better when the product requires fast interaction.

## Throughput

Throughput is the amount of work a system completes per unit of time.

Examples:

- Requests per second (RPS)
- Transactions per second (TPS)
- Messages per second
- MB/s

## Difference

| Latency | Throughput |
|---|---|
| Time per operation | Operations per unit time |
| Usually measured in ms/s | Usually RPS/TPS/etc. |
| User-perceived responsiveness | Capacity/work processing rate |

A system can have high throughput but high latency, especially with batching or asynchronous processing.

## Queue Example

A message consumer may process 10,000 messages/sec but an individual message may wait in a queue before processing. Therefore throughput and end-to-end latency are different concerns.

## Tail Latency

Use p95/p99 rather than only averages for user-facing systems.

## Interview-Ready Answer

> Latency is the time required to complete an operation, while throughput is how much work the system completes per unit time. They are related but not interchangeable. A system can increase throughput through batching or parallelism while still having high individual-request latency, so both need separate targets.
