# Q10 — What is the Role of API Gateway in Service Routing?

> **Interview Question:** What is the role of API Gateway in service routing?

## 1. Simple Hinglish Explanation

API Gateway ko microservices architecture ka **single entry point** samajho.

Frontend ko directly har microservice ka IP, port aur internal endpoint pata hona zaroori nahi hai.

Instead:

```text
Frontend / Client
        |
        ↓
   API Gateway
        |
        +------→ User Service
        |
        +------→ Order Service
        |
        +------→ Payment Service
        |
        +------→ Product Service
```

Client ek common endpoint ko call karta hai, aur API Gateway request ko **correct microservice** tak route karta hai.

Example:

```text
GET /api/users/101
        ↓
API Gateway
        ↓
User Service
```

Aur:

```text
POST /api/orders
        ↓
API Gateway
        ↓
Order Service
```

---

# 2. Problem Without API Gateway

Suppose system mein 5 microservices hain:

```text
Frontend
  |
  +----→ User Service
  +----→ Order Service
  +----→ Payment Service
  +----→ Product Service
  +----→ Notification Service
```

Frontend ko har service ka:

- URL
- Port
- Authentication mechanism
- Routing information
- Version
- Security rules

handle karna padega.

Agar service ka location change ho jaye, frontend ko update karna difficult ho sakta hai.

Isse client tightly coupled ho jata hai.

---

# 3. API Gateway Solution

API Gateway client aur internal microservices ke beech ek abstraction layer provide karta hai.

```text
                    API Gateway
                        |
          +-------------+-------------+
          |             |             |
          ↓             ↓             ↓
      User Service  Order Service  Payment Service
```

Client ko sirf gateway ka endpoint pata hota hai:

```text
https://api.example.com
```

Internal routing gateway handle karta hai.

---

# 4. API Gateway ka Main Role — Routing

Question specifically **service routing** ke baare mein hai, so interview mein routing ko clearly explain karo.

Example routing rules:

```text
/api/users/**     → User Service
/api/orders/**    → Order Service
/api/payments/**  → Payment Service
/api/products/**  → Product Service
```

Request:

```text
GET /api/orders/1001
```

Flow:

```text
Client
  |
  ↓
API Gateway
  |
  | Match: /api/orders/**
  ↓
Order Service
```

Gateway request ko correct backend service tak forward karta hai.

---

# 5. API Gateway + Service Discovery

Important interview point:

**API Gateway khud necessarily service discovery nahi hota.**

Gateway routing decision kar sakta hai, while service discovery current healthy service instances find karne mein help karta hai.

Example:

```text
Client
  |
  ↓
API Gateway
  |
  ↓
Service Discovery
  |
  +----→ Order Instance 1
  +----→ Order Instance 2
  +----→ Order Instance 3
```

Agar Order Service ke 3 instances hain, gateway/infrastructure healthy instance select karne ke liye discovery/load-balancing mechanism use kar sakta hai.

---

# 6. API Gateway ke Important Responsibilities

Routing ke alawa API Gateway commonly cross-cutting responsibilities handle kar sakta hai.

### 1. Routing

```text
/orders  → Order Service
/users   → User Service
```

### 2. Authentication

Gateway token verify kar sakta hai before request internal service tak jaaye.

```text
Client
  ↓ JWT
API Gateway
  ↓ validate
Microservice
```

### 3. Authorization

Gateway coarse-grained access rules enforce kar sakta hai, although fine-grained authorization often service ke andar bhi required hoti hai.

### 4. Rate Limiting

Example:

```text
User → 100 requests/minute
```

Limit cross hone par gateway request reject/throttle kar sakta hai.

### 5. Request Aggregation

Client ko multiple backend calls ki zarurat ho to gateway/BFF multiple services ko call karke combined response de sakta hai.

```text
Client
  ↓
API Gateway
  ├── User Service
  ├── Order Service
  └── Recommendation Service
          ↓
     Combined Response
```

### 6. Load Balancing

Multiple service instances ke beech traffic distribute karne mein gateway ya underlying load-balancing layer help kar sakti hai.

### 7. TLS Termination

Gateway edge par HTTPS/TLS terminate karke internal communication ke liye configured security model use kar sakta hai.

### 8. Logging / Correlation

Gateway common request metadata, correlation ID aur access logs capture kar sakta hai.

---

# 7. Complete Request Flow

```text
                    Internet
                       |
                       ↓
                 Load Balancer
                       |
                       ↓
                  API Gateway
                       |
            +----------+----------+
            |          |          |
            ↓          ↓          ↓
       User Service Order Service Payment Service
            |          |          |
            +----------+----------+
                       |
                  Cache / DB / Kafka
```

A typical request:

```text
1. Client sends request
2. Load balancer selects gateway instance
3. API Gateway authenticates/validates request
4. Gateway applies rate limit/security policies
5. Gateway matches route
6. Gateway discovers/selects healthy backend instance
7. Request is forwarded
8. Response comes back through gateway
9. Gateway returns response to client
```

---

# 8. API Gateway vs Load Balancer

Ye interview mein frequently confuse kiya jata hai.

| API Gateway | Load Balancer |
|---|---|
| API-level routing | Traffic distribution |
| Authentication/authorization policies | Instance selection/distribution |
| Rate limiting | Health checks |
| Request aggregation | Failover |
| API transformation | Load distribution |
| Client-facing API abstraction | Usually infrastructure/network layer |

Dono ek architecture mein saath use ho sakte hain.

```text
Client
  ↓
Load Balancer
  ↓
API Gateway instances
  ↓
Microservices
```

---

# 9. API Gateway vs Service Discovery

| API Gateway | Service Discovery |
|---|---|
| Client-facing entry point | Service location mechanism |
| Routes requests | Finds service instances |
| Authentication / rate limiting etc. | Health/instance information |
| Hides internal topology from clients | Helps services/infrastructure locate instances |

Simple memory:

**Gateway = request ko kahan bhejna hai**

**Service Discovery = service ka current instance kahan hai**

---

# 10. API Gateway vs Direct Microservice Calls

### Without Gateway

```text
Frontend
  ├──→ User Service
  ├──→ Order Service
  ├──→ Payment Service
  └──→ Product Service
```

### With Gateway

```text
Frontend
      |
      ↓
API Gateway
      |
      +──→ User Service
      +──→ Order Service
      +──→ Payment Service
      +──→ Product Service
```

Benefits:

- Client complexity reduce hoti hai
- Internal services hide hote hain
- Common policies centralize hoti hain
- Routing centrally manage hota hai
- Backend services independently evolve kar sakti hain

---

# 11. Important Trade-offs

API Gateway har system mein automatically best choice nahi hota.

### Advantages

- Single entry point
- Centralized routing
- Security policies
- Rate limiting
- Observability
- Request aggregation

### Disadvantages

- Additional network hop
- Gateway itself bottleneck ban sakta hai
- Single point of failure agar highly available design na ho
- Configuration complexity
- Too much business logic gateway mein rakhne se coupling badh sakti hai

Production mein gateway ko generally **multiple instances + load balancing + autoscaling + health checks** ke saath deploy karna chahiye.

---

# 12. Important Design Rule

API Gateway mein business logic unnecessarily nahi bharna chahiye.

Good:

```text
Authentication
Routing
Rate Limiting
Aggregation
Observability
```

Avoid putting complex domain logic such as:

```text
CalculateOrderPrice()
ProcessPaymentBusinessRules()
ApplyComplexDiscountRules()
```

Ye domain services mein rehna chahiye.

---

# 13. Interview-Ready Answer

> "API Gateway microservices architecture mein client ke liye single entry point provide karta hai. Client directly individual microservices ko call karne ke bajay gateway ko request bhejta hai. Gateway request path ke basis par correct microservice ko route karta hai. For example, `/orders/**` Order Service aur `/payments/**` Payment Service ko route kiya ja sakta hai. Gateway routing ke alawa authentication, authorization policies, rate limiting, request aggregation, load balancing integration, TLS termination aur centralized logging jaise cross-cutting concerns bhi handle kar sakta hai. Internal service instances dynamic ho sakte hain, isliye gateway service discovery aur load-balancing mechanism ke saath healthy instance tak request forward kar sakta hai."

---

# 14. 30-Second Hinglish Answer

> **"API Gateway microservices ka single entry point hota hai. Frontend ko har microservice ka IP aur port nahi pata hota. Client gateway ko request bhejta hai, gateway route ke basis par correct service ko request forward karta hai. Saath mein authentication, rate limiting, authorization, aggregation aur logging jaise common concerns handle kar sakta hai. Agar services ke multiple dynamic instances hain to gateway service discovery aur load balancing ke through healthy instance select karwa sakta hai."**

---

# 15. Follow-Up Questions

### Q. Is API Gateway same as Load Balancer?

No. Load balancer primarily traffic distribute karta hai; API Gateway API-level routing aur cross-cutting policies bhi handle karta hai.

### Q. Is API Gateway same as Service Discovery?

No. Gateway request entry/routing layer hai; service discovery service instances locate karne ka mechanism hai.

### Q. Can API Gateway become a bottleneck?

Yes. Isliye multiple gateway instances, load balancing, autoscaling and proper capacity planning use karte hain.

### Q. What if API Gateway goes down?

High availability ke liye multiple gateway instances deploy karte hain, usually load balancer ke behind.

### Q. Should business logic be placed in API Gateway?

Generally no. Gateway ko routing aur cross-cutting concerns tak limited rakhna better hai.

### Q. Can gateway aggregate multiple service responses?

Yes. Especially BFF/API-composition scenarios mein gateway or dedicated aggregation layer multiple services ko call karke combined response de sakta hai.

---

# 16. Key Memory Trick

```text
Client
  ↓
API Gateway
  ↓
Route
  ↓
Service Discovery / Load Balancing
  ↓
Healthy Service Instance
```

### One-line memory

**"API Gateway client ke liye single door hai, jo request ko correct microservice tak route karta hai aur common cross-cutting concerns handle karta hai."**

---

## Status

✅ **Q10 Solution Completed**

Next: **Q11 — Why should services communicate using service names instead of IP addresses?**
