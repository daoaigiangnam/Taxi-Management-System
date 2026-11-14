# M05 — PRICING & PAYMENT
**Status:** DRAFT · **Owner:** Finance/Pricing PO (TBD) · [Back to Overview](../../../README.md)

## Purpose and entities
Quản trị tariff, báo giá, tính cước tái lập được và giao dịch thu/hoàn tiền. Entities: TariffVersion, FareQuote, FareComponent, FareResult, FareAdjustment, PaymentTransaction, RefundTransaction, ProviderCallback.

## Feature Catalog
| ID | Feature | Priority |
|---|---|---|
| FAR-001 | Tariff setup/version/effective dates | P0 |
| FAR-002 | Quote/validity | P1 |
| FAR-003 | Final fare/breakdown | P0 |
| FAR-004 | Fare adjustment/dispute handoff | P1 |
| PAY-001 | Cash collection | P0 |
| PAY-002 | Electronic payment/callback | P1 |
| PAY-003 | Refund/partial refund | P1 |
| PAY-004 | Provider reconciliation | P1 |

## Rules
FareResult stores verified inputs/tariff/engine version/breakdown/currency/time. Money uses currency-aware precision. Callback signed/replay-protected/idempotent. Unknown outcome → reconciliation. Refund linked to original payment.

## Open decisions
Rates, distance/time source, rounding, surcharges, taxes, waiting, cash evidence, providers, retry and refund approval.
