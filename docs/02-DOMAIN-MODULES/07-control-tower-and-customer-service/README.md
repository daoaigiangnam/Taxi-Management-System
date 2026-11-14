# M07 — CONTROL TOWER & CUSTOMER SERVICE
**Status:** DRAFT · **Owner:** Operations + CSKH (TBD) · [Back to Overview](../../../README.md)

## Purpose
Góc nhìn điều hành gần thời gian thực để theo dõi Booking, nguồn lực, Offer, Assignment, Trip, ngoại lệ và yêu cầu khách hàng.

## Entities/read models
OperationalBoard, MapVehicleProjection, BookingQueueProjection, DispatchMonitor, IncidentCase, ServiceCase, Complaint, SLAClock, AgentActionAudit.

## Feature Catalog
| ID | Feature | Priority |
|---|---|---|
| CT-001 | Live overview/KPI | P0 |
| CT-002 | Map/resource freshness | P0 |
| CT-003 | Booking/dispatch queue | P0 |
| CT-004 | Manual action/controlled override | P1 |
| CT-005 | Incident/SOS monitor | P1 |
| CS-001 | Customer service case | P1 |
| CS-002 | Complaint/fare dispute handoff | P1 |
| CS-003 | SLA/escalation | P1 |

## Rules
Control Tower is not source-of-record for Booking/Trip/Payment. Invoke owner-module commands; permissions and audit mandatory. Show GPS freshness/last update. Override requires reason/policy.

## Open decisions
Map provider, refresh interval, role layouts, SLA, escalation, case taxonomy and contact policy.
