# 06. Maintainability

## Definition

Maintainability is how easily a system can be understood, changed, tested, operated and evolved over time.

## Why It Matters

Production systems live for years. A design that is fast but difficult to change can become expensive and risky.

## Important Practices

- Clear service boundaries
- Modular architecture
- Well-defined APIs
- Automated tests
- Documentation
- Consistent coding standards
- Observability
- Backward-compatible API evolution
- Infrastructure/configuration as code
- Automated deployment

## Coupling and Cohesion

**Low coupling:** components depend on each other as little as practical.

**High cohesion:** a component has a focused responsibility.

Good service boundaries generally aim for low coupling and high cohesion.

## Maintainability vs Overengineering

Do not introduce microservices, queues or complex abstractions only because they are theoretically scalable.

Choose the simplest architecture that satisfies the current requirements and leaves a practical path for growth.

## Interview-Ready Answer

> Maintainability means the system can be safely understood, modified, tested and operated as it evolves. I focus on clear boundaries, high cohesion, low coupling, stable contracts, automation, observability and documentation while avoiding unnecessary complexity.
