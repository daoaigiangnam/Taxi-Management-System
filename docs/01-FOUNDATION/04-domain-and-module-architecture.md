# FND-04 — DOMAIN VÀ KIẾN TRÚC MODULE

**Version:** 0.1 · **Status:** DRAFT · **Owner:** Solution Architect (TBD)

## 1. Kiến trúc logic

Mô hình domain-oriented modular architecture: ranh giới Module rõ trước, triển khai vật lý dạng modular monolith hay services là quyết định sau khi phân tích tải, đội ngũ, độ sẵn sàng vận hành và chi phí. Không mặc định microservices.

## 2. Domain ownership

| Module | Owns | Không được sở hữu thay |
|---|---|---|
| Customer & Booking | Customer identity/profile, Booking, BookingStop, BookingRevision | Offer, Assignment, Trip |
| Fleet & Partner | Driver, Vehicle, partner, contract versions, availability/eligibility | Booking, Trip financial facts |
| Dispatch | DispatchRun, candidate snapshot, Offer, Assignment | Driver master profile, Trip |
| Driver App & Trip | Trip, TripEvent, IncidentCase, driver-device sync metadata | Assignment decision |
| Pricing & Payment | Tariff, Quote, FareResult, PaymentTransaction, Refund | Ledger posting authority if Finance owns it |
| Finance & Settlement | Ledger, receivable/payable, settlement batches/lines, payout | Operational Trip status |
| Control Tower & CS | Operational views/actions, service cases, complaint workflow | Source-of-record state of other domains |
| Platform/IAM/Audit | User, role, policy, audit, configuration, secrets integration | Business facts owned elsewhere |
| Integration/Notification | Adapter, webhook, delivery attempt, notification | Domain truth |
| Reporting/Analytics | Read models, KPI definitions, aggregates | Transactional source-of-record writes |

## 3. Module interaction

- Synchronous API/command for operations requiring immediate validation/response.
- Domain events for cross-module facts after successful commit.
- Transactional outbox/inbox and idempotent consumers for reliable delivery.
- Reporting consumes read models/events, not direct arbitrary writes to transactional tables.
- Cross-module orchestration may use process manager/saga if long-running consistency requires it; exact patterns are TBD after technical design.

## 4. Decision: deployment topology

Candidates: modular monolith, service-oriented modular deployment, microservices. Evaluate transaction boundaries, scale, latency, independent release, team ownership, operational maturity, observability, network failure and cost. Do not choose microservices by fashion.

## 5. Architectural guardrails

- One owner per aggregate.
- No cross-module direct table updates.
- Stable API/event contract and compatibility policy.
- Explicit tenant/branch/partner data scope.
- Versioned policy snapshots for fare/contract/dispatch.
- Idempotent writes, concurrency safety and audit.
- PII separation, encryption and retention policy.
