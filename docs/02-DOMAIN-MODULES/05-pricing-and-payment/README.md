# M05 — PRICING & PAYMENT

**Status:** DRAFT · **Owner:** Finance/Pricing PO (TBD)  
**Overview:** [Project Overview](../../../00-OVERVIEW.md)

## Purpose
Quản trị tariff, báo giá, tính cước tái lập được và giao dịch thu/hoàn tiền; tách Payment khỏi Trip và Ledger.

## Entities
Tariff, TariffVersion, FareQuote, FareComponent, FareResult, FareAdjustment, PaymentTransaction, RefundTransaction, ProviderCallback.

## Features
| ID | Feature | Priority | Detail |
|---|---|---|---|
| FAR-001 | Tariff setup/version/effective period | P0 | Planned |
| FAR-002 | Quote and quote validity | P1 | Planned |
| FAR-003 | Final fare calculation and breakdown | P0 | Planned |
| FAR-004 | Fare adjustment/dispute handoff | P1 | Planned |
| PAY-001 | Cash collection confirmation workflow | P0 | Planned |
| PAY-002 | Electronic payment initiation/callback | P1 | Planned |
| PAY-003 | Refund and partial refund | P1 | Planned |
| PAY-004 | Provider reconciliation | P1 | Planned |

## Rules
- Fare result stores input facts, tariff version, engine version, breakdown, currency and calculation timestamp.
- Amounts use currency-aware minor-unit/precision rules.
- Payment callbacks are signed, replay-protected and idempotent.
- Unknown outcome goes to reconciliation; do not double charge.
- Refund is linked to original payment and must be auditable.

## Open decisions
Rate tables, distance/time source, rounding, surcharges, tax, waiting, cash evidence, payment methods/provider, retry, refund approvals and settlement timing are TBD.
