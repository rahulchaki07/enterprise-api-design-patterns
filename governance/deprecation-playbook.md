# API Deprecation Playbook


## What I Drove

- Turned deprecation into a managed program with explicit phases, ownership, and telemetry.
- Improved migration coverage by ARR segment before cutoff dates.
- Reduced sunset risk through staged enforcement and rollback readiness.

## Objective

Retire legacy APIs safely without disrupting customer operations.

## Phased approach

### Phase 1: Announce
- Publish deprecation notice with rationale and timeline
- Provide successor API and migration guide
- Notify impacted tenants and integration partners

### Phase 2: Observe
- Add usage telemetry on deprecated endpoints
- Segment usage by tenant criticality and revenue impact
- Run office hours/support outreach for high-impact customers

### Phase 3: Nudge
- Add response headers warning of sunset date
- Display admin console alerts for tenant owners
- Offer sandbox validation for migrated integrations

### Phase 4: Enforce
- Soft-disable in non-production environments first
- Final production sunset with monitored rollback window
- Keep post-sunset incident response team on standby

## Communication artifacts

- Changelog entry
- Migration checklist
- FAQ for common migration blockers
- Escalation path for contractual exceptions

## Success criteria

- >95% traffic migrated before sunset date
- Zero Sev-1 incidents attributable to deprecation
- No unmanaged strategic accounts at cutoff

## Story behind this playbook

Deprecated endpoints often linger indefinitely unless ownership and timelines are explicit. This playbook turned deprecation from an email announcement into an execution program with measurable progress and risk control.

## Program metrics

- Weekly deprecated-endpoint traffic trend (baseline -> current, % decline)
- Tenant migration coverage by ARR segment (Enterprise/Mid-market/SMB, baseline -> current)
- Time from deprecation notice to 80% migration (baseline -> current)
- Post-sunset rollback events (baseline -> current)

