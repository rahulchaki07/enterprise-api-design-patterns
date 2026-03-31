# API Review Checklist (PM Governance)


## What I Drove

- Operationalized API quality gates before engineering kickoff to reduce late-stage surprises.
- Shifted reviews toward measurable readiness: auth, reliability, lifecycle, and documentation completeness.
- Improved launch predictability by reducing downstream spec rework and QA blockers.

## Scope and purpose

Use this checklist before approving any new or changed external API contract.

## Contract quality

- [ ] Resource model is consistent with existing domain vocabulary
- [ ] Request/response schemas are explicit and validated
- [ ] Pagination, filtering, and sorting are defined for list endpoints
- [ ] Examples cover success and common error states

## Security and auth

- [ ] Auth pattern is standard and documented
- [ ] Scope model follows least-privilege design
- [ ] Tenant isolation rules are testable and enforced

## Reliability and operations

- [ ] Idempotency behavior defined for write endpoints
- [ ] Rate limits and headers are documented
- [ ] Retry and timeout behavior specified
- [ ] Monitoring and alert ownership assigned

## Lifecycle and communication

- [ ] Versioning strategy declared
- [ ] Breaking change assessment completed
- [ ] Deprecation path communicated with dates
- [ ] Migration guide drafted for affected integrators

## Why this checklist exists (story)

This checklist came from repeated late-stage surprises: missing pagination decisions, undocumented auth edge-cases, and unclear migration plans discovered after build started. Running this pre-implementation has consistently reduced rework and delayed launches.

## Impact metrics to monitor

- API rework rate after engineering kickoff (baseline -> current, % delta)
- Launch blockers discovered in final QA (baseline -> current)
- Average spec approval cycle time (days): baseline -> current
- Endpoints shipped with complete docs/examples: baseline -> current (pts delta)

