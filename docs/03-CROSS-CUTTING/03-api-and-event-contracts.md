# X-03 — API VÀ EVENT CONTRACTS

**Version:** 0.1 · **Status:** DRAFT · **Owner:** API/Integration Architect

## 1. API principles
REST/JSON is a candidate default, not a technology decision. Use explicit resource ownership, stable error schema, versioning, pagination, filtering, authorization, idempotency and correlation. Internal commands and public partner APIs need separate threat models.

## 2. API domain groups (logical, not final paths)

| Group | Candidate operations |
|---|---|
| Customer | search/get/create/update customer |
| Booking | create/get/search/revise/cancel |
| Dispatch | queue/run/retry/candidates/manual dispatch |
| Offer | send/get/accept/decline/expire |
| Assignment | activate/release/reassign |
| Fleet | driver/vehicle/availability/documents |
| Partner | partner/contract/version/eligibility |
| Trip | start/arrive/stops/complete/no-show/incident |
| Pricing | quote/tariff/final fare/adjust |
| Payment | initiate/status/callback/refund/reconcile |
| Finance | ledger query/post/adjust/reconcile |
| Settlement | calculate/review/approve/lock/pay/adjust |
| Control Tower | operational read models and authorized commands |
| IAM | users/roles/scopes/policies |
| Reporting | KPI/read models/export |

## 3. Command envelope
```json
{
  "request_id": "uuid",
  "idempotency_key": "stable-per-command",
  "expected_version": 7,
  "actor_id": "authenticated-principal",
  "source": "CALL_CENTER",
  "correlation_id": "uuid",
  "reason_code": null,
  "payload": {}
}
```

## 4. Standard response/error
Success returns resource, state, version, correlation_id. Errors include code, message, field_errors, retryable, correlation_id. Do not leak secrets/PII.

## 5. Event envelope
```json
{
  "event_id": "uuid",
  "event_type": "booking.created",
  "aggregate_type": "Booking",
  "aggregate_id": "id",
  "aggregate_version": 1,
  "occurred_at": "ISO-8601",
  "recorded_at": "ISO-8601",
  "correlation_id": "uuid",
  "data": {}
}
```

## 6. Reliability
Transactional outbox/inbox where needed; idempotent consumers; bounded retry/backoff; dead-letter and replay under control; schema compatibility; signed webhook; timeout and circuit breaker. Detailed OpenAPI/AsyncAPI schemas are created per Feature Spec after architecture decision.

## 7. Open decisions
Protocol, auth model, API gateway, sync/async boundaries, event broker, schema registry, compatibility and rate limits.
