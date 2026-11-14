# M04 — DRIVER APP & TRIP

**Status:** DRAFT · **Owner:** Driver Operations + Trip PO (TBD)  
**Overview:** [Project Overview](../../../00-OVERVIEW.md) · [E2E Flow](../../../01-FOUNDATION/03-end-to-end-business-processes.md) · [State Machines](../../../03-CROSS-CUTTING/01-state-machines-and-transition-rules.md)

## Purpose
Hỗ trợ tài xế nhận việc, đến điểm đón, thực hiện chuyến, ghi nhận sự kiện và xử lý an toàn; Trip domain ghi nhận sự thật vận hành do server xác thực.

## Entities
Trip, TripEvent, TripStopProgress, DriverDevice, OfflineEventEnvelope, IncidentCase, SOSAlert.

## Features
| ID | Feature | Priority | Detail |
|---|---|---|---|
| DRV-001 | Driver authentication/device session | P0 | Planned |
| DRV-002 | Availability and online/offline | P0 | Planned |
| DRV-003 | Offer inbox and accept/reject | P0 | Planned |
| TRP-001 | Start trip / en-route / arrived | P0 | Planned |
| TRP-002 | Trip stops and progress | P0 | Planned |
| TRP-003 | Complete trip and evidence | P0 | Planned |
| TRP-004 | Offline event queue and synchronization | P0 | Planned |
| TRP-005 | SOS, incident and safety workflow | P1 | Planned |
| TRP-006 | Cancellation/no-show/dispute | P1 | Planned |

## Rules
- Driver App commands are server-validated.
- Offline events have client UUID, device timestamp and sequence; backend deduplicates and checks transition.
- Do not display final success before server acknowledgement.
- GPS freshness and network connectivity do not directly mutate trip/availability state.
- Trip must reference one effective Assignment.

## Open decisions
Supported devices/OS, offline duration, GPS frequency, evidence requirements, no-show wait, contact masking, SOS routing, incident severity and retention are TBD.
