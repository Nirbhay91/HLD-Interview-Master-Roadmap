# URL Shortener — HLD Interview Solution

## 1. Problem Statement

Design a URL shortening service similar to Bitly.

Example:

```text
https://example.com/very/long/url
              ↓
https://short.ly/aB91xK
```

When a user opens the short URL, the service should redirect the user to the original URL.

---

## 2. Functional Requirements

### Must Have

- Create a short URL for a long URL.
- Redirect a short URL to the original URL.
- Short codes must be unique.
- Support very high read traffic.
- URL mapping should persist.

### Nice to Have

- Custom aliases.
- Expiration time.
- Click analytics.
- User-owned URLs.
- Abuse/spam protection.

---

## 3. Non-Functional Requirements

- High availability.
- Very low redirect latency.
- Horizontally scalable.
- Durable URL mappings.
- Read-heavy workload.
- Eventual consistency is acceptable for analytics.
- Redirect service should continue working even if analytics processing is temporarily unavailable.

---

## 4. Scale Estimation

Assume:

- 100M new URLs/month.
- 10x more redirects than URL creation.
- 1B redirects/month.

### Write QPS

```text
100M / (30 × 24 × 60 × 60)
≈ 39 writes/sec
```

Peak write traffic can be estimated using a multiplier such as 5x–10x.

### Read QPS

```text
1B / (30 × 24 × 60 × 60)
≈ 386 redirects/sec
```

With a 10x peak multiplier:

```text
Peak ≈ 3,860 redirects/sec
```

The exact numbers are assumptions. In an interview, state assumptions clearly and adjust them if the interviewer provides different numbers.

---

## 5. API Design

### Create Short URL

```http
POST /api/v1/urls
Content-Type: application/json

{
  "longUrl": "https://example.com/a/very/long/path",
  "expiresAt": "2027-01-01T00:00:00Z"
}
```

Response:

```json
{
  "shortUrl": "https://short.ly/aB91xK"
}
```

### Redirect

```http
GET /{shortCode}
```

Response:

```text
HTTP 301/302 → original URL
```

### Optional Analytics

```http
GET /api/v1/urls/{shortCode}/analytics
```

---

## 6. High-Level Architecture

```text
                    Client
                       |
                       v
                Load Balancer
                       |
                       v
                 URL Service
                  /       \
                 /         \
                v           v
             Redis        URL DB
                |
                |
                +------ Cache Miss ------> URL DB
                                          |
                                          v
                                    Original URL

Create URL:
Client → URL Service → ID/Code Generator → URL DB → Redis

Redirect:
Client → Load Balancer → URL Service → Redis → 301/302
                                  |
                              Cache Miss
                                  |
                                  v
                                URL DB

Analytics:
Redirect Service → Kafka → Analytics Consumer → Analytics Store
```

---

## 7. Short Code Generation

The main design question is: **How do we generate a unique short code?**

### Option A — Auto-increment ID + Base62

Generate a unique numeric ID and encode it using Base62.

Base62 characters:

```text
0-9
A-Z
 a-z
```

Example conceptually:

```text
Numeric ID → Base62 → aB91xK
```

Advantages:

- Simple.
- Compact.
- Deterministic.
- Easy to generate unique codes when the underlying ID is unique.

Trade-off:

- Sequential IDs can reveal approximate creation volume unless the IDs are obfuscated.
- A single database sequence can become a coordination point at very large scale.

### Option B — Random code

Generate a random Base62 string and check for collision.

Advantages:

- Codes are not sequential.
- Easy to distribute.

Trade-off:

- Collision handling is required.
- As the namespace becomes more occupied, collision probability increases.

### Interview Choice

For a normal URL-shortener interview, **distributed ID generation + Base62** is a strong starting point. If scale becomes extremely large, discuss range allocation, Snowflake-style IDs, or distributed ID generators.

---

## 8. Database Design

A relational or key-value database can work.

Example logical schema:

```text
URL_MAPPING
-------------------------
short_code       PK
long_url
user_id
created_at
expires_at
status
```

### Important Index

```text
PRIMARY KEY(short_code)
```

The redirect path is always:

```text
shortCode → longURL
```

Therefore the lookup must be extremely efficient.

### Database Choice

For moderate scale, a relational database can be sufficient.

At very large scale, a distributed key-value store can be attractive because the primary access pattern is a simple key lookup.

The choice should be justified by scale, consistency, operational requirements, and query patterns—not by saying that NoSQL is always better for HLD.

---

## 9. Caching

URL shortening is typically **read-heavy**, so Redis is useful.

```text
shortCode → longURL
```

### Cache-Aside Flow

```text
Request
  ↓
Redis
  |
  +-- Hit → return URL
  |
  +-- Miss
        ↓
      DB
        ↓
     Redis
        ↓
     return URL
```

### Why Cache?

- Lower redirect latency.
- Reduce database reads.
- Absorb traffic spikes.
- Improve scalability.

### Cache Eviction

Use TTL based on URL lifetime where appropriate.

For frequently accessed URLs, caching provides a large benefit.

---

## 10. Scaling Strategy

### Application Layer

Keep URL service **stateless**.

```text
                 Load Balancer
                 /     |     \
                v      v      v
             Service Service Service
```

Instances can therefore be added horizontally.

### Database

As traffic grows:

1. Read replicas for read scaling.
2. Partition/shard by short-code hash if required.
3. Distributed key-value storage for very large workloads.

### Cache

Use a distributed Redis deployment and partition cache keys when necessary.

---

## 11. Reliability

### Database Failure

- Use replication.
- Automatic failover where supported.
- Durable storage.

### Cache Failure

The system should fall back to the database.

```text
Redis unavailable
      ↓
DB lookup
      ↓
Redirect
```

Cache should improve performance, not become the only source of truth.

### Analytics Failure

Do not block redirects on analytics.

```text
Redirect
   |
   +---- User redirect succeeds
   |
   +---- Event → Kafka → Analytics
```

If analytics consumers fail, events can be retried/reprocessed according to the chosen delivery guarantees.

---

## 12. Consistency

### URL Creation

The mapping must be durable and the short code must be unique.

### Redirect

A newly created URL may need stronger read-after-write behavior depending on product requirements.

### Analytics

Eventual consistency is normally acceptable.

Example:

```text
User clicks URL
     ↓
Redirect immediately
     ↓
Analytics event processed later
```

We prioritize redirect availability and latency over immediate analytics consistency.

---

## 13. HTTP Redirect: 301 vs 302

### 301 — Permanent Redirect

Useful when the redirect is effectively permanent and clients/CDNs can cache the redirect.

### 302 — Temporary Redirect

Useful when redirect behavior may change or when more control over caching is desired.

In an interview, explicitly ask about product semantics before choosing one.

---

## 14. Idempotency

If the create API is retried, duplicate requests can create multiple short URLs unless the API is designed for idempotency.

Possible approach:

```text
Idempotency-Key → stored result
```

On retry:

```text
same key → return previous result
```

Whether the same long URL should always return the same short URL is a separate product decision.

---

## 15. Security & Abuse Protection

Important concerns:

- Validate URLs.
- Block malicious or disallowed destinations.
- Rate-limit URL creation.
- Rate-limit suspicious redirect traffic where appropriate.
- Prevent enumeration if private URLs are supported.
- Protect analytics endpoints.
- Monitor abuse patterns.

---

## 16. Bottlenecks & Trade-offs

### Bottleneck: Database Reads

Solution:

```text
Redis + read replicas + partitioning/sharding
```

### Bottleneck: Short Code Generation

Solution:

- Distributed ID generator.
- Allocate ID ranges to service instances.
- Use Snowflake-style IDs where appropriate.

### Bottleneck: Hot URLs

A viral URL can receive huge traffic.

Solution:

- Cache aggressively.
- Replicate cache.
- Use CDN/edge caching where redirect semantics permit it.

### Trade-off

> We use caching and eventual consistency for analytics because redirect latency and availability are more important than immediately consistent click statistics.

---

## 17. Deep-Dive Follow-up Questions

### Q1. Why Redis?

Because redirect traffic is read-heavy and Redis provides fast key-value lookups, reducing database load and latency.

### Q2. What if Redis goes down?

Fall back to the database. Redis is a performance layer, not the source of truth.

### Q3. How do you guarantee unique short codes?

Use a globally unique ID generator and encode the ID using Base62, or use random codes with collision detection.

### Q4. How do you handle billions of URLs?

Horizontally scale the stateless service, distribute ID generation, partition/shard the URL store, and use distributed caching.

### Q5. How do you handle a viral URL?

Cache the mapping heavily and distribute the cache workload. For extremely high read volume, edge/CDN caching can also be considered.

### Q6. What happens if the same create request is sent twice?

Use an idempotency key if the API contract requires retries to return the same result.

---

## 18. Final 2-Minute Interview Answer

> I would design the URL shortener as a stateless service behind a load balancer. The create API accepts a long URL and generates a unique short code using a distributed ID generator followed by Base62 encoding. The mapping between short code and long URL is persisted in a durable database.
>
> For redirects, the service first checks Redis using the short code. On a cache hit, it immediately returns a 301 or 302 redirect. On a cache miss, it reads from the database, populates Redis, and then redirects the client. Since redirects are much more frequent than URL creation, this read-heavy architecture significantly reduces database load.
>
> The application layer can scale horizontally because the service is stateless. If the database becomes a bottleneck, I would introduce read replicas and eventually partition or shard the data. For analytics, I would publish click events asynchronously to Kafka so analytics failures do not affect the redirect path.
>
> I would also add rate limiting, URL validation, database replication, cache fallback, idempotency for retried create requests, and monitoring. The main trade-off is that analytics can be eventually consistent because redirect latency and availability are more important than immediate analytics accuracy.

---

## 19. Interview Diagram to Remember

```text
                         +----------------+
                         |     Client     |
                         +-------+--------+
                                 |
                                 v
                         +---------------+
                         | Load Balancer |
                         +-------+-------+
                                 |
                    +------------+------------+
                    |                         |
                    v                         v
              +-----------+             +-----------+
              | URL Svc 1 |             | URL Svc N |
              +-----+-----+             +-----+-----+
                    |                         |
                    +------------+------------+
                                 |
                         +-------v-------+
                         |     Redis     |
                         +-------+-------+
                                 |
                           Cache Miss
                                 |
                         +-------v-------+
                         |    URL DB     |
                         +-------+-------+
                                 |
                                 v
                         Original Long URL

                         Analytics Path

                         URL Service
                              |
                              v
                            Kafka
                              |
                              v
                     Analytics Consumers
                              |
                              v
                       Analytics Store
```

---

## 20. What to Derive Instead of Memorizing

Remember these decisions, not the diagram:

```text
Read-heavy
   ↓
Redis

Stateless service
   ↓
Horizontal scaling

Unique short code
   ↓
Distributed ID + Base62

Simple key lookup
   ↓
Key-value access pattern

Analytics not on critical path
   ↓
Kafka + async processing

Cache failure must not break system
   ↓
DB fallback
```

This is the first complete system-design cycle. The next problem should be solved only after this one is understood and interview-ready.
