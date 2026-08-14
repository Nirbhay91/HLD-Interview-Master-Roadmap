# 04. Availability

## Definition

Availability is the proportion of time a system is operational and able to serve requests.

`Availability = Uptime / Total Observed Time`

## Availability Targets

Availability is often expressed using percentages or "nines".

- 99% ≈ 3.65 days downtime/year
- 99.9% ≈ 8.76 hours/year
- 99.99% ≈ 52.6 minutes/year
- 99.999% ≈ 5.26 minutes/year

These are approximate annual downtime budgets.

## How to Improve Availability

### Redundancy

Avoid a single critical instance.

`Load Balancer → Instance A + Instance B`

### Multi-AZ Deployment

Run critical components across independent availability zones so one zone failure does not take down the service.

### Failover

If the primary component fails, traffic can move to a healthy replica/secondary.

### Health Checks

Load balancers and orchestrators can stop routing traffic to unhealthy instances.

### Graceful Degradation

If recommendations fail, the checkout system may still work. Critical functionality should not always depend on non-critical functionality.

## Availability vs Reliability

Availability asks: **Can the system serve now?**

Reliability asks: **Does the system perform correctly and consistently over time?**

A system can be available but unreliable—for example, returning incorrect results.

## Interview-Ready Answer

> Availability measures whether a system is operational and able to serve requests. We improve it through redundancy, health checks, failover, multi-zone deployment, replication and graceful degradation. I would first identify critical dependencies and remove single points of failure.
