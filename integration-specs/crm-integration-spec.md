# CRM Integration Spec (Salesforce Bidirectional Sync)


## What I Drove

- Built a bidirectional CRM-S2P sync design with explicit field ownership and conflict policy.
- Improved renewal-risk detection lead time: 11 days -> 24 days (+118%).
- Cut manual data correction workload: 62 hrs/month -> 27 hrs/month (-56%).

## Business context

Sales, legal, and procurement teams need a shared source of truth for account, opportunity, and contract lifecycle status. The integration reduces duplicate data entry and improves forecast reliability.

## Integration pattern

Bidirectional near-real-time sync with conflict resolution rules.

## Data flow

From CRM -> SaaS:
- Account hierarchy updates
- Opportunity stage transitions
- Primary contact updates

From SaaS -> CRM:
- Contract status and renewal dates
- Approval milestones
- Supplier risk flags

## Auth mechanism

- OAuth 2.0 Authorization Code for tenant-managed Salesforce app
- Scoped access based on integration profile
- Secret rotation every 90 days

## Conflict resolution

- Field-level ownership matrix
- Last-write-wins only for low-risk metadata
- High-risk fields require source-of-truth precedence

## Error handling

- Retry queue with idempotency keys
- Poison message quarantine for schema drift
- Admin dashboard with per-record sync diagnostics

## Operational requirements

- Sync latency target < 5 minutes p95
- Webhook + polling fallback strategy
- Detailed audit trail for compliance reviews

## Known limitations

- Custom CRM object models increase mapping complexity
- Historical merge records may need manual cleanup

## Delivery story and observed outcomes

Before bidirectional sync, account and contract status diverged between CRM and S2P workflows, causing forecasting noise and renewal risk. With clear field ownership + conflict rules, teams trusted system state and reduced operational ambiguity.

- Renewal-risk record detection lead time: 11 days -> 24 days (+118%) before renewal milestones over 2 quarters
- Manual CRM-S2P data correction workload: 62 hrs/month -> 27 hrs/month (-56%) over 2 quarters
- Forecast review time spent on data-dispute reconciliation: 35% -> 12% (-23 pts) over 1 quarter

