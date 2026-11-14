# M10 — REPORTING & ANALYTICS

**Status:** DRAFT · **Owner:** Business Intelligence/Finance/Operations (TBD)  
**Overview:** [Project Overview](../../../00-OVERVIEW.md)

## Purpose
Cung cấp KPI và báo cáo từ nguồn dữ liệu có thẩm quyền; phục vụ BOD, Operations, Fleet, Partner, CSKH và Finance.

## Features
| ID | Feature | Priority | Detail |
|---|---|---|---|
| RPT-001 | Executive overview and business KPIs | P1 | Planned |
| RPT-002 | Booking and dispatch funnel | P0 | Planned |
| RPT-003 | Driver/fleet utilization and performance | P1 | Planned |
| RPT-004 | DIRECT/PARTNER economics | P1 | Planned |
| RPT-005 | Fare/payment/ledger/settlement reconciliation | P0 | Planned |
| RPT-006 | Data export and scheduled reporting | P1 | Planned |
| RPT-007 | Data quality and lineage monitoring | P1 | Planned |

## Rules
Each KPI has formula, source, grain, filters, timezone, refresh SLA, owner and version. Drill-down must reach source records subject to permissions. Reporting read models never update transaction source tables.

## Open decisions
KPI definitions, financial reporting basis, refresh cadence, warehouse/BI platform, retention and access controls are TBD.
