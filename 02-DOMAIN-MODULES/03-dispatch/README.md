# M03 — DISPATCH

**Status:** DRAFT · **Owner:** Operations/Dispatch PO (TBD)  
**Overview:** [Project Overview](../../../00-OVERVIEW.md) · [E2E Flow](../../../01-FOUNDATION/03-end-to-end-business-processes.md) · [State Machines](../../../03-CROSS-CUTTING/01-state-machines-and-transition-rules.md)

## Purpose
Chuyển Booking đủ điều kiện thành Assignment hợp lệ, công bằng, có thể giải thích và truy vết; hỗ trợ tự động và thủ công.

## Entities
DispatchRun, DispatchPolicyVersion, CandidateSnapshot, CandidateDecision, Offer, OfferDelivery, Assignment, DispatchOverride.

## Features

| ID | Feature | Priority | Detail |
|---|---|---|---|
| DSP-001 | Dispatch queue and priority | P0 | Planned |
| DSP-002 | Candidate eligibility filtering | P0 | Planned |
| DSP-003 | Candidate ranking and explanation | P0 | Planned |
| DSP-004 | Offer creation, delivery and expiry | P0 | Planned |
| DSP-005 | Atomic offer acceptance and assignment | P0 | Planned |
| DSP-006 | Retry, no-candidate and escalation | P0 | Planned |
| DSP-007 | Manual dispatch and override | P1 | Planned |
| DSP-008 | Release, cancellation and reassignment | P1 | Planned |
| DSP-009 | Dispatch audit and performance metrics | P1 | Planned |

## Hard constraints before ranking
Driver active/eligible; vehicle service/capacity/documents; shift/availability; no resource conflict; partner/contract eligibility; zone/service compatibility; location quality/freshness per approved policy.

## Concurrency invariants
One active Assignment per Booking. Driver/Vehicle cannot hold conflicting active work. Accept requests use idempotency and atomic checks. Only one concurrent offer acceptance wins.

## Open decisions
Candidate radius, GPS freshness, ranking weights, fairness, DIRECT/PARTNER policy, offer fanout, expiry, retry, SLA, emergency override and manual-selection reason codes are TBD.
