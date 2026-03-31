# Enterprise Webhook Design (Reliability-First)


## What I Drove

- Reframed webhooks as a reliability contract (fast ACK, async processing, controlled retries, DLQ).
- Raised successful delivery within retry horizon: 99.2% -> 99.95% over 2 quarters.
- Reduced missing-webhook support tickets by 55% and customer duplicate incidents by 70%.

## Product stance

Webhooks are a delivery contract, not a "best effort" notification. In enterprise integrations, missing an event can mean a failed order, an unapproved contract, or a broken audit trail.

My default: optimize for guaranteed, observable delivery even if it adds implementation complexity.

## Design principles

- ACK fast, process async
- Sign every payload
- Treat retries as first-class behavior
- Make replay safe through idempotency
- Expose delivery status to customers

## Delivery model

1. Producer emits event to durable queue.
2. Dispatcher sends HTTPS POST with signature headers.
3. Consumer returns immediate 2xx ACK within strict timeout.
4. Business processing happens out-of-band on consumer side.
5. Producer retries failed/timeout deliveries with backoff.
6. After max attempts, event is moved to dead-letter queue (DLQ).

## Payload contract

- Stable event envelope:
  - `event_id`
  - `event_type`
  - `occurred_at`
  - `tenant_id`
  - `version`
  - `data` object
- Event types are namespaced (`contract.approved.v1`)
- Backward-compatible additions only inside `data`

## Security requirements

- HMAC-SHA256 signature header with timestamp
- Signature verification examples in docs for major languages
- Reject old timestamps to prevent replay
- Optional mTLS for high-compliance tenants

## Retry and DLQ policy

Recommended initial strategy:
- Max attempts: 10
- Backoff: exponential with jitter
- Retry horizon: up to 24 hours
- DLQ retention: 14-30 days

PM decision tradeoff:
- More retries improve resilience but can create customer-side duplicate pressure and delayed failure discovery.

## Idempotency model for consumers

- `event_id` is globally unique for producer system
- Consumer stores processed IDs with TTL
- Re-delivery of same `event_id` is treated as success/no-op

## Observability and support

### Metrics
- Delivery success rate by event type and tenant
- Mean time to successful delivery
- Retry depth distribution
- DLQ volume trend
- Endpoint timeout rate

### Customer-facing tooling
- Webhook delivery logs in admin UI
- Retry button for failed events
- Endpoint validation test event
- Secret rotation workflow with overlap window

## Common failure patterns

- Customer endpoint does heavy work before ACK
- No signature verification implemented
- Event schema changes without versioning
- Missing runbook for DLQ replay

## Recommended SLA framing

- Event publish to first attempt: near real-time (seconds)
- Eventual delivery target: within retry horizon
- Incident communication includes impacted event types and replay guidance

## Practical checklist

- [ ] Signed payloads with documented verification
- [ ] Timeouts and retries configured per environment
- [ ] DLQ replay runbook tested quarterly
- [ ] Event schema versioning policy in place
- [ ] Delivery logs visible to support and customers
- [ ] Integration certification includes webhook chaos tests

## Story from the field

During a quarter-end billing cycle, one customer endpoint intermittently timed out under load. Because we had fast ACK + async processing + controlled retries, delivery eventually completed without data loss. The customer saw delay, not failure. Without this pattern, we would have had invoice reconciliation gaps and escalations across finance and IT.

## Outcomes and metrics I track

- End-to-end webhook delivery success (within retry horizon): 99.2% -> 99.95% (+0.75 pts) over 2 quarters
- Median redelivery completion time after transient failure: 11.4 min -> 3.8 min (-67%) over 2 quarters
- Customer-side duplicate-processing incidents: index 100 -> 30 (-70%) over 2 quarters after idempotency rollout
- "Missing webhook" support tickets: index 100 -> 45 (-55%) over 2 quarters

