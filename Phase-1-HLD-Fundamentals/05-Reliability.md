# 05. Reliability

## Definition

Reliability is the ability of a system to perform its intended function correctly and consistently over time, including under failures.

## Reliability Techniques

- Replication
- Retries with bounded backoff
- Timeouts
- Circuit breakers
- Idempotency
- Durable queues
- Data backups
- Validation
- Failure isolation
- Disaster recovery

## Retry Carefully

Retries help with transient failures but can amplify load when a dependency is already overloaded.

Use:

- Timeouts
- Exponential backoff
- Jitter
- Maximum retry limits

## Idempotency

An operation is idempotent when repeating the same request does not create an unintended additional effect.

Example: a payment request with an idempotency key should not charge the customer twice when the client retries after a timeout.

## Availability vs Reliability

| Availability | Reliability |
|---|---|
| Can the system serve? | Does it work correctly and consistently? |
| Focuses on operational access | Focuses on correct behavior over time |
| Often measured as uptime | Often measured through failures/error rates and recovery behavior |

## Interview-Ready Answer

> Reliability is about consistently performing the correct operation despite failures. I design for reliability using timeouts, controlled retries, idempotency, replication, durable messaging, backups and failure isolation. The exact mechanisms depend on the failure modes and business criticality.
