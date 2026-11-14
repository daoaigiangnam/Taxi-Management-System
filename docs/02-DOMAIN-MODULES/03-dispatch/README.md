# M03 — DISPATCH
**Status:** DRAFT · **Owner:** Operations/Dispatch PO (TBD) · [Back to Overview](../../../README.md)

## Purpose
Chuyển Booking đủ điều kiện thành Assignment hợp lệ, có giải thích và truy vết; hỗ trợ tự động/thủ công.

## Entities
DispatchRun, DispatchPolicyVersion, CandidateSnapshot, CandidateDecision, Offer, OfferDelivery, Assignment, DispatchOverride.

## Feature Catalog
| ID | Feature | Priority |
|---|---|---|
| DSP-001 | Dispatch queue/priority | P0 |
| DSP-002 | Hard eligibility filtering | P0 |
| DSP-003 | Candidate ranking/explanation | P0 |
| DSP-004 | Offer creation/delivery/expiry | P0 |
| DSP-005 | Atomic accept/assignment | P0 |
| DSP-006 | Retry/no candidate/escalation | P0 |
| DSP-007 | Manual dispatch/override | P1 |
| DSP-008 | Release/cancel/reassign | P1 |
| DSP-009 | Audit and metrics | P1 |

## Invariants
Eligibility precedes ranking. Only eligible candidates are offered. One active Assignment per Booking. Driver/Vehicle cannot hold conflicting work. Accept uses idempotency and atomic checks. Manual override cannot bypass hard constraints except approved emergency policy.

## Open decisions
Radius, GPS freshness, score weights, fairness, DIRECT/PARTNER policy, offer fanout/expiry, retry/SLA, emergency override and reason codes.
