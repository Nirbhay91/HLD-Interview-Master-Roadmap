# Q17 — How Do You Secure Your Microservices?

> **Interview Question:** How do you secure your microservices?

## 1. Simple Hinglish Explanation

Microservices security ko sirf **JWT laga do** bolkar explain nahi karna chahiye.

Production-level answer mein multiple layers cover karne chahiye:

```text
Client
  ↓
HTTPS / TLS
  ↓
API Gateway
  ↓
Authentication
  ↓
Authorization
  ↓
Microservice
  ↓
Service-to-Service Authentication
  ↓
Database / Kafka / External Services
```

Main security controls:

- Authentication
- Authorization
- TLS / HTTPS
- API Gateway security
- Service-to-service authentication
- Secrets management
- Input validation
- Rate limiting
- Least privilege
- Secure database/message-broker access
- Logging & auditing
- Monitoring

OWASP recommends considering authentication and authorization at both edge and service levels, using defense in depth rather than trusting only the API Gateway. citeturn0search0turn0search7

---

# 2. Authentication vs Authorization

Ye difference sabse pehle clear rakho.

### Authentication — Who are you?

```text
User → Login
     ↓
Identity Provider
     ↓
Access Token
```

### Authorization — What are you allowed to do?

```text
User = Nirbhay
Role = ADMIN

Can DELETE /users?
        ↓
      YES / NO
```

Authentication identity establish karti hai; authorization access rights decide karti hai. citeturn0search5

---

# 3. Typical Production Architecture

```text
                    Identity Provider
                         |
                         | Access Token
                         ↓
Client → HTTPS → API Gateway
                    |
                    | Authenticate
                    | Coarse Authorization
                    | Rate Limit
                    ↓
             Microservice A
                |
                | mTLS / Service Token
                ↓
             Microservice B
                |
                ↓
             Database
```

Internal services ko direct anonymous access nahi dena chahiye.

---

# 4. API Gateway Security

API Gateway ko security ka **first layer** bana sakte hain.

Responsibilities:

- TLS termination/handling
- Authentication
- Coarse-grained authorization
- Rate limiting
- Request validation
- Routing
- Threat filtering
- Audit logging

Example:

```text
Client
  ↓
API Gateway
  ↓ Validate Access Token
  ↓ Check Scope/Role
  ↓ Rate Limit
  ↓
Order Service
```

Lekin sirf Gateway par authorization rakhna sufficient nahi hota. OWASP defense-in-depth ke liye gateway/proxy level ke saath service-level authorization bhi recommend karta hai. citeturn0search0

---

# 5. JWT / Access Token

User login ke baad Identity Provider access token issue kar sakta hai.

```text
Client
  ↓
Login
  ↓
Identity Provider
  ↓
Access Token
  ↓
API Gateway
  ↓
Microservice
```

JWT mein claims ho sakte hain:

```text
sub
scope
roles
iss
aud
exp
```

Resource service token ki signature/integrity, issuer, audience aur expiry jaise relevant claims validate karti hai.

OWASP notes that JWTs can carry claims for access control and should be integrity protected with a signature or MAC. citeturn0search7

---

# 6. Authorization at Service Level

Suppose Gateway ne request allow kar di:

```text
DELETE /orders/123
```

Order Service ko still check karna chahiye:

```text
Is this user allowed to delete this order?
```

Example:

```text
ADMIN → YES
ORDER_OWNER → Depends on business rule
NORMAL_USER → NO
```

Fine-grained/business-specific authorization service level par enforce karna important hai. citeturn0search0

---

# 7. Service-to-Service Security

Microservices ke beech communication ko bhi secure karna hai.

Do common approaches:

## A. mTLS

```text
Service A
   ⇄ TLS + Client Certificate ⇄
Service B
```

mTLS dono sides ko authenticate kar sakta hai aur communication confidentiality/integrity provide karta hai. OWASP specifically service-to-service authentication ke liye mTLS ko established pattern ke roop mein describe karta hai. citeturn0search0

## B. Service Tokens

```text
Service A
   ↓
Token Service / Identity Provider
   ↓
Service Token
   ↓
Service B
```

Token mein service identity/scopes ho sakte hain.

---

# 8. Why mTLS?

Normal TLS generally server ko client ke saamne authenticate karta hai.

mTLS mein:

```text
Client authenticates Server
AND
Server authenticates Client
```

Useful for:

- Service-to-service identity
- Preventing unauthorized internal calls
- Encryption in transit
- Strong workload identity

OWASP recommends strong authentication for internal service communication and notes TLS client authentication/mTLS as an option. citeturn0search8

---

# 9. TLS / HTTPS

Sensitive API traffic ko encrypted transport par chalana chahiye.

```text
HTTP  ❌
HTTPS ✅
```

TLS provides confidentiality and integrity and helps authenticate the server. OWASP recommends TLS for web-service communication carrying sensitive or authenticated data. citeturn0search2turn0search6

Internal traffic bhi sensitive hai:

```text
Service A → Service B
Service B → DB
Service C → Kafka
```

Environment aur threat model ke according encryption/authentication controls apply karne chahiye.

---

# 10. Secrets Management

Never hardcode:

```text
DB_PASSWORD = "password123" ❌
JWT_SECRET = "my-secret" ❌
API_KEY = "abc123" ❌
```

Instead use a dedicated secrets management mechanism.

```text
Microservice
     ↓
Secrets Manager / Vault / Cloud Secret Store
     ↓
Credential
```

Secrets should be centrally managed, access-controlled and rotated where appropriate. OWASP advises against secrets being hardcoded in source/configuration and recommends controlled storage, provisioning, auditing and rotation. citeturn0search4

---

# 11. Least Privilege

Har service ko sirf required permissions do.

Example:

```text
Order Service
   ↓
Can read/write Order DB
   ↓
Cannot directly delete Payment DB
```

Kafka example:

```text
Payment Consumer
   ↓
Read payment-events
   ↓
Cannot publish to every topic
```

Principle:

> **Minimum permission required to perform the job.**

OWASP highlights least privilege as a key microservice security design principle. citeturn0search3

---

# 12. Database Security

Database ko public internet par expose nahi karna chahiye unless there is a strong, justified architecture requirement.

Use:

- Private network/subnet
- Firewall/security groups
- TLS where supported
- Strong authentication
- Least-privileged DB users
- Encryption at rest where appropriate
- Credential rotation

Architecture:

```text
Internet
   ↓
API Gateway
   ↓
Microservice
   ↓
Private DB
```

Not:

```text
Internet → Database ❌
```

---

# 13. Kafka / Message Broker Security

Messaging layer bhi secure karna hai.

Consider:

- TLS
- Client authentication
- Topic-level authorization
- Least privilege
- Consumer/producer permissions
- Sensitive-data protection

Example:

```text
Payment Service
   ↓
TLS + Authentication
   ↓
Kafka
   ↓
Authorized Topic
```

---

# 14. Input Validation

Authentication ke baad bhi incoming data untrusted hai.

Validate:

- Request body
- Query parameters
- Path variables
- Content type
- File uploads
- Message payloads

Example:

```text
POST /orders
       ↓
Schema Validation
       ↓
Business Validation
       ↓
Process
```

Security ka matlab sirf identity/token validation nahi hai.

---

# 15. Rate Limiting

Agar attacker repeatedly API call kare:

```text
POST /login
POST /login
POST /login
...
```

Rate limiting abuse ko reduce kar sakti hai.

```text
Client
  ↓
API Gateway
  ↓
Rate Limiter
  ↓
Allowed → Service
Blocked → 429
```

OWASP's REST guidance describes rate limiting/API-key controls as mechanisms that can reduce abuse and excessive resource consumption. citeturn0search7

---

# 16. Don't Trust Internal Network

Ye important microservices security principle hai.

❌ Wrong assumption:

> "Request internal network se aa rahi hai, so it's trusted."

Better:

```text
Every service boundary
        ↓
Authenticate
        ↓
Authorize
```

API Gateway bypass prevent karne ke liye internal services ko appropriate authentication controls ke saath protect karna important hai. OWASP specifically gateway bypass aur defense-in-depth ko highlight karta hai. citeturn0search0

---

# 17. Token Propagation

External user token ko blindly har internal service ko forward karna always ideal nahi hai.

Better architecture can use:

```text
External Access Token
        ↓
Gateway / Edge
        ↓
Validate
        ↓
Trusted Internal Identity / Token
        ↓
Internal Service
```

Internal services ko required identity/context securely propagate karna chahiye.

OWASP warns that blindly propagating external access tokens internally can increase attack surface and recommends decoupling external tokens from internal identity representation where appropriate. citeturn0search0

---

# 18. Logging & Auditing

Security incidents investigate karne ke liye audit trail important hai.

Log useful security context:

```text
userId
serviceId
requestId / correlationId
action
resource
result
timestamp
```

But secrets log nahi karne:

```text
Password ❌
Access Token ❌
API Secret ❌
```

OWASP recommends structured logging, correlation IDs and filtering sensitive information from centralized logs. citeturn0search0

---

# 19. Monitoring & Security Alerts

Monitor:

- Authentication failures
- Authorization failures
- Abnormal traffic
- Repeated 401/403
- Token validation failures
- Unusual service-to-service calls
- Secret access anomalies
- High error rate

Example alert:

```text
1000 failed logins / minute
        ↓
Security Alert 🚨
```

---

# 20. Defense in Depth

Strong interview answer ka core concept:

```text
Layer 1 → TLS
Layer 2 → API Gateway
Layer 3 → Authentication
Layer 4 → Service Authorization
Layer 5 → mTLS / Service Identity
Layer 6 → DB / Kafka Access Control
Layer 7 → Secrets Management
Layer 8 → Logging / Monitoring
```

Ek layer compromise ho jaye to next layer protection de sakti hai.

OWASP recommends defense in depth across gateway/proxy, microservice and business-code levels. citeturn0search0

---

# 21. Complete Request Flow

```text
                Identity Provider
                       |
                       | Access Token
                       ↓
Client
  |
  | HTTPS
  ↓
API Gateway
  |
  | Authentication
  | Coarse Authorization
  | Rate Limit
  ↓
Order Service
  |
  | Fine-grained Authorization
  | mTLS / Service Token
  ↓
Payment Service
  |
  ↓
Private Payment DB
```

Security controls har trust boundary par apply hote hain.

---

# 22. How I Would Secure a Spring Boot Microservices System

Interview mein agar interviewer practical implementation pooche:

```text
1. Spring Security
2. OAuth 2.0 / OIDC based Identity Provider
3. JWT access tokens
4. API Gateway authentication
5. Method/endpoint authorization
6. mTLS or service tokens for internal calls
7. HTTPS everywhere
8. Secrets Manager/Vault
9. Database least privilege
10. Kafka TLS + ACLs
11. Rate limiting
12. Centralized security logging
13. Monitoring + alerts
14. Input validation
```

Exact technology organization ke cloud/platform stack par depend karegi.

---

# 23. Interview Scenario

### Interviewer:

> Suppose attacker somehow reaches Payment Service directly instead of going through API Gateway. How will you protect it?

### Strong Answer:

Main Gateway ko only security boundary nahi maanunga.

```text
Payment Service
   ↓
Validate authentication
   ↓
Authorize request
   ↓
Network policy / private access
   ↓
Service identity
```

Internal service ko direct anonymous access allow nahi karunga.

Ye **defense in depth** approach hai. citeturn0search0

---

# 24. Common Interview Mistakes

❌ "JWT se microservices secure ho jaate hain."

JWT only authentication/authorization mechanism ka part hai.

❌ "API Gateway security ke liye enough hai."

Gateway bypass scenario ke liye service-level controls bhi important hain.

❌ "Internal services trusted hain."

Internal calls ko appropriate authentication and authorization chahiye.

❌ "Password ko encrypted form mein config file mein rakh denge."

Secrets management mechanism use karo.

❌ "Authorization sirf Gateway mein karenge."

Fine-grained/business authorization service level par bhi required ho sakti hai.

---

# 25. Interview-Ready Answer

> **"I secure microservices using a defense-in-depth approach. At the edge, I use HTTPS/TLS, an API Gateway, authentication with OAuth 2.0/OIDC and access tokens, coarse-grained authorization and rate limiting. Each microservice also performs the authorization required for its own resources instead of blindly trusting the gateway. For service-to-service communication, I use strong service identity such as mTLS or service tokens. Databases and message brokers are kept private and protected with least-privilege access. Secrets are stored in a dedicated secrets-management system rather than source code. Finally, I use input validation, security logging, auditing, monitoring and alerting to detect and investigate attacks. The exact implementation depends on the platform and threat model."**

---

# 26. 30-Second Hinglish Answer

> **"Main microservices ko defense-in-depth approach se secure karunga. Client side par HTTPS, API Gateway aur OAuth/JWT authentication use karunga. Gateway par coarse authorization aur rate limiting hoga, but service level par bhi fine-grained authorization karunga. Service-to-service calls ke liye mTLS ya service tokens use kar sakte hain. DB aur Kafka ko private aur least-privilege access ke saath secure karunga. Secrets ko source code mein nahi, Secrets Manager/Vault mein rakhunga. Saath mein input validation, audit logging, monitoring aur alerting bhi maintain karunga."**

---

# 27. Memory Trick

```text
AUTHENTICATE
      ↓
AUTHORIZE
      ↓
ENCRYPT
      ↓
SERVICE IDENTITY
      ↓
LEAST PRIVILEGE
      ↓
SECRETS
      ↓
VALIDATE
      ↓
MONITOR
```

### One-line memory

**"Gateway protects the edge, services protect their own resources, and TLS + identity protect communication."**

---

# 28. Follow-Up Questions

### Q. JWT vs OAuth 2.0?

OAuth 2.0 is an authorization framework; JWT is a token format. OAuth access tokens can be JWTs, but OAuth does not require JWT.

### Q. JWT vs Session?

Session state is maintained server-side; a JWT is typically a self-contained signed token whose claims can be validated by the resource service.

### Q. API Gateway vs Authentication Service?

Identity Provider/authentication service establishes identity and issues tokens; Gateway can validate/enforce them at the edge.

### Q. How do services authenticate each other?

mTLS or service identity/access tokens are common approaches.

### Q. Why not share one secret among all services?

It increases blast radius. Prefer service-specific credentials/identity and least privilege.

### Q. What if token is stolen?

Use short-lived access tokens, secure transport, appropriate audience/scope validation, rotation/revocation strategies where applicable, and monitoring.

### Q. How do you prevent API Gateway bypass?

Keep internal services private/restricted and enforce service-level authentication/authorization as appropriate.

---

## Status

✅ **Q17 Solution Completed**

Next: **Q18 — Explain JWT authentication flow.**
