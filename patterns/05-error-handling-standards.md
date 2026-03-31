# Error Handling Standards for Enterprise APIs


## What I Drove

- Unified API errors into a single contract with stable machine codes and actionable diagnostics.
- Improved first-response incident diagnosis time: 44 min -> 24 min (-45%).
- Reduced support back-and-forth by 52% and increased self-resolution by 21 pts.

## Point of view

Error design is part of developer experience. A clean error taxonomy reduces integration time, support load, and blame loops across customer IT teams.

## Standard response envelope

```json
{
  "error": {
    "code": "validation_failed",
    "message": "Field 'currency' is invalid",
    "details": [
      {"field": "currency", "issue": "unsupported_value"}
    ],
    "request_id": "req_123",
    "docs_url": "https://docs.example.com/errors/validation_failed"
  }
}
```

## Status code mapping

- `400` malformed request
- `401` missing/invalid auth
- `403` authenticated but unauthorized
- `404` resource not found
- `409` state conflict/idempotency conflict
- `422` semantic validation errors
- `429` rate limit exceeded
- `5xx` server-side failure

## Taxonomy principles

- Stable machine codes, human-readable messages
- One canonical code per failure category
- Include actionable next step where possible

## Localization and privacy

- Messages can be localized for UI clients
- Never leak internal stack traces or secrets
- PII-safe logging keyed by `request_id`

## PM governance

- New error code additions require review
- Deprecated codes mapped and documented
- SDKs expose typed errors for major languages

## Story from the field

Before standardization, each service returned different error shapes and vague messages. Customer teams escalated "API not working" tickets with little diagnostic context. By moving to a single envelope with stable codes and `request_id`, support and engineering could triage quickly and customers could self-correct many integration issues.

## Outcomes and metrics I track

- First-response diagnosis time for API incidents: 44 min -> 24 min (-45%) over 2 quarters
- "Need more logs/details" ticket back-and-forth: index 100 -> 48 (-52%) over 2 quarters
- Self-resolution rate for integration errors: 28% -> 49% (+21 pts) over 6 months after error-doc rollout
- SDK implementation cycle time (pilot accounts): 5.4 weeks -> 3.7 weeks (-31%) over 2 releases

