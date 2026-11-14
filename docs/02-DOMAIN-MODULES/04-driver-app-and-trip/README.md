# M04 — DRIVER APP & TRIP
**Status:** DRAFT · **Owner:** Driver Operations + Trip PO (TBD) · [Back to Overview](../../../README.md)

## Purpose
Hỗ trợ tài xế nhận việc, đến điểm đón, thực hiện chuyến, ghi nhận sự kiện và xử lý an toàn. Trip server là nguồn có thẩm quyền.

## Entities
Trip, TripEvent, TripStopProgress, DriverDevice, OfflineEventEnvelope, IncidentCase, SOSAlert.

## Feature Catalog
| ID | Feature | Priority |
|---|---|---|
| DRV-001 | Authentication/device session | P0 |
| DRV-002 | Availability/online state | P0 |
| DRV-003 | Offer inbox/accept/reject | P0 |
| TRP-001 | En-route/arrive/start | P0 |
| TRP-002 | Stops/progress | P0 |
| TRP-003 | Complete trip/evidence | P0 |
| TRP-004 | Offline queue/sync | P0 |
| TRP-005 | SOS/incident | P1 |
| TRP-006 | Cancel/no-show/dispute | P1 |

## Rules
Commands are server-validated. Offline events carry client UUID/device time/sequence; backend deduplicates and checks state. No final success before acknowledgement. GPS freshness/network independent. Trip references effective Assignment.

## Open decisions
Device/OS; offline duration; GPS frequency; evidence; no-show wait; contact masking; SOS routing; severity and retention.
