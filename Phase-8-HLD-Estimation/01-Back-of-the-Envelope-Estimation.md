# Back-of-the-Envelope Estimation

## Topics
- Users
- DAU / MAU
- Requests per Second (QPS)
- Peak QPS
- Storage Estimation
- Bandwidth Estimation
- Cache Size Estimation
- Replication Overhead

## Interview Focus
- Estimate scale before architecture decisions and clearly state assumptions.

## Typical Flow
1. Estimate users and activity.
2. Convert daily traffic to average QPS.
3. Apply a peak multiplier.
4. Estimate storage and bandwidth.
5. Account for replication, indexing, and caching.
