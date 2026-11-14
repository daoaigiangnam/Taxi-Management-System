# FND-02 — MÔ HÌNH VẬN HÀNH DIRECT / PARTNER

**Version:** 0.1 · **Status:** DRAFT · **Owner:** Operations + Finance + Legal (TBD)

## 1. Nguyên tắc

DIRECT và PARTNER tham gia cùng một chuỗi Booking → Dispatch → Trip. Hệ thống dùng chung workflow và control tower nhưng giữ riêng chủ thể sở hữu nguồn lực, hợp đồng, doanh thu, công nợ và settlement.

## 2. Phân biệt các khái niệm

| Khái niệm | Ý nghĩa |
|---|---|
| booking_channel | Kênh nhận yêu cầu: hotline, app, web, API, partner portal… |
| fleet_type | Nguồn lực thực hiện: DIRECT hoặc PARTNER |
| partner_id | Chủ thể đối tác sở hữu/cung cấp nguồn lực |
| contract_version_id | Bản hợp đồng/điều khoản có hiệu lực tại thời điểm nghiệp vụ |
| assignment | Kết quả phân công tài xế và xe |
| beneficiary | Chủ thể có quyền nhận dòng tiền theo chính sách |

Không suy ra fleet_type từ booking_channel hoặc từ nhân viên tạo booking.

## 3. Phân định trách nhiệm (RACI đề xuất)

| Hoạt động | Operations | Fleet | Partner Mgmt | Finance | Legal | IT |
|---|---|---|---|---|---|---|
| Nhận và xác nhận booking | A/R | I | I | I | I | C |
| Xác nhận điều kiện xe/tài xế | C | A/R | R (partner) | I | C | C |
| Chính sách dispatch | A/R | C | C | C | I | R/C |
| Hợp đồng và điều kiện partner | I | C | A/R | C | R | C |
| Giá và phụ phí | C | I | C | A/R | C | C |
| Payment/ledger | I | I | I | A/R | C | R/C |
| Settlement partner | C | I | R | A/R | C | R/C |
| Incident/an toàn | A/R | R | C | I | C | C |
| Phân quyền và audit | C | C | C | C | C | A/R |

RACI phải được tổ chức điều chỉnh theo cơ cấu và ủy quyền thực tế.

## 4. Shared vs segregated control

**Shared:** booking intake, dispatch workflow, candidate eligibility engine framework, driver app/trip event model, control tower, notifications, common audit and analytics framework.

**Segregated:** legal entity/contract, fleet ownership, partner identity, tariff entitlement, beneficiary, revenue/cost allocation, receivable/payable, deductions, settlement approval and payout.

## 5. Chính sách cần xác nhận

- DIRECT/PARTNER có ưu tiên hoặc quota hay không.
- Partner/contract/service/zone/time eligibility.
- Revenue share, commissions, fees, deductions, tax and dispute.
- Who owns customer relationship and complaint resolution.
- Partner driver onboarding, documents, suspension, insurance and training.
- Settlement cycle, cutoff, approval, payout evidence and adjustment.
Mỗi nội dung là TBD, phải có owner và quyết định được ghi nhận.
