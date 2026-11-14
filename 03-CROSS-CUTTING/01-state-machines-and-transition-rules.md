# X-01 — STATE MACHINES VÀ QUY TẮC CHUYỂN TRẠNG THÁI
**Version:** 0.1 · **Status:** DRAFT · **Owner:** Domain Owners + Architecture

## 1. Nguyên tắc
Một aggregate có một owner và vòng đời riêng. Không trộn Booking, Offer, Assignment, Trip, Payment và Settlement vào một status field. Server xác thực mọi command; audit và idempotency bắt buộc với thao tác ghi quan trọng.

## 2. State catalog đề xuất
| Aggregate | States proposal | Owner |
|---|---|---|
| Booking | DRAFT, REQUESTED, CONFIRMED, READY_FOR_DISPATCH, DISPATCHING, ASSIGNED, IN_SERVICE, COMPLETED, CANCELLED, EXPIRED, EXCEPTION, UNFULFILLED | Booking |
| DispatchRun | CREATED, RUNNING, COMPLETED, NO_CANDIDATE, FAILED, CANCELLED, ESCALATED | Dispatch |
| Offer | CREATED, SENT, ACCEPTED, DECLINED, EXPIRED, CANCELLED, SUPERSEDED | Dispatch |
| Assignment | PENDING, ACTIVE, COMPLETED, RELEASED, CANCELLED, REASSIGNED, FAILED | Dispatch |
| Trip | CREATED, DRIVER_EN_ROUTE, ARRIVED, IN_PROGRESS, COMPLETED, CANCELLED, NO_SHOW, INCIDENT, NEEDS_REVIEW, DISPUTED | Trip |
| PaymentTransaction | PENDING, INITIATED, SUCCEEDED, FAILED, EXPIRED, CANCELLED, RECONCILIATION_REQUIRED | Payment |
| RefundTransaction | PENDING, INITIATED, SUCCEEDED, FAILED, RECONCILIATION_REQUIRED | Payment |
| SettlementBatch | DRAFT, CALCULATED, REVIEWED, APPROVED, REJECTED, ON_HOLD, LOCKED, PAID, CANCELLED | Finance/Settlement |
| DriverAvailability | OFFLINE, AVAILABLE, RESERVED, ON_TRIP, PAUSED, SUSPENDED | Fleet |
| VehicleAvailability | AVAILABLE, RESERVED, ON_TRIP, PAUSED, MAINTENANCE, SUSPENDED | Fleet |

Enum proposal only; owner approvals required. Booking REQUESTED/CONFIRMED/DISPATCHING semantics, Quote aggregate, Trip IN_TRIP vs IN_PROGRESS, Incident, Payment/Refund and no-show must be decided.

## 3. Transition guardrails
- Submit Booking only after validation.
- Each dispatch attempt creates DispatchRun; never overwrite prior attempt.
- Offer acceptance checks expiry using server time, dispatchable Booking, Driver/Vehicle eligibility and conflicts.
- Assignment activation is atomic; at most one active Assignment per Booking.
- Trip events have order/version; offline queue uses UUID/sequence/server validation/dedup.
- Payment unknown outcome → reconciliation, not false failure or repeated charge.
- Refund links original payment; Ledger correction uses reversal/adjustment.
- Locked/paid settlement is immutable; correction through adjustment.

## 4. Command envelope
```json
{
  "command_id": "uuid",
  "aggregate_id": "id",
  "expected_state": "CURRENT_STATE",
  "expected_version": 4,
  "idempotency_key": "stable-key",
  "actor_id": "user-or-service",
  "source": "DRIVER_APP|CALL_CENTER|CONTROL_TOWER|SYSTEM",
  "reason_code": "optional-code",
  "correlation_id": "uuid",
  "occurred_at": "ISO-8601"
}
```
Server checks permission → state/version → guards → atomic write → audit/outbox → response.

## 5. Mandatory invariants
One active Assignment per Booking; no conflicting resource allocation; eligibility before ranking; state+domain write atomic; idempotent command/consumer; immutable posted finance; GPS freshness/connectivity/availability separate.
