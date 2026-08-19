# Q26 — How Would Caching Help in Reducing Database Load?

## 1. Simple Hinglish Explanation

Caching ka matlab frequently accessed data ko fast storage (jaise Redis) mein temporarily rakhna hai, taaki har request ke liye database ko hit na karna pade.

Without cache:

```text
Client → Service → Database → Response
```

With cache:

```text
Client → Service → Cache
                    ├─ HIT  → Response
                    └─ MISS → Database → Cache → Response
```

Example: 100,000 cacheable requests aur 90% hit ratio ho to approximately 10,000 requests database tak pahunchengi. Isse DB read load aur latency significantly reduce ho sakti hai.

---

## 2. Why Caching?

Main benefits:

- Database read load reduce
- Lower latency
- Higher throughput
- Better scalability
- Repeated computation/fetching avoid

Important: caching primarily repeated reads ko reduce karta hai. Writes automatically reduce nahi hote unless write-through/write-back jaisi strategy deliberately use ki jaye.

---

## 3. Cache Hit vs Cache Miss

### Cache Hit

```text
Request → Cache → HIT → Response
```

Database call avoid ho jati hai.

### Cache Miss

```text
Request → Cache → MISS → DB → Populate Cache → Response
```

---

## 4. Cache Hit Ratio

```text
Hit Ratio = Cache Hits / Total Cache Requests
```

Example:

```text
Total = 100,000
Hits  = 90,000

Hit Ratio = 90%
```

Higher hit ratio generally means fewer cacheable reads reach the DB, but actual benefit workload-dependent hai.

---

## 5. What Should Be Cached?

Good candidates:

- Frequently read data
- Infrequently changing data
- Expensive-to-fetch data
- Data that can tolerate defined staleness

Examples:

```text
Product catalog
Product details
Configuration
Popular content
Reference data
User preferences
```

Strictly real-time or highly sensitive data ko blindly cache nahi karna chahiye.

---

## 6. Cache-Aside Pattern

Most common application caching approach:

```text
Request
  ↓
Check Cache
  ↓
 HIT ─────────→ Return
  ↓ MISS
Database
  ↓
Put in Cache
  ↓
Return
```

Application cache ko explicitly manage karti hai.

Example:

```text
product:123
```

Cache miss par DB se product fetch karke Redis mein store karenge.

---

## 7. Read-Through

```text
Application → Cache → Data Store
```

Cache layer miss hone par backing store se data fetch/populate kar sakti hai, depending on technology.

**Cache-aside:** application manages population.

**Read-through:** cache layer manages read-through population.

---

## 8. Write-Through

```text
Application
    ↓
  Cache
    ↓
Database
```

Write cache ke through synchronously backing store tak propagate hota hai.

Benefit: cache aur DB update path coordinated hota hai.

Trade-off: write latency/coordination complexity increase ho sakti hai.

---

## 9. Write-Back / Write-Behind

```text
Application → Cache → Async persistence → Database
```

Benefit: lower write latency.

Risk: cache failure before persistence can cause data loss, so durability requirements carefully evaluate karni hoti hain.

---

## 10. Cache Invalidation

Suppose:

```text
DB    = ₹120
Cache = ₹100
```

DB update hone ke baad old cache value serve ho sakti hai.

Common strategies:

```text
TTL
Explicit invalidation
Event-driven invalidation
Versioned keys
Write-through
```

TTL stale data ki maximum intended lifetime define kar sakta hai, but TTL alone every consistency problem solve nahi karta.

---

## 11. TTL — Time To Live

Example:

```text
product:123
TTL = 10 minutes
```

Expiry ke baad entry remove/expire ho sakti hai.

TTL selection business freshness requirement par depend karta hai.

```text
Static/reference data → longer TTL
Fast-changing inventory → shorter TTL
```

---

## 12. Eviction Policies

Cache memory limited hoti hai. Common policies:

```text
LRU — Least Recently Used
LFU — Least Frequently Used
FIFO — First In First Out
TTL expiration
```

Exact behavior cache technology/configuration par depend karta hai.

---

## 13. Redis

Redis distributed caching ke liye commonly use hota hai:

```text
Application
    ↓
   Redis
    ↓ miss
 Database
```

Use cases:

- Caching
- Counters
- Sessions
- Rate limiting
- Short-lived shared state

Redis ko automatically system of record nahi maana chahiye unless architecture explicitly usse authoritative/durable store banata hai.

---

## 14. Local Cache vs Distributed Cache

### Local

```text
App-1 → Memory
App-2 → Memory
App-3 → Memory
```

Very fast but every instance has separate data.

### Distributed

```text
App-1 ─┐
App-2 ─┼→ Redis
App-3 ─┘
```

Shared cache state milta hai, but network/operational overhead add hota hai.

---

## 15. Cache Key Design

Good:

```text
product:{productId}
product:123
```

Keys should be:

```text
Unique
Predictable
Compact
Versionable where useful
```

Unbounded user-controlled key creation se memory abuse ho sakta hai.

---

## 16. Cache Stampede / Thundering Herd

Popular key expire hoti hai:

```text
10,000 requests
      ↓
Same Cache MISS
      ↓
10,000 DB requests
      ↓
DB overload ❌
```

Solutions:

```text
Request coalescing / single-flight
Jittered TTL
Early refresh
Pre-warming
Stale-while-revalidate
```

### Request Coalescing

```text
10,000 requests
      ↓
First → DB
Others → wait/reuse result
      ↓
Cache populated
```

---

## 17. Cache Penetration

Non-existent data repeatedly request ho:

```text
product:999999
Cache MISS → DB NOT FOUND
```

Repeated requests DB ko hit karte rahenge.

Solutions:

```text
Negative caching
Bloom filter
Input validation
Rate limiting
```

### Negative Caching

Short TTL ke saath NOT_FOUND result cache kar sakte hain.

---

## 18. Cache Avalanche

Bahut saare keys same time expire/fail ho jayein:

```text
Many cache entries expire
        ↓
Huge DB traffic spike
        ↓
DB overload
```

Mitigation:

```text
TTL jitter
Staggered expiry
Cache redundancy
Warm-up
Request limiting
```

### Memory Trick

```text
Stampede  → Same hot key
Penetration → Missing data
Avalanche → Many keys
```

---

## 19. Hot Keys

One key extremely popular ho sakti hai:

```text
product:123 → 1 million requests
```

Possible solutions:

```text
Local cache
Replication
Request coalescing
CDN for suitable content
Key spreading where appropriate
```

Exact solution cache architecture par depend karega.

---

## 20. Cache Consistency

Cache stale data return kar sakti hai. Isliye business requirement define karegi ki kitni staleness acceptable hai.

```text
Strict consistency requirement
        ↓
Very careful cache strategy

Stale-tolerant data
        ↓
TTL / eventual consistency may be acceptable
```

Example:

```text
Product description → seconds of staleness may be okay
Bank balance → much stricter consistency
```

Interview line:

> **"I choose the cache strategy based on the freshness and consistency requirement of the data."**

---

## 21. Cache Update Flow

Typical cache-aside write flow:

```text
Write Request
     ↓
Update DB
     ↓
Invalidate Cache
```

Next read:

```text
Cache MISS
     ↓
DB
     ↓
Populate Cache
```

Concurrency and ordering edge cases need to be considered in high-consistency systems.

---

## 22. Cache Warming

Known traffic spike se pehle popular data cache mein load kar sakte hain:

```text
Expected Event
     ↓
Warm Popular Keys
     ↓
Traffic Starts
```

Useful for:

```text
Popular products
Scheduled events
Configuration
Reference data
```

Over-warming itself unnecessary DB/cache load create kar sakta hai.

---

## 23. Cache Failure

Redis down:

```text
Application
    ↓
Redis ❌
    ↓
Database
```

Agar every request DB par fallback kare:

```text
Cache outage
    ↓
Mass misses
    ↓
DB overload
    ↓
Cascading failure
```

Protection:

```text
Bounded cache timeout
Circuit breaker
Rate limiting
Local cache where safe
Graceful degradation
DB capacity protection
Highly available cache setup
```

---

## 24. Multi-Level Cache

```text
L1 → Local application cache
L2 → Redis
L3 → Database
```

Flow:

```text
Request
  ↓
L1
  ↓ miss
Redis
  ↓ miss
DB
```

Benefit: hot data ke liye very low latency.

Trade-off: more complexity and consistency challenges.

---

## 25. Cache vs CDN

### Redis / Application Cache

```text
Product data
Session
Counters
API results
```

### CDN

```text
Images
CSS
JS
Video
Static content
Cacheable HTTP responses
```

Both coexist kar sakte hain.

---

## 26. API Response Caching

Example:

```http
GET /products/123
```

```text
Request
  ↓
Cache
  ↓ HIT
JSON Response
```

Consider:

```text
Cache key
TTL
Invalidation
Authorization
Privacy
User-specific data
```

Shared cache mein personalized response safely isolate karna mandatory hai.

---

## 27. Back-of-the-Envelope Example

Suppose:

```text
Traffic = 100,000 QPS
Cache hit ratio = 90%
```

Approximate cacheable DB reads:

```text
100,000 × (1 - 0.90)
= 10,000 QPS
```

So cacheable read traffic roughly:

```text
100,000 QPS → 10,000 QPS DB reads
```

This is an estimate; writes, misses, evictions, refreshes and uncached queries still consume DB capacity.

---

## 28. Complete Architecture

```text
                         Clients
                            ↓
                       Load Balancer
                            ↓
                       API Gateway
                            ↓
                     Application Pods
                      /          \
                     ↓            ↓
               Local Cache      Redis
                     \            /
                      \          /
                         ↓
                       Database
```

Read flow:

```text
Request
  ↓
L1 Cache
  ↓ miss
Redis
  ↓ miss
DB
  ↓
Redis
  ↓
L1
  ↓
Response
```

---

## 29. Interview Scenario

### Interviewer:

> Your product API receives 50,000 QPS and the database is overloaded. How would caching help?

### Strong Answer

```text
1. Check whether traffic is read-heavy and cacheable.
2. Introduce cache-aside with Redis.
3. Use stable keys such as product:{id}.
4. Cache hit → return without DB.
5. Cache miss → DB → populate cache.
6. Set TTL according to freshness requirement.
7. Invalidate/update cache after writes where needed.
8. Protect against stampede with request coalescing/jitter.
9. Use negative caching for repeated NOT_FOUND cases where appropriate.
10. Monitor hit ratio, latency, misses and DB QPS.
11. Design cache failure handling so DB is not overwhelmed by a flood of misses.
```

---

## 30. Interview-Ready Answer

> **"Caching reduces database load by serving frequently requested data from a faster cache instead of querying the database for every request. I would typically use a cache-aside strategy with Redis: first check the cache, return on a hit, and on a miss read from the database and populate the cache. I would choose TTL and invalidation based on the data's freshness requirements. I would monitor cache hit ratio, latency and miss rate, and protect against cache stampede, penetration and avalanche. I would also design for cache failure because a sudden flood of misses can overload the database. Caching can significantly reduce DB read QPS, but the database still needs capacity for misses, writes and failure scenarios."**

---

## 31. 30-Second Hinglish Answer

> **"Caching frequently accessed data ko Redis jaise fast storage mein rakhta hai, jisse har request database ko hit nahi karti. Main generally cache-aside use karunga: pehle cache check, hit hua to direct response; miss hua to DB se data lekar cache mein store karunga. TTL aur invalidation freshness requirement ke according rakhunga. Stampede, penetration aur cache failure se DB ko protect karunga. Isse DB read load aur application latency significantly reduce ho sakti hai."**

---

## 32. Memory Trick

```text
CHECK CACHE
     ↓
HIT → RETURN
     ↓ MISS
READ DB
     ↓
POPULATE CACHE
     ↓
RETURN
```

### One-line memory

**"Cache Hit saves DB; Cache Miss reads DB and warms Cache."**

Production checklist:

**Cache + TTL + Invalidation + Stampede Protection + DB Protection**

---

## 33. Common Interview Mistakes

### Mistake 1

> Redis database ko replace kar deta hai.

Usually wrong. Cache and system-of-record responsibilities different hoti hain.

### Mistake 2

> Cache always consistent hota hai.

Wrong. Cache stale data serve kar sakta hai.

### Mistake 3

> TTL solves every invalidation problem.

No. TTL expiry boundary deta hai; expiry se pehle stale data possible hai.

### Mistake 4

> Cache hit ratio always 100% hoga.

No. New keys, misses, eviction and expiry exist.

### Mistake 5

> Redis down → directly DB fallback.

Without protection this can overload DB.

---

## 34. Follow-Up Questions

### Q. Cache-aside vs write-through?

Cache-aside mein application cache population manage karti hai; write-through mein cache/write layer backing store ko synchronously update karti hai.

### Q. What is cache stampede?

Many requests same popular key par simultaneously miss karke DB hit karein.

### Q. What is cache penetration?

Repeated requests for non-existent data repeatedly DB tak jaayein.

### Q. What is cache avalanche?

Many cache entries simultaneously expire/fail and backend par huge load aa jaye.

### Q. How do you handle stale data?

TTL, explicit/event-driven invalidation, versioning aur business-specific consistency strategy.

### Q. What if Redis goes down?

Bounded timeouts, DB protection, graceful degradation/local cache where safe, circuit breaking and highly available cache architecture.

### Q. How do you choose TTL?

Freshness SLA, update frequency, read frequency and acceptable staleness ke basis par.

### Q. How do you monitor cache?

Hit ratio, miss ratio, latency, memory, evictions, errors, key distribution and DB QPS.

### Q. Can caching reduce write load?

Not by default. Primarily reads reduce hote hain. Write-through/write-back writes ko affect kar sakte hain but different consistency/durability trade-offs ke saath.

### Q. Redis vs local HashMap?

Redis shared distributed state provide kar sakta hai; local cache faster ho sakta hai but instance-local hota hai.

### Q. Can personalized responses be cached?

Yes, but authorization and cache-key isolation carefully design karni hogi, warna data leak ho sakta hai.

---

## Status

✅ **Q26 Solution Completed**

Next: **Q27 — What are Cascading Failures?**
