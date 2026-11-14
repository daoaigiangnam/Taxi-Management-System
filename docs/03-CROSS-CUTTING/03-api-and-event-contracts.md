# X-03 — API VÀ EVENT CONTRACTS
**Version:** 0.1 · **Status:** DRAFT · **Owner:** API/Integration Architect

## 1. API principles
REST/JSON là ứng viên, chưa phải quyết định công nghệ. API phải có ownership, schema versioning, stable error, pagination/filter, authorization, idempotency, correlation. Internal command và partner public API cần threat model riêng.

## 2. API groups
Customer search/get/create/update; Booking create/get/search/revise/cancel; Dispatch queue/run/retry/candidates/manual; Offer send/get/accept/decline/expire; Assignment activate/release/reassign; Fleet driver/vehicle/availability/documents; Partner/contract/eligibility; Trip start/arrive/stops/complete/no-show/incident; Pricing quote/tariff/final fare/adjust; Payment initiate/status/callback/refund/reconcile; Finance ledger/post/adjust/reconcile; Settlement calculate/review/approve/lock/pay/adjust; IAM; Control Tower; Reporting.

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

## 4. Response/error
Success returns resource, state, version, correlation_id. Errors include code, message, field_errors, retryable, correlation_id. Never leak secrets/PII.

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
Outbox/inbox; idempotent consumers; bounded retry/backoff; dead-letter and controlled replay; schema compatibility; signed webhook; timeout/circuit breaker. OpenAPI/AsyncAPI schemas per approved Feature Spec.

## 7. Decisions TBD
Protocol, auth, API gateway, sync/async boundaries, event broker, schema registry, compatibility and rate limits.
