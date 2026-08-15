# URL Shortener — LLD

## 1. Objective

Design the low-level implementation of a URL Shortener service.

Example:

```text
Long URL:
https://example.com/products/category/item?id=12345

Short URL:
https://short.ly/aZ91xK
```

The HLD decides the services, database, cache and scaling strategy. This LLD focuses on the classes, interfaces, responsibilities and object interactions inside the URL Shortener service.

---

## 2. Core Functional Requirements

1. Create a short URL from a long URL.
2. Redirect a short code to the original URL.
3. Support optional custom aliases.
4. Validate the input URL.
5. Detect duplicate/custom-code conflicts.
6. Track basic metadata such as creation time and expiration.
7. Keep the design extensible for analytics and different code-generation strategies.

---

## 3. Main Use Cases

### Create Short URL

```text
Client
  ↓
UrlController
  ↓
UrlService
  ↓
CodeGenerator
  ↓
UrlRepository
```

### Redirect

```text
Client
  ↓
UrlController
  ↓
UrlService
  ↓
UrlRepository / Cache
  ↓
Original URL
```

---

## 4. Domain Model

### UrlMapping

Represents the relationship between a short code and the original URL.

```java
class UrlMapping {
    private String id;
    private String shortCode;
    private String originalUrl;
    private UrlStatus status;
    private Instant createdAt;
    private Instant expiresAt;
    private String userId;
}
```

### UrlStatus

```java
enum UrlStatus {
    ACTIVE,
    EXPIRED,
    DISABLED
}
```

---

## 5. Request / Response Objects

### CreateUrlRequest

```java
class CreateUrlRequest {
    private String originalUrl;
    private String customAlias;
    private Long expirationSeconds;
}
```

### CreateUrlResponse

```java
class CreateUrlResponse {
    private String shortUrl;
    private String shortCode;
    private Instant expiresAt;
}
```

Keeping API DTOs separate from domain entities avoids exposing internal persistence details.

---

## 6. Controller Layer

```java
class UrlController {

    private final UrlService urlService;

    public CreateUrlResponse createShortUrl(CreateUrlRequest request) {
        return urlService.createShortUrl(request);
    }

    public RedirectResponse redirect(String shortCode) {
        return urlService.resolve(shortCode);
    }
}
```

### Responsibilities

- Receive HTTP requests.
- Validate basic request format.
- Delegate business logic to `UrlService`.
- Convert domain results into API responses.

The controller should not contain code-generation, persistence or redirect business logic.

---

## 7. Service Layer

```java
interface UrlService {
    CreateUrlResponse createShortUrl(CreateUrlRequest request);
    RedirectResponse resolve(String shortCode);
}
```

Implementation:

```java
class UrlServiceImpl implements UrlService {

    private final UrlRepository urlRepository;
    private final CodeGenerator codeGenerator;
    private final UrlValidator urlValidator;
    private final UrlCache urlCache;

    @Override
    public CreateUrlResponse createShortUrl(CreateUrlRequest request) {
        urlValidator.validate(request.getOriginalUrl());

        String code = request.getCustomAlias() != null
                ? request.getCustomAlias()
                : codeGenerator.generate();

        // Check uniqueness and persist mapping.
        // Return the generated short URL.
        return null;
    }

    @Override
    public RedirectResponse resolve(String shortCode) {
        UrlMapping mapping = urlCache.get(shortCode);

        if (mapping == null) {
            mapping = urlRepository.findByShortCode(shortCode)
                    .orElseThrow(() -> new UrlNotFoundException(shortCode));
            urlCache.put(shortCode, mapping);
        }

        validateActive(mapping);
        return new RedirectResponse(mapping.getOriginalUrl());
    }
}
```

### Responsibilities

- Coordinate business logic.
- Select the code-generation strategy.
- Check URL validity and status.
- Coordinate repository and cache.
- Enforce business rules.

---

## 8. Code Generation Strategy

Do not hard-code code generation inside `UrlService`. Use the Strategy pattern.

```java
interface CodeGenerator {
    String generate();
}
```

Possible implementations:

```java
class RandomBase62CodeGenerator implements CodeGenerator {
    public String generate() {
        // Generate a collision-resistant Base62 code.
        return null;
    }
}
```

```java
class SequenceBasedCodeGenerator implements CodeGenerator {
    public String generate() {
        // Generate from a unique numeric sequence and encode using Base62.
        return null;
    }
}
```

### Why Strategy Pattern?

If the business later changes from random codes to sequence + Base62, the service layer does not need to change.

```text
UrlService
    |
    +---- CodeGenerator
             |
             +---- RandomBase62CodeGenerator
             +---- SequenceBasedCodeGenerator
```

---

## 9. Repository Layer

```java
interface UrlRepository {
    UrlMapping save(UrlMapping mapping);

    Optional<UrlMapping> findByShortCode(String shortCode);

    boolean existsByShortCode(String shortCode);

    void disable(String shortCode);
}
```

Implementation could use JPA/JDBC:

```java
class UrlRepositoryImpl implements UrlRepository {
    // Database implementation
}
```

### Why Interface?

- Dependency Inversion Principle.
- Easy unit testing with mocks.
- Database implementation can change without affecting business logic.

---

## 10. Cache Abstraction

```java
interface UrlCache {
    UrlMapping get(String shortCode);
    void put(String shortCode, UrlMapping mapping);
    void evict(String shortCode);
}
```

Implementation:

```java
class RedisUrlCache implements UrlCache {
    // Redis implementation
}
```

The LLD remains independent of Redis-specific APIs.

---

## 11. Validator

```java
interface UrlValidator {
    void validate(String url);
}
```

Implementation:

```java
class HttpUrlValidator implements UrlValidator {
    @Override
    public void validate(String url) {
        // Validate URL syntax and allowed schemes.
    }
}
```

Possible rules:

- Reject null/blank URLs.
- Allow only `http` and `https`.
- Apply maximum URL length.
- Reject malformed URLs.
- Apply application-specific security rules.

---

## 12. Exception Design

```java
class UrlNotFoundException extends RuntimeException {
    public UrlNotFoundException(String shortCode) {
        super("URL not found: " + shortCode);
    }
}
```

```java
class ShortCodeAlreadyExistsException extends RuntimeException {
}
```

```java
class InvalidUrlException extends RuntimeException {
}
```

```java
class UrlExpiredException extends RuntimeException {
}
```

A global exception handler at the API layer can translate these into appropriate HTTP responses.

---

## 13. Database Entity

A relational model could look like:

```text
url_mapping
-----------
id                PK
short_code        UNIQUE INDEX
original_url
status
user_id
created_at
expires_at
```

Important index:

```text
UNIQUE(short_code)
```

This is essential because the short code must uniquely identify one URL mapping.

---

## 14. Object Relationships

```text
UrlController
      |
      v
 UrlService
   /  |  \
  /   |   \
 v    v    v
Code  Repo Cache
Gen
  |
  +-- RandomBase62CodeGenerator
  +-- SequenceBasedCodeGenerator
```

Detailed relationship:

```text
UrlController --> UrlService
UrlService --> UrlRepository
UrlService --> CodeGenerator
UrlService --> UrlValidator
UrlService --> UrlCache
UrlRepository --> Database
UrlCache --> Redis
```

---

## 15. SOLID Principles Applied

### Single Responsibility

Each class has one primary responsibility:

- Controller → API handling
- Service → business logic
- Repository → persistence
- Generator → short-code generation
- Validator → validation
- Cache → caching

### Open/Closed

New code-generation algorithms can be added without modifying `UrlService`.

### Liskov Substitution

Any `CodeGenerator` implementation should work through the `CodeGenerator` interface.

### Interface Segregation

Small interfaces such as `CodeGenerator`, `UrlRepository`, and `UrlCache` avoid forcing implementations to depend on unrelated methods.

### Dependency Inversion

`UrlService` depends on abstractions rather than directly on Redis/JDBC implementation classes.

---

## 16. Design Patterns

### Strategy Pattern

Used for different short-code generation algorithms.

### Repository Pattern

Separates persistence from business logic.

### Factory Pattern — Optional

Useful if code-generator selection depends on configuration or runtime requirements.

### Dependency Injection

Use constructor injection:

```java
UrlServiceImpl(
    UrlRepository repository,
    CodeGenerator codeGenerator,
    UrlValidator validator,
    UrlCache cache
)
```

---

## 17. Create URL Flow

```text
POST /urls
      |
      v
UrlController
      |
      v
UrlService
      |
      +--> UrlValidator
      |
      +--> CodeGenerator
      |
      +--> UrlRepository
      |
      +--> UrlCache
      |
      v
CreateUrlResponse
```

### Important concurrency issue

Two requests must never successfully create the same custom alias.

Do not rely only on:

```java
if (!repository.existsByShortCode(code)) {
    repository.save(mapping);
}
```

because two concurrent requests can both observe `false`.

Instead, enforce uniqueness at the database level:

```text
UNIQUE(short_code)
```

and handle the duplicate-key exception safely.

---

## 18. Redirect Flow

```text
GET /{shortCode}
       |
       v
UrlController
       |
       v
UrlService
       |
       +----> Redis Cache
       |          |
       |        HIT ──────> Original URL
       |
       | MISS
       v
UrlRepository
       |
       v
Database
       |
       v
Cache + Original URL
```

This is a classic **cache-aside** flow.

---

## 19. Unit Testing Strategy

### UrlService tests

- Creates short URL successfully.
- Rejects invalid URL.
- Generates code when alias is absent.
- Uses custom alias when provided.
- Handles duplicate alias.
- Resolves existing short code.
- Throws `UrlNotFoundException` for unknown code.
- Rejects expired URL.

### CodeGenerator tests

- Generated code is non-empty.
- Generated code follows allowed character set.
- Collision handling is tested.

### Repository tests

- Save mapping.
- Find by short code.
- Unique constraint behavior.

---

## 20. Interview Questions

### Q1. Why separate Controller and Service?

To keep HTTP/API concerns separate from business logic and make the business layer independently testable.

### Q2. Why use an interface for CodeGenerator?

Because code generation is a replaceable behavior. Strategy lets us switch algorithms without modifying `UrlService`.

### Q3. Where should uniqueness be enforced?

At the database level using a unique constraint. Application-level checks alone are vulnerable to race conditions.

### Q4. Why is cache not directly used inside the controller?

Caching is part of the URL resolution business flow, so the service layer should coordinate cache and repository access.

### Q5. What happens if Redis is down?

The cache implementation should fail gracefully and the service should fall back to the repository/database, subject to the HLD availability and load constraints.

### Q6. Which design pattern is most important here?

Strategy for code generation, Repository for persistence abstraction, and Dependency Injection for loose coupling.

### Q7. How would you make code generation collision-safe?

Use a sufficiently large code space, check the database uniqueness constraint, and retry generation when a collision occurs.

---

## 21. 2-Minute LLD Interview Answer

> I would divide the URL Shortener into Controller, Service, Repository, Cache, Validator and Code Generator components. The controller handles HTTP requests, while the service contains the business logic. I would keep code generation behind a `CodeGenerator` interface so we can support random Base62 or sequence-based generation using the Strategy pattern. Persistence would be abstracted behind `UrlRepository`, and Redis behind `UrlCache`.
>
> For concurrency, I would enforce a unique constraint on `short_code` in the database instead of relying only on an application-level existence check. For redirects, I would use a cache-aside approach: first check Redis, then the database on a cache miss, and populate the cache after a successful lookup. I would use constructor-based dependency injection and keep each class focused on a single responsibility. This gives us loose coupling, testability and easy extensibility.

---

## 22. HLD vs LLD Boundary

### HLD

```text
Load Balancer
     ↓
API Gateway
     ↓
URL Service
     ↓
Redis + Database
```

### LLD

```text
UrlController
     ↓
UrlService
  /   |    \
 /    |     \
Repo Cache  CodeGenerator
             /        \
            /          \
      RandomBase62   SequenceBased
```

**Remember:** HLD defines the system components and their interactions. LLD defines how an individual component is implemented internally.
