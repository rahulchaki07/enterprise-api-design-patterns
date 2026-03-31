# Breaking Change Policy


## What I Drove

- Defined a strict breaking-change policy tied to release governance and customer communication timelines.
- Reduced production compatibility regressions through pre-release change classification.
- Improved migration accountability with measurable pre-sunset completion tracking.

## Policy intent

Protect integration stability while enabling responsible platform evolution.

## What is a breaking change

A change is breaking if a previously valid client integration can fail without modification.

### Explicitly breaking
- Removing or renaming endpoints, fields, enums
- Changing requiredness from optional to required
- Changing data types or semantic meaning
- Tightening validation rules without compatibility path

### Typically non-breaking
- Adding optional fields
- Adding new endpoints
- Expanding enum values when clients are documented to be forward-compatible

## Change control process

1. Author proposes change with compatibility assessment.
2. API governance review labels `breaking` or `non-breaking`.
3. If breaking, next major version required.
4. Publish migration guide and deprecation timeline before release.

## Required timelines

- Notice period: minimum 90 days for general changes
- Enterprise critical dependencies: 6-12 months depending on contractual obligations
- Emergency security exceptions require executive approval and customer communication plan

## Story behind the policy

This policy was shaped by incidents where seemingly small schema changes broke customer middleware in production. Formal classification and timelines shifted change management from ad-hoc announcements to accountable release governance.

## Governance outcome metrics

- Breaking changes detected pre-release vs post-release (counts + ratio per quarter)
- Customer migration completion before deprecation cutoff (baseline -> current)
- Compatibility-regression incident count (baseline -> current)
- Approved exception requests and ARR impact (baseline -> current)

