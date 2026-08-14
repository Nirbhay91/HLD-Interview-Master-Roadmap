# 02. Functional vs Non-Functional Requirements

## Functional Requirements

Functional requirements describe **what the system must do**.

Examples:

- User can create an account.
- User can upload a file.
- User can send a message.
- User can search products.
- User can place an order.

## Non-Functional Requirements

Non-functional requirements describe **how well the system must operate**.

Common NFRs:

- Scalability
- Availability
- Reliability
- Performance
- Latency
- Throughput
- Security
- Durability
- Maintainability
- Observability

## Why requirements come first

Architecture depends on requirements. A system serving 1,000 requests/day can have a very different architecture from one serving 100 million requests/day.

Before drawing architecture, clarify:

1. Functional scope
2. Users/traffic
3. Read/write ratio
4. Data size and growth
5. Latency expectations
6. Availability target
7. Consistency requirements
8. Security/compliance requirements

## Example: E-commerce

Functional:

- Browse products
- Add to cart
- Checkout
- Make payment
- Track order

Non-functional:

- Checkout should remain available during traffic spikes.
- Product reads should be low latency.
- Payment operations must be reliable and idempotent.
- Order data must not be lost.

## Interview Tip

Do not blindly ask for every possible requirement. Identify the requirements that **change the architecture** and confirm them first.

## Interview-Ready Answer

> Functional requirements define the capabilities the system provides, while non-functional requirements define quality attributes and operational constraints such as latency, scalability, availability, reliability and security. In HLD, I clarify both because NFRs often determine the architecture and technology choices.
