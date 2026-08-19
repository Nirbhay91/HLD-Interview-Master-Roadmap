# Q23 — Suppose One Service Suddenly Receives Very High Traffic. How Will You Handle It?

> **Interview Question:** Suppose one service suddenly receives very high traffic. How will you handle it?

## 1. Simple Hinglish Explanation

Agar ek microservice par suddenly traffic spike aa jaye, to main immediately sirf "more servers" nahi bolunga.

Pehle identify karunga:

```text
Is traffic legitimate?
Is service CPU bound hai?
DB bottleneck hai?
Cache miss hai?
Downstream dependency slow hai?
```

Then protection + scaling:

```text
Traffic Spike
     ↓
Rate Limit / WAF
     ↓
Load Balancer
     ↓
Horizontal Scaling
     ↓
Cache
     ↓
Async Processing
     ↓
DB / Downstream Protection
     ↓
Monitoring
```

---

# 2. First Step — Identify the Bottleneck

High traffic ka matlab automatically CPU bottleneck nahi hota.

Check:

```text
CPU
Memory
QPS
P95/P99 latency
Error rate
Thread pool
Connection pool
DB CPU
DB connections
Cache hit ratio
Downstream latency
Queue depth
```

Example:

```text
Service CPU = 95%
DB CPU = 30%
```

Likely compute bottleneck.

But:

```text
Service CPU = 30%
DB CPU = 95%
```

Scaling service instances alone may not solve the problem.

---

# 3. Step 1 — Verify Traffic

Sudden traffic could be:

```text
Normal business spike
Marketing campaign
Viral event
Bot traffic
DDoS/abuse
Buggy client retry storm
```

Therefore first understand traffic source.

Useful dimensions:

```text
Requests/sec
Users
IP/client
API endpoint
Geography
Status codes
User-Agent/client version
```

---

# 4. Step 2 — Rate Limiting

If traffic is beyond what system can safely handle:

```text
Client
  ↓
Rate Limiter
  ↓
Service
```

Example:

```text
1000 req/sec/client
```

Excess requests:

```text
HTTP 429 Too Many Requests
```

Rate limiting prevents one client/tenant from consuming all capacity.

---

# 5. Step 3 — Horizontal Scaling

If service is stateless and capacity is the bottleneck:

```text
Before:
Load Balancer
     ↓
Service-1

After:
              Load Balancer
            /      |       \
           ↓       ↓        ↓
      Service-1 Service-2 Service-3
```

More instances → more request processing capacity.

This is usually preferred over vertically scaling a single machine when the workload supports horizontal scaling.

---

# 6. Stateless Service

Horizontal scaling easiest tab hoti hai jab service stateless ho.

Bad:

```text
User session
    ↓
Memory of Service-1
```

Request Service-2 par gaya to state missing ho sakti hai.

Better:

```text
Service instances
      ↓
Shared Redis / DB / external state
```

Then any instance can handle request.

---

# 7. Step 4 — Auto Scaling

Traffic continuously increase ho raha ho to auto scaling use kar sakte hain.

Example:

```text
CPU > 70%
OR
Request rate > threshold
OR
Queue depth > threshold
        ↓
Add instances
```

Traffic reduce hone par:

```text
Scale down
```

Important:

> Auto scaling is reactive unless combined with forecasting/scheduled scaling.

---

# 8. Scaling Metric Should Match Bottleneck

CPU-only auto scaling always correct nahi hai.

Different services ke liye:

```text
CPU
Memory
Requests/sec
Concurrency
Queue depth
Latency
Custom business metric
```

Example worker service:

```text
Queue depth = 100,000
```

Queue depth may be a better scaling signal than CPU.

---

# 9. Step 5 — Load Balancing

Multiple instances ke beech traffic distribute karo:

```text
                Load Balancer
              /      |       \
             ↓       ↓        ↓
         Service-1 Service-2 Service-3
```

Health checks ensure unhealthy instances receive less/no new traffic according to the load-balancer behavior.

---

# 10. Step 6 — Caching

Agar high traffic mostly read requests hai:

```text
Client
  ↓
Service
  ↓
Cache
  ↓ cache miss
Database
```

Cache hit:

```text
Client
  ↓
Service
  ↓
Redis
  ↓
Response
```

Database load significantly reduce ho sakta hai.

---

# 11. Cache-Aside Example

```text
Request
   ↓
Check Redis
   ↓
Hit → Return

Miss
   ↓
DB
   ↓
Put in Redis
   ↓
Return
```

Good candidates:

```text
Product catalog
Configuration
Popular content
Read-heavy metadata
```

But highly dynamic data ke liye freshness requirements consider karni hongi.

---

# 12. Step 7 — CDN

Agar traffic static content ke liye hai:

```text
User
 ↓
CDN Edge
 ↓
Origin
```

Examples:

```text
Images
CSS
JS
Videos
Static files
```

CDN origin/service traffic reduce kar sakta hai.

---

# 13. Step 8 — Async Processing

Agar request expensive background operation kar rahi hai:

Bad:

```text
Client
 ↓
Service
 ↓
Long processing
 ↓
Response
```

Better:

```text
Client
 ↓
Service
 ↓
Queue / Kafka
 ↓
Worker
 ↓
Process
```

API quickly job ID return kar sakti hai:

```text
202 Accepted
jobId = 123
```

Then client:

```text
GET /jobs/123
```

or SSE/WebSocket use kar sakta hai for status updates.

---

# 14. Step 9 — Protect Database

Often application service scale karne ke baad DB bottleneck ban jata hai.

```text
100 service instances
       ↓
One DB
       ↓
DB overload
```

Use:

```text
Cache
Read replicas
Connection pooling
Query optimization
Indexes
Batching
Partitioning/sharding where justified
```

Do not blindly add application instances without checking DB capacity.

---

# 15. Connection Pool Protection

High traffic:

```text
1000 requests
   ↓
DB connection pool
   ↓
Only 100 connections
```

If requests wait indefinitely:

```text
Thread exhaustion
Latency increase
Timeouts
```

Set bounded pool sizes and request timeouts according to DB/service capacity.

---

# 16. Backpressure

Agar downstream capacity limited hai, upstream ko unlimited requests forward nahi karni chahiye.

```text
Producer
   ↓
Queue / Buffer
   ↓
Consumer
```

If queue also reaches capacity:

```text
Reject / shed load
```

Backpressure prevents uncontrolled resource growth.

---

# 17. Load Shedding

When system is overloaded, every request ko process karna possible nahi hota.

Priority define karo:

```text
Critical requests → Keep
Non-critical → Reject / degrade
```

Example:

```text
Checkout → Priority
Recommendations → Can degrade
```

This is **graceful degradation**.

---

# 18. Circuit Breaker

Agar high traffic ki wajah se downstream service unhealthy ho:

```text
Order Service
     ↓
Payment Service ❌
```

Repeated calls avoid karne ke liye circuit breaker:

```text
CLOSED
  ↓ failures
OPEN
  ↓ cooldown
HALF-OPEN
  ↓ test
CLOSED / OPEN
```

This helps prevent cascading failures.

---

# 19. Retry Storm Avoid Karo

Suppose service already overloaded hai:

```text
Request fails
   ↓
100 clients retry
   ↓
More traffic
   ↓
More failures
   ↓
More retries
```

This becomes a **retry storm**.

Use:

```text
Bounded retries
Exponential backoff
Jitter
Timeouts
Idempotency
```

And avoid retrying errors that are not transient.

---

# 20. Exponential Backoff + Jitter

Instead of:

```text
Retry after 1 sec
Retry after 1 sec
Retry after 1 sec
```

Use increasing delay:

```text
100ms
200ms
400ms
800ms
```

with jitter to avoid synchronized retry spikes.

---

# 21. Queue-Based Load Leveling

If traffic is bursty:

```text
Clients
   ↓
API
   ↓
Queue
   ↓
Workers
```

Queue absorbs temporary spikes.

Workers process at sustainable rate.

Important:

> Queue does not create infinite capacity. If producers permanently exceed consumer capacity, queue will eventually grow.

---

# 22. Read vs Write Traffic

First identify:

```text
90% reads
10% writes
```

For read-heavy:

```text
Cache
Read replicas
CDN
```

For write-heavy:

```text
Partitioning/sharding
Async processing
Batching
Queue
Write optimization
```

Architecture depends on workload.

---

# 23. Database Read Replica

If read traffic high:

```text
Service
  ↓
Read Router
 ├→ Primary
 └→ Read Replica
```

Read replicas scale read capacity, but replication lag can affect freshness.

Do not blindly send consistency-sensitive reads to replicas.

---

# 24. Sharding

If a single database cannot handle scale:

```text
Data
 ↓
Shard 1
Shard 2
Shard 3
```

Example:

```text
userId % N
```

But shard key selection is critical.

Bad shard key can create a hot partition.

---

# 25. Hot Key / Hot Partition

Suppose 90% traffic is for:

```text
productId = 123
```

One cache key/partition becomes hot.

Solutions may include:

```text
Replication
Request coalescing
Local cache
Key spreading where appropriate
Read replicas
Traffic distribution
```

Exact solution depends on storage/cache technology.

---

# 26. Autoscaling + Queue Example

Suppose image processing service:

```text
Traffic spike
    ↓
100,000 jobs
    ↓
Kafka / Queue
    ↓
Workers
```

Scale workers based on:

```text
Queue depth
Consumer lag
Processing latency
```

rather than only CPU.

---

# 27. Rate Limiting vs Load Shedding

### Rate Limiting

Controls incoming traffic:

```text
Allow 1000 req/sec
```

### Load Shedding

When overloaded, deliberately reject lower-priority work.

```text
System overloaded
      ↓
Reject non-critical requests
```

Both can be used together.

---

# 28. Graceful Degradation

Suppose recommendation service overloaded hai.

Instead of failing complete product page:

```text
Product ✅
Price ✅
Inventory ✅
Recommendations ❌
```

Return:

```text
Product + Price + Inventory
```

This keeps core functionality available.

---

# 29. Monitoring During Spike

Track:

```text
QPS
CPU
Memory
P95/P99 latency
Error rate
Saturation
DB latency
DB connections
Cache hit ratio
Queue depth
Consumer lag
```

Create alerts for important SLO violations.

---

# 30. Back-of-the-Envelope Check

Suppose:

```text
Traffic = 50,000 QPS
```

One instance can safely handle:

```text
1,000 QPS
```

Rough capacity:

```text
50,000 / 1,000 = 50 instances
```

Then add headroom based on availability and peak strategy.

For example, targeting 70% utilization:

```text
50,000 / (1,000 × 0.7)
≈ 72 instances
```

This is an estimate; real capacity must be load-tested and validated with actual bottlenecks.

---

# 31. Avoid Single Point of Failure

Bad:

```text
Load Balancer
     ↓
One Service Instance
```

Better:

```text
             Load Balancer
           /      |       \
          ↓       ↓        ↓
      Instance  Instance  Instance
```

Deploy across failure domains/availability zones where supported.

---

# 32. Complete Architecture

```text
                         Internet
                            ↓
                       CDN / WAF
                            ↓
                     Rate Limiter
                            ↓
                      Load Balancer
                            ↓
                ┌───────────┼───────────┐
                ↓           ↓           ↓
             Service-1   Service-2   Service-3
                ↓
              Cache
                ↓
           Primary / Replica DB
                ↓
             Queue/Kafka
                ↓
              Workers
```

Protection around the service:

```text
Rate Limit
Timeout
Circuit Breaker
Bulkhead
Backpressure
Load Shedding
```

---

# 33. Production Approach — Step by Step

A strong interview answer should sound like this:

### Step 1
Check whether traffic is legitimate and identify bottleneck.

### Step 2
Apply rate limiting/load shedding if capacity is being exceeded.

### Step 3
Horizontally scale the service if it is the bottleneck.

### Step 4
Use load balancing across healthy instances.

### Step 5
For read-heavy traffic, introduce/optimize cache and possibly read replicas.

### Step 6
For expensive asynchronous work, use queue/Kafka and scale workers.

### Step 7
Protect downstream dependencies with timeout, circuit breaker, bulkhead and backpressure.

### Step 8
Prevent retry storms with bounded retries, backoff and jitter.

### Step 9
Monitor QPS, latency, errors, saturation, DB and queue metrics.

### Step 10
Use capacity estimates and load testing to validate the scaling model.

---

# 34. Interview Scenario

### Interviewer:

> One Payment Service suddenly gets 10x traffic. What will you do?

### Strong Answer:

```text
1. Check whether traffic is legitimate or abusive.
2. Check CPU, memory, DB, connection pool and downstream latency.
3. Apply rate limiting if needed.
4. Horizontally scale Payment Service.
5. Put instances behind a load balancer.
6. Cache safe read-heavy data.
7. Protect DB and downstream services.
8. Use queue for non-critical asynchronous work.
9. Use timeout/circuit breaker/bulkhead.
10. Use backoff + jitter to prevent retry storms.
11. Monitor P95/P99, QPS and error rate.
12. Validate capacity with load testing and estimation.
```

---

# 35. Interview-Ready Answer

> **"If one service suddenly receives very high traffic, I would first identify whether the spike is legitimate and determine the actual bottleneck using QPS, CPU, memory, latency, error rate, database connections, cache hit ratio and downstream metrics. If the service is the bottleneck, I would horizontally scale stateless instances behind a load balancer and use autoscaling based on an appropriate metric. For read-heavy traffic I would use caching and possibly read replicas. Expensive non-critical work can be moved to Kafka or a queue and processed asynchronously. I would also protect dependencies using rate limiting, timeouts, circuit breakers, bulkheads and backpressure, while using bounded retries with exponential backoff and jitter. Finally, I would monitor P95/P99 latency, errors and saturation and validate the capacity through load testing and back-of-the-envelope estimation."**

---

# 36. 30-Second Hinglish Answer

> **"Agar ek service par suddenly high traffic aa jaye, pehle bottleneck identify karunga — CPU, DB, cache, connection pool ya downstream service. Agar application bottleneck hai to stateless instances ko horizontally scale karke load balancer ke peeche rakhunga. Read-heavy traffic ke liye cache use karunga, expensive background work ko Kafka/queue par async karunga. Rate limiting, timeout, circuit breaker, bulkhead aur backpressure se system ko protect karunga. Retry storm avoid karne ke liye exponential backoff aur jitter use karunga, aur P95/P99, QPS, error rate aur DB metrics monitor karunga."**

---

# 37. Memory Trick

```text
IDENTIFY
   ↓
LIMIT
   ↓
SCALE
   ↓
CACHE
   ↓
ASYNC
   ↓
PROTECT
   ↓
MONITOR
```

### One-line memory

**"Find the bottleneck → Control traffic → Scale → Protect dependencies → Monitor."**

---

# 38. Follow-Up Questions

### Q. Is horizontal scaling always enough?

No. DB, cache, network, connection pools or downstream services can become bottlenecks.

### Q. Why stateless services?

Any instance can handle any request, making horizontal scaling and load balancing easier.

### Q. CPU is low but latency is high. Why?

Possible DB bottleneck, downstream latency, lock contention, connection pool exhaustion, network issues or external dependency.

### Q. How do you handle write-heavy traffic?

Batching, queues, partitioning/sharding, write optimization and asynchronous processing where business semantics allow.

### Q. Why use a queue?

To absorb bursts and decouple producers from consumers; it does not provide infinite capacity.

### Q. Why not retry everything?

Retries can multiply traffic and create retry storms; non-idempotent operations can also cause duplicate effects.

### Q. What is graceful degradation?

Keep core functionality available while temporarily dropping non-critical features.

### Q. What metric should trigger autoscaling?

Depends on workload: CPU, memory, QPS, concurrency, queue depth, consumer lag or a custom metric.

### Q. How do you know how many instances are required?

Estimate QPS per instance, include utilization/headroom, then validate with load testing.

### Q. How do you prevent DB from becoming bottleneck?

Cache, read replicas, query/index optimization, connection-pool control, batching and partitioning/sharding where justified.

### Q. How do you handle a DDoS-like spike?

Use upstream protection such as CDN/WAF/provider-level DDoS controls, rate limiting and load shedding rather than relying only on application autoscaling.

---

## Status

✅ **Q23 Solution Completed**

Next: **Q24 — What is Load Balancing?**
