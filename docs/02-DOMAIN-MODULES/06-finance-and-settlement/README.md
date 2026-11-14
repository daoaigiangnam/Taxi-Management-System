# M06 — FINANCE & SETTLEMENT

**Status:** DRAFT · **Owner:** Finance/Accounting (TBD)  
**Overview:** [Project Overview](../../../00-OVERVIEW.md) · [Operating Model](../../../01-FOUNDATION/02-operating-model-direct-partner.md)

## Purpose
Ghi nhận kết quả tài chính có kiểm toán; quản lý doanh thu, chi phí, khoản phải thu/phải trả, nghĩa vụ đối tác và quy trình đối soát/chi trả.

## Entities
LedgerAccount, LedgerEntry, Receivable, Payable, PartnerSettlementBatch, SettlementLine, AdjustmentEntry, Payout, ReconciliationRun, VarianceCase.

## Features
| ID | Feature | Priority | Detail |
|---|---|---|---|
| FIN-001 | Chart/account mapping and posting rules | P0 | Planned |
| FIN-002 | Ledger posting from verified events | P0 | Planned |
| FIN-003 | Receivable/payable allocation | P1 | Planned |
| SET-001 | Partner settlement batch calculation | P0 | Planned |
| SET-002 | Settlement review/approve/lock | P0 | Planned |
| SET-003 | Payout and evidence | P1 | Planned |
| SET-004 | Variance/dispute and adjustment batch | P1 | Planned |
| FIN-004 | Financial reconciliation and export | P1 | Planned |

## Invariants
Posted entries are append-only; corrections use compensating/adjustment entries. Ledger must balance by approved accounting rules. Settlement line references Trip, LedgerEntry and contract snapshot. Locked batches are immutable. DIRECT/PARTNER economics must not be merged.

## Open decisions
Chart of accounts, accounting treatment, partner share/deductions/taxes, period/cutoff, approval matrix, payout provider/bank file and tolerances are TBD.
