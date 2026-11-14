# BKG-006 — HỦY BOOKING VÀ CHÍNH SÁCH HỦY

**Mã tài liệu:** TAXI-BKG-006  
**Module sở hữu:** M01 — Khách hàng và Đặt xe  
**Epic:** Tiếp nhận và quản lý Booking đa kênh  
**Phiên bản:** 0.1  
**Trạng thái:** DRAFT — CHỜ NGHIỆP VỤ, M02, M03, M04, M05, M06 VÀ KIẾN TRÚC REVIEW  
**Priority theo Feature Catalog:** P1  
**Business Owner:** TBD  
**Technical Owner:** M01 Owner — TBD  
**Tài liệu liên quan:** [M01 Feature Catalog](../../../02-DOMAIN-MODULES/01-customer-and-booking/README.md) · [BKG-001 — Tạo Booking tức thời](./BKG-001-create-immediate-booking.md) · [BKG-005 — Sửa đổi Booking](./BKG-005-booking-amendment-and-change-request.md) · [BKG-003 — Kiểm tra hợp lệ và vùng phục vụ](./BKG-003-validation-and-serviceability.md) · [BKG-004 — Điểm đón/điểm đến/điểm dừng](./BKG-004-pickup-destination-and-ordered-stops.md) · [X-01 — State Machine](../../../03-CROSS-CUTTING/01-state-machines-and-transition-rules.md) · [X-02 — Data Architecture](../../../03-CROSS-CUTTING/02-data-architecture-and-erd.md) · [X-03 — API/Event](../../../03-CROSS-CUTTING/03-api-and-event-contracts.md)

> **Quy ước nguồn:** KPMG 2015 mô tả hủy yêu cầu qua Tổng đài (TO.P1), App (TO.P2), điểm tiếp thị (TO.P3), SMS (TO.P5), Chat (TO.P6) và Web (TO.P7). Nguồn nêu việc tra cứu đúng yêu cầu, hủy Booking, loại yêu cầu khỏi danh sách chờ/bản đồ/danh sách trên ứng dụng tài xế và đưa xe đã đăng ký về trạng thái trống. Một số bước mô tả tài xế tự hủy đăng ký hoặc khách không còn chờ. Đây là các hành động khác nhau. KPMG 2015 không xác định đầy đủ mức phí, điều kiện hoàn tiền, trách nhiệm bồi thường, ngưỡng thời gian hoặc chính sách hủy hiện hành. Mọi giá trị/chính sách chưa có căn cứ được để TBD.

---

## 0. Quản lý phiên bản và phê duyệt

| Vai trò | Người duyệt | Trạng thái |
|---|---|---|
| Business Owner — Vận hành Taxi | TBD | Pending |
| Product Owner M01 | TBD | Pending |
| Chủ Module M02 — Đội xe và Đối tác | TBD | Pending |
| Chủ Module M03 — Điều phối | TBD | Pending |
| Chủ Module M04 — Chuyến đi | TBD | Pending |
| Chủ Module M05 — Giá cước và Thanh toán | TBD | Pending |
| Chủ Module M06 — Tài chính và Quyết toán | TBD | Pending |
| Chủ Module M08 — Phân quyền và Kiểm toán | TBD | Pending |
| QA Lead | TBD | Pending |

| Phiên bản | Nội dung |
|---|---|
| 0.1 | Đặc tả hủy Booking, giải phóng nguồn lực, phân loại hủy và tác động liên Module |

---

## 1. Bối cảnh và mục tiêu

### 1.1. Vấn đề nghiệp vụ

Khách có thể không còn nhu cầu sử dụng taxi sau khi Booking đã được tạo. Việc hủy có thể xảy ra ở nhiều thời điểm:
- Booking vừa được gửi, đang kiểm tra.
- Booking đã được tiếp nhận nhưng chưa phát Dispatch.
- M03 đã phát Offer cho tài xế.
- Tài xế đã nhận Assignment.
- Tài xế đang trên đường đến đón.
- Tài xế đã đến điểm đón và đang chờ khách.
- Trip đã bắt đầu.
- Trip đã hoàn tất nhưng khách có khiếu nại/đề nghị điều chỉnh.

Ở mỗi thời điểm, hành động và hệ quả khác nhau. Hủy trước Dispatch chủ yếu cập nhật Booking; hủy khi có Offer/Assignment phải thu hồi nguồn lực do M03 sở hữu; hủy sau khi Trip được tạo cần M04 xử lý theo trạng thái Trip. Nếu có khoản thu, hoàn hoặc phí hủy thì M05/M06 phải xử lý theo chính sách đã phê duyệt.

### 1.2. Mục tiêu

1. Cung cấp cơ chế hủy Booking thống nhất cho các kênh được hỗ trợ.
2. Xác định chính xác Booking cần hủy.
3. Xác minh quyền, trạng thái Booking, Dispatch và Trip trước khi chấp nhận.
4. Ghi nhận ai hủy, hủy khi nào, lý do, kênh và bằng chứng.
5. Hủy Booking đúng một lần, chống lặp do SMS/Webhook/retry.
6. Gửi yêu cầu M03 thu hồi Offer/Assignment và giải phóng xe đúng quy trình.
7. Phân biệt khách hủy, tài xế từ chối/hủy nhận, Điều hành hủy và hệ thống tự hết hạn.
8. Chuyển sang M04 nếu Trip đã bắt đầu hoặc cần kết thúc sớm.
9. Gọi M05/M06 khi phát sinh phí, thanh toán, hoàn tiền hoặc điều chỉnh tài chính.
10. Không tạo phí hủy hoặc hoàn tiền nếu chính sách chưa được xác nhận.
11. Bảo đảm DIRECT/PARTNER giữ đúng hợp đồng và quy tắc tài chính riêng.
12. Có audit, báo cáo, xử lý ngoại lệ và khả năng đối soát.

### 1.3. Nguyên tắc bắt buộc

- **Cancellation ≠ Driver refusal:** tài xế từ chối không làm Booking tự động bị hủy.
- **Booking cancellation ≠ Trip cancellation:** Booking chưa bắt đầu và Trip đã bắt đầu có ranh giới xử lý khác nhau.
- **Single owner:** M01 sở hữu Booking; M03 sở hữu Offer/Assignment; M04 sở hữu Trip; M05 sở hữu Payment/Refund theo X-02.
- **Release only by owner:** M01 gửi lệnh yêu cầu M03 giải phóng nguồn lực, không tự sửa trạng thái xe/tài xế.
- **No silent financial effect:** không tự thu phí/hoàn tiền khi chưa có policy và quyết định M05/M06.
- **Idempotent:** một yêu cầu hủy được xử lý lặp an toàn, không tạo nhiều lần giải phóng/hoàn tiền.
- **Audit first:** lưu lịch sử trước/sau và actor; không xóa Booking đã phát sinh.
- **State-aware:** chỉ hủy theo ma trận trạng thái được phê duyệt.

---

## 2. Nguồn nghiệp vụ và căn cứ

### 2.1. Nội dung được KPMG 2015 hỗ trợ

| Quy trình/bước | Nội dung nguồn | Kết luận có thể sử dụng |
|---|---|---|
| TO.P1.1, P1.5 | Khách gọi Tổng đài để yêu cầu, nhắc, thay đổi hoặc hủy; phân loại yêu cầu hủy | Tổng đài là kênh tiếp nhận yêu cầu hủy |
| TO.P1.10 | Tổng đài viên tra cứu yêu cầu gần nhất theo số điện thoại hoặc điểm đón và chọn yêu cầu để hủy | Cần xác định chính xác Booking trước khi hủy |
| TO.P1.11 | Kiểm tra tài xế đã đăng ký hay chưa | Trạng thái xe đã đăng ký ảnh hưởng bước tiếp theo |
| TO.P1.16 | Tài xế nhận thông báo hủy/thay đổi; hủy thì kết thúc | Tài xế phải nhận thông báo khi Booking có thay đổi/hủy |
| TO.P1.20–P1.21 | Nếu khách không tiếp tục chờ thì hủy; Booking biến khỏi danh sách chờ/bản đồ/danh sách trên xe; xe đã đăng ký trở về trống | Hủy khách chờ phải cập nhật Dispatch view và nguồn lực |
| TO.P1.25 | Khi không có xe sau các lần chỉ định, Tổng đài thông báo và tiến hành hủy | Hủy do không thể phục vụ là trường hợp khác với khách chủ động hủy |
| TO.P2.16 | Khách báo không còn nhu cầu/đã đi xe khác; tài xế hủy yêu cầu trên App, ghi chú lý do và kết thúc quy trình | Nguồn mô tả hủy tại giai đoạn tài xế liên hệ khách; cần chuẩn hóa actor/quyền ở hệ thống mới |
| TO.P2.17 | Nếu người lên xe không phải người yêu cầu, tài xế hủy đăng ký đón để đưa yêu cầu về trạng thái chưa có xe đăng ký | Hủy đăng ký đón nhầm không phải hủy Booking của khách |
| TO.P3.7/P3.9 | Tài xế có thể từ chối chỉ định; lượt sắp tài chuyển sang tài xế kế tiếp theo thứ tự | Từ chối nhận lượt là sự kiện Dispatch/Queue, không phải khách hủy |
| TO.P4.7/P4.9 | Tài xế từ chối chỉ định; Tổng đài chỉ định xe khác | Tương tự, không hủy Booking chỉ vì tài xế từ chối |
| TO.P5.1–P5.3 | SMS yêu cầu và mã đặt chỗ | SMS là kênh có mã Booking để định danh |
| TO.P5.8 | Tài xế hủy đăng ký; Booking trở về chưa có xe đăng ký, hiển thị lại trong bán kính phát; xe trở về trống; lưu lịch sử | Tài xế hủy đăng ký không đồng nghĩa khách hủy |
| TO.P5.10–P5.12 | Luồng chờ lâu, khách không tiếp tục chờ và hủy yêu cầu | Có luồng khách chủ động dừng chờ; chi tiết cú pháp/kênh phải xác nhận |
| TO.P5.22–P5.23 (theo bảng bước) | Khách phản hồi/hủy trong luồng SMS; tài xế cập nhật lý do theo quy trình | Cần ghi nhận lý do và actor; dẫn chiếu bước nguồn cần đối chiếu |
| TO.P6.1–P6.2 | Chat hỗ trợ yêu cầu/thay đổi/hủy; tổng đài viên nhập mã đặt chỗ, chọn yêu cầu; xe đã đăng ký được đưa về trạng thái trống | Chat là kênh tiếp nhận hủy và cần Booking Code |
| TO.P7.3/P7.5 | Khách có thể chọn hủy trên Web; xe đã đăng ký (nếu có) được đưa về xe trống | Web là kênh hủy; cần giải phóng xe |
| TO.P7.11–P7.12 | Mô tả SMS `HUY [Mã số đặt xe]` và hệ thống tự hủy khi nhận SMS hoặc lệnh Web | Cú pháp lịch sử; không xem là cú pháp production nếu chưa xác nhận |
| TO.P7.15–P7.16 | Khi không có xe sau thời gian chỉ định, Tổng đài thông báo khách và xử lý hủy | Hủy do không có xe cần reason riêng |

### 2.2. Những điều KPMG không xác định

Nguồn năm 2015 không đủ căn cứ để kết luận:
- Hủy miễn phí trong bao lâu trước giờ đón.
- Mức phí hủy theo thời điểm, loại xe, quãng đường hoặc kênh.
- Có thu phí khi tài xế đã đến điểm đón hay không.
- Khách không xuất hiện (No-show) có phải chịu phí không.
- Tài xế/đối tác có được bồi hoàn khi khách hủy không.
- Phí hủy được phân bổ giữa DIRECT/PARTNER như thế nào.
- Khi đã thanh toán trước thì hoàn tiền trong bao lâu.
- Hủy một phần khi Booking có nhiều xe/điểm dừng.
- Quy trình khi Trip đã IN_PROGRESS.
- Có cho khách hủy khi tài xế đã nhận Assignment nhưng chưa đến hay không.
- Các ngưỡng thời gian, số lần liên hệ, thời gian chờ và SLA.
- Ai có thẩm quyền miễn phí/điều chỉnh phí.

Tất cả phải là quyết định của Mai Linh; không được hard-code theo giả định.

### 2.3. Phân biệt các loại kết thúc

| Nghiệp vụ | Actor khởi tạo | Booking | Dispatch | Trip |
|---|---|---|---|---|
| Khách hủy Booking | Khách/đại diện hợp lệ | Có thể CANCELLED | M03 thu hồi Offer/Assignment | Chưa bắt đầu; nếu đã bắt đầu chuyển M04 |
| Điều hành hủy Booking | Điều hành có quyền | Có thể CANCELLED/UNFULFILLED theo lý do | M03 giải phóng nguồn lực | M04 xử lý nếu Trip tồn tại |
| Hệ thống hết hạn | System theo policy | EXPIRED | M03 dọn Offer/Assignment nếu còn | Thường chưa bắt đầu |
| Không thể bố trí xe | M03/Điều hành | UNFULFILLED theo X-01 | Dispatch kết thúc/không có ứng viên | Không có Trip đang chạy |
| Tài xế từ chối Offer | Tài xế | Booking vẫn còn | Offer bị từ chối; tiếp tục tìm xe | Chưa có Trip |
| Tài xế hủy Assignment | Tài xế/M03 theo quyền | Booking vẫn còn nếu khách còn nhu cầu | M03 thu hồi Assignment và tìm xe khác | Trip chưa bắt đầu hoặc Trip được M04 xử lý |
| Khách không xuất hiện | Tài xế/Điều hành theo policy | Kết quả Booking theo quy tắc | M03 đóng Assignment | M04 ghi NO_SHOW nếu đủ điều kiện |
| Trip bị gián đoạn | Tài xế/Điều hành | Không tự đổi Booking thành CANCELLED | M03 xử lý Assignment nếu cần | M04 ghi INTERRUPTED/INCIDENT |
| Trip kết thúc sớm | Tài xế/khách/Điều hành theo quyền | Không sửa Booking gốc tùy tiện | M03 xử lý nguồn lực | M04 kết thúc theo quy trình riêng |

---

## 3. Phạm vi

### 3.1. Trong phạm vi

1. Khách yêu cầu hủy Booking qua kênh được hỗ trợ.
2. Tổng đài/Điều hành hủy thay khách theo quyền.
3. Xác định đúng Booking và xác minh quyền.
4. Kiểm tra Booking State, Dispatch status và Trip status.
5. Phân loại reason code và nguồn hủy.
6. Tạo Cancellation Request và ghi nhận quyết định.
7. Cập nhật Booking State theo X-01 khi đủ điều kiện.
8. Gửi lệnh M03 thu hồi Offer/Assignment và giải phóng xe.
9. Chuyển M04 xử lý nếu Trip đã được tạo/đã đến đón/đang thực hiện.
10. Gọi M05/M06 đánh giá phí, Payment, Refund, Adjustment nếu có policy.
11. Gửi thông báo cho khách, tài xế và nhân viên liên quan.
12. Lưu audit, bằng chứng, idempotency, event và kết quả xử lý.
13. Xử lý yêu cầu hủy trùng lặp, timeout, partial failure và reconciliation.

### 3.2. Ngoài phạm vi

- Tài xế từ chối Offer và thuật toán tìm xe thay thế: M03.
- Tài xế tự hủy Assignment và tái điều phối: M03, phối hợp M04 khi Trip đã tạo.
- Chính sách No-show và bằng chứng đến điểm đón: M04/Operations; BKG-006 chỉ tích hợp kết quả.
- Hủy/kết thúc Trip đã IN_PROGRESS: M04.
- Tính Fare/Cancel Fee, thu tiền/hoàn tiền: M05.
- Ghi sổ, công nợ, đối soát, chia sẻ doanh thu: M06.
- Khiếu nại và yêu cầu miễn/điều chỉnh phí: M07/M05/M06 theo ownership.
- Sửa Booking: BKG-005.
- Hết hạn Booking đặt trước: X-01/BKG-002 policy, không mặc nhiên coi là khách hủy.

### 3.3. Kênh hủy

| Channel | Nguồn lịch sử | Cách tiếp nhận đề xuất |
|---|---|---|
| Call Center | TO.P1.10 | Nhân viên tra cứu Booking, xác minh người yêu cầu, nhập lý do |
| App | TO.P2.16 | Khách gửi lệnh hủy từ Booking detail; xác nhận lại khi policy yêu cầu |
| Marketing Point | TO.P3 | Khách báo nhân viên/Điều hành; cần xác nhận luồng hủy trực tiếp và quyền |
| SMS | TO.P5, TO.P7.11–P7.12 | Parser nhận mã Booking và cú pháp đã được Mai Linh duyệt |
| Chat | TO.P6.1–P6.2 | Tổng đài viên xác minh mã Booking và nhập lệnh hủy |
| Web | TO.P7.3/P7.5 | Khách hủy trên Booking detail, có bước xác nhận |
| Partner API | Chưa đủ căn cứ KPMG | Chỉ bật khi hợp đồng/API policy quy định rõ actor và quyền |

SMS `HUY [Mã số đặt xe]` là ví dụ từ tài liệu năm 2015, không được coi là cú pháp hiện hành nếu chưa được xác nhận.

---

## 4. Actor, quyền và Data Scope

| Actor | Hành động | Permission đề xuất | Điều kiện |
|---|---|---|---|
| Khách hàng | Yêu cầu hủy Booking của mình | `booking.cancel.request.self` | Booking thuộc khách; xác minh theo policy |
| Người được ủy quyền | Yêu cầu hủy | `booking.cancel.request.delegated` | Có căn cứ ủy quyền |
| Tổng đài viên | Hủy thay khách | `booking.cancel.on_behalf` | Xác minh khách; reason bắt buộc |
| Điều hành | Hủy Booking trong phạm vi xử lý | `booking.cancel.operational` | Có quyền theo khu vực/ca |
| Tài xế | Từ chối/hủy Assignment | Quyền M03 | Không được dùng quyền này để hủy Booking của khách |
| Tài xế | Báo khách No-show/sự cố | Quyền M04 | Phải có evidence/flow hợp lệ |
| M01 Service | Quản lý Cancellation Request/Booking State | Internal permission | Theo Booking aggregate |
| M03 Service | Thu hồi Offer/Assignment, giải phóng nguồn lực | Internal permission | M03 ownership |
| M04 Service | Hủy/đóng Trip hoặc ghi No-show/Interrupted | Internal permission | M04 ownership |
| M05 Service | Đánh giá Cancel Fee/Payment/Refund | Internal permission | M05 ownership |
| M06 Service | Hạch toán/đối soát khoản phí/hoàn | Internal permission | M06 ownership |
| Auditor/CSKH | Đọc lịch sử hủy | `booking.cancel.audit.read` | Theo data scope |

Quy tắc:
- Khách không được hủy Booking của người khác.
- Tổng đài viên phải lưu actor thực hiện và actor yêu cầu nếu thao tác thay khách.
- Tài xế không có quyền tự đổi Booking thành CANCELLED chỉ vì từ chối Offer.
- Điều hành chỉ được hủy trong phạm vi quyền, phải có reason code.
- Quyền miễn phí/điều chỉnh phí là quyền riêng; không gộp với quyền hủy.
- DIRECT/PARTNER phải kiểm tra contract context; không suy ra từ channel.

---

## 5. Mô hình dữ liệu

### 5.1. Booking State

Theo X-01, Booking State có:
`DRAFT, SUBMITTED, VALIDATING, ACCEPTED, SCHEDULED, READY_FOR_DISPATCH, FULFILLING, COMPLETED, CANCELLED, EXPIRED, REJECTED, UNFULFILLED, ON_HOLD`.

Các trạng thái được phép hủy theo X-01 hiện tại:
- DRAFT/SUBMITTED/VALIDATING/ACCEPTED/SCHEDULED/READY_FOR_DISPATCH có thể chuyển CANCELLED nếu hủy hợp lệ.
- FULFILLING không được xử lý như hủy Booking thông thường.
- COMPLETED/CANCELLED/EXPIRED/REJECTED không được hủy lần nữa theo luồng thông thường.
- UNFULFILLED cần quy tắc riêng: trạng thái đã thể hiện không phục vụ được; không tạo cancellation trùng.
- ON_HOLD chỉ actor/policy có quyền mới được quyết định.

Không tự thêm Booking State mới như `CANCEL_PENDING` hoặc `CANCELLED_BY_DRIVER` nếu chưa phê duyệt X-01. Chi tiết yêu cầu hủy được lưu ở Cancellation Request.

### 5.2. Entity BookingCancellationRequest — đề xuất

Cần Data Architect xác nhận có bổ sung Entity riêng vào X-02 hay dùng Cancellation aggregate.

| Field logic | Ý nghĩa | Bắt buộc |
|---|---|---|
| cancellation_request_id | Định danh yêu cầu hủy | Có |
| booking_id | Booking cần hủy | Có |
| request_no | Số thứ tự yêu cầu hủy | Có |
| requested_by | Actor khởi tạo | Có |
| requester_type | CUSTOMER/AGENT/OPERATOR/SYSTEM/PARTNER | Có |
| source_channel | APP/WEB/CALL_CENTER/SMS/CHAT/API | Có |
| reason_code | Lý do hủy | Có |
| reason_text | Mô tả thêm | Không |
| requested_at | Thời điểm yêu cầu | Có |
| booking_revision | Revision được yêu cầu hủy | Có |
| booking_version | Version tại lúc yêu cầu | Có |
| dispatch_context_ref | Snapshot/ref M03 | Nếu có Dispatch |
| trip_context_ref | Snapshot/ref M04 | Nếu có Trip |
| policy_version | Phiên bản chính sách áp dụng | Khi policy đã có |
| fee_evaluation_ref | Kết quả M05 | Nếu có |
| status | Trạng thái Cancellation Request | Có |
| decision_by | Người/hệ thống quyết định | Khi có |
| decision_reason | Lý do quyết định | Khi có |
| cancelled_at | Thời điểm Booking chuyển CANCELLED | Khi thành công |
| correlation_id | Truy vết liên Module | Có |

### 5.3. Cancellation Request State đề xuất

| State | Ý nghĩa |
|---|---|
| SUBMITTED | Đã tiếp nhận yêu cầu |
| VALIDATING | Đang kiểm tra quyền/trạng thái |
| IMPACT_REVIEW | Đang kiểm tra Dispatch/Trip/Finance |
| AWAITING_OPERATOR_DECISION | Chờ Điều hành quyết định |
| AWAITING_CUSTOMER_CONFIRMATION | Chờ khách xác nhận phí/điều kiện nếu policy yêu cầu |
| APPROVED | Đã được chấp thuận |
| PROCESSING | Đang cập nhật Booking và yêu cầu Module liên quan |
| COMPLETED | Đã hoàn tất hủy theo policy |
| REJECTED | Bị từ chối |
| WITHDRAWN | Khách rút yêu cầu trước khi xử lý |
| FAILED | Lỗi xử lý, cần retry/reconcile |
| PARTIALLY_COMPLETED | Có bước liên Module chưa hoàn tất |
| DUPLICATE | Yêu cầu trùng, tham chiếu request gốc |

Đây là lifecycle của Cancellation Request, không phải Booking State.

### 5.4. Reason Code đề xuất

| Reason Code | Ý nghĩa | Nguồn |
|---|---|---|
| CUSTOMER_NO_LONGER_NEEDS | Khách không còn nhu cầu | KPMG TO.P2.16 |
| CUSTOMER_USED_OTHER_SERVICE | Khách đã đi xe khác | KPMG TO.P2.16 |
| CUSTOMER_CHANGED_PLAN | Khách thay đổi kế hoạch | Đề xuất, cần duyệt |
| CUSTOMER_CANCELLED_AFTER_WAIT | Khách không tiếp tục chờ | KPMG TO.P1.20–P1.21 |
| NO_VEHICLE_AVAILABLE | Không thể bố trí xe | KPMG TO.P1.24–P1.25/TO.P7.15–P7.16 |
| CUSTOMER_REQUESTED_BY_CALL | Khách yêu cầu qua Tổng đài | Channel/reason context |
| CUSTOMER_REQUESTED_BY_APP | Khách yêu cầu qua App | Channel/reason context |
| CUSTOMER_REQUESTED_BY_SMS | Khách yêu cầu qua SMS | Channel/reason context |
| CUSTOMER_REQUESTED_BY_CHAT | Khách yêu cầu qua Chat | Channel/reason context |
| CUSTOMER_REQUESTED_BY_WEB | Khách yêu cầu qua Web | Channel/reason context |
| DUPLICATE_BOOKING | Booking trùng | Cần BKG-007/policy |
| INVALID_BOOKING_DATA | Dữ liệu sai cần xử lý | Cần policy |
| OPERATIONAL_EXCEPTION | Ngoại lệ vận hành | Điều hành phê duyệt |
| SAFETY_OR_SECURITY | Lý do an toàn/an ninh | M04/M07 policy |
| PARTNER_REQUESTED | Đối tác yêu cầu hủy | Contract/API policy |
| SYSTEM_EXPIRY | Hết hạn tự động | System policy |
| OTHER | Lý do khác | Bắt buộc mô tả |

Không dùng reason code tài xế từ chối Offer làm reason khách hủy Booking.

---

## 6. Business Rules

| Rule ID | Quy tắc | Nguồn/căn cứ | Trạng thái |
|---|---|---|---|
| BKG006-R01 | Mọi yêu cầu hủy phải tham chiếu một Booking xác định | KPMG TO.P1.10/TO.P6.2 | Draft |
| BKG006-R02 | Nếu tra cứu theo phone/pickup trả về nhiều Booking, không tự chọn mù | Thiết kế an toàn | Draft |
| BKG006-R03 | Booking Code/ID là khóa ưu tiên khi có | KPMG TO.P6.2/TO.P7 | Draft |
| BKG006-R04 | Yêu cầu hủy phải lưu actor, channel, reason, time và idempotency | Audit/đề xuất | Draft |
| BKG006-R05 | Chỉ actor có permission và data scope mới được hủy | IAM | Draft |
| BKG006-R06 | Hủy hợp lệ ở các Booking State được X-01 cho phép | X-01 | Draft |
| BKG006-R07 | FULFILLING không được hủy theo luồng Booking thông thường | X-01 | Draft |
| BKG006-R08 | COMPLETED/CANCELLED/EXPIRED/REJECTED không được hủy lặp | X-01 | Draft |
| BKG006-R09 | Hủy Booking khi có Offer/Assignment phải yêu cầu M03 thu hồi | X-02 ownership/KPMG P1.21 | Draft |
| BKG006-R10 | M01 không tự chuyển Driver/Vehicle sang AVAILABLE | M03 ownership | Draft |
| BKG006-R11 | Tài xế từ chối Offer không tự làm Booking CANCELLED | Phân biệt nghiệp vụ | Draft |
| BKG006-R12 | Tài xế hủy Assignment phải do M03 xử lý và có thể tái điều phối nếu khách còn nhu cầu | M03 ownership | Draft |
| BKG006-R13 | Hủy Booking phải loại khỏi danh sách chờ/dispatch view theo kết quả M03 | KPMG TO.P1.21 | Draft |
| BKG006-R14 | Hủy Booking phải thông báo các actor/Module liên quan theo channel và policy | KPMG TO.P1.16 | Draft |
| BKG006-R15 | Không được xử lý Trip IN_PROGRESS như hủy Booking thông thường | X-01 Trip | Draft |
| BKG006-R16 | Trip CREATED/DRIVER_EN_ROUTE/ARRIVED/WAITING_FOR_CUSTOMER phải qua ma trận quyền và điều kiện M04 | X-01 Trip | Draft |
| BKG006-R17 | NO_SHOW chỉ được xác định theo policy, thời gian chờ/liên hệ/evidence; không tự suy ra từ việc khách không phản hồi | X-01 Trip | Draft |
| BKG006-R18 | Cancel Fee chỉ được áp dụng khi policy version hiệu lực và M05 trả kết quả | M05 ownership | Draft |
| BKG006-R19 | Không thu phí hủy từ M01; M01 không tính tiền | X-02 | Draft |
| BKG006-R20 | Payment/Refund/Adjustment do M05 quản lý; Ledger/Settlement do M06 quản lý | X-02 | Draft |
| BKG006-R21 | Không hoàn tiền chỉ dựa trên Booking State CANCELLED; phải căn cứ Payment và refund policy | M05 | Draft |
| BKG006-R22 | DIRECT/PARTNER áp dụng contract/policy riêng; không gộp settlement | X-02 | Draft |
| BKG006-R23 | Một Booking chỉ có một kết quả hủy hợp lệ; yêu cầu lặp phải idempotent | X-03 | Draft |
| BKG006-R24 | Hai yêu cầu hủy đồng thời phải được xử lý bằng expected version/locking | X-03 | Draft |
| BKG006-R25 | Nếu M03 timeout, không được giả định Assignment đã giải phóng | Resilience | Draft |
| BKG006-R26 | Nếu M01 commit CANCELLED nhưng downstream chưa hoàn tất, request phải giữ trạng thái partial/reconcile rõ ràng | Saga/Outbox | Draft |
| BKG006-R27 | Hủy từ SMS phải xác thực cú pháp/mã Booking và chống replay; cú pháp KPMG 2015 cần xác nhận | KPMG TO.P7.11–P7.12 | Draft |
| BKG006-R28 | Không cho phép hủy Booking của khách khác qua lệnh SMS chỉ biết mã nếu policy xác thực chưa đủ | Security | Draft |
| BKG006-R29 | Hủy thay khách qua Call Center/Chat phải lưu nhân viên thao tác và bằng chứng xác minh | KPMG + audit | Draft |
| BKG006-R30 | Hủy do không có xe phải có reason riêng, không gán thành khách chủ động hủy | KPMG TO.P1.24–P1.25 | Draft |
| BKG006-R31 | Hết hạn Booking là EXPIRED theo policy, không tự gán CANCELLED_BY_CUSTOMER | X-01 | Draft |
| BKG006-R32 | Hủy một phần chỉ hỗ trợ khi mô hình Booking/Assignment cho phép và policy được duyệt | TBD | Draft |
| BKG006-R33 | Khi Trip đã bắt đầu, yêu cầu khách dừng chuyến phải chuyển M04 xử lý Trip interruption/early completion | X-01 | Draft |
| BKG006-R34 | Cancellation Request và audit không bị xóa cứng | X-02 | Draft |
| BKG006-R35 | Event hủy phải có booking version, cancellation request ID và correlation ID | X-03 | Draft |
| BKG006-R36 | Notification thất bại không được làm mất trạng thái hủy đã commit; phải retry/reconcile | X-03 | Draft |

---

## 7. Ma trận hủy theo trạng thái Booking/Dispatch/Trip

> Ma trận dưới đây là thiết kế đề xuất dựa trên X-01 và ownership X-02. Cần Business Owner và các Module phê duyệt.

| Booking State | Dispatch/Trip Context | Xử lý đề xuất |
|---|---|---|
| DRAFT | Chưa Dispatch | Cho phép hủy/đóng bản nháp theo quyền |
| SUBMITTED | Chưa Dispatch | Có thể hủy; kiểm tra request đã được tiếp nhận chưa |
| VALIDATING | Đang validate | Có thể hủy; hủy tác vụ validate nếu an toàn |
| ACCEPTED | Chưa có DispatchRun | Có thể hủy; ghi Cancellation Request |
| SCHEDULED | Chưa đến thời điểm Dispatch | Có thể hủy; kiểm tra policy đặt trước/fee |
| READY_FOR_DISPATCH | Chưa có Offer | Có thể hủy; thông báo M03 nếu đã tạo DispatchRun |
| READY_FOR_DISPATCH | Offer đang mở | M03 thu hồi tất cả Offer hiệu lực; chờ xác nhận kết quả theo hợp đồng |
| READY_FOR_DISPATCH | Assignment đã nhận, Trip chưa tạo | M03 hủy Assignment/giải phóng nguồn lực; xử lý thông báo tài xế |
| FULFILLING | Trip CREATED/DRIVER_EN_ROUTE | M04/M03 review; không chỉ cập nhật Booking |
| FULFILLING | Trip ARRIVED/WAITING_FOR_CUSTOMER | M04 xử lý hủy/no-show theo quyền và evidence; M05 đánh giá phí nếu có |
| FULFILLING | Trip IN_PROGRESS | Không hủy Booking; M04 xử lý INTERRUPTED/early completion |
| COMPLETED | Trip COMPLETED | Không hủy; mở case hậu kiểm/điều chỉnh nếu có |
| CANCELLED | Đã hủy | Idempotent: trả kết quả đã có; không xử lý lần hai |
| EXPIRED | Hết hạn | Không hủy lặp; nếu khách vẫn cần xe, tạo Booking mới |
| REJECTED | Không tiếp nhận | Không hủy lặp |
| UNFULFILLED | Không phục vụ được | Giữ kết quả không phục vụ; không gán nhầm khách hủy |
| ON_HOLD | Đang chờ xác minh | Chỉ actor có quyền quyết định sau khi xử lý lý do Hold |

### 7.1. Các ranh giới cần giữ

- **Offer chưa được nhận:** M03 thu hồi Offer.
- **Assignment đã được nhận nhưng Trip chưa bắt đầu:** M03 xử lý hủy Assignment và nguồn lực; cần ghi nhận tài xế đã nhận thông báo.
- **Trip đã tạo nhưng tài xế chưa đến:** M04 phải quyết định trạng thái Trip; M03 đồng bộ nguồn lực.
- **Tài xế đã đến và đang chờ:** phải kiểm tra bằng chứng/điều kiện trước khi đánh dấu No-show hoặc hủy.
- **Trip đang chạy:** không được dùng Booking CANCELLED để che lấp hành trình đã thực hiện.
- **Trip hoàn tất:** không sửa lịch sử; mọi tranh chấp/điều chỉnh qua quy trình hậu kiểm.

---

## 8. Phân loại hủy và chính sách phí

### 8.1. Nhóm loại hủy

| Nhóm | Ví dụ | Booking outcome đề xuất | Owner quyết định |
|---|---|---|---|
| Customer initiated | Khách không còn nhu cầu | CANCELLED nếu đủ điều kiện | M01/Operations |
| Customer after wait | Khách không muốn chờ nữa | CANCELLED | Operations/M03 |
| Operator initiated | Điều hành hủy do tình huống vận hành | CANCELLED/UNFULFILLED theo reason | Operations |
| System expiry | Hết hạn đặt trước/chờ xác nhận | EXPIRED | M01/Operations |
| No vehicle available | Không tìm được xe | UNFULFILLED theo X-01 | M03 |
| Driver refusal | Tài xế từ chối Offer | Booking giữ trạng thái phù hợp; Dispatch tiếp tục | M03 |
| Driver cancels Assignment | Tài xế hủy nhận chuyến | Booking còn nhu cầu nếu chưa hủy; M03 tái điều phối | M03 |
| Customer no-show | Khách không xuất hiện | Trip NO_SHOW theo M04; Booking projection theo policy | M04/Operations |
| Trip interrupted | Sự cố/không thể tiếp tục | Trip INTERRUPTED/INCIDENT | M04 |

### 8.2. Chính sách phí hủy — chưa được xác nhận

Hệ thống cần hỗ trợ cấu hình policy theo các chiều có thể áp dụng:
- DIRECT/PARTNER.
- Khu vực/đơn vị khai thác.
- Loại dịch vụ/loại xe.
- Booking immediate/scheduled.
- Thời điểm khách yêu cầu hủy so với thời gian đón.
- Dispatch state/Assignment state.
- Trip state (đã đến/chờ khách/đã bắt đầu).
- Lý do hủy và bên chịu trách nhiệm.
- Hợp đồng khách hàng/đối tác.
- Chính sách miễn/giảm phí được duyệt.

**Không đặt ngưỡng thời gian hay số tiền mặc định.** Các tham số phải do Business Owner và Finance phê duyệt.

### 8.3. Ma trận phí cần Mai Linh quyết định

| Tình huống | Có phí? | Mức phí/công thức | Người chịu | Owner |
|---|---|---|---|---|
| Khách hủy trước Dispatch | TBD | TBD | TBD | Operations/M05 |
| Khách hủy khi Offer đang mở | TBD | TBD | TBD | Operations/M03/M05 |
| Khách hủy sau khi tài xế nhận | TBD | TBD | TBD | Operations/M05 |
| Khách hủy khi tài xế đang đến | TBD | TBD | TBD | Operations/M05 |
| Khách hủy sau khi tài xế đến | TBD | TBD | TBD | Operations/M04/M05 |
| Khách No-show | TBD | TBD | TBD | Operations/M04/M05 |
| Không có xe phục vụ | Không tự thu khách; cần policy | TBD | TBD | Operations |
| Tài xế hủy Assignment | Không tự coi là khách hủy | TBD | TBD | M02/M03 |
| Sự cố an toàn/xe hỏng | TBD | TBD | TBD | Operations/M04 |
| Đối tác hủy Booking | Theo hợp đồng | TBD | TBD | M02/M06 |

### 8.4. Quy trình tính phí/hoàn tiền

1. M01 tạo Cancellation Request và gửi context sang M05.
2. M05 xác định policy/tariff/contract version hiệu lực và trạng thái Payment.
3. M05 trả kết quả: no fee, cancel fee, refund eligible, adjustment required hoặc manual review.
4. Nếu khách phải xác nhận phí/điều kiện, M01 gửi nội dung qua channel.
5. Khi đủ điều kiện, M05 thực hiện Payment/Refund theo API riêng.
6. M06 nhận event tài chính để hạch toán/đối soát theo quy định.
7. M01 chỉ hiển thị kết quả tài chính do M05/M06 cung cấp.
8. Nếu refund thất bại, Booking vẫn giữ kết quả hủy hợp lệ; Refund có trạng thái riêng để retry/reconcile.

Không gộp Booking cancellation và Refund vào một transaction xuyên Module.

---

## 9. Luồng nghiệp vụ

### 9.1. Tiền điều kiện

- Booking tồn tại.
- Actor có quyền và data scope.
- Xác định được trạng thái Booking.
- Có Dispatch/Trip context nếu Booking đã được giao cho M03/M04.
- Reason code hợp lệ.
- Kênh có thể xác thực yêu cầu.
- Nếu có policy phí, policy version được xác định rõ.

### 9.2. Luồng chính — Khách hủy Booking trước khi Trip bắt đầu

| Bước | Actor | Hành động | Hệ thống xử lý | Kết quả |
|---|---|---|---|---|
| 1 | Khách/nhân viên | Chọn Booking cần hủy | Tra cứu theo ID/Code hoặc tiêu chí cho phép | Booking target |
| 2 | M01 | Xác minh quyền | Kiểm tra customer ownership/employee scope | Được phép/từ chối |
| 3 | Khách/nhân viên | Chọn lý do và xác nhận hủy | Thu reason/evidence | Cancellation Request |
| 4 | M01 | Kiểm tra state/version | Kiểm tra Booking, Dispatch, Trip context | VALIDATING |
| 5 | M01 | Đánh giá tác động | Xác định có Offer/Assignment/Trip/Payment | IMPACT_REVIEW |
| 6 | M03/M04 | Review trạng thái vận hành | M03 thu hồi nguồn lực; M04 quyết định nếu Trip đã tạo | Kết quả Module |
| 7 | M05 | Đánh giá phí/hoàn nếu policy có | Trả fee/refund eligibility | Fee result |
| 8 | Khách/nhân viên | Xác nhận điều kiện nếu cần | Lưu bằng chứng xác nhận | Approved |
| 9 | M01 | Áp dụng hủy Booking | Cập nhật state CANCELLED theo X-01 và ghi audit | Booking cancelled |
| 10 | M01 | Phát Event | Outbox phát BookingCancelled | Consumer nhận |
| 11 | M03/M04/M05/M09 | Đồng bộ tác động | Giải phóng nguồn lực, xử lý Trip/Payment/Notification | Kết quả liên Module |
| 12 | M01 | Tổng hợp trạng thái | Hoàn tất hoặc đánh dấu partial/reconcile | Cancellation Request result |
| 13 | Channel | Thông báo | Gửi kết quả cho khách/nhân viên | Kết thúc |

**Lưu ý:** Trình tự commit Booking và xử lý M03/M04 cần được chốt trong Saga. Không được thiết kế theo giả định distributed transaction. Có thể dùng trạng thái Cancellation Request PROCESSING/PARTIALLY_COMPLETED để phản ánh tác vụ đang xử lý, nhưng Booking State không tự thêm enum mới.

### 9.3. Luồng hủy trước khi Dispatch

1. Khách/nhân viên gửi lệnh hủy.
2. M01 kiểm tra Booking State và quyền.
3. Xác nhận chưa có Offer/Assignment hiệu lực.
4. M01 ghi Cancellation Request.
5. M01 cập nhật Booking CANCELLED theo X-01.
6. M01 phát BookingCancelled qua Outbox.
7. M03 nhận event để đóng DispatchRun/loại khỏi danh sách tìm xe nếu đã tạo.
8. M09 gửi thông báo.
9. M05 chỉ được gọi nếu có policy/Payment liên quan.

### 9.4. Luồng hủy khi Offer đang mở

1. M01 nhận lệnh hủy.
2. M03 trả danh sách Offer còn hiệu lực theo Booking.
3. M03 thu hồi Offer, ngăn tài xế nhận thêm.
4. M03 xác nhận kết quả thu hồi hoặc trả trạng thái pending.
5. M01 áp dụng hủy theo Saga policy.
6. Event đồng bộ tới các consumer.
7. Nếu Offer được nhận đồng thời trong lúc hủy, xử lý race condition theo expected version/locking ở M03; không tự giả định Offer đã thu hồi.

### 9.5. Luồng hủy sau khi Assignment đã được nhận nhưng Trip chưa bắt đầu

1. M01 tiếp nhận Cancellation Request.
2. M03 kiểm tra Assignment hiệu lực và trạng thái Driver/Vehicle.
3. M03 quyết định thu hồi Assignment và giải phóng nguồn lực theo policy.
4. Nếu Trip đã được tạo, M04 phải xác nhận cách đóng Trip tương ứng.
5. M01 áp dụng Booking cancellation khi đủ điều kiện theo Saga.
6. M05 đánh giá Cancel Fee/Refund nếu policy có.
7. M09 thông báo khách và tài xế.
8. Ghi lại trạng thái Assignment/Trip trước và sau; không xóa lịch sử.

### 9.6. Luồng khách hủy khi tài xế đã đến/đang chờ

1. Khách hoặc tổng đài gửi yêu cầu hủy.
2. M01 lấy Trip status từ M04.
3. Nếu Trip ARRIVED/WAITING_FOR_CUSTOMER, chuyển M04 kiểm tra quyền và điều kiện.
4. M04 ghi nhận việc khách hủy hoặc kết quả No-show theo policy, evidence và thời gian chờ.
5. M03 đóng/giải phóng Assignment theo kết quả M04.
6. M05 đánh giá phí/hoàn theo policy.
7. M01 cập nhật Booking projection theo kết quả chuẩn.
8. Gửi thông báo và lưu audit.

Không tự chuyển sang NO_SHOW chỉ vì khách không phản hồi hoặc tài xế báo không thấy khách.

### 9.7. Luồng yêu cầu hủy khi Trip đã IN_PROGRESS

1. M01 phát hiện Trip IN_PROGRESS.
2. BKG-006 không chuyển Booking thành CANCELLED bằng luồng hủy trước chuyến.
3. Chuyển yêu cầu tới M04 Trip Change/Trip Interruption.
4. M04 đánh giá an toàn, điểm dừng, khả năng kết thúc sớm và actor có quyền.
5. M04 ghi TripEvent/Trip state tương ứng (ví dụ INTERRUPTED/INCIDENT hoặc kết thúc theo policy).
6. M05 tính FareResult theo hành trình thực tế và policy.
7. M06 xử lý điều chỉnh tài chính nếu có.
8. M01 chỉ nhận kết quả projection; không ghi đè lịch sử Booking/Trip.

### 9.8. Luồng tài xế từ chối/hủy nhận chuyến — không phải khách hủy

1. Tài xế từ chối Offer hoặc hủy Assignment qua ứng dụng tài xế.
2. M03 ghi Offer/Assignment event và reason.
3. M03 cập nhật nguồn lực và tìm tài xế khác nếu Booking còn nhu cầu.
4. Booking tiếp tục trạng thái phù hợp; không chuyển CANCELLED.
5. Nếu không tìm được xe theo policy, M03 phát kết quả NO_CANDIDATE/UNFULFILLED để M01 xử lý.
6. Khách được thông báo về tình hình xe; không báo “khách đã hủy”.

### 9.9. Luồng hủy do không có xe

1. M03/Điều hành xác định không có xe đủ điều kiện sau các bước đã được duyệt.
2. M03 phát kết quả không thể phục vụ.
3. Tổng đài thông báo khách và cho phép tiếp tục chờ/phương án khác nếu policy có.
4. Nếu khách không tiếp tục chờ, reason là CUSTOMER_CANCELLED_AFTER_WAIT.
5. Nếu hệ thống/Điều hành kết thúc vì không thể phục vụ, reason là NO_VEHICLE_AVAILABLE và Booking outcome theo X-01 là UNFULFILLED.
6. Không ghi nhận nhầm khách chủ động hủy khi nguyên nhân là thiếu xe.

### 9.10. Luồng SMS hủy

1. SMS Gateway nhận tin nhắn.
2. Parser xác định cú pháp theo cấu hình đã duyệt.
3. Tách mã Booking và thông tin xác thực/chống giả mạo nếu được yêu cầu.
4. Tra cứu Booking và kiểm tra trạng thái.
5. Tạo idempotency key theo message/provider reference.
6. M01 xử lý như Cancellation Request chuẩn.
7. SMS Gateway trả xác nhận đã nhận yêu cầu hoặc kết quả xử lý; không báo đã hủy nếu mới chỉ tiếp nhận.
8. Lưu inbound/outbound message reference và delivery status.

Cú pháp lịch sử KPMG: `HUY [Mã số đặt xe]`. Phải xác nhận lại trước khi dùng.

### 9.11. Luồng thay thế

| Mã | Tình huống | Xử lý |
|---|---|---|
| ALT-01 | Khách có Booking Code | Tra cứu trực tiếp |
| ALT-02 | Khách không có Code | Tra cứu theo phone/pickup; nếu nhiều kết quả thì xác nhận |
| ALT-03 | Khách hủy trước khi Dispatch | M01 xử lý; M03 đóng lượt nếu có |
| ALT-04 | Khách hủy khi Offer đang mở | M03 thu hồi Offer |
| ALT-05 | Khách hủy sau Assignment | M03 review Assignment |
| ALT-06 | Khách hủy khi Trip chưa bắt đầu | M04 review nếu Trip đã tạo |
| ALT-07 | Khách muốn tiếp tục chờ thay vì hủy | Không hủy; giữ Booking và cập nhật yêu cầu |
| ALT-08 | Khách rút yêu cầu hủy trước khi commit | WITHDRAWN nếu policy cho phép |
| ALT-09 | Điều hành hủy do sự cố khai thác | Reason OPERATIONAL_EXCEPTION |
| ALT-10 | Hủy do không có xe | UNFULFILLED, không gán khách chủ động hủy |
| ALT-11 | Partner gửi lệnh hủy | Xác thực partner, contract và idempotency |
| ALT-12 | Booking có nhiều xe | Xử lý toàn bộ hay một phần theo policy/mô hình dữ liệu |

### 9.12. Luồng ngoại lệ

| Mã | Tình huống | Error/Handling |
|---|---|---|
| EX-01 | Không tìm thấy Booking | BOOKING_NOT_FOUND |
| EX-02 | Nhiều Booking khớp tiêu chí | BOOKING_MATCH_AMBIGUOUS |
| EX-03 | Actor không có quyền | PERMISSION_DENIED |
| EX-04 | Booking đã CANCELLED | Trả kết quả hủy trước đó; idempotent |
| EX-05 | Booking COMPLETED | BOOKING_NOT_CANCELLABLE |
| EX-06 | Booking EXPIRED/REJECTED | Không hủy lặp |
| EX-07 | Booking FULFILLING/Trip IN_PROGRESS | TRIP_CHANGE_REQUIRED |
| EX-08 | Trip ARRIVED/WAITING_FOR_CUSTOMER | TRIP_REVIEW_REQUIRED |
| EX-09 | Assignment thay đổi đồng thời | DISPATCH_CONTEXT_CHANGED |
| EX-10 | M03 thu hồi Offer thất bại | DISPATCH_CANCELLATION_PENDING |
| EX-11 | M03 timeout | DOWNSTREAM_SERVICE_TIMEOUT; query/retry |
| EX-12 | Assignment đã được tài xế nhận đồng thời khi khách hủy | Concurrency conflict; M03 quyết định |
| EX-13 | M04 không xác định được Trip state | TRIP_STATE_UNKNOWN |
| EX-14 | M04 từ chối hủy Trip | TRIP_CANCELLATION_REJECTED |
| EX-15 | Khách chưa xác nhận phí/điều kiện | CANCELLATION_CONFIRMATION_REQUIRED |
| EX-16 | M05 không tính được phí | FEE_EVALUATION_FAILED; không tự thu |
| EX-17 | Payment/Refund đang xử lý | PAYMENT_STATUS_PENDING |
| EX-18 | Refund thất bại | REFUND_FAILED; retry/reconcile |
| EX-19 | Partner không được phép hủy Booking | PARTNER_PERMISSION_DENIED |
| EX-20 | SMS sai cú pháp | SMS_SYNTAX_INVALID |
| EX-21 | SMS không xác định Booking | BOOKING_CODE_INVALID |
| EX-22 | SMS bị replay/trùng | IDEMPOTENCY_REPLAY |
| EX-23 | Hai lệnh hủy đồng thời | CANCELLATION_CONFLICT |
| EX-24 | Client timeout sau khi M01 commit | REQUEST_RESULT_UNKNOWN; query idempotency |
| EX-25 | M01 commit nhưng M03/M04 chưa hoàn tất | CANCELLATION_PARTIAL |
| EX-26 | Notification không gửi được | NOTIFICATION_DELIVERY_FAILED; retry |
| EX-27 | Không đủ bằng chứng No-show | NO_SHOW_EVIDENCE_INSUFFICIENT |
| EX-28 | Khách yêu cầu hủy một phần | PARTIAL_CANCELLATION_NOT_SUPPORTED nếu chưa bật |
| EX-29 | Reason code không hợp lệ | CANCELLATION_REASON_INVALID |
| EX-30 | Booking PARTNER hết quyền hủy theo hợp đồng | CONTRACT_POLICY_DENIED |
| EX-31 | Booking ON_HOLD | OPERATOR_REVIEW_REQUIRED |
| EX-32 | Hủy do thiếu xe bị ghi nhầm khách hủy | CANCELLATION_REASON_MISMATCH; yêu cầu sửa qua quyền đặc biệt/audit |

### 9.13. Hậu điều kiện

**Hủy thành công**
- Booking chuyển CANCELLED khi được phép theo X-01.
- Cancellation Request có kết quả COMPLETED.
- M03 xác nhận Offer/Assignment được xử lý theo yêu cầu.
- M04 xác nhận Trip đã xử lý nếu tồn tại.
- M05/M06 có trạng thái phí/hoàn/đối soát rõ ràng nếu phát sinh.
- Khách/tài xế/nhân viên nhận thông báo phù hợp.
- Audit/Event có thể truy vết.

**Hủy chưa hoàn tất**
- Cancellation Request ở PROCESSING/PARTIALLY_COMPLETED/FAILED.
- Không báo khách “đã hủy hoàn toàn” khi nguồn lực/Trip còn trạng thái chưa rõ.
- Có retry/reconciliation/escalation.
- Không tạo phí/hoàn tiền trùng.

---

## 10. UI/UX

### 10.1. Màn hình Booking Detail

Hiển thị:
- Booking Code/ID.
- Booking State.
- Dispatch status: Offer/Assignment từ M03.
- Trip status từ M04 nếu có.
- Payment status từ M05 nếu có.
- Channel, fleet_type DIRECT/PARTNER và contract context ở mức quyền cho phép.
- Nút Hủy chỉ hiển thị khi policy/state cho phép.
- Nếu không cho hủy, hiển thị lý do và hướng dẫn liên hệ.

### 10.2. Dialog xác nhận hủy

Nội dung:
- Mã Booking và tóm tắt chuyến.
- Lý do hủy.
- Cảnh báo tác động: tài xế/xe đã nhận, xe đang đến, có thể có điều kiện phí nếu policy đã được duyệt.
- Phí/hoàn tiền chỉ hiển thị từ M05 với policy version.
- Xác nhận rõ hành động; không dùng một nút chung cho “Tài xế từ chối” và “Khách hủy”.

### 10.3. Màn hình Điều hành/Tổng đài

- Tra cứu Booking và xem Dispatch/Trip context.
- Chọn reason code.
- Ghi chú và evidence.
- Phân biệt Customer Requested / Operator Cancel / No Vehicle / System Expiry / Driver Refusal.
- Hiển thị trạng thái thu hồi Offer/Assignment.
- Hiển thị kết quả fee/refund từ M05.
- Cho phép retry/reconcile theo quyền; không cho sửa trực tiếp state của M03/M04.

### 10.4. Message Code đề xuất

| Code | Nội dung |
|---|---|
| CANCEL-001 | Bạn có chắc chắn muốn hủy yêu cầu đặt xe này? |
| CANCEL-002 | Yêu cầu hủy đã được tiếp nhận |
| CANCEL-003 | Booking đã được hủy |
| CANCEL-004 | Không thể hủy ở trạng thái hiện tại |
| CANCEL-005 | Tài xế đã nhận chuyến; hệ thống đang kiểm tra khả năng hủy |
| CANCEL-006 | Tài xế đang trên đường đến điểm đón |
| CANCEL-007 | Chuyến đi đã bắt đầu; vui lòng liên hệ Tổng đài |
| CANCEL-008 | Yêu cầu đang chờ Điều hành xác nhận |
| CANCEL-009 | Không tìm thấy mã đặt xe |
| CANCEL-010 | Yêu cầu hủy đã được xử lý trước đó |
| CANCEL-011 | Chưa thể xác nhận đã hủy; hệ thống đang đồng bộ với bộ phận điều hành |
| CANCEL-012 | Không tìm được xe phục vụ; yêu cầu đã được kết thúc theo quy trình |
| CANCEL-013 | Có điều kiện phí hủy; vui lòng xác nhận theo thông tin hiển thị |
| CANCEL-014 | Yêu cầu hoàn tiền đang được xử lý |

Nội dung cuối cùng phải được Business Owner duyệt.

---

## 11. API và Event Contract

> Endpoint/payload là đề xuất, phải đồng bộ X-03 trước khi triển khai.

### 11.1. Tạo Cancellation Request

**Command:** `CreateBookingCancellationRequest`  
**Endpoint tham khảo:** `POST /api/v1/bookings/{booking_id}/cancellation-requests`

Request minh họa:
```json
{
  "expected_version": 8,
  "reason_code": "CUSTOMER_NO_LONGER_NEEDS",
  "reason_text": "Khách không còn nhu cầu",
  "source_channel": "APP",
  "requester": {
    "type": "CUSTOMER",
    "id": "customer-id"
  },
  "idempotency_key": "client-generated-key"
}
```

### 11.2. Review cancellation impact

**Command nội bộ:** `ReviewBookingCancellationImpact`

M01 lấy context từ M03/M04/M05:
```json
{
  "booking_id": "booking-id",
  "cancellation_request_id": "cancel-request-id",
  "booking_state": "READY_FOR_DISPATCH",
  "dispatch_context": {
    "active_offers": 2,
    "active_assignment": false
  },
  "trip_context": null,
  "financial_context": {
    "payment_status": "NOT_PAID"
  }
}
```

Số liệu trong ví dụ chỉ minh họa cấu trúc, không phải dữ liệu vận hành.

### 11.3. Approve/Reject/Withdraw

Endpoint tham khảo:
- `POST /api/v1/bookings/{booking_id}/cancellation-requests/{request_id}/approve`
- `POST /api/v1/bookings/{booking_id}/cancellation-requests/{request_id}/reject`
- `POST /api/v1/bookings/{booking_id}/cancellation-requests/{request_id}/withdraw`

Các command phải có actor, expected_version, reason và idempotency phù hợp.

### 11.4. Downstream commands

- M03: `RevokeOffersForBooking`, `CancelAssignment`, `ReleaseDispatchResources`.
- M04: `ReviewTripCancellation`, `CancelTrip`, `RecordNoShow`, `InterruptTrip` — tên command phải do M04 xác nhận.
- M05: `EvaluateCancellationFee`, `CreateRefundRequest` — theo API M05.
- M06: nhận event tài chính để hạch toán/đối soát, không cho M01 ghi trực tiếp ledger.

### 11.5. Response minh họa

```json
{
  "cancellation_request_id": "cancel-request-id",
  "booking_id": "booking-id",
  "status": "PROCESSING",
  "booking_state": "READY_FOR_DISPATCH",
  "dispatch_release_status": "PENDING",
  "trip_action_status": "NOT_REQUIRED",
  "fee_evaluation_status": "NOT_REQUIRED",
  "next_action": "WAIT_FOR_DISPATCH_CONFIRMATION"
}
```

Không trả kết quả `COMPLETED` khi M03/M04 có bước bắt buộc chưa xác nhận.

### 11.6. Event đề xuất

- `BookingCancellationRequested`
- `BookingCancellationApproved`
- `BookingCancellationRejected`
- `BookingCancelled`
- `BookingCancellationPartiallyCompleted`
- `BookingCancellationFailed`
- `BookingCancellationExpired`
- `BookingUnfulfilled` (kết quả không phục vụ, không đồng nghĩa khách hủy)
- `CancellationFeeEvaluated` (M05 ownership)
- `RefundRequested` / `RefundCompleted` (M05 ownership)

Event `BookingCancelled` tối thiểu có:
- event_id, event_type, event_version, occurred_at.
- booking_id, booking_version.
- cancellation_request_id.
- cancellation_reason_code.
- cancelled_by/requester_type.
- source_channel.
- dispatch_context_ref/trip_context_ref nếu có.
- policy_version nếu có.
- correlation_id/causation_id.

Dùng Transactional Outbox, consumer idempotent, Saga/compensation và reconciliation. Không giả định distributed transaction.

### 11.7. Error Codes đề xuất

| Code | Ý nghĩa |
|---|---|
| BOOKING_NOT_FOUND | Không tìm thấy Booking |
| BOOKING_MATCH_AMBIGUOUS | Có nhiều Booking phù hợp |
| BOOKING_NOT_CANCELLABLE | Booking không được phép hủy |
| TRIP_CHANGE_REQUIRED | Trip đã bắt đầu |
| TRIP_REVIEW_REQUIRED | Trip cần M04 review |
| CANCELLATION_REQUEST_NOT_FOUND | Không tìm thấy yêu cầu hủy |
| CANCELLATION_CONFLICT | Xung đột version/yêu cầu đồng thời |
| CANCELLATION_REASON_INVALID | Lý do hủy không hợp lệ |
| DISPATCH_CANCELLATION_PENDING | M03 chưa xác nhận thu hồi |
| TRIP_CANCELLATION_REJECTED | M04 từ chối |
| FEE_EVALUATION_FAILED | M05 không đánh giá được phí |
| REFUND_FAILED | Hoàn tiền thất bại |
| PARTNER_PERMISSION_DENIED | Đối tác không có quyền |
| SMS_SYNTAX_INVALID | Cú pháp SMS sai |
| IDEMPOTENCY_REPLAY | Yêu cầu đã xử lý |
| CANCELLATION_PARTIAL | Hủy chưa hoàn tất toàn bộ |

---

## 12. Bảo mật, Audit và dữ liệu cá nhân

### 12.1. Audit bắt buộc

| Audit Action | Khi nào |
|---|---|
| BOOKING_CANCELLATION_REQUESTED | Tạo yêu cầu hủy |
| BOOKING_CANCELLATION_VALIDATED | Kiểm tra quyền/state |
| BOOKING_CANCELLATION_IMPACT_REVIEWED | Review M03/M04/M05 |
| BOOKING_CANCELLATION_APPROVED | Được duyệt |
| BOOKING_CANCELLATION_REJECTED | Bị từ chối |
| BOOKING_CANCELLATION_WITHDRAWN | Khách rút yêu cầu |
| BOOKING_CANCELLED | Booking chuyển CANCELLED |
| BOOKING_CANCELLATION_PARTIAL | Downstream chưa hoàn tất |
| BOOKING_CANCELLATION_RETRIED | Retry |
| BOOKING_UNFULFILLED | Không thể phục vụ |
| DRIVER_OFFER_DECLINED | Tài xế từ chối Offer — M03 |
| DRIVER_ASSIGNMENT_CANCELLED | Tài xế hủy Assignment — M03 |
| TRIP_NO_SHOW_RECORDED | M04 ghi No-show |
| TRIP_INTERRUPTED | M04 ghi gián đoạn |
| CANCELLATION_FEE_EVALUATED | M05 đánh giá phí |
| REFUND_REQUESTED/COMPLETED/FAILED | M05 quản lý Refund |

Audit lưu actor, requester, channel, reason, timestamp, Booking version, policy version, Dispatch/Trip reference, decision và correlation ID. Không ghi token/bí mật.

### 12.2. Quyền riêng tư

- Số điện thoại/địa chỉ khách được masking theo vai trò.
- Tài xế chỉ nhận thông tin cần thiết cho Assignment/Trip.
- Không gửi thông tin khách qua log SMS/chat không được bảo vệ.
- Bằng chứng liên hệ/No-show phải lưu theo retention và access policy.
- Refund/payment data không được sao chép sang M01 nếu chỉ cần trạng thái tham chiếu.
- Tuân thủ X-04 khi hoàn thiện.

---

## 13. NFR và vận hành

| NFR | Yêu cầu |
|---|---|
| Integrity | Booking/Dispatch/Trip/Finance không mâu thuẫn sau hủy |
| Idempotency | Lệnh hủy lặp không tạo nhiều cancellation/refund/release |
| Concurrency | Chống race giữa khách hủy và tài xế nhận Assignment |
| Reliability | Outbox, retry, dead-letter/escalation |
| Consistency | Cancellation Request có trạng thái liên Module rõ ràng |
| Auditability | Truy xuất được ai hủy, vì sao, trạng thái trước/sau |
| Security | RBAC, customer ownership, partner scope |
| Privacy | Masking/retention theo X-04 |
| Observability | Correlation ID xuyên M01/M03/M04/M05/M06 |
| Recovery | Reconcile Assignment/Trip/Refund khi timeout |
| Performance | SLA hủy và giải phóng nguồn lực chốt theo X-05 |
| Availability | M03/M04/M05 lỗi không dẫn đến báo thành công giả |

Metrics đề xuất:
- booking_cancellation_request_total theo channel/reason.
- booking_cancellation_completed_total.
- booking_cancellation_rejected_total.
- booking_cancellation_pending_duration.
- booking_cancellation_dispatch_release_latency.
- booking_cancellation_trip_review_latency.
- booking_cancellation_fee_evaluation_total.
- booking_cancellation_refund_pending_total.
- booking_cancellation_partial_total.
- booking_cancellation_duplicate_total.
- booking_unfulfilled_total.
- driver_offer_declined_total (M03 metric, không cộng vào customer cancellation).
- driver_assignment_cancelled_total (M03 metric).
- trip_no_show_total (M04 metric).

---

## 14. Kiểm thử và Acceptance Criteria

### 14.1. Test Cases

| Test ID | Tình huống | Kết quả mong đợi |
|---|---|---|
| BKG006-TC-001 | Khách hủy Booking bằng ID/Code hợp lệ | Tạo Cancellation Request đúng Booking |
| BKG006-TC-002 | Tra cứu theo phone/pickup có nhiều kết quả | Không tự chọn; yêu cầu xác nhận |
| BKG006-TC-003 | Booking không tồn tại | BOOKING_NOT_FOUND |
| BKG006-TC-004 | Khách hủy Booking của người khác | PERMISSION_DENIED |
| BKG006-TC-005 | Nhân viên ngoài data scope hủy | PERMISSION_DENIED |
| BKG006-TC-006 | Hủy Booking DRAFT | Xử lý theo X-01 |
| BKG006-TC-007 | Hủy Booking SUBMITTED | Chuyển CANCELLED nếu hợp lệ |
| BKG006-TC-008 | Hủy Booking VALIDATING | Dừng/đóng validation an toàn |
| BKG006-TC-009 | Hủy Booking ACCEPTED chưa Dispatch | M01 cập nhật; M03 nhận event nếu cần |
| BKG006-TC-010 | Hủy Booking SCHEDULED | Hủy theo policy đặt trước |
| BKG006-TC-011 | Hủy READY_FOR_DISPATCH chưa có Offer | M03 đóng DispatchRun nếu đã có |
| BKG006-TC-012 | Hủy khi Offer đang mở | M03 thu hồi Offer |
| BKG006-TC-013 | Tài xế nhận Offer đồng thời khách hủy | Concurrency xử lý đúng; không có Assignment mồ côi |
| BKG006-TC-014 | Hủy sau khi Assignment đã nhận | M03 thu hồi Assignment theo policy |
| BKG006-TC-015 | M03 timeout khi thu hồi | Request pending/partial; không báo hoàn tất |
| BKG006-TC-016 | Tài xế từ chối Offer | Booking không chuyển CANCELLED |
| BKG006-TC-017 | Tài xế hủy Assignment | M03 xử lý; Booking giữ nhu cầu nếu khách chưa hủy |
| BKG006-TC-018 | Khách hủy khi Trip CREATED | M04 review Trip |
| BKG006-TC-019 | Khách hủy khi tài xế DRIVER_EN_ROUTE | M04/M03 xử lý theo policy |
| BKG006-TC-020 | Khách hủy khi Trip ARRIVED | M04 kiểm tra điều kiện |
| BKG006-TC-021 | Khách hủy khi WAITING_FOR_CUSTOMER | M04 xử lý; No-show cần evidence |
| BKG006-TC-022 | Khách yêu cầu hủy khi IN_PROGRESS | Chuyển M04; không CANCELLED Booking theo luồng thường |
| BKG006-TC-023 | Trip đã COMPLETED | Không hủy; mở quy trình hậu kiểm |
| BKG006-TC-024 | Booking đã CANCELLED | Trả kết quả cũ; không xử lý lần hai |
| BKG006-TC-025 | Booking EXPIRED | Không hủy lặp |
| BKG006-TC-026 | Booking REJECTED | Không hủy lặp |
| BKG006-TC-027 | Booking UNFULFILLED do thiếu xe | Không ghi thành khách hủy |
| BKG006-TC-028 | Khách không tiếp tục chờ | Reason CUSTOMER_CANCELLED_AFTER_WAIT |
| BKG006-TC-029 | Điều hành hủy do sự cố | Reason OPERATIONAL_EXCEPTION và actor |
| BKG006-TC-030 | Không có xe phục vụ | Outcome UNFULFILLED theo X-01 |
| BKG006-TC-031 | SMS đúng cú pháp đã duyệt | Tạo Cancellation Request |
| BKG006-TC-032 | SMS sai cú pháp | SMS_SYNTAX_INVALID |
| BKG006-TC-033 | SMS mã Booking không tồn tại | BOOKING_CODE_INVALID |
| BKG006-TC-034 | SMS gửi lặp | Idempotent; không hủy/hoàn nhiều lần |
| BKG006-TC-035 | Chat/Call Center hủy thay khách | Lưu agent, requester và evidence |
| BKG006-TC-036 | Web/App khách xác nhận hủy | Lưu confirmation |
| BKG006-TC-037 | M05 không có policy phí | Không tự tính/thu phí |
| BKG006-TC-038 | M05 trả Cancel Fee | Hiển thị theo policy version; yêu cầu xác nhận nếu cần |
| BKG006-TC-039 | Refund thất bại | Booking vẫn hủy; Refund retry/reconcile |
| BKG006-TC-040 | M06 nhận event tài chính | Hạch toán theo source event; không trùng |
| BKG006-TC-041 | Partner hủy Booking không thuộc contract scope | PARTNER_PERMISSION_DENIED |
| BKG006-TC-042 | DIRECT/PARTNER cancellation | Giữ contract context và tài chính tách biệt |
| BKG006-TC-043 | Hai lệnh hủy cùng lúc | Một kết quả duy nhất theo version/locking |
| BKG006-TC-044 | Client timeout sau commit | Tra cứu idempotency trả kết quả gốc |
| BKG006-TC-045 | M01 commit, M03 chưa xác nhận | PARTIALLY_COMPLETED/PROCESSING |
| BKG006-TC-046 | Notification lỗi | Retry; không mất trạng thái hủy |
| BKG006-TC-047 | No-show thiếu evidence | Không chuyển NO_SHOW |
| BKG006-TC-048 | Hủy một phần Booking nhiều xe khi feature chưa bật | Từ chối rõ ràng |
| BKG006-TC-049 | Reason code không hợp lệ | CANCELLATION_REASON_INVALID |
| BKG006-TC-050 | Booking ON_HOLD | Chỉ actor có quyền mới xử lý |
| BKG006-TC-051 | Hủy lặp sau khi Assignment đã giải phóng | Không giải phóng xe lần hai |
| BKG006-TC-052 | M01/M03/M04 đối soát sau outage | Phát hiện và sửa lệch bằng reconciliation có audit |

### 14.2. Acceptance Criteria

**AC-01 — Đúng Booking**
- Given khách/nhân viên yêu cầu hủy
- When hệ thống tra cứu
- Then phải xác định đúng một Booking
- And nếu nhiều kết quả mơ hồ thì không tự chọn.

**AC-02 — Kiểm tra quyền và trạng thái**
- Given actor gửi yêu cầu
- When M01 kiểm tra quyền/Booking State
- Then chỉ cho phép hủy theo permission, data scope và X-01.

**AC-03 — Phân biệt customer cancel và driver refusal**
- Given tài xế từ chối Offer/hủy Assignment
- When M03 ghi nhận
- Then Booking không tự chuyển CANCELLED nếu khách vẫn có nhu cầu.

**AC-04 — Giải phóng nguồn lực**
- Given Booking có Offer/Assignment hiệu lực
- When khách hủy hợp lệ
- Then M03 phải thu hồi/giải phóng theo ownership
- And M01 không tự sửa Driver/Vehicle state.

**AC-05 — Trip boundary**
- Given Trip IN_PROGRESS
- When khách yêu cầu hủy
- Then BKG-006 chuyển M04 xử lý
- And không chuyển Booking CANCELLED theo luồng hủy trước chuyến.

**AC-06 — No-show**
- Given khách không xuất hiện
- When chưa đủ thời gian/evidence theo policy
- Then không được đánh dấu NO_SHOW.

**AC-07 — Financial policy**
- Given cancellation có thể phát sinh phí/hoàn
- When M05 chưa có policy version hoặc kết quả
- Then M01 không tự thu/hoàn tiền.

**AC-08 — Partial failure**
- Given M03/M04/M05 timeout/failure
- When cancellation đang xử lý
- Then trạng thái partial/pending phải được lưu
- And không báo hoàn tất giả.

**AC-09 — Idempotency**
- Given SMS/API retry hoặc nhiều lệnh hủy lặp
- When cùng idempotency reference
- Then chỉ một kết quả hủy và một chuỗi xử lý tài chính hợp lệ.

**AC-10 — Audit**
- Given một Booking đã hủy
- When auditor tra cứu
- Then xem được actor, requester, channel, reason, thời điểm, policy, Dispatch/Trip context và kết quả tài chính.

**AC-11 — DIRECT/PARTNER**
- Given Booking thuộc DIRECT hoặc PARTNER
- When hủy
- Then giữ đúng contract/fleet context và không gộp quy tắc tài chính.

### 14.3. Definition of Done riêng

- [ ] Business Owner duyệt Cancellation Reason Catalog.
- [ ] Duyệt ma trận hủy theo Booking/Dispatch/Trip State.
- [ ] Duyệt phân quyền khách/Tổng đài/Điều hành/Partner.
- [ ] M03 duyệt thu hồi Offer/Assignment và race condition.
- [ ] M04 duyệt Trip cancellation/No-show/Interrupted boundary.
- [ ] M05 duyệt Cancel Fee/Payment/Refund policy.
- [ ] M06 duyệt accounting/receivable/settlement effects.
- [ ] X-01 xác nhận Booking State transitions.
- [ ] X-02 xác nhận Cancellation Request/Entity.
- [ ] X-03 xác nhận API/Event/Saga/Outbox/Idempotency.
- [ ] X-04 xác nhận IAM/Audit/PII/retention.
- [ ] X-05 xác nhận SLA/NFR/monitoring.
- [ ] QA hoàn tất Test Cases.
- [ ] Business UAT và các Module ký nghiệm thu.

---

## 15. Cấu hình và triển khai

### 15.1. Configuration cần có

| Configuration | Ý nghĩa | Owner | Trạng thái |
|---|---|---|---|
| CANCELLATION_ALLOWED_STATES | Booking State được hủy | M01/Operations | TBD |
| CANCELLATION_REASON_CODES | Lý do hủy | Operations | Draft |
| CANCELLATION_CHANNELS | Kênh hỗ trợ hủy | Product/Operations | TBD |
| CUSTOMER_IDENTITY_POLICY | Xác minh khách | CSKH/ATTT | TBD |
| OPERATOR_CANCEL_PERMISSION | Quyền Điều hành hủy | Operations/IAM | TBD |
| SMS_CANCEL_SYNTAX | Cú pháp SMS | Product/Operations | TBD |
| SMS_AUTHENTICATION_POLICY | Xác thực SMS | ATTT | TBD |
| DISPATCH_RELEASE_POLICY | Thu hồi Offer/Assignment | M03 | TBD |
| TRIP_CANCEL_POLICY | Quy tắc hủy Trip | M04 | TBD |
| NO_SHOW_POLICY | Điều kiện/evidence No-show | M04/Operations | TBD |
| CANCEL_FEE_POLICY | Phí hủy | M05/Finance | TBD |
| REFUND_POLICY | Hoàn tiền | M05/Finance | TBD |
| PARTNER_CANCELLATION_POLICY | Hủy theo hợp đồng đối tác | M02/M06 | TBD |
| CANCELLATION_CONFIRMATION_POLICY | Yêu cầu xác nhận | Product/Operations | TBD |
| CANCELLATION_SLA | SLA xử lý | Operations/SRE | TBD |

Không hard-code thời hạn, mức phí, số lần liên hệ, thời gian chờ hoặc số lần retry.

### 15.2. Triển khai theo giai đoạn

1. **Giai đoạn 1:** Hủy Booking trước Dispatch; App/Web/Call Center.
2. **Giai đoạn 2:** Tích hợp M03 thu hồi Offer/Assignment; SMS/Chat.
3. **Giai đoạn 3:** Tích hợp M04 cho Trip CREATED/DRIVER_EN_ROUTE/ARRIVED/WAITING_FOR_CUSTOMER.
4. **Giai đoạn 4:** Tích hợp M05/M06 cho Cancel Fee/Payment/Refund/Settlement.
5. **Giai đoạn 5:** Partner API, No-show, hủy một phần và ngoại lệ nâng cao sau khi policy được duyệt.

Mỗi giai đoạn có feature flag, audit và reconciliation. Không bật thu phí hủy trước khi chính sách, hợp đồng và UAT được phê duyệt.

---

## 16. Rủi ro, quyết định và TBD

| ID | Nội dung cần quyết định | Ảnh hưởng | Owner đề xuất |
|---|---|---|---|
| BKG006-DEC-01 | Booking State nào khách được tự hủy? | UX/Booking | Operations |
| BKG006-DEC-02 | Booking State nào Tổng đài được hủy thay khách? | Operations/IAM | Operations |
| BKG006-DEC-03 | Điều hành có quyền hủy Booking ở giai đoạn nào? | Governance | COO |
| BKG006-DEC-04 | Có cho hủy khi Offer đang mở không? | Dispatch | M03 |
| BKG006-DEC-05 | Khi tài xế đã nhận Assignment, khách hủy cần xác nhận gì? | Dispatch | M03/Operations |
| BKG006-DEC-06 | Khi tài xế đang trên đường, có giới hạn hủy hay điều kiện phí không? | Operations/Fare | Operations/M05 |
| BKG006-DEC-07 | Khi tài xế đã đến, xử lý khách hủy và No-show thế nào? | Trip/CSKH | M04 |
| BKG006-DEC-08 | Ai có quyền xác nhận No-show? | Governance | M04/Operations |
| BKG006-DEC-09 | Thời gian chờ khách và bằng chứng No-show là gì? | Trip/Legal | M04 |
| BKG006-DEC-10 | Có phí hủy trước Dispatch không? | Pricing | M05/Finance |
| BKG006-DEC-11 | Có phí hủy sau khi tài xế nhận Assignment không? | Pricing | M05/Finance |
| BKG006-DEC-12 | Có phí hủy khi tài xế đang đến/đã đến không? | Pricing | M05/Finance |
| BKG006-DEC-13 | Mức phí/công thức tính theo loại xe/khu vực/dịch vụ? | Pricing | M05 |
| BKG006-DEC-14 | Ai chịu phí khi không có xe phục vụ? | Customer policy | Operations |
| BKG006-DEC-15 | Có bồi hoàn tài xế/đối tác khi khách hủy không? | Partner finance | M02/M06 |
| BKG006-DEC-16 | Có hoàn tiền tự động nếu khách đã trả trước không? | Payment | M05 |
| BKG006-DEC-17 | SLA hoàn tiền và phương thức hoàn? | Finance/CSKH | M05/M06 |
| BKG006-DEC-18 | Ai có quyền miễn/giảm phí hủy? | Approval matrix | Finance/Operations |
| BKG006-DEC-19 | Quy tắc phí hủy DIRECT và PARTNER khác nhau thế nào? | Contract/Finance | M02/M06 |
| BKG006-DEC-20 | Có cho hủy một phần Booking nhiều xe không? | Data/Dispatch | M01/M03 |
| BKG006-DEC-21 | SMS hủy dùng cú pháp nào hiện hành? | Channel | Product/Operations |
| BKG006-DEC-22 | SMS chỉ có Booking Code có đủ xác thực không? | Security/Fraud | ATTT |
| BKG006-DEC-23 | Chat/Call Center cần xác minh khách ở mức nào? | Privacy/Fraud | CSKH/ATTT |
| BKG006-DEC-24 | Partner API có được hủy Booking sau Assignment không? | Contract/API | M02/M01 |
| BKG006-DEC-25 | Nếu M03 không phản hồi thu hồi Assignment thì Booking state xử lý ra sao? | Saga | M01/M03 |
| BKG006-DEC-26 | Nếu M04 từ chối Trip cancellation thì chuyển trạng thái nào? | Trip | M04 |
| BKG006-DEC-27 | Hủy Booking khi Trip đã bắt đầu có được phép trong trường hợp nào? | Safety/Trip | M04/Operations |
| BKG006-DEC-28 | Hủy do khách đổi ý khác với khách không liên lạc được thế nào? | Reporting | Operations |
| BKG006-DEC-29 | Reason code nào bắt buộc theo từng channel? | Reporting | Operations |
| BKG006-DEC-30 | Hạn lưu Cancellation Request/evidence/audit? | Compliance | Data Owner/ATTT |
| BKG006-DEC-31 | SLA xử lý cancellation partial và escalation tới ai? | Operations/SRE | M01/M03/M04 |
| BKG006-DEC-32 | Booking UNFULFILLED do thiếu xe có cần gửi thông báo và quy trình hỗ trợ nào? | Customer Experience | Operations/M07 |

---

## 17. Tiêu chí nghiệm thu Feature

- [ ] Hủy được Booking qua các channel đã được phê duyệt.
- [ ] Tra cứu và xác định đúng Booking.
- [ ] Permission/data scope được kiểm tra.
- [ ] Cancellation Request có lifecycle riêng.
- [ ] Booking State chuyển đúng theo X-01.
- [ ] Phân biệt Customer Cancel, Driver Refusal, Assignment Cancel, No-show, Unfulfilled và Trip Interrupted.
- [ ] M03 thu hồi Offer/Assignment và giải phóng nguồn lực.
- [ ] M04 xử lý Trip theo đúng trạng thái.
- [ ] M05 là nguồn duy nhất đánh giá Cancel Fee/Payment/Refund.
- [ ] M06 xử lý kế toán/đối soát theo event tài chính.
- [ ] Không tự đặt phí/thời gian/điều kiện hủy khi policy chưa duyệt.
- [ ] DIRECT/PARTNER tách biệt theo contract.
- [ ] Có API/Event/Saga/Outbox/Idempotency.
- [ ] Có concurrency control và reconciliation.
- [ ] Có audit/PII/retention theo X-04.
- [ ] NFR/SLA/monitoring/runbook được duyệt theo X-05.
- [ ] QA hoàn tất Test Cases và Business UAT ký xác nhận.

### 17.1. Kết luận hiện tại

**Trạng thái:** DRAFT — Chưa đạt READY/DoR.

**Blocker chính:**
1. Chưa chốt ma trận quyền hủy theo Booking/Dispatch/Trip State.
2. Chưa chốt chính sách Cancel Fee và No-show.
3. Chưa chốt quy trình M03 thu hồi Assignment và giải phóng xe.
4. Chưa chốt ranh giới hủy Booking với Trip Cancellation/Interruption.
5. Chưa chốt Payment/Refund/Finance đối với DIRECT/PARTNER.
6. Chưa chốt cú pháp và xác thực SMS.
7. Chưa chốt Saga/reconciliation khi các Module xử lý một phần.

**Không triển khai thu phí hủy, tự động ghi No-show hoặc tự giải phóng tài xế/xe từ M01 trước khi các Module sở hữu và Business Owner phê duyệt.**

---

**Quay lại:** [Feature Specifications](../README.md) · [M01 — Customer & Booking](../../../02-DOMAIN-MODULES/01-customer-and-booking/README.md)
