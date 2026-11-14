# M07 — CONTROL TOWER & CUSTOMER SERVICE

**Status:** DRAFT · **Owner:** Operations + CSKH (TBD)  
**Overview:** [Project Overview](../../../00-OVERVIEW.md)

## Purpose
Cung cấp góc nhìn điều hành thời gian gần thực; giúp điều phối viên/giám sát theo dõi Booking, nguồn lực, Offer, Assignment, Trip, ngoại lệ và hỗ trợ khách hàng.

## Entities/read models
OperationalBoard, MapVehicleProjection, BookingQueueProjection, DispatchMonitor, IncidentCase, ServiceCase, Complaint, SLAClock, AgentActionAudit.

## Features
| ID | Feature | Priority | Detail |
|---|---|---|---|
| CT-001 | Live operational overview and KPI | P0 | Planned |
| CT-002 | Map and resource freshness | P0 | Planned |
| CT-003 | Booking/dispatch queue management | P0 | Planned |
| CT-004 | Manual action and controlled override | P1 | Planned |
| CT-005 | Incident/SOS monitoring | P1 | Planned |
| CS-001 | Customer service case | P1 | Planned |
| CS-002 | Complaint, refund/fare dispute handoff | P1 | Planned |
| CS-003 | SLA and escalation monitoring | P1 | Planned |

## Rules
Control Tower is not source-of-record for Booking/Trip/Payment. Actions invoke owner-module commands, permissions and audit. Display GPS freshness and last update prominently. Manual overrides require reason and policy.

## Open decisions
Map provider, refresh interval, role layouts, SLA definitions, escalations, agent scripts, case taxonomy and customer contact policy are TBD.
