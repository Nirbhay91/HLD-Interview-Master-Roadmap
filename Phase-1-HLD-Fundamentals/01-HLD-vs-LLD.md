# 01. HLD vs LLD

## 1. What is HLD?

High-Level Design (HLD) describes the **system architecture and major building blocks** without going into implementation-level code details.

It answers:

- What components do we need?
- How do components communicate?
- Where does data live?
- How do we scale the system?
- How do we achieve availability and reliability?
- What are the important trade-offs?

Typical HLD components:

- Clients
- Load balancers
- API gateways
- Application/services
- Caches
- Databases
- Message brokers
- Object storage
- Search systems
- Monitoring and logging

## 2. What is LLD?

Low-Level Design (LLD) converts a component or service into an implementation-oriented design.

It answers:

- What classes/interfaces are required?
- What methods and responsibilities exist?
- What design patterns should be used?
- What are the object relationships?
- How is a particular module implemented?

## 3. HLD vs LLD

| HLD | LLD |
|---|---|
| System architecture | Component/class design |
| Focuses on services and infrastructure | Focuses on classes, methods and objects |
| Scalability and availability | Encapsulation and code structure |
| Data flow between components | Object interaction |
| Technology choices | Design patterns and interfaces |
| Usually diagrams | Usually class/sequence diagrams |

## 4. Example: URL Shortener

### HLD

`Client → Load Balancer → URL Service → Cache/Database`

Questions include:

- How do we generate unique short URLs?
- How do we handle millions of redirects?
- Should redirects use cache?
- How do we partition the database?

### LLD

Inside URL Service we may design:

- `UrlController`
- `UrlService`
- `UrlRepository`
- `UrlGenerator`
- `UrlMapping`

## 5. Interview Rule

When asked to design a system, do **not jump directly to classes or code**.

Start with:

1. Requirements
2. Scale/traffic assumptions
3. APIs
4. Data model
5. High-level architecture
6. Deep dive into critical components
7. Bottlenecks and failure handling
8. Trade-offs

## 6. Interview-Ready Answer

> HLD defines the overall architecture of a system—its major components, communication, data storage, scalability, availability and reliability. LLD goes one level deeper and defines the internal design of individual components using classes, interfaces, methods and design patterns. In a system-design interview, I would start with HLD and then deep-dive into LLD only for the critical component being discussed.

## 7. Common Follow-ups

- When would you move from monolith to microservices?
- How do you decide whether a component should be a separate service?
- How do HLD and LLD influence each other?
- What trade-offs matter more at HLD level?
