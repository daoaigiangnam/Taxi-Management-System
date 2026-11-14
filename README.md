# TAXI MANAGEMENT SYSTEM

> **ĐIỂM TRUY CẬP DUY NHẤT — PROJECT OVERVIEW**  
> Trang này là mục lục trung tâm. Từ đây có thể xem tổng thể, mở từng Module, đọc đặc tả Feature và quay lại trang này.

**Repository:** [Taxi-Management-System](https://github.com/daoaigiangnam/Taxi-Management-System)  
**Branch:** `main` · **Documentation baseline:** 0.1 · **Status:** DRAFT — cần Business/Architecture review.

## 1. Đọc từ đây
- **[01 — Thiết kế nghiệp vụ và hệ thống toàn diện Taxi Mai Linh (cơ sở KPMG Taxi tương lai)](./01-FULL-BUSINESS-SYSTEM-BLUEPRINT.md)** — cơ sở nghiệp vụ, 9 quy trình tương lai, nhóm yêu cầu TO.A–TO.J, bản đồ Module và tiêu chuẩn đặc tả cho đội phát triển.
- **[00 — OVERVIEW: Tổng quan hệ thống](./00-OVERVIEW.md)**
- [Danh mục 10 Module](#2-module-catalog)
- [Thiết kế xuyên Module](#3-thiet-ke-xuyen-module)
- [Feature Specifications](./docs/05-FEATURE-SPECS/README.md)
- [Sơ đồ hệ thống](#4-so-do-he-thong)

## 2. Module Catalog

| Module | Phạm vi | Tài liệu chi tiết |
|---|---|---|
| M01 — Customer & Booking | Khách hàng, tiếp nhận yêu cầu, đặt/sửa/hủy xe | [M01](./02-DOMAIN-MODULES/01-customer-and-booking/README.md) |
| M02 — Fleet & Partner | Tài xế, xe, đội xe, đối tác, hợp đồng | [M02](./02-DOMAIN-MODULES/02-fleet-and-partner/README.md) |
| M03 — Dispatch | Eligibility, Candidate, Ranking, Offer, Assignment | [M03](./02-DOMAIN-MODULES/03-dispatch/README.md) |
| M04 — Driver App & Trip | Ứng dụng tài xế, GPS, hành trình, sự cố | [M04](./02-DOMAIN-MODULES/04-driver-app-and-trip/README.md) |
| M05 — Pricing & Payment | Biểu giá, tính cước, thu/hoàn tiền | [M05](./02-DOMAIN-MODULES/05-pricing-and-payment/README.md) |
| M06 — Finance & Settlement | Ledger, công nợ, đối soát, thanh toán đối tác | [M06](./02-DOMAIN-MODULES/06-finance-and-settlement/README.md) |
| M07 — Control Tower & CSKH | Điều hành, giám sát, khiếu nại, SLA | [M07](./02-DOMAIN-MODULES/07-control-tower-and-customer-service/README.md) |
| M08 — Platform, IAM & Audit | Người dùng, phân quyền, cấu hình, nhật ký | [M08](./02-DOMAIN-MODULES/08-platform-iam-audit/README.md) |
| M09 — Integration & Notification | Maps, SMS, Voice, Payment, Partner API | [M09](./02-DOMAIN-MODULES/09-integration-and-notification/README.md) |
| M10 — Reporting & Analytics | KPI, Dashboard, báo cáo | [M10](./02-DOMAIN-MODULES/10-reporting-and-analytics/README.md) |

## 3. Thiết kế xuyên Module
- [State Machines & Transition Rules](./03-CROSS-CUTTING/01-state-machines-and-transition-rules.md)
- [Data Architecture & ERD](./03-CROSS-CUTTING/02-data-architecture-and-erd.md)
- [API & Event Contracts](./03-CROSS-CUTTING/03-api-and-event-contracts.md)
- Security, NFR, Delivery Governance: sẽ bổ sung và liên kết tại đây sau khi xác nhận cấu trúc.

## 4. Sơ đồ hệ thống
Quy trình nghiệp vụ chuẩn:

```mermaid
flowchart LR
 A["Customer / Channel"] --> B["Customer & Booking"]
 B --> C["Fleet & Partner"]
 C --> D["Dispatch"]
 D --> E["Driver App & Trip"]
 E --> F["Pricing & Payment"]
 F --> G["Finance & Settlement"]
 D --> H["Control Tower / CSKH"]
 E --> H
 G --> I["Reporting"]
 H --> I
```

## 5. Nguyên tắc kiến trúc nền tảng
1. DIRECT và PARTNER dùng chung nền tảng vận hành/điều phối; hợp đồng, doanh thu, công nợ, khấu trừ và settlement phải tách bạch.
2. `booking_channel` độc lập `fleet_type`.
3. Booking, DispatchRun, Offer, Assignment và Trip là các đối tượng/vòng đời riêng.
4. Eligibility filter trước Candidate ranking.
5. Một Booking tối đa một Assignment hiệu lực tại một thời điểm.
6. Module sở hữu dữ liệu của mình; không ghi trực tiếp vào dữ liệu nguồn của Module khác.
7. Lệnh ghi quan trọng cần authorization, validation, idempotency, concurrency control và audit.
8. Ledger và dữ liệu tài chính đã ghi sổ không bị sửa/xóa tùy tiện; điều chỉnh phải truy vết.
9. Chính sách chưa được phê duyệt phải ghi TBD, không biến giả định thành quy tắc chính thức.

## 6. Trình tự triển khai
Foundation & E2E → Domain/Module boundaries → State/Data/API/Security/NFR → Feature Specs → Implementation → Test/UAT → Release.
**Lát cắt ưu tiên:** Booking → Dispatch → Trip. Chưa bắt đầu code khi Feature chưa đạt Definition of Ready.

## 7. Trạng thái và quản trị tài liệu
Tất cả nội dung hiện là DRAFT baseline. Mỗi quyết định cần owner/approver; mỗi Feature cần business flow, UI/UX states, validation, rules, API/event, DB, permissions, errors, tests và acceptance evidence.


## 8. Cách đọc và thiết kế — từ tổng quan đến mức DEV triển khai

Tài liệu được tổ chức theo **5 tầng**, không đi thẳng từ danh sách Module sang code:

1. **Tầng A — System Overview:** mục tiêu, phạm vi, actor, hệ thống liên quan, nguyên tắc và ranh giới.
2. **Tầng B — End-to-End Business Flows:** các luồng xuyên hệ thống theo vai trò; luồng chính, luồng thay thế, ngoại lệ và điểm bàn giao giữa Module.
3. **Tầng C — Module Design:** mục tiêu Module, phạm vi sở hữu/không sở hữu, actor, dữ liệu, quy tắc nghiệp vụ, trạng thái, phụ thuộc và luồng nội bộ.
4. **Tầng D — Feature Specification:** đặc tả từng chức năng đủ chi tiết cho BA/UX/DEV/QA cùng hiểu và thực hiện.
5. **Tầng E — Build & Acceptance:** API/DB/UI, test cases, nghiệm thu, triển khai và bằng chứng hoàn thành.

### Thứ tự đọc bắt buộc cho đội dự án
`README → 00 Overview → E2E Flow → Module Overview → Module Detailed Flows → Feature Spec → API/Data/UI/Permission → Test & Acceptance`.

### Mỗi Module phải có bộ tài liệu
- `README.md`: Module Overview, phạm vi, actor, ownership, entities, dependencies, feature catalog.
- `01-business-rules.md`: thuật ngữ, quy tắc, điều kiện, công thức, quyết định và TBD.
- `02-workflows.md`: luồng chính, thay thế, ngoại lệ, swimlane và điểm bàn giao.
- `03-state-machine.md`: trạng thái, transition, guard, actor, side effect, lỗi.
- `04-data-model.md`: entity, field dictionary, quan hệ, constraint, ownership, audit.
- `05-permissions.md`: role/action/data scope và maker-checker.
- `06-integrations.md`: API, command, event, contract, idempotency, retry.
- `features/`: một file cho mỗi Feature theo mẫu thống nhất.

### Tiêu chuẩn để DEV nhận Feature
DEV không nhận việc chỉ từ tên chức năng hoặc mockup. Feature phải có: mục tiêu và phạm vi; actor/quyền; precondition/postcondition; happy path; alternate/exception/recovery; business rules; state transitions; UI states; field validation; API request/response/error; data changes/migration; audit; concurrency/idempotency; security/NFR; test cases; Given/When/Then acceptance criteria; dependency và decision links.

**Nguyên tắc:** phần nào chưa có căn cứ nghiệp vụ phải ghi `TBD + người quyết định + hạn chốt`; không tự điền bằng suy đoán của DEV.

---
**Quay lại đầu trang:** [Taxi Management System — README](./README.md)
