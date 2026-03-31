# enterprise-api-design-patterns


## What I Drove

- Built a PM-led enterprise API playbook focused on decisions that move delivery, risk, and revenue outcomes.
- Captured reusable patterns spanning auth, reliability, versioning, governance, and cross-system integrations.
- Anchored the portfolio in measurable impact (go-live success, incident reduction, migration completion, onboarding speed).

**API design specs, integration patterns, and governance frameworks for enterprise SaaS platforms**

> This repo documents how I think about API architecture as a Product Manager — not as implementation code, but as product decisions that have long-term platform consequences.

---

## Why this exists

API design is a PM responsibility that gets under-invested. Most API decisions get made by engineering in isolation, without considering:

- What does the developer experience (DX) feel like for integration partners?
- How does this API design affect our ability to version and evolve the platform?
- What security and governance requirements will enterprise customers impose?
- How does the integration model affect support burden and implementation costs?

This repo is a reference for those decisions — patterns I've encountered, specs I've written, and frameworks I use when evaluating API and integration design choices.

---

## Contents

```
enterprise-api-design-patterns/
├── README.md
├── patterns/
│   ├── 01-auth-oauth-enterprise.md      # OAuth 2.0 flows for enterprise SaaS
│   ├── 02-webhook-design.md             # Reliable webhook architecture
│   ├── 03-api-versioning-strategy.md    # Versioning approaches + tradeoffs
│   ├── 04-idempotency-patterns.md       # Handling retries in distributed systems
│   ├── 05-error-handling-standards.md   # Error taxonomy and response design
│   └── 06-rate-limiting-enterprise.md   # Rate limiting for B2B API products
├── integration-specs/
│   ├── erp-integration-spec.md          # SAP / Oracle integration pattern
│   ├── crm-integration-spec.md          # Salesforce bidirectional sync pattern
│   └── middleware-evaluation.md         # MuleSoft vs Apigee vs DIY tradeoffs
├── governance/
│   ├── api-review-checklist.md          # PM checklist for API spec review
│   ├── breaking-change-policy.md        # What constitutes a breaking change
│   └── deprecation-playbook.md          # How to deprecate an API endpoint
└── diagrams/
    ├── oauth-flow-enterprise.png
    ├── webhook-retry-pattern.png
    └── bidirectional-sync-architecture.png
```

---

## Core Patterns (Quick Reference)

### Auth: OAuth 2.0 for Enterprise SaaS

The pattern I've used most in enterprise integrations. Key decisions that matter at the product level:

**Token type selection:**

| Scenario | Token type | Why |
|----------|-----------|-----|
| User-context API calls | Authorization Code flow | User grants delegated access |
| Server-to-server integration | Client Credentials flow | No user context needed |
| Embedded integrations (ERP) | Authorization Code + PKCE | Secure for server environments |

**Enterprise-specific considerations:**
- Tenant isolation: each enterprise customer should have scoped tokens — one customer's token must never access another customer's data
- Token refresh strategy: enterprise customers often have integration middleware that doesn't handle refresh gracefully — build in clear expiry signaling
- Scopes: design scopes at the resource level, not the operation level — `contracts:read` not `get_contracts`

### Webhook Design: Reliability-first

Most webhook implementations are too optimistic. Enterprise systems need:

```
Producer → [signed payload] → Consumer endpoint
                                      │
                    ┌─────────────────┘
                    │
              ┌─────▼──────┐
              │  Immediate  │  Return 200 immediately, never block
              │    ACK      │  on business logic processing
              └─────┬──────┘
                    │
              ┌─────▼──────┐
              │   Async     │  Process in background queue
              │  processor  │  This is where your business logic lives
              └─────┬──────┘
                    │
              ┌─────▼──────┐
              │   Retry     │  Exponential backoff: 1s, 2s, 4s, 8s...
              │   policy    │  Dead letter queue after N failures
              └─────────────┘
```

PM decision: what's your retry limit and dead-letter strategy? Enterprise customers need guaranteed delivery SLAs.

### API Versioning: The PM Perspective

I've seen three versioning strategies in enterprise SaaS. My view on each:

**URL versioning** (`/v1/contracts`, `/v2/contracts`):
- Simple for developers and documentation
- Encourages abandoning old versions (good and bad)
- What I'd choose for external APIs that external customers integrate against

**Header versioning** (`API-Version: 2024-01-01`):
- Stripe's approach — elegant and date-based
- Harder to discover, but excellent for frequent iteration
- What I'd consider for internal platform APIs

**Query param versioning** (`?version=2`):
- Avoid. Leaks into logs, breaks caching, feels second-class

**Breaking change definition** (the hard PM question):
See [`governance/breaking-change-policy.md`](./governance/breaking-change-policy.md) for the full framework. Short version: adding fields is safe, removing or renaming is a breaking change, changing types is always breaking.

---

## Integration Spec Format

I write integration specs in this format — it captures the decisions that matter for product and engineering alignment:

```markdown
# [System] Integration Spec

## Business context
Why are we integrating? What user problem does this solve?

## Integration pattern
Bidirectional sync / One-way push / Event-driven / Polling

## Data flow
What data moves in which direction, and when?

## Auth mechanism
How does authentication work across system boundaries?

## Error handling
What happens when the integration fails? Who gets notified? What's the retry policy?

## Operational requirements
Monitoring, alerting, SLA commitments

## Known limitations
What can't this integration do? What tradeoffs did we accept?
```

---

## The PM Checklist for API Review

Before signing off on any API spec:

- [ ] Is auth using a standard pattern (OAuth 2.0, API keys with rotation)?
- [ ] Are error codes consistent with existing endpoints?
- [ ] Is pagination designed for large result sets?
- [ ] Is the versioning strategy defined and followed?
- [ ] What's the rate limit and is it documented?
- [ ] Is the developer documentation complete before engineering starts?
- [ ] Have integration partners reviewed the spec?
- [ ] Is there a deprecation plan for endpoints this replaces?

---

## Background

I built these patterns working in enterprise integration roles at TCS (MuleSoft, Apigee, IBM API Connect) and as a Product Manager at Avasant where I led the design of the integration architecture for an enterprise S2P/CLM platform integrating with SAP, Salesforce, and ServiceNow.

The goal here isn't to teach API design — it's to document how a PM thinks about API decisions differently from an engineer.

---

## Portfolio Lens: Outcomes I Optimize For

These patterns are not theoretical templates; they come from enterprise delivery constraints where product decisions have measurable operational impact. The metrics I optimize for across integrations:

- First-attempt integration go-live success (baseline -> current, pts change, quarterly)
- Time-to-onboard for enterprise customers/partners (baseline -> current, days)
- Sev incident volume tied to auth/retries/schema changes (baseline -> current, % delta)
- Support tickets caused by unclear API contracts (baseline -> current, % delta)
- Migration completion before version sunset/deprecation deadlines (baseline -> current, pts delta)

When these metrics improve, revenue realization gets faster, implementation risk drops, and trust with enterprise architecture teams compounds.

