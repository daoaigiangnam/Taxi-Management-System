# M09 — INTEGRATION & NOTIFICATION
**Status:** DRAFT · **Owner:** IT/Integration (TBD) · [Back to Overview](../../../README.md)

## Purpose
Adapter với hệ thống/nhà cung cấp bên ngoài và phân phối thông báo; không sở hữu domain truth.

## Domains
Maps/geocoding/routing; SMS/voice/IVR; push/email; payment adapter; partner API/webhook; ERP/accounting export.

## Feature Catalog
| ID | Feature | Priority |
|---|---|---|
| INT-001 | Provider registry/secret reference | P1 |
| INT-002 | Map/geocode/route adapter | P0 |
| INT-003 | SMS/voice/push/email delivery | P1 |
| INT-004 | Partner API/webhook gateway | P1 |
| INT-005 | Payment provider adapter framework | P1 |
| INT-006 | Retry/dead-letter/delivery monitor | P0 |

## Rules
Translate provider contracts into canonical internal contracts; signed webhooks, idempotency, bounded retry, DLQ/repair and secret management. Provider failure must not fabricate domain state.

## Open decisions
Provider, API quotas, SLA/cost, data residency, retry/timeout and support contract.
