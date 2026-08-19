# Q19 — How Do Services Authenticate With Each Other?

> **Interview Question:** How do services authenticate with each other?

## 1. Simple Hinglish Explanation

Microservices mein ek service ko doosri service ko call karte waqt sirf ye prove nahi karna hota ki request valid hai; usse apni **service identity** bhi prove karni hoti hai.

Example:

```text
Order Service
      ↓
Payment Service
```

Payment Service ko verify karna hai:

> "Kya request genuinely Order Service se aa rahi hai?"

Isse **service-to-service authentication** ya **machine-to-machine authentication** kehte hain.

Common approaches:

```text
1. OAuth 2.0 Client Credentials
2. mTLS
3. Service Identity / Workload Identity
4. Signed service tokens
5. API keys (limited/simple use cases)
```

Production microservices mein usually **strong service identity + TLS + authorization** combine kiya jata hai.

---

# 2. User Authentication vs Service Authentication

Ye distinction interview mein important hai.

### User → Service

```text
User
 ↓
Access Token
 ↓
API
```

Identity:

```text
user-123
```

### Service → Service

```text
Order Service
 ↓
Service Credential / Identity
 ↓
Payment Service
```

Identity:

```text
order-service
```

Service-to-service call mein user identity bhi separately propagate/delegate ki ja sakti hai when business authorization requires it.

---

# 3. Recommended Pattern — OAuth 2.0 Client Credentials

Machine-to-machine APIs ke liye **Client Credentials Grant** common pattern hai.

Flow:

```text
Order Service
      |
      | client credentials
      ↓
Authorization Server
      |
      | Access Token
      ↓
Order Service
      |
      | Authorization: Bearer <token>
      ↓
Payment Service
```

Order Service khud ko authenticate karke short-lived access token obtain karti hai.

---

# 4. Step-by-Step OAuth Client Credentials Flow

## Step 1 — Service Has an Identity

```text
Order Service
client_id = order-service
```

Secret/private credential securely managed hota hai.

❌ Source code mein hard-code nahi karna chahiye.

Use:

```text
Secret Manager
Vault
Cloud IAM / Workload Identity
Kubernetes Secret mechanisms
```

architecture ke according.

---

## Step 2 — Token Request

Order Service authorization server ko token request bhejti hai.

Conceptually:

```http
POST /oauth2/token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
```

Authentication mechanism authorization server ke configuration ke according hota hai.

---

# 5. Step 3 — Authorization Server Issues Token

Authorization server service identity validate karta hai.

Then access token issue hota hai:

```text
Order Service
      ↓
Access Token
```

Token mein scopes/audience jaise claims ho sakte hain:

```json
{
  "sub": "order-service",
  "aud": "payment-api",
  "scope": "payment:charge"
}
```

Exact claims identity provider par depend karte hain.

---

# 6. Step 4 — Service Calls Another Service

```http
POST /payments
Authorization: Bearer <service-access-token>
```

Payment Service token validate karegi.

---

# 7. Step 5 — Authentication + Authorization

Payment Service checks:

```text
Token signature
      ↓
Issuer
      ↓
Audience
      ↓
Expiry
      ↓
Scope
      ↓
Service identity
```

Then:

```text
order-service
      ↓
Has payment:charge scope?
      ↓
YES → Allow
NO  → Reject
```

**Authentication:** Service kaun hai?

**Authorization:** Service ko kya karne ki permission hai?

---

# 8. mTLS — Another Important Approach

**mTLS = Mutual TLS**.

Normal TLS mein:

```text
Client → verifies Server
```

mTLS mein:

```text
Client ↔ Server
```

Dono sides certificates ke through identity prove karti hain.

Flow:

```text
Order Service
   |
   | Client Certificate
   ↓
TLS Handshake
   ↑
   | Server Certificate
   |
Payment Service
```

Both sides authenticate each other.

---

# 9. mTLS vs OAuth Token

| Feature | mTLS | OAuth 2.0 Access Token |
|---|---|---|
| Identity | Certificate | Token claims |
| Layer | Transport | Application/API authorization |
| Mutual authentication | ✅ | Not inherently |
| Authorization scopes | Not naturally represented | ✅ |
| Rotation | Certificate rotation | Token/key rotation |
| Best use | Strong workload identity | API authorization |

Many production systems can use both:

```text
mTLS
  +
OAuth access token
```

mTLS establishes workload identity and encrypted channel; OAuth token can express API-level permissions.

---

# 10. Service Identity

Modern platforms often give workloads a first-class identity.

Example concept:

```text
Order Service
   ↓
Workload Identity
   ↓
order-service identity
```

Then infrastructure can issue credentials/certificates/tokens based on workload identity.

This is preferable to manually distributing long-lived secrets where possible.

---

# 11. API Key Approach

Simple approach:

```http
X-API-Key: abc123
```

Service validates key.

Advantages:

- Simple
- Easy to implement

Problems:

- Key rotation management
- Long-lived credential risk
- Limited identity/authorization semantics
- Harder at large scale

Therefore API keys may be acceptable for some controlled integrations, but are usually not the strongest general-purpose service-to-service identity mechanism.

---

# 12. Signed Service Tokens

A service can use signed tokens to prove identity.

```text
Order Service
    ↓
Signed Token
    ↓
Payment Service
    ↓
Verify Signature
```

Asymmetric signing can be useful:

```text
Private Key
    ↓
Sign

Public Key
    ↓
Verify
```

Private signing keys should never be distributed to every consumer.

---

# 13. TLS Is Not Optional

Authentication mechanism ke saath communication channel ko secure karna bhi important hai.

```text
Service A
   ↓ HTTPS/TLS
Service B
```

TLS provides encryption in transit and server authentication; mTLS additionally authenticates the client workload.

❌ Internal network ko automatically trusted assume nahi karna chahiye.

---

# 14. Service Authentication + Authorization

Suppose:

```text
Order Service
```

Payment API ko call karti hai.

Sirf authenticated hona enough nahi hai.

Payment Service policy:

```text
order-service
  → payment:charge ✅

notification-service
  → payment:charge ❌
```

This is **least privilege**.

---

# 15. Complete Production Architecture

```text
                         ┌─────────────────────┐
                         │ Authorization Server │
                         └──────────┬──────────┘
                                    │
                              Access Token
                                    │
                                    ↓
┌───────────────┐       TLS       ┌────────────────┐
│ Order Service │ ──────────────→ │ Payment Service│
│               │   Bearer Token  │                │
└───────────────┘                 └───────┬────────┘
                                          │
                                   Validate +
                                   Authorize
```

Alternative/combined:

```text
Order Service
     ↓
 mTLS identity
     +
 OAuth access token
     ↓
Payment Service
```

---

# 16. What If Order Service Token Is Stolen?

Possible protections:

```text
Short-lived access token
        ↓
Limited scopes
        ↓
Audience restriction
        ↓
TLS
        ↓
Credential rotation
        ↓
Monitoring
```

Example:

```text
Token:
aud = payment-api
scope = payment:charge
exp = short-lived
```

Token ko unrelated service par use nahi karna chahiye if audience validation is correctly enforced.

---

# 17. Token Audience Is Important

Suppose token intended for:

```text
aud = payment-api
```

Agar Inventory Service request receive karti hai:

```text
Inventory API
```

and audience validation requires `inventory-api`, token reject hona chahiye.

This prevents a valid token issued for one resource from being blindly accepted by another.

---

# 18. Service Credential Rotation

Long-lived credentials dangerous hain.

Production mein:

```text
Credential
   ↓
Rotate
   ↓
New Credential
   ↓
Old Credential Expire/Revoke
```

For certificates:

```text
Certificate Rotation
```

For OAuth:

```text
Short-lived Access Token
Refresh / Re-authenticate
```

Machine-to-machine token flows commonly obtain new access tokens rather than using a long-lived access token indefinitely.

---

# 19. Service-to-Service Call Through API Gateway

Internal calls ke liye har architecture mein API Gateway mandatory nahi hai.

Possible:

```text
Client
  ↓
API Gateway
  ↓
Order Service
  ↓
Payment Service
```

Order → Payment direct internal call ho sakti hai, with service authentication.

Or architecture may intentionally route internal traffic through a gateway/service mesh depending on operational requirements.

**Important:** Gateway routing and service identity are separate concerns.

---

# 20. Service Mesh

Service mesh environments mein identity/security infrastructure layer handle kar sakti hai.

Conceptually:

```text
Order Service
     |
   Sidecar
     |
   mTLS
     |
   Sidecar
     |
Payment Service
```

Mesh can provide features such as:

- Workload identity
- mTLS
- Certificate rotation
- Traffic policy
- Observability

Application-level authorization is still an architectural concern and should not be assumed solved merely because mTLS exists.

---

# 21. User Token vs Service Token

Suppose user request:

```text
User
 ↓
Order Service
 ↓
Payment Service
```

Two possible models:

### Model A — Propagate User Token

```text
Same user access token
      ↓
Payment Service
```

Useful when downstream service needs user context/authorization.

### Model B — Service Identity

```text
Order Service
      ↓
Service credential/token
      ↓
Payment Service
```

Useful when Payment Service only needs to know that a trusted Order Service is calling.

### Model C — Delegated Identity / Token Exchange

A service may obtain a token representing the required delegated permissions instead of blindly forwarding the original bearer token.

Choice depends on authorization requirements.

---

# 22. Common Mistakes

### ❌ Mistake 1
"Internal network secure hai, authentication ki need nahi."

Wrong.

### ❌ Mistake 2
"API Gateway authenticate kar raha hai, internal services don't need validation."

Too simplistic.

### ❌ Mistake 3
"API key is always best."

Not for large-scale workload identity/security.

### ❌ Mistake 4
"mTLS alone solves authorization."

mTLS authenticates workloads and secures transport; it does not automatically express all business/API permissions.

### ❌ Mistake 5
"Service token never expires."

Long-lived bearer credentials increase risk.

---

# 23. Interview Scenario

### Interviewer:

> Order Service needs to call Payment Service. How will you secure it?

### Strong Answer:

```text
1. Give Order Service a strong workload identity.
2. Use TLS for encrypted communication.
3. Obtain a short-lived access token using OAuth Client Credentials if OAuth is used.
4. Restrict token audience to Payment API.
5. Grant only required scope, e.g. payment:charge.
6. Payment Service validates signature, issuer, audience and expiry.
7. Payment Service checks authorization/scope.
8. Rotate credentials/keys and monitor failures.
```

For stronger workload identity:

```text
mTLS + OAuth
```

can be considered depending on architecture.

---

# 24. Interview-Ready Answer

> **"For service-to-service communication, I would give each service a strong identity and use TLS for secure communication. A common approach is OAuth 2.0 Client Credentials, where the calling service authenticates with an authorization server and receives a short-lived access token with a specific audience and scopes. The target service validates the token signature, issuer, audience and expiry, and then checks authorization. For stronger workload-level authentication, mTLS can be used so both services authenticate each other. In a production system I would also apply least privilege, credential or certificate rotation, monitoring and avoid relying only on API Gateway validation."**

---

# 25. 30-Second Hinglish Answer

> **"Service-to-service authentication mein har service ki ek strong identity hoti hai. Commonly OAuth Client Credentials use karke calling service short-lived access token leti hai aur target service ko Bearer token ke saath call karti hai. Target service signature, issuer, audience, expiry aur scopes validate karti hai. Strong workload authentication ke liye mTLS use kar sakte hain, jisme dono services certificates se authenticate hoti hain. Saath mein TLS, least privilege, credential rotation aur monitoring important hain."**

---

# 26. Memory Trick

```text
SERVICE IDENTITY
      ↓
TLS / mTLS
      ↓
TOKEN
      ↓
VALIDATE
      ↓
AUTHORIZE
      ↓
ALLOW / DENY
```

### One-line memory

**"Identify the service → Secure the channel → Validate the credential → Check permission."**

---

# 27. Follow-Up Questions

### Q. OAuth Client Credentials kab use karte hain?

Machine-to-machine/service-to-service access ke liye, jab user ki interactive login involved nahi hoti.

### Q. mTLS kya provide karta hai?

Encrypted TLS channel + mutual certificate-based workload authentication.

### Q. mTLS aur OAuth ko saath use kar sakte hain?

Yes. mTLS workload/channel identity ke liye aur OAuth token API-level authorization ke liye use ho sakta hai.

### Q. API Gateway authenticate kar raha hai to service ko dobara validate kyun karna chahiye?

Defense in depth ke liye; internal network aur downstream service ko independently protected rakhna useful hai.

### Q. API key use kar sakte hain?

Controlled/simple integrations mein possible hai, but large distributed systems mein stronger identity, rotation and authorization mechanisms usually preferred hote hain.

### Q. Service token mein audience kya hai?

Token kis resource/API ke liye intended hai. Resource server audience validate karke token misuse reduce kar sakta hai.

### Q. User JWT aur service token mein difference?

User JWT user identity/permissions represent kar sakta hai; service token workload/service identity represent karta hai. Exact model architecture par depend karta hai.

---

## Status

✅ **Q19 Solution Completed**

Next: **Q20 — How does a frontend application communicate with backend services in a microservices architecture?**
