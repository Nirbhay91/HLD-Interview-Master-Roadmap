# Q21 — Why Do We Use API Gateway Instead of Directly Calling Microservices?

> **Interview Question:** Why do we use API Gateway instead of directly calling microservices?

## 1. Simple Hinglish Explanation

Agar frontend directly har microservice ko call kare:

```text
Frontend
 ├──→ Order Service
 ├──→ Payment Service
 ├──→ Inventory Service
 └──→ User Service
```

to frontend ko internal service topology, endpoints, authentication, CORS aur versioning ka knowledge rakhna padega.

API Gateway ek **single controlled entry point** provide karta hai:

```text
                 API Gateway
                /     |      \
               ↓      ↓       ↓
            Order  Payment  Inventory
```

Isse client aur internal microservices ke beech **loose coupling + centralized edge controls** milte hain.

---

# 2. Main Reasons

API Gateway commonly use hota hai for:

1. **Single Entry Point**
2. **Routing**
3. **Authentication**
4. **Authorization / policy enforcement**
5. **Rate Limiting**
6. **TLS termination**
7. **Request aggregation**
8. **Load balancing integration**
9. **Observability / Correlation**
10. **API versioning and controlled exposure**
11. **Protocol/header transformation where required**
12. **Protecting internal topology**

---

# 3. Single Entry Point

Without Gateway:

```text
Frontend
 ↓
Multiple public service endpoints
```

With Gateway:

```text
Frontend
    ↓
api.example.com
    ↓
API Gateway
    ↓
Internal Services
```

Frontend ko multiple service endpoints manage nahi karne padte.

---

# 4. Service Topology Hidden

❌ Direct:

```text
Frontend
   ↓
order-service:8081
```

Agar service ka:

- Port change ho
- Host change ho
- Number of instances change ho
- Service move ho
- Deployment topology change ho

frontend impact ho sakta hai.

Gateway ke saath:

```text
Frontend
   ↓
api.example.com/orders
   ↓
Gateway
   ↓
Current healthy Order instances
```

Internal topology frontend se decoupled rehti hai.

---

# 5. Routing

Gateway URL ko service ke saath map kar sakta hai:

```text
/api/orders/**
       ↓
Order Service

/api/payments/**
       ↓
Payment Service

/api/products/**
       ↓
Product Service
```

Dynamic environments mein gateway service discovery/load balancing integration ke through healthy instances select kar sakta hai.

---

# 6. Authentication

Gateway edge par access token validate kar sakta hai.

```text
Client
  ↓ JWT
API Gateway
  ↓ Validate
Microservice
```

But important:

> **Gateway authentication does not automatically remove the need for service-level security.**

Critical services should still enforce their own authorization/security boundaries according to the architecture.

---

# 7. Authorization

Gateway common policies enforce kar sakta hai:

```text
Admin API
   ↓
admin scope/role required
```

But resource/business authorization often belongs in the service that owns the resource.

Example:

```text
Can user-123 modify order-456?
```

Ye decision Order Service ke business context mein better evaluate ho sakta hai.

---

# 8. Rate Limiting

Gateway centralized rate limiting provide kar sakta hai:

```text
100 req/min/user
```

or:

```text
10,000 req/min/client
```

Benefits:

```text
Abuse protection
Traffic control
Backend overload reduction
Fair usage
```

Distributed gateways ke case mein shared/distributed rate limiting mechanism ki zarurat ho sakti hai.

---

# 9. TLS Termination

Gateway TLS connection terminate kar sakta hai:

```text
Client
  ↓ HTTPS
Gateway
  ↓
Internal services
```

Internal traffic ko automatically insecure nahi maana chahiye; service-to-service TLS/mTLS may still be required.

---

# 10. Request Aggregation

Suppose frontend dashboard ko chahiye:

```text
User
Orders
Recommendations
```

Without aggregation:

```text
Frontend
  ├→ User Service
  ├→ Order Service
  └→ Recommendation Service
```

With aggregation/BFF:

```text
Frontend
   ↓
Gateway / BFF
   ├→ User Service
   ├→ Order Service
   └→ Recommendation Service
   ↓
Aggregated Response
   ↓
Frontend
```

Isse client-side round trips reduce ho sakte hain.

**Note:** Pure API Gateway aur BFF same thing nahi hain. Complex client-specific aggregation often BFF concern hota hai.

---

# 11. API Versioning

Gateway controlled version routing support kar sakta hai:

```text
/api/v1/orders
      ↓
Order Service v1

/api/v2/orders
      ↓
Order Service v2
```

Isse client migration gradual ho sakti hai.

---

# 12. Canary / Controlled Routing

Gateway traffic routing policies support kar sakta hai.

Example:

```text
95% → v1
5%  → v2
```

New version ko gradually expose kar sakte hain.

Exact capability gateway/platform par depend karti hai.

---

# 13. Observability

Gateway request metadata capture kar sakta hai:

```text
Request ID
Correlation ID
Latency
HTTP status
Route
Client information
```

Flow:

```text
Client
 ↓
Gateway
 ↓
Order Service
 ↓
DB
```

Same correlation/trace context propagate karne se distributed debugging easier hoti hai.

---

# 14. Protocol / Header Transformation

Gateway kuch cases mein:

```text
Header transformation
Path rewriting
Protocol translation
Request normalization
```

kar sakta hai.

Example:

```text
/api/orders
     ↓
/internal/v2/order-service/orders
```

But business logic gateway mein excessive amount mein nahi rakhna chahiye.

---

# 15. Security Boundary

Gateway public traffic aur internal network ke beech controlled boundary provide kar sakta hai:

```text
Internet
   ↓
WAF
   ↓
API Gateway
   ↓
Private Services
```

Internal services ko directly internet se expose na karke attack surface reduce kiya ja sakta hai.

---

# 16. Load Balancing

Gateway/load-balancing layer multiple instances ke across traffic distribute kar sakti hai:

```text
Gateway
  ↓
 ┌───────┬───────┬───────┐
 ↓       ↓       ↓
Order-1 Order-2 Order-3
```

Health-aware routing available platform ke according configure ki ja sakti hai.

---

# 17. Failure Handling

Gateway edge-level policies provide kar sakta hai, but retry carefully use karna chahiye.

For example:

```text
Gateway
  ↓
Order Service timeout
  ↓
Retry only when safe
```

Unsafe non-idempotent operations ko blindly retry karna duplicate effects create kar sakta hai.

Service-level timeout/circuit-breaker policies may also be needed.

---

# 18. API Gateway vs Direct Calling

| Concern | Direct Microservice Calls | API Gateway |
|---|---|---|
| Public entry point | Multiple | Usually one controlled entry |
| Routing | Client handles more | Centralized |
| Auth edge policy | Distributed | Centralized edge layer |
| Rate limiting | Repeated implementation | Central policy |
| Internal topology | More exposed | Hidden/abstracted |
| Aggregation | Client-side | Gateway/BFF can aggregate |
| Observability | More scattered | Common edge telemetry |
| Version routing | Client complexity | Central policy possible |
| Service coupling | Higher | Lower at client boundary |
| Extra hop | ❌ | ✅ |

---

# 19. Does API Gateway Remove All Problems?

No.

Gateway khud ek critical component ban sakta hai.

```text
Client
  ↓
Gateway
  ↓
Services
```

If poorly designed:

```text
Gateway failure
      ↓
Entire API unavailable
```

Therefore Gateway itself should be:

- Horizontally scalable
- Highly available
- Stateless where practical
- Monitored
- Rate controlled
- Properly configured

---

# 20. API Gateway Can Become a Bottleneck

Suppose traffic:

```text
1M requests/sec
      ↓
Single gateway instance
```

Bad design.

Better:

```text
             Load Balancer
             /    |    \
            ↓     ↓     ↓
        Gateway Gateway Gateway
```

Scale gateway horizontally.

---

# 21. API Gateway vs Service Mesh

Interview mein ye difference useful hai.

### API Gateway

Primarily north-south traffic:

```text
Client
  ↓
Gateway
  ↓
Services
```

### Service Mesh

Primarily east-west service-to-service traffic:

```text
Service A
   ↔
Service B
   ↔
Service C
```

Mesh can provide workload identity, mTLS, traffic policies and telemetry depending on implementation.

---

# 22. API Gateway vs BFF

### API Gateway

General-purpose API entry point.

### BFF

Frontend-specific backend.

```text
Web
 ↓
Web BFF
 ↓
Services

Mobile
 ↓
Mobile BFF
 ↓
Services
```

BFF often handles client-specific aggregation and response shaping.

---

# 23. Real-World Example

E-commerce:

```text
                     Internet
                         ↓
                        WAF
                         ↓
                  Load Balancer
                         ↓
                   API Gateway
                         ↓
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
   Order Service   Product Service   User Service
        ↓                ↓                ↓
     Order DB        Product DB        User DB
```

Gateway can handle:

```text
Authentication
Rate limiting
Routing
Observability
Versioning
```

Services handle:

```text
Business logic
Resource authorization
Database operations
```

---

# 24. Why Not Put Business Logic in Gateway?

Bad design:

```text
Gateway
 ↓
1000 lines business logic
 ↓
Services
```

Problems:

- Gateway becomes tightly coupled
- Hard to test
- Hard to deploy
- Single critical bottleneck
- Business ownership becomes unclear

Better:

```text
Gateway
 ↓
Routing + Edge Policies
 ↓
Service
 ↓
Business Logic
```

---

# 25. When Might Direct Service Calls Be Acceptable?

"Never directly call microservices" too absolute hai.

Direct calls can be acceptable when:

- Controlled internal environment
- Specialized trusted clients
- No need for common edge policies
- Architecture intentionally exposes multiple APIs
- Service mesh/other boundary provides required controls

But public browser/mobile clients ke liye stable API boundary generally cleaner hota hai.

---

# 26. Interview Scenario

### Interviewer:

> Why would you not let the frontend directly call Order, Payment and Inventory services?

### Strong Answer:

```text
Because it would couple the frontend to internal service topology.
It would also duplicate authentication, rate limiting,
CORS, routing and observability concerns across services.
```

Then:

```text
Frontend
   ↓
API Gateway / BFF
   ↓
Microservices
```

And add:

> "The gateway is not a replacement for service-level authorization or service-to-service security."

---

# 27. Interview-Ready Answer

> **"We use an API Gateway to provide a controlled API boundary between clients and internal microservices. Instead of exposing every service directly, the client calls a stable gateway endpoint. The gateway can handle routing, authentication, rate limiting, TLS termination, observability, version routing and, when appropriate, request aggregation. This reduces client-to-service coupling and hides internal topology. However, I would keep business logic in the services and still enforce resource authorization and service-to-service security downstream. The gateway itself should also be horizontally scalable and highly available because it becomes a critical entry point."**

---

# 28. 30-Second Hinglish Answer

> **"API Gateway use karne ka main reason hai frontend ko internal microservices se decouple karna. Frontend ek stable gateway endpoint call karta hai aur gateway routing, authentication, rate limiting, TLS aur observability jaise common concerns handle karta hai. Isse internal service topology hide rehti hai aur client complexity kam hoti hai. Lekin business logic gateway mein nahi rakhna chahiye, aur services ko apni authorization aur service-to-service security maintain karni chahiye."**

---

# 29. Memory Trick

```text
SINGLE ENTRY
     ↓
ROUTING
     ↓
AUTH
     ↓
RATE LIMIT
     ↓
OBSERVABILITY
     ↓
MICROSERVICES
```

### One-line memory

**"Gateway protects and routes the edge; services own the business logic."**

---

# 30. Follow-Up Questions

### Q. Is API Gateway mandatory in microservices?

No. It is an architectural pattern; whether to use it depends on client, security, routing and operational requirements.

### Q. Does Gateway replace service discovery?

No. Gateway can integrate with service discovery, but they solve different problems.

### Q. Does Gateway replace service-to-service authentication?

No. Internal service security is a separate concern.

### Q. Can Gateway aggregate responses?

Yes, but complex client-specific aggregation is often better suited to a BFF/composition layer.

### Q. What happens if Gateway goes down?

It can become a critical failure point, so run multiple instances behind load balancing and design for high availability.

### Q. Can Gateway contain business logic?

Keep business logic in domain services. Gateway should primarily handle edge concerns and lightweight request policies.

### Q. API Gateway vs Load Balancer?

A load balancer primarily distributes traffic among healthy instances; an API Gateway adds API-aware concerns such as routing policies, authentication, rate limiting and transformations.

### Q. API Gateway vs Service Mesh?

Gateway mainly handles north-south client traffic; service mesh mainly handles east-west service communication.

---

## Status

✅ **Q21 Solution Completed**

Next: **Q22 — How do you aggregate responses from multiple services?**
