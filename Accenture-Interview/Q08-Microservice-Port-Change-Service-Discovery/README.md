# Q8 — Microservice ka Port Change Ho Jaye To Dusra Service Correct Instance Kaise Identify Karega?

> **Interview Question:** Suppose the port of a microservice changes. How will another service identify and send requests to the correct instance?

## 1. Simple Hinglish Explanation

Microservices architecture mein hum kisi service ko **fixed IP address + fixed port** se call nahi karna chahte.

Agar `Order Service` ko `Payment Service` call karna hai, aur Payment Service ka port ya IP change ho gaya, to Order Service ka hardcoded configuration break ho jayega.

Is problem ko solve karne ke liye hum **Service Discovery** use karte hain.

Order Service simply bolega:

```text
payment-service
```

Aur Service Discovery mechanism current healthy instance ka address provide karega, for example:

```text
payment-service
      ↓
10.0.1.15:8081
```

Agar next deployment mein port change hokar:

```text
10.0.2.20:9090
```

ho jaye, Order Service ko apna code change karne ki zarurat nahi hoti.

---

## 2. Problem Without Service Discovery

Suppose:

```text
Order Service
      |
      | http://10.0.1.15:8081
      ↓
Payment Service
```

Ab Payment Service ka instance restart hua aur naya address mila:

```text
10.0.2.20:9090
```

Order Service abhi bhi old address call karega:

```text
10.0.1.15:8081 ❌
```

Result:

```text
Connection Failed
```

Isliye distributed environment mein hardcoded IP/port par depend karna reliable nahi hai.

---

# 3. Service Discovery Solution

Service Discovery ke through services apna current location register karti hain.

```text
                    Service Registry
                         |
             +-----------+-----------+
             |                       |
      payment-service          order-service
      10.0.1.15:8081           10.0.1.16:8080
```

Payment Service start hone par registry mein register karega:

```text
Service Name: payment-service
Host: 10.0.1.15
Port: 8081
Status: HEALTHY
```

Agar instance change hota hai:

```text
payment-service
Host: 10.0.2.20
Port: 9090
Status: HEALTHY
```

Registry updated information provide karegi.

---

# 4. Request Flow

```text
Client
  |
  ↓
Order Service
  |
  | 1. Need Payment Service
  ↓
Service Discovery / Registry
  |
  | 2. Return healthy instance
  ↓
10.0.2.20:9090
  |
  ↓
Payment Service
```

Important point:

**Order Service ko actual IP/port remember nahi karna padta. Woh service name ke through service ko identify karta hai.**

---

# 5. Multiple Instances Ho To?

Real production system mein Payment Service ke multiple instances ho sakte hain:

```text
payment-service
   |
   +--> 10.0.1.10:8080
   +--> 10.0.1.11:8080
   +--> 10.0.1.12:8080
```

Service Discovery healthy instances ki list de sakta hai.

Uske baad load-balancing mechanism kisi ek instance ko select karega.

```text
Order Service
      |
      ↓
payment-service
      |
      ↓
Service Discovery
      |
      +--> Instance 1
      +--> Instance 2
      +--> Instance 3
```

---

# 6. Health Checking

Sirf registered hona enough nahi hai.

Agar instance crash ho gaya hai, registry/load-balancer ko usko traffic dena band karna chahiye.

Example:

```text
Instance 1 → HEALTHY ✅
Instance 2 → HEALTHY ✅
Instance 3 → DOWN ❌
```

Traffic sirf healthy instances ko milega.

Spring Boot environment mein health information commonly Actuator health endpoints aur infrastructure health checks ke through expose/verify ki ja sakti hai.

---

# 7. Client-Side vs Server-Side Discovery

## Client-Side Discovery

Client/service khud registry se instances obtain karta hai aur instance select karta hai.

```text
Order Service
      |
      ↓
Registry
      |
      ↓
Healthy Instances
      |
      ↓
Order Service selects one
      |
      ↓
Payment Service
```

## Server-Side Discovery

Client ek stable endpoint/load balancer ko call karta hai. Discovery aur instance selection infrastructure handle karta hai.

```text
Order Service
      |
      ↓
Load Balancer / Gateway
      |
      ↓
Service Discovery
      |
      +--> Payment Instance 1
      +--> Payment Instance 2
```

Interview mein dono ka difference explain karna important hai.

---

# 8. Kubernetes Environment Mein

Kubernetes mein usually services ko direct Pod IP se call nahi karte.

```text
Order Service
      |
      ↓
payment-service
      |
      ↓
Kubernetes Service
      |
      +--> Payment Pod 1
      +--> Payment Pod 2
      +--> Payment Pod 3
```

Pods replace/restart ho sakte hain aur unke IPs change ho sakte hain, lekin Kubernetes Service ek stable service endpoint provide karta hai.

Isliye application ko individual Pod IP manage nahi karna padta.

---

# 9. API Gateway Se Relation

API Gateway aur Service Discovery same cheez nahi hain.

**API Gateway:**
- Entry point provide karta hai
- Routing
- Authentication/Authorization
- Rate limiting
- Request aggregation etc.

**Service Discovery:**
- Service instances ka current location discover karne mein help karta hai
- Healthy instances identify karne mein help karta hai

Typical flow:

```text
Frontend
   |
   ↓
API Gateway
   |
   ↓
Service Discovery
   |
   ↓
Payment Service Instance
```

---

# 10. Interview-Ready Answer

> "Microservices mein hum services ko hardcoded IP address aur port se call nahi karte, because instances frequently restart, scale aur redeploy ho sakte hain. Isliye hum Service Discovery use karte hain. Har service apne current host, port aur health status ko service registry ya platform mechanism mein register karti hai. Calling service logical service name, for example `payment-service`, use karti hai. Service discovery healthy instances ka address provide karta hai, aur load-balancing mechanism ek instance select karta hai. Is approach se port ya IP change hone par calling service ko code change nahi karna padta. Kubernetes environment mein Kubernetes Service aur DNS bhi isi problem ko solve karte hain by providing a stable service endpoint in front of dynamic Pods."

---

# 11. Follow-Up Questions

### Q. Why not use IP address directly?

Because IP/port can change due to restart, deployment, scaling or rescheduling.

### Q. What happens if one instance goes down?

Health checks detect the unhealthy instance and traffic should be routed only to healthy instances.

### Q. What happens when service scales from 2 to 10 instances?

New instances register/become discoverable and the load-balancing mechanism distributes traffic across healthy instances.

### Q. Is API Gateway the same as Service Discovery?

No. Gateway is mainly an entry/routing and cross-cutting layer; service discovery finds service instances.

### Q. How does Kubernetes solve this?

A Kubernetes Service provides a stable endpoint/DNS name while Pods behind it can be dynamically created, removed or replaced.

---

# 12. Key Points to Remember

```text
Hardcoded IP/Port ❌
        ↓
Service Name ✅
        ↓
Service Discovery
        ↓
Healthy Instance
        ↓
Load Balancing
        ↓
Request
```

### One-line memory trick

**"Service name stable hota hai, instance location dynamic ho sakti hai — Service Discovery dynamic location find karwata hai."**

---

## Status

✅ **Q8 Solution Completed**

Next: **Q9 — Why do we use Service Discovery?**
