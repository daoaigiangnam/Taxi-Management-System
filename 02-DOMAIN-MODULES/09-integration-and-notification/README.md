# M09 — INTEGRATION & NOTIFICATION

**Status:** DRAFT · **Owner:** IT/Integration (TBD)  
**Overview:** [Project Overview](../../../00-OVERVIEW.md)

## Purpose
Tích hợp nhà cung cấp/hệ thống bên ngoài và phân phối thông báo; không nắm quyền sở hữu dữ liệu nghiệp vụ gốc.

## Domains
Map/geocoding/routing; SMS/voice/IVR; push/email; payment provider adapters; partner APIs/webhooks; ERP/accounting export; monitoring integrations.

## Features
| ID | Feature | Priority | Detail |
|---|---|---|---|
| INT-001 | Provider registry and credentials references | P1 | Planned |
| INT-002 | Map/geocode/route adapter | P0 | Planned |
| INT-003 | SMS/voice/push/email delivery | P1 | Planned |
| INT-004 | Partner API and webhook gateway | P1 | Planned |
| INT-005 | Payment provider adapter framework | P1 | Planned |
| INT-006 | Retry, dead-letter and delivery monitoring | P0 | Planned |

## Rules
Adapters translate provider contracts to canonical internal contracts. Use signed webhooks, idempotency, retry with bounded policy, dead-letter/repair workflow and secrets manager. Provider failures must not create false domain state.

## Open decisions
Provider selection, API limits, SLA, cost, data residency, timeout/retry and support contracts are TBD.
