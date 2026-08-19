# Q18 — Explain JWT Authentication Flow

> **Interview Question:** Explain JWT authentication flow.

## 1. Simple Hinglish Explanation

**JWT (JSON Web Token)** ek compact token format hai jo client ko authentication ke baad server ko apni identity aur claims prove karne ke liye use karne diya ja sakta hai.

Typical flow:

```text
Client
  |
  | 1. Login credentials
  ↓
Auth Server / Authentication Service
  |
  | 2. Validate credentials
  ↓
 3. Issue Access Token
  |
  ↓
Client
  |
  | 4. Authorization: Bearer <token>
  ↓
API Gateway / Resource Server
  |
  | 5. Validate token
  ↓
Protected API
  |
  ↓
Response
```

Important distinction:

- **Authentication** = user kaun hai?
- **Authorization** = user ko kya karne ki permission hai?

JWT mainly identity/claims ko securely carry karne ka token format hai; authorization decision resource server ki policy par depend karta hai.

---

# 2. Complete JWT Flow

## Step 1 — User Login

Client credentials bhejta hai:

```http
POST /auth/login
Content-Type: application/json

{
  "username": "nirbhay",
  "password": "******"
}
```

Authentication server credentials validate karta hai.

```text
Client
  ↓
Auth Service
  ↓
User Store / Identity Provider
```

Password ko plain text mein store nahi karna chahiye; secure password hashing use honi chahiye.

---

# 3. Step 2 — Token Issue

Credentials valid hain to authorization server/access-token issuer access token issue karta hai.

Conceptually:

```text
Access Token
     ↓
Client
```

JWT ke claims mein example:

```json
{
  "sub": "user-123",
  "iss": "https://auth.example.com",
  "aud": "orders-api",
  "exp": 1760000000,
  "scope": "orders:read orders:write"
}
```

Exact claims application aur identity provider par depend karte hain.

---

# 4. JWT Structure

JWT commonly 3 Base64URL-encoded parts se bana hota hai:

```text
HEADER.PAYLOAD.SIGNATURE
```

Example shape:

```text
xxxxx.yyyyy.zzzzz
```

## Header

Usually algorithm/type information:

```json
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "key-1"
}
```

## Payload

Claims:

```json
{
  "sub": "user-123",
  "iss": "auth-service",
  "aud": "orders-api",
  "exp": 1760000000
}
```

## Signature

Signature token ki integrity/authenticity verify karne mein help karti hai.

Conceptually:

```text
sign(
  base64url(header) + "." + base64url(payload),
  signing key
)
```

**Important:** JWT payload encrypted nahi hota by default. Base64URL encoding encryption nahi hai. Sensitive information payload mein nahi rakhni chahiye unless a suitable encryption mechanism such as JWE is deliberately used.

---

# 5. Step 3 — Client Stores/Handles Token

Client access token ko safely handle karta hai and protected requests mein send karta hai.

Browser-based applications ke storage choice security architecture par depend karti hai. XSS/CSRF risks ko consider karna important hai.

A common server-managed browser session approach is:

```text
HttpOnly + Secure + SameSite Cookie
```

For APIs, bearer access tokens are commonly sent using the Authorization header.

---

# 6. Step 4 — Client Calls Protected API

```http
GET /orders
Authorization: Bearer <access-token>
```

Flow:

```text
Client
  ↓
API Gateway
  ↓
Order Service
```

---

# 7. Step 5 — Token Validation

Gateway or resource service token validate karta hai.

Important checks:

```text
1. Signature
2. Token expiry (exp)
3. Issuer (iss)
4. Audience (aud)
5. Required scopes/roles
6. Token type / algorithm policy
```

Signature valid hona alone enough nahi hai.

Example:

```text
Signature valid ✅
exp valid ✅
iss valid ✅
aud valid ❌
```

Token reject hona chahiye if audience policy require karti hai.

---

# 8. Signature Verification

Two common approaches:

### Symmetric

```text
Same secret
   ↓
Sign + Verify
```

Example family:

```text
HS256
```

### Asymmetric

```text
Private Key
   ↓
Sign

Public Key
   ↓
Verify
```

Example:

```text
RS256
```

Microservices environments mein asymmetric signing useful ho sakta hai because resource services ko verification ke liye public key deni hoti hai, signing private key nahi.

---

# 9. Step 6 — Authorization

Token valid hai, but ab check karna hai:

> Kya user ko ye operation perform karne ki permission hai?

Example:

```text
GET /orders
scope = orders:read
```

Allowed.

But:

```text
DELETE /orders/123
```

requires:

```text
orders:delete
```

Agar permission nahi hai:

```text
403 Forbidden
```

---

# 10. Authentication vs Authorization

| Concept | Meaning | Example |
|---|---|---|
| Authentication | Who are you? | Valid JWT for user-123 |
| Authorization | What can you do? | `orders:write` |

Typical result:

```text
No/invalid token → 401 Unauthorized
Valid token but insufficient permission → 403 Forbidden
```

---

# 11. 401 vs 403 — Interview Favorite

### 401 Unauthorized
Authentication missing/invalid/failed.

Examples:

```text
Missing token
Expired token
Invalid signature
Invalid issuer/audience
```

### 403 Forbidden
Authentication is accepted, but authorization fails.

```text
Token valid
Permission insufficient
```

Memory:

```text
401 → Identity problem
403 → Permission problem
```

---

# 12. Access Token vs Refresh Token

Production systems often separate these.

## Access Token

Used for API access.

Usually short-lived.

```text
Client
 ↓
API
```

## Refresh Token

Used to obtain a new access token when the access token expires.

```text
Refresh Token
     ↓
Auth Server
     ↓
New Access Token
```

Refresh token ko access token se higher sensitivity ke saath protect karna chahiye.

---

# 13. Refresh Flow

```text
Client
  |
  | Access Token expired
  ↓
Auth Server
  |
  | Refresh Token
  ↓
Validate refresh token
  |
  ↓
Issue new Access Token
  |
  ↓
Client
```

Depending on the authorization system, refresh token rotation and revocation can be used to reduce replay risk.

---

# 14. JWT Is Usually Stateless

JWT-based API authentication can avoid storing a server-side session for every access-token request.

```text
Request
  ↓
JWT
  ↓
Validate locally
```

But this does **not** mean the entire authentication system is magically state-free.

Refresh-token state, revocation, key management, session policies and identity-provider state may still exist.

---

# 15. JWT vs Session

| Feature | JWT Access Token | Server Session |
|---|---|---|
| Client sends | Token | Session ID/cookie |
| Server lookup | Can validate token locally | Usually session store lookup |
| Horizontal scaling | Convenient | Requires shared/session strategy |
| Immediate revocation | More involved | Usually straightforward |
| Token size | Can be larger | Small session ID |
| Main risk | Token theft/replay | Session theft/fixation |

Neither is automatically more secure. Security depends on implementation and threat model.

---

# 16. JWT in Microservices

Typical architecture:

```text
                    Auth Server
                        |
                        | Access Token
                        ↓
Client → API Gateway → Service A
                         |
                         ├→ Service B
                         └→ Service C
```

Resource services can validate the access token according to their own trust and authorization requirements.

**Important:** Gateway validation alone should not automatically mean internal services trust every request. Service-to-service authentication and authorization should be designed explicitly.

---

# 17. Token Propagation

Suppose:

```text
Client
  ↓ JWT
Gateway
  ↓
Order Service
  ↓
Payment Service
```

Internal call mein same end-user token propagate karna possible hai, but blindly forwarding bearer tokens is not always the best design.

Depending on the architecture, service-to-service credentials, token exchange, or delegated authorization may be more appropriate.

---

# 18. JWT Security Best Practices

### 1. Use HTTPS

Bearer token network par plain HTTP se nahi bhejna chahiye.

```text
HTTPS / TLS
```

### 2. Short-lived access tokens

Token theft ka exposure window reduce hota hai.

### 3. Validate all important claims

At minimum, architecture ke according:

```text
iss
sub
aud
exp
```

and required scopes/roles.

### 4. Do not trust arbitrary algorithms

Accepted algorithms explicitly configure karo.

### 5. Protect signing keys

Private keys/secrets source code mein hard-code nahi karne chahiye.

### 6. Key rotation

Signing keys periodically rotate karo and `kid`/JWKS strategy use kar sakte ho.

### 7. Minimize claims

JWT ko unnecessarily large ya sensitive data ka container mat banao.

---

# 19. Token Revocation Problem

JWT access token stateless hone ki wajah se ek challenge hota hai:

```text
Token issued
   ↓
User compromised
   ↓
Token still valid until expiry
```

Possible approaches:

- Short-lived access tokens
- Refresh-token revocation
- Refresh-token rotation
- Centralized deny-list where justified
- Key rotation for broad emergency invalidation
- Risk-based/session controls

Trade-off samajhna important hai: aggressive revocation state add kar sakta hai.

---

# 20. JWT Logout

Simple logout ka meaning client side token discard karna ho sakta hai, but if a bearer access token is already stolen and still valid, client logout alone necessarily invalidates that token.

Stronger logout/revocation design may involve refresh-token revocation and server-side session/token state.

---

# 21. Spring Boot High-Level Flow

Spring Security based resource server ka conceptual flow:

```text
HTTP Request
     ↓
Bearer Token Filter
     ↓
JWT Decoder / Authentication Provider
     ↓
Signature + Claims Validation
     ↓
Authentication Created
     ↓
Authorization Rules
     ↓
Controller
```

Example configuration conceptually:

```java
http
    .authorizeHttpRequests(auth -> auth
        .requestMatchers("/public/**").permitAll()
        .requestMatchers("/orders/**").hasAuthority("SCOPE_orders:read")
        .anyRequest().authenticated()
    )
    .oauth2ResourceServer(oauth2 -> oauth2.jwt());
```

Exact configuration depends on Spring Security version and identity-provider setup.

---

# 22. Complete Production Flow

```text
                   ┌──────────────────┐
                   │  Identity/Auth   │
                   │     Server       │
                   └────────┬─────────┘
                            │
                      Access Token
                            │
                            ↓
┌──────────┐        ┌────────────────┐
│  Client  │ ─────→ │  API Gateway   │
└──────────┘  JWT   └───────┬────────┘
                            │
                     Validate / Route
                            │
                            ↓
                  ┌──────────────────┐
                  │  Order Service   │
                  └────────┬─────────┘
                           │
                     Authorization
                           │
                           ↓
                       Order DB
```

For internal service calls:

```text
Order Service
     ↓
Service Identity / Delegated Token
     ↓
Payment Service
     ↓
Validate + Authorize
```

---

# 23. Common Mistakes

### ❌ Mistake 1
"JWT is encrypted."

Wrong by default.

JWT is typically signed; JWS payload is encoded, not encrypted.

### ❌ Mistake 2
"JWT validation means user is authorized for everything."

Wrong.

Authentication and authorization are separate.

### ❌ Mistake 3
"API Gateway validates JWT, so internal services don't need security."

Too simplistic.

### ❌ Mistake 4
"JWT can never be revoked."

Too absolute. Revocation is more involved than server-side sessions, but systems can use short expiry, refresh-token revocation/rotation, deny lists or key rotation depending on requirements.

### ❌ Mistake 5
"Just put user password inside JWT."

Never do this.

---

# 24. Interview Scenario

### Interviewer:

> User logs in successfully. How does the request reach a protected microservice?

### Strong Flow:

```text
1. User sends credentials
2. Auth Server validates credentials
3. Access token is issued
4. Client sends Bearer token
5. Gateway/resource server validates token
6. Signature + claims are checked
7. Authorization policy is evaluated
8. Request reaches protected endpoint
9. Service returns response
```

If token invalid:

```text
401
```

If token valid but permission insufficient:

```text
403
```

---

# 25. Interview-Ready Answer

> **"In a typical JWT authentication flow, the client first authenticates with an authorization server. After successful authentication, the server issues an access token, often a signed JWT. The client sends that token with protected API requests using the Bearer Authorization header. The API gateway or resource server validates the token signature and important claims such as issuer, audience and expiry, and then applies authorization rules such as scopes or roles. If the token is invalid, the request is rejected with 401; if the token is valid but the user lacks permission, it is rejected with 403. In a microservices environment, service-to-service authentication and authorization should also be explicitly designed rather than relying only on gateway validation."**

---

# 26. 30-Second Hinglish Answer

> **"JWT flow mein user pehle Auth Server par login karta hai. Credentials valid hone par Auth Server signed access token issue karta hai. Client har protected API request ke saath `Authorization: Bearer <token>` bhejta hai. Gateway ya resource service signature, expiry, issuer, audience aur required scope/role validate karti hai. Valid token aur permission hone par request service tak jaati hai. Invalid authentication par 401 aur insufficient permission par 403 milta hai. Microservices mein internal service-to-service security bhi separately design karni chahiye."**

---

# 27. Memory Trick

```text
LOGIN
  ↓
AUTHENTICATE
  ↓
ISSUE JWT
  ↓
SEND BEARER TOKEN
  ↓
VALIDATE
  ↓
AUTHORIZE
  ↓
ACCESS API
```

### One-line memory

**"Login → Token → Send → Validate → Authorize → API."**

---

# 28. Follow-Up Questions

### Q. What are the three parts of JWT?

Header, Payload and Signature.

### Q. Is JWT encrypted?

Not by default. A signed JWT provides integrity/authenticity; encryption requires an encryption mechanism such as JWE.

### Q. Why use refresh token?

To obtain a new access token without asking the user to repeatedly provide credentials.

### Q. What is the difference between 401 and 403?

401 means authentication failed/missing; 403 means authentication succeeded but authorization failed.

### Q. What happens if JWT expires?

The access request is rejected; the client may obtain a new access token using the refresh flow if the refresh token/session is still valid.

### Q. Where should JWT be validated?

At the gateway and/or resource services according to the security architecture. Critical services should not blindly trust an unverified identity assertion from another component.

### Q. Why use RS256 instead of HS256?

Asymmetric signing can allow many resource services to verify tokens using a public key without distributing the signing private key.

### Q. Can we revoke a JWT immediately?

Not inherently through the token itself. You need an explicit revocation strategy such as short-lived access tokens, refresh-token revocation, deny lists, or key rotation depending on the scope of revocation required.

---

## Status

✅ **Q18 Solution Completed**

Next: **Q19 — How do services authenticate with each other?**
