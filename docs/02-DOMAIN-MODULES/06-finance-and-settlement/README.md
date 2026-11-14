# M06 — FINANCE & SETTLEMENT
**Status:** DRAFT · **Owner:** Finance/Accounting (TBD) · [Back to Overview](../../../README.md)

## Purpose
Ghi nhận kết quả tài chính có kiểm toán; quản lý doanh thu, chi phí, công nợ, nghĩa vụ partner, đối soát và chi trả.

## Entities
LedgerAccount, LedgerEntry, Receivable, Payable, SettlementBatch, SettlementLine, AdjustmentEntry, Payout, ReconciliationRun, VarianceCase.

## Feature Catalog
| ID | Feature | Priority |
|---|---|---|
| FIN-001 | Account mapping/posting rules | P0 |
| FIN-002 | Ledger posting from verified events | P0 |
| FIN-003 | Receivable/payable allocation | P1 |
| SET-001 | Partner settlement calculation | P0 |
| SET-002 | Review/approve/lock | P0 |
| SET-003 | Payout/evidence | P1 |
| SET-004 | Variance/dispute/adjustment | P1 |
| FIN-004 | Financial reconciliation/export | P1 |

## Invariants
Posted ledger append-only; correction by compensating entries; ledger balances per approved rules; line references Trip/LedgerEntry/contract snapshot; locked batches immutable; DIRECT/PARTNER economics separate.

## Open decisions
Chart of accounts, accounting treatment, partner share/deduction/tax, period/cutoff, maker-checker, payout evidence and tolerance.
