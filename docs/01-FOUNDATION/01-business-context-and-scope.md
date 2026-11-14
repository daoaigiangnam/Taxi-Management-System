# FND-01 — BỐI CẢNH NGHIỆP VỤ VÀ PHẠM VI

**Version:** 0.1 · **Status:** DRAFT · **Owner:** Sponsor/Business Owner (TBD)

## 1. Bối cảnh

Nền tảng quản lý taxi cần kết nối các hoạt động: tiếp nhận yêu cầu, quản trị khách hàng và nguồn lực xe, điều phối, thực hiện chuyến, tính/thu cước, quản lý tài chính, đối soát, chăm sóc khách và báo cáo.

Phạm vi tổ chức gồm nguồn xe DIRECT và PARTNER. Mục tiêu kiến trúc là dùng chung năng lực vận hành nhưng không làm mất sự phân biệt về chủ thể hợp đồng và dòng tiền.

## 2. Mục tiêu nghiệp vụ cần đo lường

| Mục tiêu | Chỉ số đo lường đề xuất | Baseline/Target |
|---|---|---|
| Tăng tỷ lệ đáp ứng booking | Fulfilled bookings / eligible bookings | TBD |
| Rút ngắn thời gian phân xe | Time from dispatch start to active assignment | TBD |
| Giảm hủy và không có xe | Cancellation / unfulfilled rate | TBD |
| Tăng độ chính xác ETA | ETA error distribution | TBD |
| Tăng năng suất nguồn lực | Trips, utilization, idle time by fleet | TBD |
| Minh bạch tài chính | Fare/payment/ledger/settlement reconciliation variance | TBD |
| Nâng chất lượng dịch vụ | Complaints, CSAT, incident resolution | TBD |
| Tăng khả năng kiểm soát | Audit completeness, operational data freshness | TBD |

Không gán target số trước khi lấy baseline và được owner phê duyệt.

## 3. Phạm vi

**In scope:** Customer/Booking; Fleet/Partner; Dispatch; Driver App/Trip; Pricing/Payment; Finance/Settlement; Control Tower/CSKH; IAM/Audit; Integration/Notification; Reporting.

**Out of scope until approved:** payroll, HRM, workshop/garage, insurance claims, accounting general ledger replacement, loyalty, delivery/logistics, advertising, third-party open marketplace.

## 4. Stakeholders

Sponsor; Operations/Dispatch; Fleet; Partner Management; Call Center/CSKH; Finance/Accounting; Legal/Compliance; Security/IT; Product/BA; UX; Architecture/Engineering; QA; Operations Support.

## 5. Assumptions and constraints

- Shared dispatch does not mean pooled finance or contracts.
- Customer channel does not define fleet source.
- Pricing/settlement rules must be versioned and snapshotted.
- Server validates state transitions.
- Historical financial and audit records are append-only/corrected by linked entries.
- Vendor, infrastructure, scale and regulatory requirements remain TBD.

## 6. Success gates

1. Business objectives and scope approved.
2. E2E process and operating responsibility approved.
3. Domain/data ownership and architecture approved.
4. Critical feature specifications ready and traceable.
5. Test/UAT evidence accepted.
6. Operations, DR, security and finance sign-off obtained.
