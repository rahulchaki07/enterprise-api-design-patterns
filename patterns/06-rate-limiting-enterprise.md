# Rate Limiting for Enterprise B2B APIs


## What I Drove

- Implemented tier-aware, tenant-level rate controls to protect shared performance.
- Improved peak p95 latency: 1.9s -> 1.2s (-37%) while traffic grew ~34%.
- Reduced noisy-neighbor incidents: 7/quarter -> 1/quarter (-86%).

## Why this matters

Rate limiting protects platform stability, but in B2B contexts it also shapes contract negotiations and customer satisfaction.

## PM stance

Use fair-use controls with transparent tiers, predictable burst behavior, and clear escalation paths for enterprise workloads.

## Recommended model

- Primary limit: token bucket per tenant + endpoint group
- Secondary limit: concurrent request cap for expensive operations
- Optional per-client credentials limit inside tenant

## Tiering example

| Plan tier | Sustained RPM | Burst | Notes |
|---|---:|---:|---|
| Standard | 600 | 1200 | Shared workloads |
| Enterprise | 3000 | 6000 | Dedicated priority lanes |
| Strategic | Custom | Custom | Contractual SLA-backed |

## Response headers

- `X-RateLimit-Limit`
- `X-RateLimit-Remaining`
- `X-RateLimit-Reset`
- `Retry-After` on `429`

## Retry guidance for customers

- Exponential backoff with jitter
- Honor `Retry-After`
- Avoid synchronized retries from middleware fleets

## Governance and ops

- Monitor top tenants by throttled volume
- Alert on sudden `429` spikes per endpoint
- Provide self-serve usage dashboard
- Document temporary limit increase request process

## Story from the field

One high-volume tenant periodically flooded expensive endpoints during nightly jobs, degrading performance for everyone. Introducing tier-aware rate limits + burst controls protected shared stability while giving that tenant a clear path to a contractual higher throughput tier.

## Outcomes and metrics I track

- Platform p95 latency at peak: 1.9s -> 1.2s (-37%) while peak traffic grew ~34% over 2 quarters
- Noisy-neighbor incidents: 7/quarter -> 1/quarter (-86%) over 2 quarters
- Unexplained `429` variance incidents: 10/quarter -> 2/quarter (-80%) after tiered limits launch
- Capacity-tier expansion conversion among eligible enterprise tenants: 18% -> 33% (+15 pts) over 2 quarters

