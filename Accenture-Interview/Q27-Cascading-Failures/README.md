# Q27 — What are Cascading Failures?

> **Interview Question:** What are Cascading Failures?

## 1. Simple Hinglish Explanation

Cascading Failure ka matlab hai:

> **Ek service/component ka failure gradually dusri dependent services ko overload ya fail kar de, aur failure chain reaction ki tarah poore system mein spread ho jaye.**

Simple example:

```text
Payment Service
      ↓
Payment DB slow
      ↓
Payment requests slow
      ↓
Order Service waits
      ↓
Order threads/connections occupied
      ↓
Order Service overloaded
      ↓
API Gateway requests fail
      ↓
Overall system degraded ❌
```

Yaani **one failure → dependency pressure → another failure → more pressure → wider outage**.

---

# 2. Why Does Cascading Failure Happen?

Distributed systems mein services ek doosre par dependent hoti hain:

```text
Order
  ↓
Payment
  ↓
Payment DB
```

Agar downstream dependency slow/fail ho jaye aur upstream service usko unlimited wait/retry kare, to resources exhaust ho sakte hain.

Common reasons:

- No timeout
- Unlimited retries
- Retry storm
- Connection pool exhaustion
- Thread pool exhaustion
- Queue buildup
- Slow database
- Dependency failure
- Synchronous dependency chain
- No rate limiting
- No circuit breaker
- Unbounded concurrency
- Poor resource isolation

---

# 3. Simple Real-World Example

Suppose:

```text
100 Order requests/sec
```

Order Service Payment Service ko call karta hai.

Normally:

```text
Payment latency = 50 ms
```

Suddenly Payment DB slow ho gaya:

```text
Payment latency = 5 sec
```

Ab Order Service ke requests 5 seconds tak wait karne lagenge.

Agar 100 requests/sec aa rahi hain:

```text
100 × 5 sec ≈ 500 requests
```

concurrently in-flight ho sakti hain before considering other limits.

Agar Order Service ke threads/connections limited hain, woh exhaust ho sakte hain.

Then:

```text
Payment slow
   ↓
Order slow
   ↓
Threads exhausted
   ↓
Order unavailable
```

This is cascading failure.

---

# 4. Classic Microservices Example

```text
                API Gateway
                     ↓
                Order Service
                     ↓
                Payment Service
                     ↓
                Payment DB
```

Payment DB becomes slow:

```text
Payment DB ❌/Slow
      ↓
Payment Service Slow
      ↓
Order Service waits
      ↓
Order resources exhausted
      ↓
Gateway receives failures/timeouts
      ↓
Users see errors
```

---

# 5. Cascading Failure vs Single Failure

### Single Failure

Only one component fails:

```text
Payment Service ❌
```

Other services remain healthy because failure is isolated.

### Cascading Failure

```text
Payment ❌
   ↓
Order degraded
   ↓
Gateway degraded
   ↓
More services impacted
```

### Memory

> **Single failure stays isolated. Cascading failure spreads.**

---

# 6. Main Mechanism: Resource Exhaustion

Most cascading failures eventually resource exhaustion tak pahunch sakte hain.

Resources:

```text
Threads
Connections
CPU
Memory
File descriptors
DB connections
HTTP connections
Queue capacity
```

Example:

```text
Downstream slow
      ↓
Requests wait longer
      ↓
More concurrent requests
      ↓
Threads/connections occupied
      ↓
Resource exhaustion
      ↓
Service fails
```

---

# 7. No Timeout = Dangerous

Suppose Order Service Payment ko call karta hai:

```text
Order → Payment
```

Payment response nahi de raha.

Without timeout:

```text
Request waits...
Request waits...
Request waits...
```

Hundreds/thousands of requests accumulate kar sakti hain.

Better:

```text
Payment call timeout = 2 sec
```

Then:

```text
Payment slow
   ↓
Timeout
   ↓
Fail fast / fallback
```

Important:

> Timeout alone doesn't fix the dependency; it limits how long resources remain occupied.

---

# 8. Retry Storm

Suppose Payment temporarily fails.

100 requests fail:

```text
100 failures
   ↓
Each retries 3 times
   ↓
300 extra requests
```

Now Payment is even more overloaded.

```text
Failure
  ↓
Retry
  ↓
More Load
  ↓
More Failure
  ↓
More Retry
```

This creates a feedback loop.

---

# 9. Why Retry Without Backoff Is Dangerous

Bad:

```text
Failure
 ↓
Retry immediately
 ↓
Failure
 ↓
Retry immediately
```

Better:

```text
Retry
 ↓
Exponential Backoff
 ↓
Jitter
 ↓
Retry
```

Example conceptual delays:

```text
100 ms
200 ms
400 ms
800 ms
```

Actual values depend on system and policy.

---

# 10. Circuit Breaker

Circuit Breaker cascading failure prevent karne ka important pattern hai.

States:

```text
CLOSED
  ↓ failures exceed threshold
OPEN
  ↓ wait
HALF-OPEN
  ↓ test request
CLOSED / OPEN
```

### Closed

Requests normally flow:

```text
Order → Payment
```

### Open

Payment repeatedly failing:

```text
Order
  ↓
Circuit Breaker OPEN
  ↓
Do NOT call Payment
```

Fail fast.

### Half-Open

After wait period, limited test requests allow ki jaati hain.

---

# 11. Circuit Breaker Helps How?

Without:

```text
Order → Payment ❌
Order → Payment ❌
Order → Payment ❌
Order → Payment ❌
```

With circuit breaker:

```text
Order
 ↓
Circuit OPEN
 ↓
Fast failure / fallback
```

This protects:

```text
Threads
Connections
CPU
Latency budget
Downstream service
```

---

# 12. Bulkhead Pattern

Bulkhead resources ko isolate karta hai.

Example:

```text
Order Service
 ├── Payment Pool
 ├── Inventory Pool
 └── Notification Pool
```

Agar Payment slow hai:

```text
Payment Pool exhausted
```

but Inventory pool still available ho sakta hai.

Without isolation:

```text
Payment slow
   ↓
All shared threads consumed
   ↓
Entire Order Service impacted
```

### Memory

> **Bulkhead = ek dependency ko poore service ke resources consume karne se rokna.**

---

# 13. Rate Limiting

Agar dependency already overloaded hai, unlimited traffic bhejna situation worse karega.

```text
Incoming Traffic
      ↓
Rate Limiter
      ↓
Allowed Requests
      ↓
Dependency
```

Excess traffic:

```text
Reject / Delay / Queue
```

This protects system capacity.

---

# 14. Load Shedding

When system is overloaded, some requests intentionally reject/drop ki ja sakti hain.

```text
Overload
   ↓
Load Shedding
   ↓
Reject low-priority traffic
   ↓
Protect critical operations
```

Example:

```text
Payment → High priority
Recommendation → Low priority
```

If overloaded, recommendation traffic degrade ho sakta hai while payment remains available.

---

# 15. Graceful Degradation

Dependency unavailable ho to complete application fail karne ke bajay reduced functionality provide karna.

Example:

```text
Recommendation Service ❌
        ↓
Checkout still works
        ↓
Show cached/default recommendations
```

Another example:

```text
Profile Service slow
        ↓
Use cached profile data
```

Only when stale data is acceptable.

---

# 16. Asynchronous Communication

Synchronous chain:

```text
Order
 ↓
Payment
 ↓
Notification
 ↓
Analytics
```

One slow service can block the chain.

Async:

```text
Order
 ↓
Kafka
 ├→ Payment
 ├→ Notification
 └→ Analytics
```

Producer can continue after publishing an event, depending on business semantics.

Async does not eliminate failure; it can reduce synchronous coupling and provide buffering.

---

# 17. Queue as a Shock Absorber

Suppose:

```text
Incoming = 10,000 events/sec
Consumer = 5,000 events/sec
```

Queue:

```text
Producer
   ↓
Queue
   ↓
Consumers
```

Backlog temporarily grows instead of immediately overwhelming consumers.

Then:

```text
Scale Consumers
   ↓
Process Backlog
```

But queue capacity and acceptable message age must also be considered.

---

# 18. Connection Pool Exhaustion

Example:

```text
Max DB connections = 100
```

Requests keep waiting because DB is slow.

Eventually:

```text
100 connections occupied
      ↓
New request waits
      ↓
Timeout
      ↓
More retries
      ↓
More load
```

This can become a feedback loop.

Mitigation:

```text
Connection pool limits
Timeouts
Query optimization
Backpressure
Circuit breaker
Bulkhead
```

---

# 19. Thread Pool Exhaustion

Suppose:

```text
Thread pool = 200
```

Downstream calls take too long.

```text
200 threads waiting
      ↓
No threads available
      ↓
New requests queue/fail
```

Service becomes unhealthy even if CPU isn't necessarily 100%.

This is why:

> **CPU alone is not enough to detect service health.**

---

# 20. Queue Buildup

If producer rate > consumer processing rate:

```text
Producer = 10k/s
Consumer = 5k/s
```

Backlog grows:

```text
Queue
████████████████
```

Eventually:

```text
Memory / disk / queue capacity
          ↓
Exhaustion
```

Monitor:

```text
Queue depth
Consumer lag
Oldest message age
Processing rate
```

---

# 21. Database as Cascading Failure Source

Common scenario:

```text
Application
    ↓
Database slow
```

Then:

```text
DB latency ↑
    ↓
Connection usage ↑
    ↓
Application requests wait
    ↓
Thread pool ↑
    ↓
Latency ↑
    ↓
Timeouts
    ↓
Retries
    ↓
More DB load
```

This is a classic feedback loop.

---

# 22. Cache Failure Can Cause Cascading Failure

Suppose Redis suddenly fails:

```text
Redis ❌
  ↓
All requests become cache misses
  ↓
Database traffic spikes
  ↓
Database overload
  ↓
Application latency increases
  ↓
Requests timeout
```

Therefore:

> **Caching itself can become a failure amplifier if fallback is not protected.**

---

# 23. Auto Scaling Can Sometimes Amplify a Failure

Autoscaling is useful, but blindly scaling app instances may not fix a downstream bottleneck.

```text
DB overloaded
   ↓
App latency ↑
   ↓
Autoscaler adds more app instances
   ↓
More DB connections/queries
   ↓
DB even more overloaded
```

This can make the outage worse.

Important interview point:

> **Scaling the symptom instead of the bottleneck can amplify failure.**

---

# 24. Dependency Graph

Suppose:

```text
             API
              ↓
            Order
          /   |   \
         ↓    ↓    ↓
    Payment Inventory Notification
       ↓
      DB
```

Payment DB failure can affect:

```text
Payment
   ↓
Order
   ↓
API
```

But if Notification is asynchronous and isolated:

```text
Notification failure
        ↓
Notification only
```

Architecture should minimize blast radius.

---

# 25. Blast Radius

Blast radius = failure ka impact kitne components/users tak spread hota hai.

Good design:

```text
Failure
 ↓
Small isolated area
```

Bad design:

```text
Failure
 ↓
Many services
 ↓
Entire platform
```

Ways to reduce blast radius:

```text
Bulkheads
Isolation
Rate limits
Circuit breakers
Separate pools
Cell-based architecture
Regional isolation
```

---

# 26. Timeouts Must Be Layered Carefully

Suppose:

```text
API timeout = 3 sec
Order → Payment timeout = 2 sec
Payment → DB timeout = 5 sec
```

This can create poor behavior because downstream timeout exceeds upstream deadline.

Better principle:

> **Timeout/deadline budgets should be coordinated across the request chain.**

Conceptually:

```text
Client deadline = 3 sec
        ↓
Order budget < 3 sec
        ↓
Payment budget < remaining time
        ↓
DB budget < remaining time
```

Exact values depend on workload.

---

# 27. Deadline Propagation

Distributed request ke saath deadline propagate ki ja sakti hai.

```text
Client
 deadline = 2 sec
      ↓
Order
 remaining = 1.8 sec
      ↓
Payment
 remaining = 1.2 sec
```

If deadline is nearly exhausted:

```text
Don't start expensive work
```

This prevents useless work and resource consumption.

---

# 28. Retry Only Safe Operations

Retry blindly dangerous hai.

Example:

```text
POST /payment
```

Network timeout hua. Client ko pata nahi payment processed hui ya nahi.

Retry can create duplicate payment.

Use:

```text
Idempotency Key
```

Example:

```text
Idempotency-Key: abc123
```

Then repeated request same logical operation ko duplicate nahi karegi, assuming service implements idempotency correctly.

---

# 29. Retry Budget

Distributed systems mein retries ko bounded rakhna chahiye.

Instead of:

```text
Unlimited retries ❌
```

Use:

```text
Maximum attempts
Maximum elapsed time
Exponential backoff
Jitter
Retry only transient errors
```

This limits retry amplification.

---

# 30. Observability

Cascading failures detect karne ke liye monitor:

```text
Request rate
Error rate
Latency
P95 / P99
CPU
Memory
Thread pool
Connection pool
DB latency
Queue depth
Consumer lag
Circuit breaker state
Retry count
```

Distributed tracing:

```text
Trace
 ├─ API
 ├─ Order
 ├─ Payment
 └─ DB
```

Can show where latency/failure originates.

---

# 31. SLI / SLO

Important signals:

```text
Availability
Latency
Error rate
Throughput
```

SLO define karta hai desired reliability target.

Example:

```text
99.9% successful requests
P99 latency < defined target
```

During overload, SLO helps prioritize reliability decisions.

---

# 32. How to Prevent Cascading Failures?

Interview mein complete answer:

```text
Timeouts
Retries + Exponential Backoff + Jitter
Circuit Breaker
Bulkhead
Rate Limiting
Load Shedding
Backpressure
Async Queues
Graceful Degradation
Connection/Thread Pool limits
Idempotency
Health Checks
Autoscaling
Monitoring & Tracing
```

But important:

> **No single pattern is enough. These controls work together.**

---

# 33. Resilience Architecture

```text
                     Client
                       ↓
                Rate Limiter / WAF
                       ↓
                 Load Balancer
                       ↓
                  API Gateway
                       ↓
                Order Service
                       ↓
             ┌── Timeout ──┐
             │ Circuit     │
             │ Breaker     │
             │ Bulkhead    │
             └─────┬───────┘
                   ↓
             Payment Service
                   ↓
              Payment DB
```

Async side:

```text
Order
  ↓
Kafka / Queue
  ↓
Notification Workers
```

This limits synchronous dependency chains and isolates failure domains.

---

# 34. Interview Scenario

### Interviewer:

> Payment Service is down. How will you prevent the entire Order Service from going down?

### Strong Answer

```text
1. Set a bounded timeout on Payment calls.
2. Use a circuit breaker so repeated failures stop calls temporarily.
3. Isolate Payment resources using a bulkhead/thread or connection pool.
4. Retry only transient failures with bounded exponential backoff and jitter.
5. Use async processing if the business flow allows it.
6. Provide a fallback/graceful degradation where business semantics allow.
7. Use rate limiting and load shedding under overload.
8. Protect DB and other downstream dependencies from retry amplification.
9. Monitor latency, errors, retries and circuit state.
```

Then add:

> **"The key objective is to contain the failure and reduce the blast radius instead of allowing one unhealthy dependency to consume all Order Service resources."**

---

# 35. Interview-Ready Answer

> **"A cascading failure occurs when the failure or severe slowdown of one component causes dependent components to become overloaded or fail as well, spreading the problem through the system. For example, if Payment Service becomes slow, Order Service may keep requests waiting until its thread or connection pools are exhausted. Timeouts, retries and retry storms can make the situation worse. I prevent cascading failures using bounded timeouts, limited retries with exponential backoff and jitter, circuit breakers, bulkheads, rate limiting, backpressure, asynchronous queues, graceful degradation and proper resource limits. I also use monitoring and distributed tracing to detect where the failure starts and design services to limit the blast radius."**

---

# 36. 30-Second Hinglish Answer

> **"Cascading failure tab hota hai jab ek service ka failure ya slowdown uski dependent service ko overload kar deta hai aur failure chain reaction ki tarah spread hota hai. Example, Payment slow hua, Order Service requests wait karengi, threads aur connections exhaust ho sakte hain, phir Order Service bhi fail ho sakti hai. Isko prevent karne ke liye timeout, circuit breaker, bounded retry with exponential backoff and jitter, bulkhead, rate limiting, backpressure, async processing aur graceful degradation use karte hain. Main goal hota hai failure ko contain karna aur blast radius limit karna."**

---

# 37. Memory Trick

```text
ONE SERVICE SLOWS
        ↓
REQUESTS WAIT
        ↓
THREADS / CONNECTIONS EXHAUST
        ↓
TIMEOUTS / RETRIES
        ↓
MORE LOAD
        ↓
MORE SERVICES FAIL
```

### Prevention

```text
TIMEOUT
   ↓
CIRCUIT BREAKER
   ↓
BULKHEAD
   ↓
RATE LIMIT
   ↓
BACKPRESSURE
   ↓
GRACEFUL DEGRADATION
```

### One-line memory

> **"Cascading Failure = one dependency failure spreads because resources are not isolated or protected."**

---

# 38. Common Interview Mistakes

### Mistake 1

> "Circuit breaker prevents every cascading failure."

Incomplete. It is one protection mechanism among several.

### Mistake 2

> "Just add retries."

Dangerous. Unbounded retries can amplify load.

### Mistake 3

> "Just autoscale."

Wrong if DB/downstream dependency is the bottleneck.

### Mistake 4

> "Timeout means failure is solved."

Timeout limits waiting time; it doesn't fix the underlying dependency.

### Mistake 5

> "Use synchronous communication everywhere."

Synchronous calls create dependency chains; async processing can reduce coupling where business semantics allow.

---

# 39. Follow-Up Questions

### Q. What is the difference between cascading failure and retry storm?

Retry storm is a mechanism that can amplify load during failures; cascading failure is the broader spread of failure across components.

### Q. How does circuit breaker help?

It stops repeated calls to an unhealthy dependency after a failure threshold and allows controlled recovery testing.

### Q. What is Bulkhead pattern?

It isolates resources so one dependency cannot consume all service capacity.

### Q. Why are retries dangerous?

They can multiply traffic against an already unhealthy dependency.

### Q. Why use exponential backoff and jitter?

To spread retries over time and reduce synchronized retry bursts.

### Q. Can autoscaling cause cascading failure?

Yes, if scaling app instances increases pressure on an already overloaded downstream DB/service.

### Q. How does caching failure cause cascading failure?

Cache outage can turn many requests into DB misses, causing a sudden DB traffic spike.

### Q. How does async communication help?

It can remove synchronous blocking dependencies and provide buffering, but it introduces eventual consistency and queue-management considerations.

### Q. What metrics detect cascading failures?

Latency, error rate, retries, thread/connection pool usage, DB latency, queue depth, consumer lag and dependency health.

### Q. What is blast radius?

The scope of users/components affected by a failure.

---

## Status

✅ **Q27 Solution Completed**

Next: **Q28 — How do you prevent cascading failures in a microservices architecture?**
