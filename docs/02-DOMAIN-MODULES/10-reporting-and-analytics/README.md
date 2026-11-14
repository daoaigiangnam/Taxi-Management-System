# M10 — REPORTING & ANALYTICS
**Status:** DRAFT · **Owner:** BI/Finance/Operations (TBD) · [Back to Overview](../../../README.md)

## Purpose
Cung cấp KPI/báo cáo cho BOD, Operations, Fleet, Partner, CSKH và Finance từ nguồn dữ liệu có thẩm quyền.

## Feature Catalog
| ID | Feature | Priority |
|---|---|---|
| RPT-001 | Executive overview/business KPI | P1 |
| RPT-002 | Booking/dispatch funnel | P0 |
| RPT-003 | Driver/fleet utilization | P1 |
| RPT-004 | DIRECT/PARTNER economics | P1 |
| RPT-005 | Fare/payment/ledger/settlement reconciliation | P0 |
| RPT-006 | Export/scheduled reporting | P1 |
| RPT-007 | Data quality/lineage | P1 |

## Rules
Each KPI has formula, source, grain, filters, timezone, refresh SLA, owner and version. Drill-down follows permissions. Reporting read model never updates transactional tables.

## Open decisions
KPI definitions, financial basis, refresh cadence, BI platform, retention and access.
