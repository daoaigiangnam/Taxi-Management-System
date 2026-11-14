# X-01 — STATE MACHINES VÀ QUY TẮC CHUYỂN TRẠNG THÁI

**Version:** 0.1 · **Status:** DRAFT · **Owner:** Domain Owners + Architecture

## 1. Nguyên tắc
Một aggregate có một owner và một vòng đời riêng. Không trộn Booking, Offer, Assignment, Trip, Payment và Settlement vào một status field. Server xác thực mọi command; audit và idempotency là bắt buộc với thao tác ghi quan trọng.

## 2. State catalog đề xuất

| Aggregate | States đề xuất | Owner |
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

**Cảnh báo:** đây là enum proposal; chưa được nghiệp vụ/kiến trúc duyệt. Booking dùng REQUESTED/CONFIRMED/DISPATCHING thế nào cần chốt. Các state aggregate có thể được mô hình bằng state + reason_code thay vì thêm enum không kiểm soát.

## 3. Quy tắc chuyển đổi tối thiểu
- Submit Booking chỉ sau validate; không hợp lệ thì không tạo yêu cầu dispatch.
- DispatchRun tạo mới cho mỗi attempt; không ghi đè lịch sử tìm xe.
- Offer acceptance chỉ thắng khi chưa hết hạn theo server time, Booking còn dispatchable, Driver/Vehicle đủ điều kiện và chưa có Assignment thắng.
- Assignment activation là atomic transaction; tối đa một Assignment hiệu lực trên Booking.
- Trip events theo thứ tự hợp lệ; offline queue có UUID, sequence, server validation và deduplication.
- Payment unknown outcome phải reconciliation, không đánh dấu thất bại giả hoặc thu lặp.
- Refund là giao dịch liên kết với payment gốc; Ledger correction là bút toán đảo/điều chỉnh.
- Settlement LOCKED/PAID không sửa lịch sử; phát sinh adjustment riêng.

## 4. Command transition envelope
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

Server checks permission → current state/version → guards → atomic write → audit/outbox → response. API details belong to X-03.

## 5. Mandatory invariants
- One active Assignment per Booking.
- No conflicting active resource allocation.
- Eligibility before ranking.
- State transition and domain write commit atomically.
- Idempotent consumer and command retry.
- No mutation of posted financial history.
- GPS freshness, connectivity and resource availability are separate.

## 6. Open state decisions
Booking post-submit state; quote as state vs own aggregate; DISPATCHING vs DispatchRun projection; meaning of UNFULFILLED; Assignment PENDING necessity; Trip IN_TRIP vs IN_PROGRESS; Incident as case vs Trip state; payment/refund lifecycle; no-show; cancellation; resource reservation semantics.
