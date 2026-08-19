# Q20 — How Does a Frontend Application Communicate With Backend Services in a Microservices Architecture?

> **Interview Question:** How does a frontend application communicate with backend services in a microservices architecture?

## 1. Simple Hinglish Explanation

Frontend ko normally directly har microservice ko call nahi karna chahiye.

Typical architecture:

```text
Frontend
   ↓
CDN / WAF
   ↓
Load Balancer
   ↓
API Gateway / BFF
   ↓
Microservices
   ↓
DB / Cache / Kafka
```

Frontend ek **public API entry point** ko call karta hai, usually API Gateway ya **Backend for Frontend (BFF)**.

Gateway/BFF request ko appropriate backend service tak route karta hai.

---

# 2. Example

Suppose e-commerce application hai:

```text
React / Angular / Mobile App
             ↓
        API Gateway
             ↓
    ┌────────┼─────────┐
    ↓        ↓         ↓
 Order    Product    User
 Service  Service    Service
```

Frontend ko ideally ye nahi pata hona chahiye ki:

```text
Order Service = 10.0.1.5
Product Service = 10.0.2.8
```

Frontend ko stable API endpoint pata ho:

```text
api.example.com
```

Gateway internally routing handle karta hai.

---

# 3. Complete Request Flow

Example:

```http
GET /api/orders
Authorization: Bearer <access-token>
```

Flow:

```text
Browser
  ↓
DNS
  ↓
CDN / WAF
  ↓
Load Balancer
  ↓
API Gateway / BFF
  ↓
Order Service
  ↓
Order DB
  ↓
Response
  ↓
Frontend
```

---

# 4. Step 1 — Frontend Sends HTTP Request

Frontend JavaScript/API client request bhejta hai.

Example:

```javascript
fetch('/api/orders', {
  method: 'GET',
  headers: {
    'Authorization': 'Bearer <access-token>'
  }
});
```

Usually APIs REST/HTTP use karti hain, but GraphQL, gRPC-Web, WebSocket, SSE etc. bhi architecture ke according use ho sakte hain.

---

# 5. Step 2 — DNS

Frontend public domain ko resolve karta hai:

```text
api.example.com
       ↓
DNS
       ↓
Public endpoint
```

Frontend ko internal service IPs expose nahi karne chahiye.

---

# 6. Step 3 — CDN / WAF

Request edge infrastructure se pass ho sakti hai.

### CDN
Static content ke liye useful:

```text
JS
CSS
Images
Videos
```

### WAF
Common web attacks ke against protection provide kar sakta hai.

```text
SQL Injection
XSS patterns
Malicious requests
```

Exact protection WAF rules/configuration par depend karti hai.

---

# 7. Step 4 — Load Balancer

Traffic multiple gateway/BFF instances mein distribute ho sakta hai.

```text
              Load Balancer
               /    |    \
              ↓     ↓     ↓
gateway-1   gateway-2   gateway-3
```

This helps horizontal scaling and availability.

---

# 8. Step 5 — API Gateway

Gateway common edge responsibilities handle kar sakta hai:

- Routing
- Authentication/token validation
- Rate limiting
- TLS termination
- Request/response policies
- Observability
- CORS policy where appropriate
- Load balancing/service discovery integration

Example:

```text
/api/orders/*
       ↓
Order Service

/api/products/*
       ↓
Product Service

/api/users/*
       ↓
User Service
```

Gateway ko business logic ka dumping ground nahi banana chahiye.

---

# 9. API Gateway vs BFF

Dono related but different patterns hain.

## API Gateway

General entry point:

```text
Frontend
   ↓
API Gateway
   ↓
Services
```

## BFF — Backend for Frontend

Different frontend clients ke liye tailored backend:

```text
Web App ─────→ Web BFF ──→ Services
Mobile App ──→ Mobile BFF ─→ Services
```

BFF response ko frontend-specific shape mein aggregate/transform kar sakta hai.

---

# 10. Why BFF?

Suppose mobile app ko sirf:

```text
name
price
image
```

chahiye.

Web app ko:

```text
name
price
image
reviews
recommendations
inventory
```

chahiye.

Agar same API sabko huge response dega to unnecessary data transfer ho sakta hai.

BFF client-specific response provide kar sakta hai.

---

# 11. Authentication

Frontend protected API call mein credential/access token bhej sakta hai.

Example:

```http
Authorization: Bearer <access-token>
```

Gateway/resource server token validate karta hai.

Typical flow:

```text
Login
 ↓
Authorization Server
 ↓
Access Token
 ↓
Frontend
 ↓
Protected API
```

Browser-based applications ke liye token/session storage aur CSRF/XSS protections carefully design karni chahiye.

---

# 12. Authorization

Authentication ke baad authorization check hota hai.

Example:

```text
GET /orders
orders:read
```

Allowed.

But:

```text
DELETE /orders/123
orders:delete
```

required hai.

Frontend UI button hide kar sakta hai, but **security enforcement backend par hona chahiye**.

Important interview line:

> **Frontend authorization is a UX feature; backend authorization is the security boundary.**

---

# 13. Service Routing

Gateway service name ke basis par route kar sakta hai:

```text
/api/orders
     ↓
order-service

/api/payment
     ↓
payment-service
```

Service Discovery ke saath:

```text
Gateway
  ↓
Service Registry
  ↓
order-service instances
```

Dynamic instances ke case mein registry/load-balancing mechanism current healthy instances choose karne mein help karta hai.

---

# 14. Frontend Should Not Know Internal Topology

❌ Bad design:

```text
Frontend
  ├→ order-service:8081
  ├→ payment-service:8082
  └→ inventory-service:8083
```

Problems:

- Internal topology exposed
- Service changes affect frontend
- More authentication complexity
- More CORS configuration
- More public attack surface
- Harder versioning

Better:

```text
Frontend
    ↓
api.example.com
    ↓
Gateway/BFF
    ↓
Internal Services
```

---

# 15. Response Aggregation

Suppose product page requires:

```text
Product
Reviews
Inventory
Recommendations
```

BFF/API composition layer can call multiple services:

```text
                BFF
          /      |      \
         ↓       ↓       ↓
     Product  Reviews  Inventory
         \       |       /
          \      |      /
             Response
                ↓
             Frontend
```

This can reduce frontend round trips, but introduces aggregation latency and failure handling concerns.

---

# 16. Synchronous vs Asynchronous Communication

Frontend user interactions are often request/response based:

```text
Frontend
   ↓ HTTP
Backend
   ↓
Response
```

For long-running/background work:

```text
Frontend
   ↓
POST /reports
   ↓
Backend
   ↓
Kafka / Queue
   ↓
Worker
```

Frontend may then:

```text
GET /reports/{id}
```

or receive progress/results through SSE/WebSocket depending on requirements.

---

# 17. WebSocket / SSE

Normal REST request:

```text
Client → Request → Server
Client ← Response ← Server
```

### WebSocket
Bidirectional real-time communication.

Useful for:

- Chat
- Live collaboration
- Real-time status

### SSE
Server → client streaming over HTTP.

Useful for:

- Notifications
- Live updates
- Progress streams

Choice depends on communication direction and requirements.

---

# 18. Error Handling

Backend ko consistent error response dena chahiye.

Example:

```json
{
  "code": "ORDER_NOT_FOUND",
  "message": "Order not found",
  "correlationId": "abc-123"
}
```

Frontend:

```text
4xx → user/request problem
5xx → server problem
```

Exact status-code policy API contract ke according define honi chahiye.

---

# 19. Correlation ID

Request tracing ke liye:

```text
Frontend
   ↓
X-Correlation-ID: abc-123
   ↓
Gateway
   ↓
Order Service
   ↓
Payment Service
```

Logs mein same ID hone se distributed request trace karna easier hota hai.

Modern systems distributed tracing ke liye trace context propagation bhi use karte hain.

---

# 20. CORS

Agar frontend aur API different origins par hain:

```text
Frontend:
https://app.example.com

API:
https://api.example.com
```

Browser CORS rules apply kar sakta hai.

Backend ko allowed origins/methods/headers carefully configure karne chahiye.

❌ Production mein blindly:

```text
Access-Control-Allow-Origin: *
```

use nahi karna chahiye when credentialed browser requests are involved.

---

# 21. Rate Limiting

Frontend/API traffic ko protect karne ke liye gateway par rate limiting ho sakti hai.

Example:

```text
100 requests/minute/user
```

or:

```text
1000 requests/minute/client
```

Rate limit key:

```text
User ID
API Key
IP
Client ID
```

requirements ke according choose hota hai.

---

# 22. API Versioning

Frontend aur backend independently deploy ho sakte hain.

Versioning useful hai:

```text
/api/v1/orders
/api/v2/orders
```

Better compatibility practices include backward-compatible evolution where possible.

Breaking changes ko carefully manage karna chahiye.

---

# 23. Pagination

Frontend ko huge dataset ek request mein nahi dena chahiye.

Example:

```http
GET /api/orders?page=1&size=20
```

Large/high-scale systems mein cursor-based pagination bhi useful ho sakti hai:

```http
GET /api/orders?cursor=abc123&limit=20
```

---

# 24. Caching

Common cache layers:

```text
Browser Cache
    ↓
CDN Cache
    ↓
Gateway/BFF Cache
    ↓
Service Cache
    ↓
Database
```

Cacheability carefully define karni chahiye, especially personalized/authenticated data ke liye.

---

# 25. Security Architecture

Recommended high-level flow:

```text
               Identity Provider
                     ↓
                  Token
                     ↓
Frontend ──HTTPS──→ Gateway/BFF
                     ↓
               AuthN / Policies
                     ↓
               Internal Services
                     ↓
            Service-to-Service Auth
```

Internal service communication ko automatically trusted nahi maana chahiye.

---

# 26. Mobile + Web + Multiple Clients

Large system mein:

```text
                 API Gateway
                /     |      \
               /      |       \
           Web BFF  Mobile BFF  Partner API
              |         |           |
              +---------+-----------+
                        ↓
                  Microservices
```

BFFs client-specific requirements handle kar sakte hain while core services business capabilities expose karte hain.

---

# 27. Complete Production Architecture

```text
                       ┌───────────────┐
                       │ Identity      │
                       │ Provider      │
                       └───────┬───────┘
                               │
                              Token
                               │
┌─────────────┐       ┌────────▼────────┐
│ Web / Mobile│ ────→ │ CDN / WAF / LB  │
└─────────────┘       └────────┬────────┘
                               ↓
                       ┌───────────────┐
                       │ API Gateway / │
                       │ BFF           │
                       └───────┬───────┘
                               ↓
             ┌─────────────────┼────────────────┐
             ↓                 ↓                ↓
       Order Service     Product Service   User Service
             ↓                 ↓                ↓
          Order DB          Product DB        User DB
             \                 |                /
              +----------- Kafka / Events ------+
```

---

# 28. Why Frontend Should Usually Not Directly Call Every Microservice

### Problem 1 — Security
More public endpoints.

### Problem 2 — Coupling
Frontend internal service topology se coupled ho jayega.

### Problem 3 — CORS
Multiple services ke liye browser CORS complexity increase hoti hai.

### Problem 4 — Authentication
Multiple endpoints ke credentials/security policies manage karna harder hota hai.

### Problem 5 — Aggregation
Frontend ko multiple service calls manage karne padenge.

### Problem 6 — Versioning
Internal service changes frontend ko directly affect kar sakte hain.

---

# 29. But Is Direct Service Access Always Wrong?

Not absolutely.

Some architectures intentionally expose multiple APIs or use specialized BFFs. The important requirement is:

```text
Public API Boundary
      ↓
Explicit Security
      ↓
Stable Contract
      ↓
Controlled Service Exposure
```

Interview mein "never" bolne ke bajay requirements-based answer dena better hai.

---

# 30. Interview Scenario

### Interviewer:

> Frontend ko Order, Product aur Payment services access karni hain. How will you design communication?

### Strong Answer:

```text
Frontend
   ↓ HTTPS
API Gateway / BFF
   ↓
Authentication + Authorization
   ↓
Service Routing
   ↓
Order / Product / Payment Services
```

If one screen needs data from multiple services:

```text
BFF
 ↓
Parallel service calls where safe
 ↓
Aggregate response
 ↓
Frontend
```

For long-running operations:

```text
Frontend
 ↓
POST job
 ↓
Queue/Kafka
 ↓
Worker
 ↓
GET status / SSE / WebSocket
```

---

# 31. Interview-Ready Answer

> **"In a microservices architecture, I would normally expose a stable API boundary such as an API Gateway or a Backend-for-Frontend rather than exposing every internal microservice directly to the frontend. The frontend communicates over HTTPS using APIs such as REST or GraphQL, and the gateway or BFF handles routing, authentication, rate limiting and other cross-cutting concerns. It then routes requests to the appropriate microservice using service discovery or load balancing. If a UI needs data from multiple services, a BFF or aggregation layer can compose the response. For real-time requirements we can use WebSocket or SSE, and for long-running asynchronous operations we can use a queue/event system with status polling or streaming. Internal service-to-service communication must also have its own security model."**

---

# 32. 30-Second Hinglish Answer

> **"Frontend usually directly har microservice ko call nahi karta. Frontend HTTPS ke through API Gateway ya BFF ko call karta hai. Gateway authentication, rate limiting aur routing handle karke request ko appropriate microservice tak bhejta hai. Agar ek screen ko multiple services ka data chahiye to BFF response aggregate kar sakta hai. Real-time use cases ke liye WebSocket/SSE aur long-running operations ke liye asynchronous queue + status API use kar sakte hain. Internal service-to-service communication ko bhi separately secure karna hota hai."**

---

# 33. Memory Trick

```text
FRONTEND
   ↓
HTTPS
   ↓
GATEWAY / BFF
   ↓
AUTH + RATE LIMIT
   ↓
ROUTING
   ↓
MICROSERVICE
   ↓
DB / CACHE / KAFKA
```

### One-line memory

**"Frontend talks to the API boundary; the API boundary talks to the microservices."**

---

# 34. Follow-Up Questions

### Q. Why use API Gateway?

Stable public entry point, routing and common cross-cutting policies.

### Q. What is BFF?

Backend-for-Frontend — client-specific backend layer that can tailor and aggregate APIs for a particular frontend.

### Q. Can frontend directly call microservices?

Technically yes, but exposing internal topology creates coupling and security/operational complexity. Requirements determine the appropriate boundary.

### Q. REST vs GraphQL?

REST exposes resource-oriented endpoints; GraphQL lets clients request a specific response shape. Choice depends on API and client requirements.

### Q. When use WebSocket?

Bidirectional real-time communication such as chat or collaboration.

### Q. When use SSE?

Server-to-client streaming such as notifications or progress updates.

### Q. How do you handle a long-running request?

Return an operation/job ID, process asynchronously, and expose status/results via polling or streaming.

### Q. Should frontend know service IPs?

No. It should use a stable public API endpoint/contract rather than internal IP addresses.

---

## Status

✅ **Q20 Solution Completed**

Next: **Q21 — Why do we use API Gateway instead of directly calling microservices?**
