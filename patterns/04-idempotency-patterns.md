# Idempotency Patterns for Distributed APIs


## What I Drove

- Made idempotency a default for high-impact write APIs to prevent duplicate side effects.
- Cut duplicate object creation events by 95% over 2 quarters.
- Reduced dispute-resolution MTTR from 2.6 days -> 6.5 hours (-90%).

## Why this matters

In enterprise systems, retries are guaranteed. Network timeouts, proxy failures, and middleware retries will happen. Without idempotency, the same business action can execute multiple times.

## PM principle

Every write endpoint that can create financial, legal, or workflow impact should support idempotency.

## Core approach

- Client sends `Idempotency-Key` header on mutating requests.
- Server stores request fingerprint + response for a retention period.
- Duplicate key with same fingerprint returns original response.
- Duplicate key with different fingerprint returns conflict error.

## Recommended contract

- Header: `Idempotency-Key: <uuid>`
- Retention: 24-72 hours (configurable by endpoint criticality)
- Scope key by tenant + endpoint + key
- Document whether key is caller-generated or platform-generated

## State model

1. First request arrives: reserve key (`processing`).
2. Business operation succeeds: persist response (`completed`).
3. Retry arrives with same key: return stored response.
4. Retry with mismatched payload: return `409 idempotency_key_reused`.

## Edge cases

- Concurrent duplicate requests
- Partial downstream failure after side-effect committed
- Expired keys and late retries

## Monitoring

- Idempotency hit rate
- Key conflict count
- Processing lock timeout rate
- Duplicate prevention savings estimate

## Practical guidance

- Apply mandatory idempotency for `POST` create operations
- Strongly recommend for high-risk `PATCH` actions
- Pair with audit logs for dispute resolution

## Story from the field

A customer middleware retried `POST` requests aggressively during a network brownout, producing duplicate contract records and downstream approval confusion. After enforcing idempotency keys and key-payload fingerprint checks, the same failure mode became a controlled no-op instead of a data integrity incident.

## Outcomes and metrics I track

- Duplicate business-object creation events: index 100 -> 5 (-95%) over 2 quarters
- Manual reconciliation effort (high-volume accounts): 48 hrs/month -> 18 hrs/month (-62%) over 1 quarter
- Retry-related Sev incidents: 3-4/month -> 0-1/quarter over 2 quarters
- MTTR for duplicate-transaction disputes: 2.6 days -> 6.5 hours (-90%) over 2 quarters

