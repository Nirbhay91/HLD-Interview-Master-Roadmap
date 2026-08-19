# Q24 — What is Load Balancing?

> **Interview Question:** What is Load Balancing?

## 1. Simple Hinglish Explanation

Load Balancing ka simple meaning hai:

> **Incoming traffic ko multiple healthy servers/instances ke across distribute karna, taaki ek single server overload na ho.**

Example:

```text
                 Load Balancer
                /      |      \
               ↓       ↓       ↓
          Server-1  Server-2  Server-3
```

Agar 3000 requests aa rahi hain aur 3 servers hain, load balancer traffic ko available capacity ke according distribute kar sakta hai.

---

# 2. Why Do We Need Load Balancing?

Main reasons:

1. **High Availability**
2. **Horizontal Scalability**
3. **Fault Tolerance**
4. **Better Resource Utilization**
5. **Traffic Distribution**
6. **Failover**
7. **Zero/Low-Downtime Deployments support**

Without load balancing:

```text
Client
  ↓
One Server
  ↓
Overload ❌
```

With load balancing:

```text
Clients
   ↓
Load Balancer
   ↓
┌───────┬───────┬───────┐
↓       ↓       ↓
S1      S2      S3
```

---

# 3. Real-World Example

Suppose an Order Service has 5 instances:

```text
Order-1
Order-2
Order-3
Order-4
Order-5
```

Client ko in instances ke individual IP/port ka knowledge nahi chahiye.

```text
Client
   ↓
orders.example.com
   ↓
Load Balancer
   ↓
Healthy Order Instance
```

Load balancer backend instances ke across requests distribute karta hai.

---

# 4. Load Balancer Works at Which Layer?

Two important types:

```text
L4 Load Balancer
L7 Load Balancer
```

---

# 5. L4 Load Balancing

L4 = **Transport Layer**.

Usually based on:

```text
IP
Port
TCP
UDP
Connection information
```

Example:

```text
Client
  ↓ TCP :443
L4 LB
  ↓
Backend Server
```

It does not normally understand HTTP path/business-level information.

### Advantages

- Fast
- Lower processing overhead
- Protocol independent at transport level
- Good for high-throughput traffic

---

# 6. L7 Load Balancing

L7 = **Application Layer**.

HTTP/HTTPS request ko understand kar sakta hai.

Can route based on:

```text
Host
Path
HTTP method
Headers
Cookies
Other application-level attributes
```

Example:

```text
/api/orders/**
       ↓
Order Service

/api/payments/**
       ↓
Payment Service
```

### Advantages

- Content-aware routing
- API routing
- Header/cookie based policies
- Better application-level control

Trade-off:

> L7 processing generally has more overhead than simple L4 forwarding.

---

# 7. L4 vs L7

| Feature | L4 | L7 |
|---|---|---|
| Layer | Transport | Application |
| Understands HTTP path | No | Yes |
| Routing | IP/Port/connection | URL/header/host etc. |
| Processing | Lower | Higher |
| Flexibility | Lower | Higher |
| Typical use | TCP/UDP traffic | HTTP APIs/web apps |

### Memory

**L4 = Connection level**

**L7 = HTTP/Application level**

---

# 8. Load Balancing Algorithms

Common algorithms:

```text
Round Robin
Weighted Round Robin
Least Connections
IP Hash
Consistent Hashing
```

---

# 9. Round Robin

Requests sequentially instances ko distribute hote hain.

```text
Request 1 → S1
Request 2 → S2
Request 3 → S3
Request 4 → S1
Request 5 → S2
```

Pattern:

```text
S1 → S2 → S3 → S1 → S2 → S3
```

### Best when

Servers roughly similar capacity ke hain aur requests ka cost relatively similar hai.

---

# 10. Weighted Round Robin

Agar servers ki capacity different hai:

```text
S1 = weight 5
S2 = weight 3
S3 = weight 2
```

Then S1 ko more traffic milega.

Useful when:

```text
Different machine sizes
Canary deployments
Gradual traffic shifting
```

---

# 11. Least Connections

Load balancer us server ko choose karta hai jiske paas currently fewer active connections hain.

```text
S1 → 20 connections
S2 → 8 connections
S3 → 12 connections

Next → S2
```

Useful when request/connection duration variable ho.

Important:

> Least connections is not automatically the best algorithm for every workload; request cost and connection behavior matter.

---

# 12. IP Hash

Client IP ko hash karke backend select kiya ja sakta hai.

```text
hash(clientIP) % N
```

Same client generally same backend ko map ho sakta hai while mapping remains stable.

Use case:

```text
Session affinity / sticky behavior
```

But backend membership changes can remap clients unless a consistent hashing strategy or other mechanism is used.

---

# 13. Consistent Hashing

Consistent hashing data/clients ko hash ring par map karta hai.

```text
          Hash Ring
       ┌─────────────┐
       │ S1          │
       │      S2     │
       │             │
       │ S3          │
       └─────────────┘
```

Node add/remove hone par ideally limited keys remap hote hain compared with naive `hash(key) % N`.

Commonly useful for:

```text
Distributed caches
Partitioning
Sticky routing scenarios
```

---

# 14. Health Checks

Load balancer ko pata hona chahiye kaunsa instance healthy hai.

Example:

```text
GET /health
```

Result:

```text
S1 → Healthy ✅
S2 → Unhealthy ❌
S3 → Healthy ✅
```

Traffic:

```text
S1 ✅
S3 ✅
```

S2 ko new traffic stop/avoid kiya ja sakta hai according to LB behavior.

---

# 15. Health Check Types

### Liveness

Process/application alive hai?

### Readiness

Kya instance traffic receive karne ke liye ready hai?

### Deep health check

Dependencies bhi check kar sakta hai.

But caution:

> Agar health endpoint har dependency ko aggressively check kare aur dependency fail ho, to healthy instances bhi unnecessarily traffic se remove ho sakte hain.

Use health checks according to the deployment platform and failure model.

---

# 16. Failover

Suppose:

```text
S1 ❌
S2 ✅
S3 ✅
```

Load balancer S1 ko remove karke traffic healthy instances par bhej sakta hai.

```text
Client
  ↓
LB
 / \
S2  S3
```

This improves availability.

---

# 17. Load Balancing + Auto Scaling

Load balancing aur auto scaling complementary hain.

```text
                Load Balancer
              /      |       \
             ↓       ↓        ↓
            S1      S2       S3
                              ↑
                         Auto Scaling
                              ↓
                              S4
```

Auto scaling instances add karta hai.

Load balancer traffic distribute karta hai.

---

# 18. Load Balancer + Service Discovery

Microservices mein service instances dynamically change ho sakte hain.

```text
Service Registry
      ↓
Healthy Instances
      ↓
Load Balancer
      ↓
Selected Instance
```

Depending on architecture, service discovery and load balancing client-side ya server-side ho sakte hain.

---

# 19. Client-Side Load Balancing

Client service instances discover karke khud select karta hai.

```text
Client
  ↓
Service Discovery
  ↓
S1 / S2 / S3
```

Client has more responsibility.

---

# 20. Server-Side Load Balancing

Client ek stable endpoint ko call karta hai:

```text
Client
  ↓
Load Balancer
  ↓
S1 / S2 / S3
```

Load balancer backend selection karta hai.

### Memory

```text
Client-side = Client chooses
Server-side = Load Balancer chooses
```

---

# 21. Sticky Sessions

Sticky session ka matlab same client ko repeatedly same backend instance se route karna.

```text
User A
  ↓
S1

User A
  ↓
S1
```

Possible mechanisms:

```text
Cookie-based affinity
IP-based affinity
```

### Problem

Agar S1 down ho gaya:

```text
Sticky User
     ↓
S1 ❌
```

Session continuity impact ho sakti hai.

Therefore modern stateless services often avoid application state being tied to one instance.

---

# 22. Stateless vs Sticky Session

### Stateless

```text
Request 1 → S1
Request 2 → S2
Request 3 → S3
```

Any instance can handle request.

### Sticky

```text
User → S1
User → S1
User → S1
```

Useful in specific legacy/session-heavy scenarios, but adds routing dependency.

---

# 23. Load Balancing and Database

Important interview trap:

> Load balancing application servers does NOT automatically solve database scaling.

Example:

```text
LB
 ├→ S1
 ├→ S2
 ├→ S3
 └→ S4
      ↓
   One DB ❌
```

DB can still be the bottleneck.

Possible DB strategies:

```text
Read replicas
Caching
Partitioning
Sharding
Query optimization
```

---

# 24. Load Balancing and WebSocket

WebSocket is long-lived connection based.

Load balancing needs to account for:

```text
Connection lifetime
Connection affinity
Connection-aware routing
```

Depending on architecture, sticky sessions or a shared messaging layer may be needed.

For example:

```text
Client
  ↓ WebSocket
LB
  ↓
Chat Instance
  ↓
Redis/Kafka
```

---

# 25. Load Balancing During Deployment

Suppose old version:

```text
S1 v1
S2 v1
S3 v1
```

New version:

```text
S4 v2
```

Canary style:

```text
95% → v1
5%  → v2
```

Traffic shifting capability depends on the load balancer/platform.

---

# 26. Blue-Green Deployment

```text
Blue → Current
Green → New
```

Load balancer traffic switch kar sakta hai:

```text
Before:
LB → Blue

After:
LB → Green
```

Rollback comparatively quick ho sakta hai by switching traffic back, provided deployment/data compatibility supports it.

---

# 27. Global Load Balancing

Multi-region architecture:

```text
                 Global LB / DNS
                 /            \
                ↓              ↓
          Region A          Region B
          /    \             /    \
        S1      S2          S3      S4
```

Routing may consider:

```text
Latency
Health
Geography
Capacity
Failover policy
```

---

# 28. Load Balancer Failure

Load balancer itself should not become a single point of failure.

Bad:

```text
Client
  ↓
One LB ❌
```

Better:

```text
          LB Layer
        /          \
      LB1          LB2
        \          /
          Services
```

Managed cloud load balancers generally provide high availability, but exact implementation depends on provider.

---

# 29. Capacity and Connection Limits

Load balancer bhi finite capacity rakhta hai.

Consider:

```text
Connections/sec
Concurrent connections
Bandwidth
TLS handshakes
Request rate
Backend connection limits
```

At very high scale, load-balancing architecture itself may need horizontal/distributed capacity.

---

# 30. Load Balancer vs Reverse Proxy

### Load Balancer

Primary focus:

```text
Traffic distribution
Health checks
Failover
```

### Reverse Proxy

Sits in front of backend and can provide:

```text
Routing
TLS termination
Caching
Compression
Security policies
```

In practice, one component can perform both roles depending on the technology.

---

# 31. Load Balancer vs API Gateway

### Load Balancer

```text
Distribute traffic
Health checks
Failover
```

### API Gateway

```text
API routing
Authentication/policies
Rate limiting
Request transformation
API composition where appropriate
Observability
```

They can coexist:

```text
Client
 ↓
Load Balancer
 ↓
API Gateway
 ↓
Microservices
```

Exact placement depends on platform/design.

---

# 32. Complete Architecture

```text
                           Internet
                              ↓
                         DNS / CDN / WAF
                              ↓
                      Load Balancer (L7)
                              ↓
                    API Gateway / Reverse Proxy
                              ↓
             ┌────────────────┼────────────────┐
             ↓                ↓                ↓
          Order-1          Order-2          Order-3
             ↓                ↓                ↓
                         Database / Cache
```

If Order-2 fails:

```text
Health Check → Unhealthy
             ↓
LB removes/avoids Order-2
             ↓
Traffic → Order-1 / Order-3
```

---

# 33. Interview Scenario

### Interviewer:

> You have 10 instances of an Order Service. How will you distribute incoming requests?

### Strong Answer:

```text
I would place the service instances behind a load balancer.
The load balancer would perform health checks and route traffic
only to healthy instances. The routing algorithm would depend on
the workload—for example, round robin for similar stateless instances,
or least connections when connection duration varies significantly.
For different instance capacities, weighted routing can be used.
```

Then add:

> "The service should preferably be stateless so any healthy instance can process any request."

---

# 34. Interview-Ready Answer

> **"Load balancing is the process of distributing incoming traffic across multiple healthy service instances to improve availability, scalability and fault tolerance. A load balancer can use algorithms such as round robin, weighted round robin, least connections or hash-based routing depending on the workload. It also performs health checks so unhealthy instances can be removed from traffic. At L4 it works primarily with transport-level information such as IP and port, while L7 can make application-aware decisions using HTTP properties such as host, path or headers. In a microservices architecture, I would typically keep services stateless and combine load balancing with autoscaling, service discovery and monitoring."**

---

# 35. 30-Second Hinglish Answer

> **"Load balancing ka matlab incoming requests ko multiple healthy server instances ke beech distribute karna hai, taaki ek server overload na ho aur system highly available rahe. Load balancer health checks karta hai aur unhealthy instance ko traffic se remove kar sakta hai. Common algorithms Round Robin, Weighted Round Robin, Least Connections aur Hash-based routing hain. L4 IP/port level par kaam karta hai, jabki L7 HTTP path, host ya headers ke basis par routing kar sakta hai. Microservices mein stateless services + load balancer + autoscaling ka combination commonly use hota hai."**

---

# 36. Memory Trick

```text
TRAFFIC
   ↓
DISTRIBUTE
   ↓
HEALTH CHECK
   ↓
FAILOVER
   ↓
SCALE
```

### One-line memory

**"Load Balancer = Distribute traffic + Check health + Route to healthy instances."**

---

# 37. Follow-Up Questions

### Q. What is L4 vs L7 load balancing?

L4 routes using transport-level information such as IP/port; L7 understands application protocols such as HTTP and can route using host/path/headers.

### Q. Round Robin vs Least Connections?

Round Robin cycles through instances. Least Connections selects an instance with fewer active connections. Choice depends on workload characteristics.

### Q. What happens when one server goes down?

Health checks detect the unhealthy instance and the load balancer avoids sending new traffic to it according to its configured behavior.

### Q. Why stateless services?

Any healthy instance can handle a request, making horizontal scaling and failover easier.

### Q. What is sticky session?

A routing mechanism that keeps a client associated with a particular backend instance for some period.

### Q. Is sticky session always recommended?

No. It can create uneven load and reduce failover flexibility. Stateless architecture is usually preferred when practical.

### Q. Does load balancer solve DB scaling?

No. It distributes application traffic; database capacity must be designed separately.

### Q. Load Balancer vs API Gateway?

LB primarily distributes traffic and handles health/failover; API Gateway adds API-aware edge policies such as auth, rate limiting and routing.

### Q. Can load balancer do canary deployment?

Many platforms can support weighted/controlled traffic routing, but exact capabilities depend on the LB/platform.

### Q. What if load balancer itself fails?

Use a highly available/redundant LB layer or managed service designed for HA.

### Q. Client-side vs server-side load balancing?

Client-side selects the instance; server-side load balancer selects the instance on behalf of the client.

---

## Status

✅ **Q24 Solution Completed**

Next: **Q25 — How does Auto Scaling work?**
