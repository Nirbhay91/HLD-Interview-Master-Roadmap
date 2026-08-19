# Q22 — How Do You Aggregate Responses From Multiple Services?

> **Interview Question:** How do you aggregate responses from multiple services?

## 1. Simple Hinglish Explanation

Kabhi frontend ko ek screen ke liye multiple microservices ka data chahiye hota hai.

Example — Product Details Page:

```text
Product Service
Inventory Service
Review Service
Recommendation Service
```

Frontend agar sabko individually call kare:

```text
Frontend
 ├──→ Product Service
 ├──→ Inventory Service
 ├──→ Review Service
 └──→ Recommendation Service
```

to client complexity, network round trips aur failure handling increase ho sakti hai.

Isliye ek **API Composition / Aggregation layer / BFF** multiple services ko call karke ek combined response frontend ko de sakta hai.

```text
Frontend
    ↓
BFF / Aggregator
  ┌─┼────┬─────────┐
  ↓ ↓    ↓         ↓
Product Inventory Reviews Recommendations
  └─┴────┴─────────┘
          ↓
   Aggregated Response
          ↓
       Frontend
```

---

# 2. Main Goal

Response aggregation ka goal hai:

```text
Multiple Service Responses
          ↓
   Combine / Compose
          ↓
   Client-friendly Response
```

It can reduce client-side orchestration and simplify frontend APIs.

---

# 3. Real-World Example

Suppose Amazon-like product page hai.

Frontend needs:

```text
Product details
Price
Inventory
Reviews
Recommendations
```

Data different services mein hai:

```text
Product Service
Inventory Service
Review Service
Recommendation Service
```

Aggregator:

```text
                  BFF
             /    |    |    \
            ↓     ↓    ↓     ↓
       Product Inventory Review Recommendation
            \     |    |     /
             \    |    |    /
               Combined
                  ↓
              Frontend
```

---

# 4. Sequential Aggregation

Simple approach:

```text
Aggregator
   ↓
Product Service
   ↓
Inventory Service
   ↓
Review Service
   ↓
Response
```

Problem:

Total latency roughly becomes:

```text
T ≈ T1 + T2 + T3
```

Agar:

```text
Product = 100 ms
Inventory = 150 ms
Reviews = 200 ms
```

then sequential calls can approach:

```text
450 ms + overhead
```

when all are independent.

---

# 5. Parallel Aggregation

Agar calls independent hain to parallelize kar sakte hain:

```text
                 Aggregator
              /      |      \
             ↓       ↓       ↓
         Product Inventory Reviews
             \       |       /
              \      |      /
                Combined
```

Latency approximately:

```text
T ≈ max(T1, T2, T3) + overhead
```

Example:

```text
Product = 100 ms
Inventory = 150 ms
Reviews = 200 ms
```

Parallel case roughly:

```text
200 ms + overhead
```

This is one of the most important interview optimization points.

---

# 6. Dependency Graph

Parallelization tabhi safe hai jab dependencies independent hon.

Example:

```text
Product ─────┐
             ├──→ Aggregator
Inventory ───┤
Reviews ──────┘
```

Parallel possible.

But:

```text
Product
  ↓
Inventory
  ↓
Recommendation
```

Here dependency hai, so completely parallel execution possible nahi hai.

Interview mein pehle dependency identify karo.

---

# 7. Aggregator API Example

Frontend:

```http
GET /api/product-page/123
```

Aggregator internally:

```text
GET /products/123
GET /inventory/123
GET /reviews/123
GET /recommendations/123
```

Combined response:

```json
{
  "product": {
    "id": "123",
    "name": "Laptop",
    "price": 75000
  },
  "inventory": {
    "available": true,
    "quantity": 12
  },
  "reviews": {
    "rating": 4.5,
    "count": 1200
  },
  "recommendations": [
    "101",
    "102",
    "103"
  ]
}
```

Frontend ko ek response mil gaya.

---

# 8. Where Should Aggregation Live?

Common choices:

### Option 1 — API Gateway

```text
Frontend
   ↓
Gateway
   ↓
Multiple Services
```

Simple aggregation ke liye possible hai.

### Option 2 — BFF

```text
Frontend
   ↓
BFF
   ↓
Multiple Services
```

Client-specific response shaping ke liye often better.

### Option 3 — Dedicated Aggregation Service

```text
Frontend
   ↓
Product-Page Aggregator
   ↓
Multiple Services
```

Complex composition ke liye useful.

**Interview point:** Business/domain logic ko generic gateway mein dump karne ke bajay BFF/composition service use karna cleaner ho sakta hai.

---

# 9. API Gateway vs BFF vs Aggregator

| Component | Primary Responsibility |
|---|---|
| API Gateway | Edge routing/policies |
| BFF | Frontend-specific API composition |
| Aggregation Service | Complex multi-service composition |
| Domain Service | Business/domain logic |

Real architecture mein responsibilities overlap kar sakti hain, but separation of concerns maintain karna important hai.

---

# 10. Failure Handling

Most important problem:

> What if one downstream service fails?

Example:

```text
Product ✅
Inventory ✅
Reviews ❌
Recommendations ✅
```

Aggregator ko decide karna hai:

```text
Whole request fail?
OR
Partial response?
```

Decision requirement par depend karta hai.

---

# 11. Mandatory vs Optional Data

Suppose:

```text
Product = mandatory
Inventory = mandatory
Reviews = optional
Recommendations = optional
```

Then:

```text
Product fails → Request fails
Inventory fails → Request may fail
Reviews fails → Return without reviews
Recommendations fails → Return without recommendations
```

This is **graceful degradation**.

---

# 12. Partial Response

Example:

```json
{
  "product": {...},
  "inventory": {...},
  "reviews": null,
  "recommendations": [...],
  "warnings": [
    "Reviews temporarily unavailable"
  ]
}
```

User ko core page still mil sakta hai.

---

# 13. Timeout

Aggregator ko har downstream call ke liye timeout define karna chahiye.

Bad:

```text
No timeout
   ↓
Slow Service
   ↓
Aggregator waits
   ↓
Frontend waits
```

Better:

```text
Product timeout = 300ms
Reviews timeout = 500ms
```

Exact timeout SLA/latency budget ke according decide hota hai.

---

# 14. Retry

Transient failures ke liye retry possible hai.

```text
Call
 ↓
Timeout/Transient error
 ↓
Retry
```

But blindly retry dangerous hai.

Especially:

```text
POST /payment
```

jaise non-idempotent operations duplicate effect create kar sakte hain.

Retries should use bounded attempts, backoff and idempotency where appropriate.

---

# 15. Circuit Breaker

Agar downstream service continuously fail ho rahi hai:

```text
Aggregator
    ↓
Reviews Service ❌
```

Circuit breaker repeatedly failing calls ko temporarily stop kar sakta hai.

```text
CLOSED
  ↓ failures
OPEN
  ↓ wait
HALF-OPEN
  ↓ test
CLOSED / OPEN
```

Isse cascading failures reduce karne mein help mil sakti hai.

---

# 16. Bulkhead

Suppose Reviews Service slow hai.

Agar unlimited resources usko de diye:

```text
Reviews slow
   ↓
Threads exhausted
   ↓
Aggregator impacted
   ↓
Other requests impacted
```

Bulkhead/resource isolation se ek dependency ka failure entire aggregator ko exhaust karne se prevent kiya ja sakta hai.

---

# 17. Fan-Out Problem

Aggregator ek request ko multiple downstream requests mein fan-out karta hai.

```text
1 incoming request
       ↓
  5 downstream calls
```

At high traffic:

```text
10,000 incoming requests
       ↓
50,000 downstream calls
```

This can multiply load.

Therefore aggregation architecture mein fan-out carefully control karna important hai.

---

# 18. Caching

Frequently requested data cache kiya ja sakta hai.

```text
Frontend
   ↓
Aggregator
   ↓
Cache
   ↓ cache miss
Services
```

Example:

```text
Product details → cacheable
Recommendations → often cacheable
Inventory → freshness requirements high
```

Cache strategy data consistency requirements par depend karegi.

---

# 19. Request Deduplication

Same data multiple downstream calls se avoid kiya ja sakta hai.

Example:

```text
Page request
   ↓
Product data required by 3 components
```

Aggregator internally one Product Service call karke response reuse kar sakta hai.

---

# 20. Parallel Calls With Time Budget

Suppose overall API SLA:

```text
500 ms
```

Aggregator budget distribute kar sakta hai:

```text
Product      200ms
Inventory    150ms
Reviews      200ms
```

But total budget mein:

```text
Network overhead
Serialization
Aggregation
```

bhi include karna padega.

Interview mein important line:

> **"I would design downstream timeouts from the end-to-end latency budget, not choose them independently."**

---

# 21. Observability

Aggregator mein distributed tracing important hai.

```text
Trace ID: abc123

Aggregator
 ├─ Product → 80ms
 ├─ Inventory → 120ms
 ├─ Reviews → 450ms
 └─ Recommendations → 90ms
```

Ab easily identify ho jayega ki latency Reviews service ki wajah se hai.

Metrics:

```text
Success rate
Error rate
Latency P95/P99
Downstream timeout rate
Partial response rate
Fan-out count
```

---

# 22. Correlation / Trace Propagation

```text
Frontend
  ↓ trace context
Aggregator
  ↓ trace context
Product Service
  ↓
Inventory Service
```

Same distributed trace se complete request journey observe kar sakte hain.

---

# 23. Data Consistency

Different services independently update ho sakte hain.

Aggregator response may temporarily represent:

```text
Product updated
Inventory slightly stale
Recommendations older
```

Isliye aggregation layer generally **read composition** karta hai; it should not pretend that combining responses creates a distributed transaction.

If data must be atomically changed across services, separate distributed-transaction/Saga design is needed.

---

# 24. Security

Aggregator ko downstream calls securely perform karni chahiye.

```text
Frontend
  ↓ User Token
Aggregator
  ↓ Service Identity / Delegated Token
Services
```

Blindly forwarding user bearer tokens everywhere is not always appropriate.

Use explicit service-to-service authentication and authorization.

---

# 25. N+1 Problem

Bad aggregation design:

```text
Get 100 products
      ↓
100 inventory calls
```

This creates:

```text
1 + N calls
```

Better approaches:

```text
Batch API
Bulk endpoint
Batch query
Dedicated read model
Cache
```

Example:

```http
POST /inventory/batch

{
  "productIds": [1,2,3,4]
}
```

---

# 26. Pagination in Aggregated APIs

Suppose:

```text
Products = 20
Reviews = 1000
```

Do not aggregate huge unbounded responses.

Use:

```text
Pagination
Cursor
Limits
```

Example:

```http
GET /product-page/123/reviews?cursor=abc&limit=20
```

---

# 27. Response Contract

Aggregator should expose a stable client-friendly contract.

Internal services can evolve:

```text
Product Service v1 → v2
```

while aggregator maintains:

```text
GET /product-page/{id}
```

This reduces frontend coupling to internal service schemas.

---

# 28. Sequential vs Parallel — Interview Comparison

| Approach | Latency | Complexity | Use |
|---|---|---|---|
| Sequential | Higher | Low | Dependent calls |
| Parallel | Lower | Medium | Independent calls |
| Batch | Efficient | Medium | Many similar items |
| Cache | Very low on hit | Medium | Frequently reused data |

Best answer:

> **"I will parallelize independent downstream calls, but preserve ordering where dependencies exist."**

---

# 29. Complete Architecture

```text
                         Frontend
                            |
                            | GET /product-page/123
                            ↓
                    ┌────────────────┐
                    │ API Gateway /  │
                    │      BFF       │
                    └───────┬────────┘
                            |
                       Aggregator
                     /      |       \
                    /       |        \
                   ↓        ↓         ↓
              Product   Inventory   Reviews
               Service    Service    Service
                   \        |         /
                    \       |        /
                     \      |       /
                      └── Combined ─┘
                            |
                            ↓
                    Client Response
```

With reliability:

```text
Aggregator
  ├→ Timeout
  ├→ Retry (safe operations)
  ├→ Circuit Breaker
  ├→ Bulkhead
  ├→ Cache
  └→ Partial Response
```

---

# 30. Interview Scenario

### Interviewer:

> Product page needs Product, Inventory, Reviews and Recommendations. How will you design it?

### Strong Answer:

```text
1. Create a BFF/aggregation endpoint.
2. Identify dependencies between downstream calls.
3. Call independent services in parallel.
4. Set timeout based on overall latency budget.
5. Use circuit breaker for unstable dependencies.
6. Retry only safe/transient operations.
7. Decide mandatory vs optional dependencies.
8. Return partial response for optional failures.
9. Use caching where freshness allows.
10. Add distributed tracing and downstream latency metrics.
11. Avoid N+1 calls using batch APIs.
```

---

# 31. Interview-Ready Answer

> **"When a client needs data from multiple microservices, I would usually use an API composition layer such as a BFF or dedicated aggregator. The aggregator identifies independent downstream calls and executes them in parallel to reduce latency, while respecting dependencies. I would define strict timeouts, use retries only for safe transient failures, and apply circuit breakers and bulkheads to prevent cascading failures. For optional data, I can return a partial response instead of failing the whole request. I would also use caching and batch APIs where appropriate and add distributed tracing to understand downstream latency. The aggregator should provide a stable client-facing contract without moving core business logic into the aggregation layer."**

---

# 32. 30-Second Hinglish Answer

> **"Agar frontend ko multiple microservices ka data chahiye, to main BFF ya aggregation service use karunga. Independent services ko parallel call karke response combine karunga, jisse latency reduce hogi. Har downstream call ke liye timeout, safe retries aur circuit breaker use karunga. Mandatory service fail ho to request fail ho sakti hai, lekin optional service fail ho to partial response de sakte hain. High traffic mein cache aur batch APIs use karunga aur distributed tracing se downstream latency monitor karunga."**

---

# 33. Memory Trick

```text
DECOMPOSE
   ↓
PARALLELIZE
   ↓
TIMEOUT
   ↓
RETRY SAFELY
   ↓
CIRCUIT BREAKER
   ↓
PARTIAL RESPONSE
   ↓
CACHE / BATCH
   ↓
TRACE
   ↓
COMBINE
```

### One-line memory

**"Split → Parallelize → Protect → Degrade → Combine."**

---

# 34. Follow-Up Questions

### Q. Why parallel calls?

Independent calls ki latency roughly maximum downstream latency ke around ho sakti hai instead of sum of all latencies.

### Q. What if one service fails?

Mandatory/optional classification ke basis par full failure ya partial response decide karenge.

### Q. Why timeout?

Slow dependency ko indefinitely wait karke thread/resource exhaustion prevent karne ke liye.

### Q. Why circuit breaker?

Repeatedly failing dependency ko call karna temporarily stop karke cascading failure reduce karne ke liye.

### Q. What is N+1 problem?

Ek parent response ke liye N individual downstream calls create hona. Batch APIs/read models/cache se reduce kar sakte hain.

### Q. Should aggregation logic be in API Gateway?

Simple aggregation possible hai, but complex client-specific composition often BFF/dedicated aggregator mein better separation deta hai.

### Q. Does aggregation guarantee consistency?

No. Multiple service responses combine karna distributed transaction nahi hai. Data eventual/stale ho sakta hai depending on service design.

### Q. How do you avoid aggregator becoming a bottleneck?

Stateless horizontal scaling, caching, bounded concurrency, timeouts, load balancing and careful fan-out control.

### Q. What if downstream services have dependencies?

Dependency graph identify karke dependent calls sequential/ordered rakhenge; independent calls parallelize karenge.

---

## Status

✅ **Q22 Solution Completed**

Next: **Q23 — Suppose one service suddenly receives very high traffic. How will you handle it?**
