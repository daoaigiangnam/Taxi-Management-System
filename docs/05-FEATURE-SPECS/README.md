# FEATURE SPECIFICATIONS — DANH MỤC VÀ HƯỚNG DẪN

**Mã tài liệu:** TAXI-FS-INDEX  
**Phiên bản:** 0.1  
**Trạng thái:** DỰ THẢO — CHỜ PHÊ DUYỆT  
**Mục tiêu:** Điểm quản lý tập trung cho đặc tả từng Feature thuộc 10 Module.

## 1. Mẫu đặc tả chuẩn

- [00 — Mẫu Feature Specification](./00-feature-specification-template.md)

Mỗi Feature phải có một file đặc tả riêng, không gộp nhiều Feature độc lập vào một tài liệu lớn. Khi tạo file, giữ nguyên các phần áp dụng; ghi N/A kèm lý do nếu không áp dụng. Không xóa các mục kiểm soát bắt buộc về quyền, dữ liệu, lỗi, test và nghiệm thu.

## 2. Cấu trúc thư mục đề xuất

```text
docs/
└── 05-FEATURE-SPECS/
    ├── README.md
    ├── 00-feature-specification-template.md
    ├── M01-customer-and-booking/
    │   ├── CUS-001-customer-profile-search.md
    │   ├── BKG-001-create-immediate-booking.md
    │   ├── BKG-002-create-scheduled-booking.md
    │   └── ...
    ├── M02-fleet-and-partner/
    ├── M03-dispatch/
    ├── M04-driver-app-and-trip/
    ├── M05-pricing-and-payment/
    ├── M06-finance-and-settlement/
    ├── M07-control-tower-and-customer-service/
    ├── M08-platform-iam-audit/
    ├── M09-integration-and-notification/
    └── M10-reporting-and-analytics/
```

Các thư mục Module sẽ được tạo khi Feature đầu tiên của từng Module bắt đầu đặc tả; không tạo file rỗng chỉ để đủ cấu trúc.

## 3. Quy ước mã Feature

- `CUS-xxx`: Customer.
- `BKG-xxx`: Booking.
- Các Module khác dùng prefix theo Feature Catalog đã được Module Owner phê duyệt.
- ID là duy nhất và ổn định; không đổi ID chỉ vì đổi tên hiển thị.
- User Story/Task/Defect có mã riêng, liên kết về Feature ID.

## 4. Trạng thái Feature

| Trạng thái | Ý nghĩa |
|---|---|
| PLANNED | Có trong catalog, chưa đặc tả |
| DRAFT | Đang xây dựng đặc tả |
| IN_REVIEW | Đang chờ review |
| READY | Đạt Definition of Ready |
| IN_DEV | Đang phát triển |
| IN_QA | Đang kiểm thử |
| UAT | Đang nghiệm thu nghiệp vụ |
| ACCEPTED | Được nghiệm thu |
| DEPRECATED | Ngừng sử dụng, có phương án thay thế |

Không chuyển sang READY chỉ vì đã viết xong tài liệu. Phải đạt checklist DoR trong X-06.

## 5. Thứ tự ưu tiên đặc tả M01

Theo Feature Catalog hiện có trong [M01 README](../../02-DOMAIN-MODULES/01-customer-and-booking/README.md):

| ID | Feature | Priority hiện tại | Trạng thái đặc tả |
|---|---|---:|---|
| CUS-001 | Hồ sơ và tra cứu khách hàng | P1 | PLANNED |
| [BKG-001](./M01-customer-and-booking/BKG-001-create-immediate-booking.md) | Tạo Booking tức thời | P0 | DRAFT v0.1 |
| BKG-002 | Tạo Booking đặt trước | P1 | PLANNED |
| [BKG-003](./M01-customer-and-booking/BKG-003-validation-and-serviceability.md) | Kiểm tra hợp lệ và vùng phục vụ | P0 | DRAFT v0.1 |
| [BKG-004](./M01-customer-and-booking/BKG-004-pickup-destination-and-ordered-stops.md) | Điểm đón/điểm đến/các điểm dừng theo thứ tự | P0 | DRAFT v0.1 |
| [BKG-005](./M01-customer-and-booking/BKG-005-booking-amendment-and-change-request.md) | Sửa đổi Booking / yêu cầu thay đổi | P1 | DRAFT v0.1 |
| [BKG-006](./M01-customer-and-booking/BKG-006-cancel-booking-and-cancellation-policy.md) | Hủy Booking / chính sách hủy | P1 | DRAFT v0.1 |
| [BKG-007](./M01-customer-and-booking/BKG-007-duplicate-booking-detection-and-warning.md) | Phát hiện/cảnh báo Booking trùng | P2 | DRAFT v0.1 |
| BKG-008 | Tra cứu/chi tiết/dòng thời gian Booking | P0 | PLANNED |
| BKG-009 | Yêu cầu/tham chiếu báo giá | P1 | PLANNED |

**Lưu ý:** Priority và tên Feature được giữ theo catalog M01 hiện tại. Đây chưa phải xác nhận rằng toàn bộ chính sách, field và quy tắc đã được nghiệp vụ phê duyệt.

## 6. Nguyên tắc đặc tả

1. Bám quy trình TO.P1–TO.P9 và các nguồn nghiệp vụ có thể truy vết.
2. Tách rõ: nội dung nguồn KPMG 2015, quyết định Mai Linh đã phê duyệt, đề xuất thiết kế và TBD.
3. Không sao chép quy tắc của kênh này sang kênh khác nếu nguồn không hỗ trợ.
4. Không tự tạo enum State, field, API, error code hoặc threshold trái với X-01 đến X-05.
5. Module sở hữu dữ liệu là nơi duy nhất được quyền thay đổi dữ liệu nguồn.
6. DIRECT/PARTNER dùng chung nền tảng vận hành nhưng giữ cách ly scope và tài chính.
7. Feature chỉ READY khi đạt DoR trong X-06.
8. Feature chỉ ACCEPTED khi đạt DoD, QA và UAT tương ứng.

## 7. Liên kết quản trị

- [X-01 — State Machine](../../03-CROSS-CUTTING/01-state-machines-and-transition-rules.md)
- [X-02 — Data Architecture & ERD](../../03-CROSS-CUTTING/02-data-architecture-and-erd.md)
- [X-03 — API & Event Contracts](../../03-CROSS-CUTTING/03-api-and-event-contracts.md)
- [X-04 — Security, IAM & Audit](../../03-CROSS-CUTTING/04-security-iam-and-audit.md)
- [X-05 — Non-Functional Requirements](../../03-CROSS-CUTTING/05-non-functional-requirements.md)
- [X-06 — Delivery Governance](../../03-CROSS-CUTTING/06-delivery-governance.md)

**Nguyên tắc chốt:** Một Feature phải đủ rõ để Business xác nhận, UX thiết kế, DEV triển khai và QA kiểm thử theo cùng một cách hiểu.
