# 00 — OVERVIEW HỆ THỐNG TAXI MANAGEMENT

**Document ID:** TAXI-ARC-000  
**Version:** 0.1  
**Status:** DRAFT  
**Owner:** Product/Business + Solution Architecture (TBD)  
**Repository:** [Taxi-Management-System](https://github.com/daoaigiangnam/Taxi-Management-System)  
**Entry point:** [README — Single Project Overview](../../README.md)

## 1. Tầm nhìn và mục tiêu

Xây dựng nền tảng quản lý hoạt động taxi xuyên suốt từ tiếp nhận nhu cầu, bố trí nguồn xe, thực hiện chuyến, tính cước, thanh toán đến tài chính, đối soát, chăm sóc khách hàng và phân tích điều hành.

Hệ thống phục vụ hai nhóm nguồn lực:
- **DIRECT:** đội xe/tài xế do doanh nghiệp trực tiếp quản lý.
- **PARTNER:** xe/tài xế thuộc đơn vị đối tác theo hợp đồng.

Hai nhóm sử dụng nền tảng booking/dispatch/trip thống nhất; nghĩa vụ hợp đồng, doanh thu, công nợ, khấu trừ và thanh toán đối tác phải được phân biệt.

## 2. Phạm vi năng lực

### In scope — đề xuất
- Customer profile và các kênh đặt xe.
- Booking tức thời/đặt trước, địa chỉ, điểm đón, điểm đến, điểm dừng, sửa/hủy.
- Quản lý Driver, Vehicle, Fleet, Partner, Contract và điều kiện khai thác.
- Dispatch tự động/thủ công: eligibility, candidate ranking, Offer và Assignment.
- Driver App, GPS/location, Trip lifecycle, offline sync, incident/SOS.
- Pricing/Fare, Payment/Refund, Ledger, Receivable/Payable, Settlement.
- Control Tower, Customer Service/Case, notification và integration.
- IAM/RBAC, data scope, audit, reporting, monitoring và operations.

### Out of scope — chưa quyết định
Các nội dung như quản lý gara chuyên sâu, bảo hiểm/claim, payroll, HRM, kế toán tổng hợp, loyalty, quảng cáo, marketplace mở, giao hàng hoặc vận tải ngoài taxi cần quyết định riêng. Không mặc định đưa vào bản đầu.

## 3. Stakeholders và trách nhiệm

| Nhóm | Trách nhiệm cần xác nhận |
|---|---|
| Ban điều hành/Sponsor | Mục tiêu, phạm vi, ngân sách, cổng quyết định |
| Khối Vận hành/Điều độ | Quy trình booking, phân xe, trip, hủy, no-show, SLA |
| Đội xe | Hồ sơ và điều kiện Driver/Vehicle |
| Đối tác | Hợp đồng, nguồn lực, vùng hoạt động, đối soát |
| CSKH/Call Center | Tiếp nhận, thay đổi, khiếu nại, incident |
| Tài chính/Kế toán | Tariff, payment, ledger, receivable, settlement |
| Pháp chế/ATTT | Quyền riêng tư, lưu trữ, hợp đồng, tuân thủ |
| Product/BA/UX | Requirement, workflow, wireframe, acceptance |
| Architect/Engineering | Domain, API, data, deployment, NFR |
| QA/Operations | Test, release, monitoring, support, DR |

## 4. Các nguyên tắc domain

- `booking_channel` (HOTLINE/APP/WEB/API/...) không đồng nhất với `fleet_type` (DIRECT/PARTNER).
- Booking là yêu cầu; Assignment là phân công; Trip là lần thực hiện.
- DispatchRun là một lần chạy điều phối; Offer là lời mời tới một tài xế cụ thể.
- Fare/Quote, PaymentTransaction, LedgerEntry và SettlementLine có vòng đời/tính bất biến riêng.
- Driver/Vehicle availability độc lập với GPS freshness và connectivity.
- Mỗi aggregate có owner và nguồn dữ liệu chuẩn; không cho nhiều Module tự sửa cùng một dữ liệu.

## 5. Luồng nghiệp vụ tổng thể

1. **Acquire request:** khách gửi nhu cầu qua App/Web/Call Center/API.
2. **Validate & quote:** kiểm tra thông tin, vùng phục vụ, loại dịch vụ, thời điểm; báo giá nếu chính sách cho phép.
3. **Create Booking:** tạo ID, reference, audit, version.
4. **Dispatch:** lấy nguồn xe; eligibility filter; ranking; phát Offer.
5. **Assignment:** tài xế nhận; server xác thực lại điều kiện và đảm bảo atomic single-winner.
6. **Trip execution:** tài xế di chuyển, đến điểm đón, bắt đầu, thực hiện, hoàn tất hoặc xử lý ngoại lệ.
7. **Fare finalization:** tính cước theo tariff snapshot và dữ liệu chuyến được xác nhận.
8. **Payment:** thu tiền/ghi nhận tiền mặt/callback/hoàn tiền/đối soát.
9. **Finance:** Ledger, doanh thu/công nợ; phân loại DIRECT/PARTNER.
10. **Settlement:** tính, rà soát, duyệt, khóa và chi trả PARTNER.
11. **After-service:** biên nhận, đánh giá, CSKH/case, khiếu nại.
12. **Analytics:** KPI, báo cáo, hiệu quả điều phối và tài chính.

## 6. Domain và boundary cấp cao

| Domain | Aggregate chính | System of Record đề xuất |
|---|---|---|
| Customer | Customer, ContactPoint, SavedPlace | Customer Module |
| Booking | Booking, BookingStop, BookingRevision | Booking Module |
| Fleet | Driver, Vehicle, DriverVehicleLink, Shift, Eligibility | Fleet Module |
| Partner | Partner, Contract, ContractVersion | Partner Management |
| Dispatch | DispatchRun, CandidateSnapshot, Offer, Assignment | Dispatch Module |
| Trip | Trip, TripEvent, IncidentCase | Trip Module |
| Pricing | TariffVersion, FareQuote, FareResult, FareAdjustment | Pricing Module |
| Payment | PaymentTransaction, RefundTransaction, ProviderCallback | Payment Module |
| Finance | LedgerEntry, Receivable, Payable, Adjustment | Finance Module |
| Settlement | SettlementBatch, SettlementLine, Payout | Settlement Module |
| Customer Service | ServiceCase, Complaint, Resolution | Customer Service |
| Platform | User, Role, Policy, AuditEvent, Config | Platform/IAM |

Các aggregate và owner là thiết kế logic ban đầu; phải được Architect/PO xác nhận trước schema vật lý.

## 7. Quy tắc DIRECT/PARTNER

| Khía cạnh | Shared | Riêng biệt |
|---|---|---|
| Booking/Dispatch/Trip | Quy trình và engine chung | Chỉ khác điều kiện nguồn lực theo policy |
| Driver/Vehicle | Model năng lực chung | Chủ sở hữu, giấy tờ, quản lý, partner_id |
| Pricing | Cùng engine tính | Tariff/contract terms theo trường hợp |
| Payment | Adapter và transaction framework chung | Beneficiary, luồng tiền |
| Finance | Quy tắc audit chung | Revenue/cost/receivable/payable |
| Settlement | Quy trình dữ liệu chuẩn | PARTNER contract, share, deduction, payout |
| Reporting | KPI vận hành chung | Bộ lọc và báo cáo economics theo fleet type |

Không gán DIRECT/PARTNER từ kênh booking hoặc người tạo yêu cầu. Phân loại dựa trên Assignment/Trip và nguồn lực có hiệu lực.

## 8. Phụ thuộc cốt lõi

- Booking cần Customer, service catalog, location/geocoding, policy và pricing estimate (nếu áp dụng).
- Dispatch cần Booking + Fleet/Partner eligibility + realtime location/availability.
- Assignment cần transaction/concurrency guarantee.
- Trip cần Assignment, Driver App, event/GPS.
- Fare cần Trip facts + Tariff snapshot.
- Payment cần Fare + provider/cash policy.
- Finance cần Fare/Payment/contract snapshots.
- Settlement cần Ledger/contract snapshot và approval workflow.
- Reporting cần event/warehouse model và data quality.
- IAM/Audit/Observability là nền tảng xuyên suốt, không phải tính năng bổ sung cuối dự án.

## 9. Các quyết định bắt buộc chưa chốt

Xem [Decision Register](../04-DELIVERY-GOVERNANCE/04-decision-register.md). Không triển khai giả định cho:
- DIRECT/PARTNER priority và tỷ lệ phân phối.
- Vùng phục vụ, bán kính, GPS freshness, scoring, timeout, retry.
- Tariff, phụ phí, làm tròn, phí chờ/hủy/no-show.
- Payment methods, refund, cash collection.
- Partner share/deduction/settlement calendar.
- SLA/KPI target, traffic/load, RPO/RTO.
- Provider, hosting, maps, telephony, SMS/push.
- Privacy/legal retention và chính sách dữ liệu.

## 10. Cổng duyệt

- G0 Business scope: Sponsor/Business owners.
- G1 Operating model & E2E: Operations, Fleet, Partner, CSKH, Finance.
- G2 Architecture baseline: Architecture, Security, IT.
- G3 Feature readiness: Product/BA/UX/Engineering/QA.
- G4 UAT and release: Business sign-off, Finance reconciliation, Operations readiness.

Tài liệu này là bản tổng quan điều hướng, không thay thế Feature Specs hoặc phê duyệt chuyên môn.
