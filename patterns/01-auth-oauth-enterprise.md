# OAuth 2.0 for Enterprise SaaS APIs


## What I Drove

- Standardized enterprise OAuth model across tenant-scoped integrations to reduce security and delivery risk.
- Cut security review cycle time: 6.0 weeks -> 2.5 weeks (-58%) over 2 quarters.
- Improved first-attempt go-live success: 62% -> 88% (+26 pts) while reducing auth Sev-2 volume by 40%.

## Why this pattern matters

In enterprise SaaS, authentication design is not just a security decision; it is a go-to-market decision. A weak auth model slows security reviews, blocks procurement, and increases implementation risk for customers integrating through middleware like MuleSoft, Boomi, or custom iPaaS stacks.

My point of view: choose boring, standard OAuth patterns and make tenant isolation non-negotiable.

## PM goals and non-goals

### Goals
- Support both user-context and machine-context integrations
- Pass enterprise security assessments quickly
- Keep onboarding friction low for partner developers
- Make credential rotation and incident response operationally simple

### Non-goals
- Inventing proprietary auth protocols
- Supporting legacy grant types (implicit/password grants)
- Treating auth as a one-time implementation detail

## Decision framework

### 1) Select grant type by integration context

| Integration scenario | Recommended grant | Why this is the right tradeoff |
|---|---|---|
| User acting in UI + API | Authorization Code + PKCE | User consent + modern security defaults |
| Backend service to backend API | Client Credentials | Clear ownership, no user dependency |
| ERP/CRM connector managed by admins | Authorization Code + PKCE + long-lived refresh tokens | Strong security with manageable operations |
| Internal service mesh | mTLS + JWT assertions (if needed) | Performance + service identity controls |

### 2) Scope design principles

- Use resource-level scopes (`contracts:read`, `suppliers:write`)
- Keep scope vocabulary stable to avoid backward compatibility issues
- Avoid operation-specific scope sprawl (`get_contracts`, `list_contracts`)
- Define default scopes for least privilege

### 3) Tenant isolation rules

- Every token carries tenant context claims
- Authorization checks always validate both subject and tenant
- No cross-tenant admin bypasses in shared APIs
- Audit logs include tenant ID, client ID, and scope set

## Reference architecture

1. Integration admin creates OAuth client in tenant admin console.
2. Client credentials or consent flow issued against tenant-scoped authorization server.
3. Access token includes:
   - `iss`, `aud`, `sub`
   - `tenant_id`
   - `scope`
   - `exp`, `iat`, `jti`
4. API gateway validates signature, expiry, audience, and revocation status.
5. API service enforces scope + tenant boundary before any data access.

## Security controls expected by enterprise buyers

- PKCE required for public clients
- Rotating refresh tokens
- Secret rotation policy and API endpoint for key roll
- IP allowlists for high-risk tenants (optional feature tier)
- Signed JWT access tokens with key rotation (`kid`, JWKS endpoint)
- Replay protection for sensitive operations (nonce/idempotency key pairing)

## Operational playbook

### Token lifetime policy
- Access tokens: 10-30 minutes
- Refresh tokens: 30-90 days with rotation
- Emergency revocation: under 5 minutes propagation target

### Failure modes to design for
- Clock skew between customer middleware and auth server
- Broken refresh token handling in third-party connectors
- Token cache stampedes during renewal windows

### Metrics I track
- Auth success rate by tenant/client
- Refresh failure rate
- Invalid scope errors by endpoint
- Token issuance latency p95/p99
- Revocation propagation time

## API contract recommendations

- `401` for invalid/expired token
- `403` for valid token without required scope
- Standard error payload with machine-readable `code`

Example:

```json
{
  "error": {
    "code": "insufficient_scope",
    "message": "Token lacks contracts:write scope",
    "request_id": "req_8f2b..."
  }
}
```

## Common anti-patterns (and what I do instead)

- **Anti-pattern:** one global client credential per enterprise customer
  - **Instead:** issue per-environment, per-integration credentials with clear ownership
- **Anti-pattern:** extremely long-lived access tokens
  - **Instead:** short-lived access tokens + robust refresh
- **Anti-pattern:** scope explosion by endpoint
  - **Instead:** domain-centric scope model aligned to product capabilities

## Launch checklist

- [ ] Security reviewed grant types and token format
- [ ] Tenant claim enforced in all services
- [ ] Scope matrix documented per endpoint
- [ ] Credential rotation tested in staging
- [ ] Revocation and incident drill completed
- [ ] Developer docs include copy-paste curl examples

## Personal PM stance

If an auth model cannot be explained in one architecture slide and one onboarding guide, it is too complex for enterprise scale. Complexity in auth always reappears as support burden, security risk, or delayed revenue.

## Story from the field

In one enterprise rollout, a global customer initially requested a single shared credential across all regions "for speed." We pushed back and enforced tenant + environment scoped clients (prod, non-prod, regional split). It added one week to onboarding but prevented a critical cross-region data exposure during a later connector misconfiguration.

## Outcomes and metrics I track

- Security review cycle time: 6.0 weeks -> 2.5 weeks (-58%) over 2 quarters after OAuth standardization
- First-attempt integration go-live success: 62% -> 88% (+26 pts) over 2 quarters
- Auth-related Sev-2 incident volume: index 100 -> 60 (-40%) quarter-over-quarter
- Credential rotation within SLA: 81% -> 97% (+16 pts) over 6 months

