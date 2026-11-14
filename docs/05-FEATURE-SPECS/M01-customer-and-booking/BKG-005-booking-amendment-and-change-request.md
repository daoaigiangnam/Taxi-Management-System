# BKG-005 — SỬA ĐỔI BOOKING VÀ QUẢN LÝ YÊU CẦU THAY ĐỔI

**Mã tài liệu:** TAXI-BKG-005  
**Module sở hữu:** M01 — Khách hàng và Đặt xe  
**Epic:** Tiếp nhận và quản lý Booking đa kênh  
**Phiên bản:** 0.1  
**Trạng thái:** DRAFT — CHỜ NGHIỆP VỤ, M03, M04, M05 VÀ KIẾN TRÚC REVIEW  
**Priority theo Feature Catalog:** P1  
**Business Owner:** TBD  
**Technical Owner:** M01 Owner — TBD  
**Tài liệu liên quan:** [M01 Feature Catalog](../../../02-DOMAIN-MODULES/01-customer-and-booking/README.md) · [BKG-001 — Tạo Booking tức thời](./BKG-001-create-immediate-booking.md) · [BKG-003 — Kiểm tra hợp lệ và vùng phục vụ](./BKG-003-validation-and-serviceability.md) · [BKG-004 — Điểm đón/điểm đến/điểm dừng](./BKG-004-pickup-destination-and-ordered-stops.md) · [X-01 — State Machine](../../../03-CROSS-CUTTING/01-state-machines-and-transition-rules.md) · [X-02 — Data Architecture](../../../03-CROSS-CUTTING/02-data-architecture-and-erd.md) · [X-03 — API/Event](../../../03-CROSS-CUTTING/03-api-and-event-contracts.md)

> **Quy ước nguồn:** KPMG 2015 mô tả khách yêu cầu thay đổi qua Tổng đài (TO.P1), App (TO.P2), Chat (TO.P6) và Web (TO.P7). TO.P1.8 nêu tra cứu yêu cầu gần nhất theo số điện thoại/điểm đón; nếu đã có xe đăng ký thì cập nhật cho xe, nếu chưa có thì xử lý như yêu cầu mới. TO.P6.2 mô tả nhập mã đặt chỗ và chọn yêu cầu để chỉnh sửa; nếu có xe đăng ký thì cập nhật thông tin cho xe. Các quy trình có chi tiết khác nhau và chưa định nghĩa đầy đủ cách tài xế xác nhận phiên bản thay đổi. KPMG 2015 là nguồn lịch sử, không phải chính sách hiện hành.

---

## 0. Quản lý phiên bản và phê duyệt

| Vai trò | Người duyệt | Trạng thái |
|---|---|---|
| Business Owner — Điều hành/Vận hành | TBD | Pending |
| Product Owner M01 | TBD | Pending |
| Technical Owner M01 | TBD | Pending |
| Chủ Module M03 — Điều phối | TBD | Pending |
| Chủ Module M04 — Chuyến đi | TBD | Pending |
| Chủ Module M05 — Giá cước | TBD | Pending |
| QA Lead | TBD | Pending |

| Phiên bản | Nội dung |
|---|---|
| 0.1 | Đặc tả yêu cầu sửa Booking, revision, change impact và đồng bộ liên Module |

---

## 1. Bối cảnh và mục tiêu

### 1.1. Vấn đề nghiệp vụ

Booking sau khi tạo có thể cần thay đổi do khách cập nhật địa chỉ, thời gian, loại xe, số lượng xe, điểm đến, ghi chú hoặc các yêu cầu khác. Việc thay đổi có thể xảy ra ở các thời điểm rất khác nhau:
- Booking vừa được tiếp nhận, chưa phát điều phối.
- M03 đã phát Offer nhưng chưa có tài xế nhận.
- Tài xế đã nhận Offer/Assignment.
- Tài xế đang trên đường đến đón khách.
- Tài xế đã đến điểm đón nhưng khách chưa lên xe.
- Trip đã bắt đầu.
- Trip đã hoàn tất.

Cùng một nội dung thay đổi nhưng tác động ở mỗi thời điểm không giống nhau. Ví dụ, đổi Pickup trước khi Dispatch có thể chỉ cần kiểm tra lại địa chỉ và vùng phục vụ; đổi Pickup sau khi tài xế đã nhận chuyến có thể buộc M03 thu hồi/đánh giá Assignment và M04 cập nhật kế hoạch Trip. Đổi Destination có thể tác động đến báo giá của M05.

Nếu M01 ghi đè trực tiếp dữ liệu hiện hành mà không tạo revision, các Module có thể sử dụng các phiên bản hành trình khác nhau; tài xế có thể tiếp tục chạy theo thông tin cũ; giá cước và lịch sử không thể giải thích.

### 1.2. Mục tiêu

1. Cung cấp một cơ chế thống nhất để khách/nhân viên yêu cầu thay đổi Booking.
2. Tra cứu đúng Booking cần thay đổi, tránh cập nhật nhầm Booking gần nhất.
3. Lưu mọi thay đổi theo BookingRevision, giữ nguyên lịch sử trước đó.
4. Kiểm tra Booking State, quyền, version và nguồn yêu cầu trước khi chấp nhận thay đổi.
5. Đánh giá tác động đến M03 Dispatch, M04 Trip, M05 Pricing và BKG-003/BKG-004.
6. Yêu cầu xác nhận lại khi thay đổi có ảnh hưởng đáng kể đến khách/tài xế.
7. Không để M01 tự ý ghi dữ liệu thuộc Module khác.
8. Hỗ trợ xử lý thay đổi đồng thời, retry và Event idempotent.
9. Phân biệt yêu cầu thay đổi, thay đổi đã được chấp nhận và thay đổi đã được thực thi.
10. Cung cấp lịch sử đủ để CSKH/Điều hành giải thích khiếu nại.

### 1.3. Nguyên tắc thiết kế

- **Append-only history:** không ghi đè mất dấu phiên bản Booking đã được xác nhận.
- **State-aware:** khả năng sửa phụ thuộc trạng thái Booking và trạng thái Dispatch/Trip liên quan.
- **Impact before commit:** xác định Module bị ảnh hưởng trước khi áp dụng thay đổi.
- **No false success:** không báo đã thay đổi thành công khi M03/M04/M05 chưa xác nhận phần việc bắt buộc.
- **Single owner:** M01 là nguồn dữ liệu chuẩn của Booking; M03/M04/M05 là nguồn chuẩn cho miền dữ liệu riêng.
- **Customer confirmation:** yêu cầu xác nhận lại nếu policy yêu cầu; lưu bằng chứng xác nhận.
- **No silent propagation:** tài xế/khách phải nhận thông tin phiên bản mới qua kênh được phê duyệt.
- **DIRECT/PARTNER separation:** thay đổi Booking không được làm thay đổi fleet_type hoặc ranh giới hợp đồng/tài chính nếu không có command/policy riêng.

---

## 2. Nguồn nghiệp vụ và mức độ căn cứ

### 2.1. Nội dung được KPMG 2015 hỗ trợ

| Nguồn | Nội dung nguồn mô tả | Kết luận được phép dùng |
|---|---|---|
| TO.P1.1 | Khách gọi Tổng đài để yêu cầu, nhắc, thay đổi hoặc hủy dịch vụ | Call Center là một kênh tiếp nhận thay đổi |
| TO.P1.5 | Phân loại yêu cầu mới, nhắc/thay đổi, hủy | Thay đổi là loại yêu cầu riêng, không đồng nhất với Booking mới |
| TO.P1.8 | Tra cứu theo số điện thoại; nếu số khác thì dựa vào điểm đón; chọn yêu cầu gần nhất để chỉnh sửa. Nếu đã có xe đăng ký thì cập nhật cho xe; chưa có xe thì xử lý như yêu cầu mới | Cần tra cứu và phân biệt tình trạng đã có xe đăng ký/chưa có xe |
| TO.P1.9 | Kiểm tra tài xế đã đăng ký hay chưa | Tình trạng Dispatch ảnh hưởng luồng thay đổi |
| TO.P2.1–P2.3 | App nhận Pickup, Destination tùy chọn và dùng Destination để ước tính lộ trình/cước | Thay đổi địa điểm có thể ảnh hưởng route/fare estimate |
| TO.P2.10–P2.11 | Tài xế xác nhận nhận đón; tài xế có thể hủy khi đang trên đường; hệ thống thông báo khách và hiển thị xe khác | Assignment có vòng đời riêng; cần đồng bộ khi thay đổi |
| TO.P2.16 | Khách có thể thông báo hủy; tài xế hủy trên App và ghi chú lý do | KPMG mô tả hành động hủy, không đủ để suy ra quy trình amendment sau Trip Start |
| TO.P6.1–P6.2 | Chat hỗ trợ yêu cầu/thay đổi/hủy; nhập mã đặt chỗ, chọn yêu cầu; nếu có xe đăng ký thì cập nhật thông tin cho xe | Chat cần Booking reference và xử lý thay đổi có tài xế liên quan |
| TO.P7.1–P7.3 | Web nhập dữ liệu, tạo mã đặt chỗ và yêu cầu khách kiểm tra/xác nhận; cần sửa thì quay lại nhập thông tin | Web có bước kiểm tra/xác nhận thông tin trước khi tiếp tục |
| TO.P7.5 | Khách hủy trên Web; hệ thống đưa xe đã đăng ký về trạng thái trống nếu có | Hủy có tác động tới nguồn lực M03 |
| TO.P7.20–P7.22 (theo bảng bước) | Tài xế có thể hủy nhận đón; hệ thống thông báo khách và hiển thị xe khác | Trạng thái tài xế/Assignment phải được phân biệt với Booking amendment |

### 2.2. Nội dung KPMG chưa đặc tả đầy đủ

KPMG 2015 không xác định rõ:
- Danh sách trường nào khách được tự thay đổi ở từng kênh.
- Mốc thời gian khóa thay đổi theo từng trạng thái.
- Cách tài xế xác nhận đã nhận phiên bản Booking mới.
- Cách xử lý khi tài xế không phản hồi thay đổi.
- Có cần khách xác nhận lại khi đổi Pickup/Destination/loại xe hay không.
- Thay đổi nào phải thu hồi Offer/Assignment và phát DispatchRun mới.
- Thay đổi nào phải báo giá lại, giữ giá cũ hay thu phí chênh lệch.
- Quy trình sửa Booking khi Trip đã bắt đầu.
- Cách giải quyết hai yêu cầu sửa đồng thời.
- SLA xử lý change request và cơ chế escalation.

Các nội dung trên là thiết kế đề xuất/TBD, không được trình bày như chính sách Mai Linh hiện hành.

### 2.3. Phân biệt bốn khái niệm

| Khái niệm | Ý nghĩa | Owner |
|---|---|---|
| Change Request | Yêu cầu thay đổi do khách/nhân viên gửi | M01 quản lý yêu cầu |
| Booking Revision | Phiên bản dữ liệu Booking sau khi thay đổi được áp dụng | M01 |
| Dispatch Impact | Tác động đến Candidate/Offer/Assignment/DispatchRun | M03 |
| Trip Change | Thay đổi kế hoạch hoặc diễn biến khi Trip đã hình thành/đang chạy | M04 |

Một Change Request không tự động đồng nghĩa với Booking Revision đã được áp dụng. Một Booking Revision cũng không có nghĩa M03/M04 đã thực thi xong tác động.

---

## 3. Phạm vi

### 3.1. Trong phạm vi

1. Tra cứu Booking cần thay đổi theo Booking ID/Booking Code và tiêu chí được phép.
2. Tạo Change Request với actor, channel, reason, requested changes.
3. Kiểm tra quyền, Booking State, revision/version và trạng thái liên quan từ M03/M04.
4. Xác định các trường được phép thay đổi theo policy.
5. Validate dữ liệu mới qua BKG-003/BKG-004 và các Feature liên quan.
6. Tạo impact assessment cho M03/M04/M05.
7. Yêu cầu khách/tài xế/nhân viên xác nhận khi cần.
8. Chấp nhận/từ chối/đóng Change Request.
9. Tạo BookingRevision khi thay đổi được áp dụng theo transaction/command đã định nghĩa.
10. Phát Event để consumer đồng bộ.
11. Lưu lịch sử trước/sau, lý do và bằng chứng.
12. Hỗ trợ retry, concurrency, idempotency và timeout.

### 3.2. Ngoài phạm vi

- Hủy Booking hoàn chỉnh và chính sách phí hủy: BKG-006.
- Tạo Booking mới: BKG-001/BKG-002.
- Tối ưu/tìm xe/phát Offer/thu hồi Assignment: M03.
- Thay đổi Trip đã bắt đầu, cập nhật TripStopProgress/GPS/TripEvent: M04.
- Tính lại FareQuote/FareResult/Payment/Refund: M05.
- Sửa hợp đồng DIRECT/PARTNER hoặc chuyển fleet_type: M02/M06 theo nghiệp vụ riêng.
- Quản lý khiếu nại/SLA hỗ trợ: M07.
- Quản lý Identity/Permission/Audit nền tảng: M08.

### 3.3. Nhóm trường có thể yêu cầu thay đổi

| Nhóm | Ví dụ | Module/Feature liên quan | Trạng thái đề xuất |
|---|---|---|---|
| Pickup | Địa chỉ, tọa độ, hướng dẫn đón | BKG-003/BKG-004/M03 | Có điều kiện |
| Destination | Địa chỉ đích | BKG-003/BKG-004/M05 | Có điều kiện |
| Ordered Stops | Thêm/xóa/reorder VIA | BKG-004/M03/M04/M05 | Chỉ khi policy bật |
| Pickup Time | Thời gian đón | BKG-002/M03 | Có điều kiện |
| Vehicle Type | 4/7 chỗ, loại dịch vụ | M02/M03 | Có điều kiện |
| Vehicle Brand/Feature | Hiệu xe, yêu cầu đặc điểm | M02/M03 | Có điều kiện |
| Vehicle Quantity | Số lượng xe | M02/M03 | Có điều kiện; có thể cần tách Booking |
| Contact Point | Số điện thoại liên hệ | Customer/ContactPoint | Theo quyền/OTP/policy |
| Customer Note | Ghi chú phục vụ | M03/M04 | Có điều kiện |
| Booking Channel | App/Web/Call Center/SMS/Chat | M01 | Không sửa sau tạo |
| Fleet Type | DIRECT/PARTNER | M02/Contract | Không cho sửa qua BKG-005 |
| Booking ID/Code | Định danh Booking | M01 | Không sửa |
| Fare/Tariff | Giá/biểu giá | M05 | Không sửa trực tiếp trong M01 |

Danh sách trường được phép thay đổi phải được Business Owner chốt theo từng Booking State và channel.

---

## 4. Actor, quyền và Data Scope

| Actor | Hành động | Permission đề xuất | Điều kiện |
|---|---|---|---|
| Khách App/Web | Tạo Change Request | `booking.change.request.self` | Booking thuộc khách |
| Khách App/Web | Xác nhận/từ chối thay đổi | `booking.change.confirm.self` | Có request đang chờ |
| Tổng đài viên | Tra cứu và tạo request thay mặt khách | `booking.change.request.on_behalf` | Xác minh khách và ghi reason |
| Điều hành | Review thay đổi có tác động Dispatch | `booking.change.review.dispatch` | Theo khu vực/ca |
| Tài xế | Xác nhận đã nhận thông tin mới | Permission của M03/M04 | Đúng Assignment |
| M01 Service | Validate, lưu Change Request/Revision | Internal permission | Theo Booking aggregate |
| M03 Service | Đánh giá/thu hồi/cập nhật Offer/Assignment | Internal API | M03 ownership |
| M04 Service | Đánh giá Trip Change | Internal API | M04 ownership |
| M05 Service | Requote/đánh giá Fare | Internal API | M05 ownership |
| Auditor/CSKH | Đọc lịch sử thay đổi | `booking.change.audit.read` | Theo data scope |

Quy tắc:
- Khách không được sửa Booking của khách khác.
- Nhân viên chỉ thao tác trong đơn vị/khu vực/ca được cấp.
- Không cho phép client gửi `fleet_type`, giá, trạng thái Dispatch hoặc trạng thái Trip để cập nhật trực tiếp.
- Mọi thao tác thay mặt khách phải lưu actor thực hiện và customer confirmation/evidence theo policy.
- Không chia sẻ số điện thoại khách cho tài xế nếu không có quyền và căn cứ nghiệp vụ.

---

## 5. Mô hình dữ liệu và Change Request

### 5.1. Entity BookingRevision

X-02 xác định M01 sở hữu Booking và BookingRevision. BKG-005 sử dụng Revision để lưu phiên bản dữ liệu Booking sau khi thay đổi.

Các thuộc tính logic đề xuất:
- `booking_revision_id`
- `booking_id`
- `revision_no`
- `based_on_revision_no`
- `change_request_id`
- `change_type`
- `change_reason`
- `changed_fields`
- `before_snapshot_ref`
- `after_snapshot_ref`
- `actor_id`
- `source_channel`
- `created_at`
- `correlation_id`

Revision đã được sử dụng bởi Dispatch/Trip/Fare không được ghi đè.

### 5.2. Entity BookingChangeRequest — đề xuất

Đây là Entity mới đề xuất, cần Data Architect xác nhận có nên bổ sung vào X-02 hay biểu diễn bằng Change Request aggregate riêng.

| Field logic | Ý nghĩa | Bắt buộc |
|---|---|---|
| change_request_id | Định danh request | Có |
| booking_id | Booking mục tiêu | Có |
| request_no | Số thứ tự request trong Booking | Có |
| requested_by | Actor yêu cầu | Có |
| requester_type | CUSTOMER / AGENT / SYSTEM / OTHER theo policy | Có |
| source_channel | APP / WEB / CALL_CENTER / CHAT / SMS / API | Có |
| reason_code | Mã lý do | Có |
| reason_text | Mô tả bổ sung | Không |
| requested_changes | Tập field/value đề nghị thay đổi | Có |
| based_on_revision | Revision khách đang nhìn thấy | Có |
| booking_version | Aggregate version tại thời điểm yêu cầu | Có |
| impact_snapshot | Kết quả đánh giá tác động | Có sau review |
| status | Trạng thái Change Request | Có |
| confirmation_required | Có cần xác nhận thêm không | Có |
| confirmation_deadline | Hạn xác nhận nếu có | TBD |
| decision_by | Người/hệ thống quyết định | Sau review |
| decision_reason | Lý do chấp nhận/từ chối | Khi có decision |
| applied_revision | Revision được tạo khi áp dụng | Khi thành công |
| created_at/updated_at | Thời điểm | Có |

### 5.3. Change Request State đề xuất

| State | Ý nghĩa |
|---|---|
| SUBMITTED | Đã tiếp nhận yêu cầu |
| VALIDATING | Đang kiểm tra dữ liệu/quyền/trạng thái |
| IMPACT_REVIEW | Đang đánh giá tác động liên Module |
| AWAITING_CUSTOMER_CONFIRMATION | Chờ khách xác nhận |
| AWAITING_DRIVER_CONFIRMATION | Chờ tài xế xác nhận khi policy yêu cầu |
| AWAITING_OPERATOR_DECISION | Chờ Điều hành xử lý |
| APPROVED | Được chấp nhận nhưng chưa chắc đã áp dụng hoàn tất |
| APPLYING | Đang áp dụng thay đổi và đồng bộ liên Module |
| APPLIED | Đã áp dụng thành công |
| REJECTED | Bị từ chối |
| EXPIRED | Hết thời hạn xử lý/xác nhận |
| WITHDRAWN | Người yêu cầu rút lại |
| FAILED | Thất bại kỹ thuật/nghiệp vụ cần xử lý |

**Lưu ý:** Đây là lifecycle của Change Request, không phải Booking State Machine. Không thêm các state này vào Booking State. State cuối cùng cần đồng bộ với chuẩn workflow nền tảng.

### 5.4. Snapshot và Diff

Mỗi Change Request phải lưu:
- Revision gốc mà người yêu cầu dựa vào.
- Tập trường đề nghị thay đổi (patch).
- Giá trị trước thay đổi.
- Giá trị mới được đề nghị.
- Giá trị mới thực tế được áp dụng.
- Kết quả validate và impact review.
- Kết quả xác nhận/từ chối.
- Revision tạo ra sau khi áp dụng.

Không nhận một payload thay thế toàn bộ Booking mà không có kiểm soát field whitelist. Không cho phép client sửa trường hệ thống như trạng thái, ID, fleet_type, fare, driver assignment.

### 5.5. Quan hệ với X-01/X-02

- Booking vẫn dùng lifecycle X-01.
- BookingRevision là lịch sử phiên bản dữ liệu.
- Change Request có lifecycle riêng.
- DispatchRun/Offer/Assignment vẫn do M03 quản lý.
- Trip/TripStopProgress/TripEvent vẫn do M04 quản lý.
- FareQuote/FareResult vẫn do M05 quản lý.
- Event/Outbox theo X-03.
- Không tạo một trạng thái Booking “CHANGE_PENDING” nếu chưa có phê duyệt kiến trúc; dùng Change Request riêng và projection nếu UI cần hiển thị.

---

## 6. Quy tắc nghiệp vụ (Business Rules)

| Rule ID | Quy tắc | Nguồn | Trạng thái |
|---|---|---|---|
| BKG005-R01 | Change Request phải tham chiếu một Booking xác định | KPMG TO.P1.8/TO.P6.2 + thiết kế | Draft |
| BKG005-R02 | Không chọn Booking chỉ dựa vào “yêu cầu gần nhất” nếu có nhiều kết quả không đủ chắc chắn | Đề xuất an toàn dữ liệu | Draft |
| BKG005-R03 | Booking Code/ID là khóa tra cứu ưu tiên khi khách cung cấp | KPMG TO.P6.2/TO.P7 | Draft |
| BKG005-R04 | Nếu tra cứu theo phone/pickup trả về nhiều Booking phù hợp, yêu cầu nhân viên/khách xác nhận đúng Booking | Đề xuất | Draft |
| BKG005-R05 | Change Request phải có actor, source_channel, reason_code và based_on_revision | Đề xuất audit/version | Draft |
| BKG005-R06 | Mọi thay đổi được áp dụng phải tạo BookingRevision mới | X-01/X-02 | Draft |
| BKG005-R07 | Revision cũ không bị ghi đè/xóa | X-02 data invariant | Draft |
| BKG005-R08 | Chỉ cho phép thay đổi field nằm trong whitelist theo policy | Đề xuất security/data integrity | Draft |
| BKG005-R09 | Không cho BKG-005 sửa Booking ID, channel gốc, fleet_type, giá, Assignment/Trip state | Ownership X-02 | Draft |
| BKG005-R10 | Thay đổi Pickup phải qua BKG-003 và BKG-004 | Feature dependency | Draft |
| BKG005-R11 | Thay đổi Destination/Stops phải qua BKG-003/BKG-004 | Feature dependency | Draft |
| BKG005-R12 | Thay đổi Pickup/Time/Vehicle sau khi có Offer/Assignment phải đánh giá impact qua M03 | X-01/X-02 | Draft |
| BKG005-R13 | Thay đổi sau khi Trip bắt đầu không được ghi trực tiếp vào Booking; chuyển M04 Trip Change | X-01 ownership | Draft |
| BKG005-R14 | Thay đổi có tác động giá phải gọi M05 đánh giá lại; M01 không tự tính giá | X-02 ownership | Draft |
| BKG005-R15 | Chỉ báo APPLIED khi các bước đồng bộ bắt buộc đã đạt kết quả theo policy | Đề xuất consistency | Draft |
| BKG005-R16 | Nếu M03/M04/M05 không phản hồi, Change Request không được giả thành công | Đề xuất resilience | Draft |
| BKG005-R17 | Mọi thay đổi gửi cho tài xế phải có revision/version tham chiếu và delivery/ack status | Đề xuất | Draft |
| BKG005-R18 | Tài xế xác nhận nhận thông tin mới không đồng nghĩa với chấp nhận thay đổi điều phối nếu policy yêu cầu M03 quyết định | Đề xuất ownership | Draft |
| BKG005-R19 | Khách phải xác nhận lại khi thay đổi thuộc nhóm cần confirmation theo policy | TBD | Draft |
| BKG005-R20 | Mọi Change Request phải kiểm tra Booking State và trạng thái Dispatch/Trip liên quan | X-01 | Draft |
| BKG005-R21 | Change Request dùng expected version; xung đột phải từ chối hoặc yêu cầu tải lại, không last-write-wins | X-03/đề xuất | Draft |
| BKG005-R22 | Retry cùng idempotency key không tạo Change Request/Revision trùng | X-03 | Draft |
| BKG005-R23 | DIRECT/PARTNER không bị suy ra hoặc thay đổi từ channel hay người sửa | X-02 | Draft |
| BKG005-R24 | Hủy Booking là BKG-006; không dùng amendment để né chính sách hủy | Feature boundary | Draft |
| BKG005-R25 | Thay đổi số lượng xe có thể cần tách/điều chỉnh Booking theo chính sách; không tự tăng Assignment trong M01 | TBD/M03 | Draft |
| BKG005-R26 | Không gửi thông tin cá nhân nhạy cảm cho tài xế ngoài phạm vi cần thiết | X-04 | Draft |
| BKG005-R27 | Change Request bị từ chối/hết hạn/rút lại vẫn phải giữ lịch sử | Audit | Draft |
| BKG005-R28 | Nếu một phần liên Module đã áp dụng nhưng bước khác thất bại, phải có saga/compensation hoặc trạng thái cần xử lý; không rollback giả định qua service boundary | X-03 | Draft |
| BKG005-R29 | Thay đổi Booking sau khi tài xế đã nhận phải thông báo rõ revision mới và xử lý xác nhận theo policy | KPMG TO.P1.8 + gap | Draft |
| BKG005-R30 | Không được thay đổi một Booking đã COMPLETED/CANCELLED/EXPIRED/REJECTED bằng luồng thông thường | X-01 | Draft |

---

## 7. Ma trận cho phép thay đổi theo trạng thái

> Đây là ma trận thiết kế đề xuất để review. Không phải chính sách đã được Mai Linh xác nhận.

| Booking State | Tình trạng M03/M04 | Khả năng tạo Change Request | Cách xử lý đề xuất |
|---|---|---|---|
| DRAFT | Chưa Dispatch | Có | Sửa trực tiếp qua revision |
| SUBMITTED | Chưa Dispatch | Có | Validate lại trước khi ACCEPTED |
| VALIDATING | Đang validate | Có điều kiện | Version guard; restart validation nếu input thay đổi |
| ACCEPTED | Chưa có DispatchRun hiệu lực | Có | Validate và tạo revision |
| READY_FOR_DISPATCH | Dispatch chưa phát Offer | Có | M03 xác nhận chưa có tác động hoặc tạo DispatchRun mới |
| READY_FOR_DISPATCH | Offer đang hiệu lực | Có | M03 đánh giá thu hồi/refresh Offer |
| READY_FOR_DISPATCH | Assignment đã được nhận | Có điều kiện | M03 review; tài xế/khách xác nhận theo policy |
| SCHEDULED | Chưa đến thời điểm Dispatch | Có | Kiểm tra thời gian và policy; giữ SCHEDULED |
| SCHEDULED | Đang/đã kích hoạt Dispatch | Có điều kiện | Áp dụng ma trận Dispatch tương ứng |
| FULFILLING | Trip chưa bắt đầu thực tế nhưng Assignment/driver en route | Có điều kiện | M03/M04 review; không sửa âm thầm |
| FULFILLING | Trip đã STARTED | Không qua Booking Amendment thông thường | Chuyển M04 Trip Change |
| COMPLETED | Trip hoàn tất | Không | Mở case điều chỉnh/khiếu nại theo quy trình riêng |
| CANCELLED/EXPIRED/REJECTED/UNFULFILLED | Kết thúc | Không | Tạo Booking mới hoặc quy trình phục hồi riêng nếu được duyệt |
| ON_HOLD | Chờ xử lý | Có điều kiện | Chỉ actor được phân quyền và theo lý do Hold |

### 7.1. Nguyên tắc cần giữ

- Không suy ra Dispatch status chỉ từ Booking State.
- M01 phải đọc projection/API M03 để biết Offer/Assignment đang ở trạng thái nào.
- “Đã có xe đăng ký” trong KPMG là mô tả nghiệp vụ lịch sử; hệ thống mới phải ánh xạ chính xác sang Offer/Assignment hiện hành của M03.
- Trip đã bắt đầu là ranh giới quan trọng: thay đổi hành trình thực tế thuộc M04, không phải sửa dữ liệu Booking gốc.
- Thời gian khóa thay đổi, ngưỡng trước giờ đón và ngoại lệ cần Mai Linh quyết định.

---

## 8. Change Impact Matrix theo loại thay đổi

| Nhóm thay đổi | BKG-003 | BKG-004 | M03 Dispatch | M04 Trip | M05 Pricing |
|---|---|---|---|---|---|
| Pickup address/coordinate | Resolve + serviceability | Cập nhật Pickup | Đánh giá Candidate/Offer/Assignment; có thể thu hồi/phát lại | Cập nhật Trip Plan nếu chưa bắt đầu | Đánh giá lại quote nếu route/fare ảnh hưởng |
| Destination | Resolve + policy destination | Cập nhật Destination | Thông báo nếu ảnh hưởng điều kiện Dispatch | Cập nhật plan nếu chưa bắt đầu | Requote theo policy |
| VIA/order | Resolve từng Stop | Cập nhật Stop list/revision | Review ảnh hưởng Dispatch | Trip Plan update nếu chưa bắt đầu | Requote theo policy |
| Pickup time | Kiểm tra giờ phục vụ nếu policy | Không nhất thiết đổi Stop | Đánh giá eligibility/time window/Offer | Cập nhật kế hoạch Trip | Kiểm tra tariff/time band |
| Vehicle type/brand/features | Có thể cần kiểm tra service capability | Không | Re-evaluate eligibility/candidate | Cập nhật thông tin phục vụ | Requote nếu giá phụ thuộc loại xe |
| Vehicle quantity | Không | Không | M03 đánh giá nguồn lực; có thể tách request | Trip/Assignment riêng | Quote theo số lượng nếu hỗ trợ |
| Contact phone | Không | Không | Cập nhật thông tin liên hệ cần thiết cho tài xế | Cập nhật contact context theo policy | Thường không |
| Customer note | Không | Có thể cập nhật instruction | Thông báo tài xế nếu cần | Cập nhật Trip instruction nếu chưa bắt đầu | Không hoặc tùy nội dung |
| Fleet type | Không | Không | Không được đổi bằng BKG-005 | Không | Không |
| Fare amount | Không | Không | Không | Không | M05 sở hữu; không sửa từ M01 |

### 8.1. Phân loại tác động

- **LOW:** chỉ thay đổi thông tin không ảnh hưởng điều phối/giá/Trip; vẫn tạo revision và audit.
- **MEDIUM:** cần validate lại và thông báo các Module liên quan.
- **HIGH:** ảnh hưởng Pickup, thời gian, loại xe, Stops, giá hoặc Assignment; cần M03/M04/M05 review và có thể cần xác nhận.
- **PROHIBITED:** không được sửa bằng BKG-005 (ví dụ Trip đã bắt đầu, Booking đã kết thúc, fleet_type, giá).

Mức LOW/MEDIUM/HIGH chỉ là phân loại kỹ thuật đề xuất; danh mục field và severity phải được phê duyệt.

---

## 9. Luồng nghiệp vụ

### 9.1. Tiền điều kiện

- Booking tồn tại và actor có quyền truy cập.
- Xác định được Booking chính xác bằng ID/Code hoặc kết quả tra cứu đã xác nhận.
- Booking chưa ở trạng thái bị cấm sửa.
- Client có revision/version mới nhất hoặc chấp nhận xử lý conflict.
- Các Module liên quan có API/projection để đánh giá impact.
- Nếu thay đổi địa điểm, BKG-003/BKG-004 sẵn sàng.

### 9.2. Luồng chính — Khách yêu cầu thay đổi

| Bước | Actor | Hành động | Hệ thống xử lý | Kết quả |
|---|---|---|---|---|
| 1 | Khách/nhân viên | Chọn Booking cần sửa | Tra cứu theo Booking ID/Code hoặc tiêu chí được phép | Booking target |
| 2 | M01 | Xác minh quyền và Booking | Kiểm tra owner/data scope/state/version | Được phép hoặc từ chối |
| 3 | Khách/nhân viên | Chọn trường và nhập giá trị mới | Kiểm tra whitelist và định dạng | Change payload |
| 4 | M01 | Tạo Change Request | Lưu actor, channel, reason, base revision/version, idempotency key | SUBMITTED |
| 5 | M01 | Validate thay đổi | Gọi BKG-003/BKG-004 và kiểm tra rule | VALIDATING |
| 6 | M01 | Thu thập trạng thái liên quan | Đọc Dispatch/Trip projection từ M03/M04 | Impact context |
| 7 | M01 | Đánh giá tác động | Xác định Module cần review | IMPACT_REVIEW |
| 8 | M03/M04/M05 | Review theo ownership | Trả quyết định/điều kiện/quote review | Impact decisions |
| 9 | M01/Channel | Yêu cầu xác nhận | Gửi diff cho khách/tài xế/Điều hành nếu policy yêu cầu | Awaiting confirmation |
| 10 | Khách/nhân viên | Xác nhận thay đổi | Lưu bằng chứng và actor | Approved/ready |
| 11 | M01 | Áp dụng thay đổi Booking | Tạo BookingRevision trong transaction; cập nhật projection | APPLYING |
| 12 | M01 | Phát Event | Outbox phát BookingAmended/ChangeApplied theo X-03 | Consumer đồng bộ |
| 13 | M03/M04/M05 | Thực thi tác động | Thu hồi/refresh Dispatch, cập nhật Trip Plan, requote theo policy | Consumer result |
| 14 | M01 | Tổng hợp kết quả | Chỉ đánh dấu APPLIED khi điều kiện hoàn tất theo policy | APPLIED hoặc cần xử lý |
| 15 | Channel | Thông báo kết quả | Gửi kết quả và revision mới | Khách/nhân viên biết kết quả |

### 9.3. Luồng thay đổi trước Dispatch

1. Khách/nhân viên chọn Booking.
2. M01 kiểm tra chưa có DispatchRun/Offer/Assignment hiệu lực.
3. Validate dữ liệu mới qua Feature liên quan.
4. Tạo Revision mới.
5. Nếu có ảnh hưởng giá, gọi M05 theo policy.
6. Booking tiếp tục trạng thái phù hợp theo X-01.
7. M01 phát Event cho các consumer.

### 9.4. Luồng thay đổi khi Offer đang hiệu lực

1. M01 tiếp nhận Change Request.
2. M03 xác định Offer nào đang hiệu lực.
3. M03 quyết định giữ, thu hồi hoặc phát lại Offer theo policy.
4. M01 không phát Revision cuối cùng nếu cần M03 chấp thuận trước.
5. Khi đủ điều kiện, M01 commit Revision.
6. M03 xử lý DispatchRun/Offer tương ứng.
7. Khách được thông báo kết quả.

### 9.5. Luồng thay đổi sau khi tài xế đã nhận

1. M01 nhận yêu cầu và lấy Assignment status từ M03.
2. Phân loại mức độ thay đổi.
3. M03 đánh giá tài xế có tiếp tục nhận Assignment với thông tin mới hay không.
4. Nếu cần, yêu cầu tài xế xác nhận đã nhận thông tin mới; xác nhận này không thay thế quyết định M03.
5. Nếu tài xế từ chối/không phản hồi theo policy, M03 xử lý Assignment/Dispatch.
6. M05 đánh giá lại giá nếu cần.
7. M01 áp dụng Revision khi điều kiện liên Module đạt.
8. Hệ thống thông báo kết quả rõ ràng cho khách và tài xế.

### 9.6. Luồng khi Trip đã bắt đầu

1. M01 phát hiện Booking liên kết Trip đã STARTED.
2. BKG-005 từ chối sửa Requested Booking theo luồng thông thường.
3. Hướng người yêu cầu sang Trip Change flow của M04.
4. M04 quyết định có cho phép thay đổi hành trình/điểm dừng trong Trip hay không.
5. M05 đánh giá tác động giá theo Trip/Fare policy.
6. M01 không ghi đè BookingRevision gốc; chỉ liên kết Change Request/Trip Change reference nếu thiết kế được duyệt.

### 9.7. Luồng thay thế

| Mã | Tình huống | Xử lý | Kết quả |
|---|---|---|---|
| ALT-01 | Khách cung cấp Booking Code | Tra cứu trực tiếp | Chọn đúng Booking |
| ALT-02 | Khách không có Code, tra cứu theo phone/pickup | Hiển thị kết quả phù hợp, yêu cầu xác nhận nếu nhiều kết quả | Không tự chọn mơ hồ |
| ALT-03 | Chỉ thay đổi ghi chú không ảnh hưởng vận hành | Áp dụng theo whitelist và policy | Revision mới |
| ALT-04 | Đổi Pickup | BKG-003 resolve/check; BKG-004 cập nhật | Impact M03/M04/M05 |
| ALT-05 | Đổi Destination | BKG-003/BKG-004 validate | Requote nếu policy |
| ALT-06 | Đổi thời gian | Kiểm tra booking type/time window | M03 đánh giá lại |
| ALT-07 | Đổi loại xe | Kiểm tra khả năng cung cấp | M03 và M05 review |
| ALT-08 | Đổi số lượng xe | Kiểm tra cấu trúc Booking và nguồn lực | M03 quyết định |
| ALT-09 | Khách rút Change Request trước khi áp dụng | Chuyển WITHDRAWN | Không tạo Revision |
| ALT-10 | Khách từ chối xác nhận | REJECTED/WITHDRAWN theo policy | Giữ Revision cũ |
| ALT-11 | Tổng đài sửa thay mặt khách | Lưu actor và bằng chứng yêu cầu | Audit đầy đủ |
| ALT-12 | Tài xế không xác nhận revision mới | M03 xử lý timeout/refusal theo policy | Không báo APPLIED sai |

### 9.8. Luồng ngoại lệ

| Mã | Tình huống | Phản hồi | Phục hồi |
|---|---|---|---|
| EX-01 | Không tìm thấy Booking | BOOKING_NOT_FOUND | Kiểm tra lại ID/Code |
| EX-02 | Nhiều Booking trùng tiêu chí | BOOKING_MATCH_AMBIGUOUS | Khách/nhân viên chọn đúng |
| EX-03 | Actor không có quyền | PERMISSION_DENIED | Kiểm tra scope |
| EX-04 | Booking đã kết thúc | BOOKING_NOT_AMENDABLE | Quy trình mới/case riêng |
| EX-05 | Trip đã bắt đầu | TRIP_CHANGE_REQUIRED | Chuyển M04 |
| EX-06 | Field không nằm whitelist | FIELD_NOT_AMENDABLE | Loại field hoặc quy trình khác |
| EX-07 | Giá trị mới không hợp lệ | VALIDATION_FAILED | Sửa dữ liệu |
| EX-08 | Địa điểm không resolve | LOCATION_NOT_RESOLVED | BKG-003/manual review |
| EX-09 | Ngoài Service Area | SERVICEABILITY_FAILED | Policy/manual review |
| EX-10 | Version đã cũ | STATE_VERSION_CONFLICT | Reload và gửi lại |
| EX-11 | Offer/Assignment thay đổi trong lúc review | DISPATCH_CONTEXT_CHANGED | M03 review lại |
| EX-12 | M03 từ chối thay đổi | DISPATCH_CHANGE_REJECTED | Giữ Revision cũ/đề xuất phương án |
| EX-13 | M04 xác định Trip đã bắt đầu | TRIP_CHANGE_REQUIRED | Chuyển M04 |
| EX-14 | M05 không thể requote | QUOTE_REEVALUATION_FAILED | Retry/manual review |
| EX-15 | Khách không xác nhận trước deadline | CHANGE_CONFIRMATION_EXPIRED | Đóng request, giữ Revision cũ |
| EX-16 | Tài xế không xác nhận | DRIVER_CONFIRMATION_TIMEOUT | M03 quyết định theo policy |
| EX-17 | Notification không gửi được | NOTIFICATION_DELIVERY_FAILED | Retry; không rollback Booking nếu policy cho phép |
| EX-18 | M03/M04/M05 timeout | DOWNSTREAM_SERVICE_TIMEOUT | Retry/query status; không báo thành công |
| EX-19 | Event phát lặp | DUPLICATE_EVENT | Consumer idempotent |
| EX-20 | Một consumer thành công, consumer khác lỗi | CHANGE_PROPAGATION_PARTIAL | Saga/compensation/escalation |
| EX-21 | Hai Change Request cùng lúc | CHANGE_REQUEST_CONFLICT | Một request thắng theo version; request còn lại review lại |
| EX-22 | Client retry cùng idempotency key | IDEMPOTENCY_REPLAY | Trả kết quả cũ, không tạo bản ghi trùng |
| EX-23 | Nhân viên cố đổi fleet_type | FIELD_NOT_AMENDABLE | Từ chối; quy trình M02/Contract riêng |
| EX-24 | Khách yêu cầu hủy thay vì sửa | CANCEL_FLOW_REQUIRED | Chuyển BKG-006 |
| EX-25 | Booking đã được sửa ở channel khác | STATE_VERSION_CONFLICT | Tải revision mới |
| EX-26 | M01 commit thành công nhưng response timeout | REQUEST_RESULT_UNKNOWN | Tra cứu idempotency/status trước khi retry |

### 9.9. Hậu điều kiện

**Thành công**
- Change Request có trạng thái APPLIED.
- BookingRevision mới được tạo.
- Các Module liên quan đã xử lý theo policy hoặc có trạng thái hoàn tất rõ ràng.
- Khách/nhân viên nhận được kết quả và revision.
- Audit/Event được lưu.

**Không thành công**
- Revision gốc không bị ghi đè.
- Không báo thay đổi thành công nếu chưa đủ điều kiện.
- Change Request giữ trạng thái lỗi/chờ xử lý phù hợp.
- Có thể retry an toàn hoặc chuyển Điều hành xử lý.
- Không làm mất trạng thái Offer/Assignment/Trip của Module sở hữu.

---

## 10. UI/UX

### 10.1. Màn hình tra cứu Booking

- Tìm theo Booking Code/ID.
- Hỗ trợ phone/pickup/time trong phạm vi permission.
- Hiển thị nhiều kết quả khi có trùng, không tự chọn mù.
- Hiển thị tóm tắt: kênh, thời điểm, Pickup, Destination, loại xe, Booking State.
- Hiển thị Dispatch/Assignment/Trip status từ read projection, có timestamp/version.
- Không hiển thị dữ liệu cá nhân quá mức cần thiết.

### 10.2. Màn hình yêu cầu thay đổi

- Hiển thị dữ liệu hiện tại (read-only snapshot).
- Chỉ cho phép chỉnh field được policy bật.
- Hiển thị giá trị cũ và giá trị mới cạnh nhau.
- Chọn reason code; có ô ghi chú.
- Hiển thị cảnh báo tác động: điều phối lại, tài xế xác nhận, báo giá lại, cần tổng đài duyệt.
- Hiển thị trạng thái xử lý Change Request.
- Có nút rút yêu cầu nếu chưa áp dụng và policy cho phép.
- Không cho client sửa trạng thái hệ thống.

### 10.3. Màn hình xác nhận thay đổi

Khách cần thấy:
- Nội dung thay đổi.
- Thông tin giá mới nếu M05 trả kết quả và policy yêu cầu.
- Trạng thái tài xế/xe có thể thay đổi hay không.
- Hạn xác nhận nếu có.
- Hành động xác nhận/từ chối.

Tài xế cần thấy:
- Booking reference.
- Revision hiện tại và revision mới.
- Những trường thay đổi liên quan đến nhiệm vụ.
- Hành động xác nhận đã nhận thông tin hoặc từ chối theo quy trình M03.
- Không hiển thị toàn bộ dữ liệu khách nếu không cần.

### 10.4. Message Code đề xuất

| Code | Nội dung |
|---|---|
| CHANGE-001 | Không tìm thấy yêu cầu đặt xe |
| CHANGE-002 | Có nhiều yêu cầu phù hợp; vui lòng chọn đúng chuyến |
| CHANGE-003 | Yêu cầu thay đổi đã được tiếp nhận |
| CHANGE-004 | Yêu cầu đang chờ xác nhận |
| CHANGE-005 | Thay đổi cần Tổng đài/Điều hành kiểm tra |
| CHANGE-006 | Thay đổi có thể ảnh hưởng đến xe đã nhận chuyến |
| CHANGE-007 | Cần cập nhật lại báo giá |
| CHANGE-008 | Yêu cầu thay đổi đã được áp dụng |
| CHANGE-009 | Không thể thay đổi ở trạng thái hiện tại |
| CHANGE-010 | Chuyến đi đã bắt đầu; vui lòng liên hệ tài xế/Tổng đài |
| CHANGE-011 | Thông tin đặt xe đã được cập nhật ở nơi khác; vui lòng tải lại |
| CHANGE-012 | Chưa thể xác nhận thay đổi; hệ thống đang kiểm tra với bộ phận điều hành |

Nội dung cuối cùng cần Business Owner phê duyệt.

---

## 11. API và Event Contract

> Endpoint/payload là đề xuất, chưa phải hợp đồng production. Đồng bộ X-03.

### 11.1. Tạo Change Request

**Command:** `CreateBookingChangeRequest`  
**Endpoint tham khảo:** `POST /api/v1/bookings/{booking_id}/change-requests`

Request minh họa:
```json
{
  "based_on_revision": 4,
  "expected_version": 7,
  "reason_code": "CUSTOMER_PICKUP_CHANGE",
  "reason_text": "Khách yêu cầu đổi địa chỉ đón",
  "changes": {
    "pickup": {
      "location_input": {
        "input_text": "Địa chỉ mới do khách cung cấp",
        "source": "CUSTOMER_ENTERED"
      }
    }
  },
  "source_channel": "CALL_CENTER",
  "idempotency_key": "client-generated-key"
}
```

### 11.2. Impact Review

**Command nội bộ:** `ReviewBookingChangeImpact`

M01 điều phối yêu cầu review qua API của M03/M04/M05. Không gọi thẳng DB của Module khác.

Kết quả logic:
```json
{
  "change_request_id": "change-request-id",
  "impact_level": "HIGH",
  "reviews": [
    {
      "module": "M03",
      "decision": "REQUIRES_REDISPATCH",
      "reason_code": "PICKUP_CHANGED"
    },
    {
      "module": "M05",
      "decision": "REQUOTE_REQUIRED",
      "reason_code": "ROUTE_OR_FARE_IMPACT"
    }
  ],
  "confirmation_required": true
}
```

Các giá trị enum chỉ là ví dụ; cần chuẩn hóa với X-03 và từng Module.

### 11.3. Confirm/Reject/Withdraw

Đề xuất:
- `POST /api/v1/bookings/{booking_id}/change-requests/{change_request_id}/confirm`
- `POST /api/v1/bookings/{booking_id}/change-requests/{change_request_id}/reject`
- `POST /api/v1/bookings/{booking_id}/change-requests/{change_request_id}/withdraw`

Mỗi command phải có actor, expected_version và idempotency key khi phù hợp.

### 11.4. Apply Change

**Command nội bộ:** `ApplyBookingChangeRequest`

M01 chỉ tạo Revision sau khi:
- Request còn hiệu lực.
- Booking State cho phép.
- Các xác nhận bắt buộc đã hoàn tất.
- M03/M04/M05 trả quyết định cần thiết.
- Expected version vẫn đúng.

### 11.5. Response minh họa

```json
{
  "change_request_id": "change-request-id",
  "booking_id": "booking-id",
  "status": "AWAITING_OPERATOR_DECISION",
  "based_on_revision": 4,
  "current_revision": 4,
  "impact_level": "HIGH",
  "confirmation_required": true,
  "next_action": "WAIT_FOR_OPERATOR"
}
```

Không trả status APPLIED trước khi Revision và các bước bắt buộc đã hoàn tất.

### 11.6. Event đề xuất

- `BookingChangeRequested`
- `BookingChangeImpactReviewed`
- `BookingChangeConfirmationRequested`
- `BookingChangeApproved`
- `BookingAmended`
- `BookingChangeRejected`
- `BookingChangeExpired`
- `BookingChangePropagationFailed`

Không cần phát tất cả event nếu không có consumer; chuẩn hóa Event Catalog theo X-03.

Event `BookingAmended` tối thiểu có:
- event_id, event_type, occurred_at.
- booking_id, booking_revision, aggregate_version.
- change_request_id, changed_fields.
- actor/source_channel/change_reason.
- correlation_id/causation_id.
- snapshot/reference cần cho consumer.

Dùng Transactional Outbox và consumer idempotent. Với các bước liên Module, định nghĩa Saga/compensation rõ ràng; không giả định distributed transaction.

### 11.7. Error Codes đề xuất

| Code | Ý nghĩa |
|---|---|
| BOOKING_NOT_FOUND | Không tìm thấy Booking |
| BOOKING_MATCH_AMBIGUOUS | Có nhiều Booking phù hợp |
| BOOKING_NOT_AMENDABLE | Booking không cho phép sửa |
| TRIP_CHANGE_REQUIRED | Trip đã bắt đầu; chuyển M04 |
| FIELD_NOT_AMENDABLE | Field không được phép sửa |
| CHANGE_REQUEST_NOT_FOUND | Không tìm thấy Change Request |
| CHANGE_REQUEST_CONFLICT | Change Request xung đột |
| CHANGE_CONFIRMATION_EXPIRED | Quá hạn xác nhận |
| DISPATCH_CHANGE_REJECTED | M03 từ chối thay đổi |
| DRIVER_CONFIRMATION_TIMEOUT | Tài xế không xác nhận |
| QUOTE_REEVALUATION_FAILED | M05 không thể đánh giá lại |
| DOWNSTREAM_SERVICE_TIMEOUT | Module liên quan timeout |
| CHANGE_PROPAGATION_PARTIAL | Đồng bộ liên Module chưa hoàn tất |
| STATE_VERSION_CONFLICT | Version đã thay đổi |
| IDEMPOTENCY_KEY_REUSED | Idempotency key dùng sai payload |
| CANCEL_FLOW_REQUIRED | Yêu cầu thuộc luồng hủy |

---

## 12. Bảo mật, Audit và dữ liệu cá nhân

### 12.1. Audit bắt buộc

| Audit Action | Khi nào |
|---|---|
| BOOKING_CHANGE_REQUESTED | Tạo Change Request |
| BOOKING_CHANGE_VALIDATED | Validate hoàn tất |
| BOOKING_CHANGE_IMPACT_REVIEWED | Module review |
| BOOKING_CHANGE_CUSTOMER_CONFIRMED | Khách xác nhận |
| BOOKING_CHANGE_DRIVER_ACKNOWLEDGED | Tài xế xác nhận đã nhận thông tin |
| BOOKING_CHANGE_APPROVED | Được chấp nhận |
| BOOKING_CHANGE_REJECTED | Bị từ chối |
| BOOKING_CHANGE_WITHDRAWN | Rút yêu cầu |
| BOOKING_REVISION_CREATED | Tạo Revision |
| BOOKING_CHANGE_APPLIED | Áp dụng hoàn tất |
| BOOKING_CHANGE_PROPAGATION_FAILED | Đồng bộ liên Module lỗi |
| BOOKING_CHANGE_RETRIED | Retry |
| BOOKING_CHANGE_EXPIRED | Hết hạn |

Audit lưu actor, channel, timestamp, reason, before/after snapshot/reference, revision, change_request_id, correlation ID và decision. Không ghi token/bí mật.

### 12.2. Quyền riêng tư

- Số điện thoại/địa chỉ được masking theo vai trò.
- Tổng đài viên chỉ xem dữ liệu trong data scope.
- Tài xế chỉ nhận trường cần cho việc đón/chở khách.
- Lưu bằng chứng khách xác nhận theo retention được phê duyệt.
- Không gửi dữ liệu cá nhân vào log kỹ thuật không được bảo vệ.
- Quy tắc retention/masking theo X-04.

---

## 13. NFR và vận hành

| NFR | Yêu cầu |
|---|---|
| Data Integrity | Không ghi đè revision cũ; không áp dụng một phần không kiểm soát |
| Consistency | Change Request/Revision có version rõ ràng |
| Concurrency | Expected version/optimistic locking |
| Idempotency | Retry không tạo request/revision trùng |
| Reliability | Có Outbox, retry, dead-letter/escalation theo X-03 |
| Availability | Module downstream lỗi không tạo false success |
| Performance | SLA tạo request và impact review chốt trong X-05 |
| Auditability | Tái hiện ai đổi gì, khi nào, vì sao, được ai duyệt |
| Security | RBAC, data scope, masking, least privilege |
| Observability | Correlation ID xuyên M01/M03/M04/M05 |
| Recovery | Có cơ chế query status/reconcile khi response timeout |
| Compatibility | Consumer xử lý revision/event version mới |

Metrics đề xuất:
- booking_change_request_total theo channel/reason.
- booking_change_applied_total.
- booking_change_rejected_total theo reason.
- booking_change_pending_duration.
- booking_change_dispatch_review_latency.
- booking_change_quote_review_latency.
- booking_change_driver_ack_latency.
- booking_change_version_conflict_total.
- booking_change_propagation_failure_total.
- booking_change_after_assignment_total.
- booking_change_after_trip_start_attempt_total.
- booking_change_idempotency_replay_total.

---

## 14. Kiểm thử và Acceptance Criteria

### 14.1. Test Cases

| Test ID | Tình huống | Kết quả mong đợi |
|---|---|---|
| BKG005-TC-001 | Khách tìm Booking bằng ID/Code hợp lệ | Tìm đúng Booking |
| BKG005-TC-002 | Tra cứu bằng phone/pickup có nhiều kết quả | Không tự chọn; yêu cầu xác nhận |
| BKG005-TC-003 | Booking không tồn tại | BOOKING_NOT_FOUND |
| BKG005-TC-004 | Khách sửa Booking của người khác | PERMISSION_DENIED |
| BKG005-TC-005 | Nhân viên ngoài data scope sửa Booking | PERMISSION_DENIED |
| BKG005-TC-006 | Sửa field trong whitelist khi DRAFT | Tạo Change Request/Revision hợp lệ |
| BKG005-TC-007 | Sửa field không được phép | FIELD_NOT_AMENDABLE |
| BKG005-TC-008 | Sửa Booking chưa Dispatch | Validate và apply theo policy |
| BKG005-TC-009 | Sửa khi Offer đang hiệu lực | M03 impact review |
| BKG005-TC-010 | Sửa khi Assignment đã nhận | M03 review; không sửa âm thầm |
| BKG005-TC-011 | Tài xế xác nhận đã nhận revision mới | Lưu acknowledgement đúng Assignment/revision |
| BKG005-TC-012 | Tài xế không xác nhận trong thời hạn | M03 xử lý theo policy; không báo APPLIED sai |
| BKG005-TC-013 | Đổi Pickup | BKG-003 resolve/check và M03 review |
| BKG005-TC-014 | Đổi Destination | BKG-003/BKG-004 validate |
| BKG005-TC-015 | Đổi VIA/order | BKG-004 validate và M03/M04/M05 review |
| BKG005-TC-016 | Đổi Pickup ngoài Service Area | Từ chối/review theo BKG-003 |
| BKG005-TC-017 | Đổi Pickup khi Assignment đang en route | M03 review; không cập nhật trực tiếp |
| BKG005-TC-018 | Đổi thời gian đón | Validate và M03 đánh giá lại |
| BKG005-TC-019 | Đổi loại xe | M02/M03/M05 review |
| BKG005-TC-020 | Đổi số lượng xe | M03 quyết định; không tự tạo Assignment |
| BKG005-TC-021 | Đổi contact phone | Kiểm tra quyền/xác minh theo policy |
| BKG005-TC-022 | Đổi customer note | Audit và thông báo đúng consumer |
| BKG005-TC-023 | Đổi fleet_type | FIELD_NOT_AMENDABLE |
| BKG005-TC-024 | Đổi giá cước qua M01 | FIELD_NOT_AMENDABLE |
| BKG005-TC-025 | M05 yêu cầu requote | Chờ kết quả M05 trước khi hoàn tất theo policy |
| BKG005-TC-026 | M05 timeout | Không báo APPLIED; retry/reconcile |
| BKG005-TC-027 | M03 từ chối thay đổi | Giữ revision cũ; thông báo lý do |
| BKG005-TC-028 | M03 timeout | Change Request ở trạng thái chờ/lỗi có thể phục hồi |
| BKG005-TC-029 | Trip đã STARTED | TRIP_CHANGE_REQUIRED; chuyển M04 |
| BKG005-TC-030 | Trip đã COMPLETED | Không sửa Booking thông thường |
| BKG005-TC-031 | Booking CANCELLED | Không cho sửa |
| BKG005-TC-032 | Khách rút request trước apply | WITHDRAWN; không tạo Revision |
| BKG005-TC-033 | Khách không xác nhận đúng hạn | EXPIRED theo policy |
| BKG005-TC-034 | Hai Change Request cùng expected_version | Chỉ một request được apply; request kia conflict |
| BKG005-TC-035 | Retry cùng idempotency key và payload | Trả kết quả cũ, không tạo trùng |
| BKG005-TC-036 | Dùng idempotency key cũ với payload khác | IDEMPOTENCY_KEY_REUSED |
| BKG005-TC-037 | M01 commit thành công nhưng client timeout | Tra cứu được kết quả cũ qua idempotency |
| BKG005-TC-038 | M03 thành công, M05 lỗi | Propagation partial; saga/reconcile |
| BKG005-TC-039 | Notification thất bại | Retry; không mất Revision |
| BKG005-TC-040 | Event phát lặp | Consumer xử lý idempotent |
| BKG005-TC-041 | Tổng đài sửa thay khách | Lưu agent và evidence/reason |
| BKG005-TC-042 | Khách yêu cầu hủy thay vì sửa | Chuyển BKG-006 |
| BKG005-TC-043 | Lịch sử Revision | Xem được before/after và actor |
| BKG005-TC-044 | DIRECT/PARTNER Booking amendment | Giữ nguyên fleet_type và contract context |
| BKG005-TC-045 | Booking thay đổi từ kênh khác trong lúc review | Version conflict; tải lại |
| BKG005-TC-046 | Booking State ON_HOLD | Chỉ actor/policy cho phép mới sửa |

### 14.2. Acceptance Criteria

**AC-01 — Xác định đúng Booking**
- Given khách/nhân viên gửi yêu cầu thay đổi
- When hệ thống tra cứu Booking
- Then phải xác định duy nhất Booking mục tiêu
- And nếu có nhiều kết quả mơ hồ, không tự chọn một Booking

**AC-02 — Revision bất biến**
- Given một Booking có revision hiện tại
- When Change Request được áp dụng
- Then tạo revision mới
- And revision cũ vẫn truy xuất được

**AC-03 — State-aware**
- Given Booking/Dispatch/Trip ở trạng thái cụ thể
- When actor gửi thay đổi
- Then hệ thống áp dụng policy tương ứng
- And không dùng một quy tắc sửa chung cho mọi trạng thái

**AC-04 — Dispatch impact**
- Given Booking có Offer/Assignment hiệu lực
- When thay đổi tác động tới Pickup/Time/Vehicle/Stops
- Then M03 phải review
- And M01 không tự sửa Offer/Assignment

**AC-05 — Trip boundary**
- Given Trip đã bắt đầu
- When actor yêu cầu sửa Booking
- Then BKG-005 không ghi Requested Booking
- And chuyển sang M04 Trip Change

**AC-06 — Fare impact**
- Given thay đổi có thể ảnh hưởng giá
- When Change Request được review
- Then M05 đánh giá lại theo policy
- And M01 không tự tính/sửa giá

**AC-07 — Confirmation**
- Given policy yêu cầu khách/tài xế xác nhận
- When chưa có xác nhận hợp lệ
- Then Change Request không được đánh dấu APPLIED

**AC-08 — Consistency**
- Given một Module downstream timeout/failure
- When Change Request đang áp dụng
- Then hệ thống lưu trạng thái propagation/recovery
- And không báo thành công giả

**AC-09 — Idempotency/concurrency**
- Given request retry hoặc hai request đồng thời
- When cùng idempotency key/version
- Then không tạo revision trùng hoặc ghi đè mất thay đổi

**AC-10 — Audit**
- Given bất kỳ Change Request nào
- When auditor tra cứu
- Then có thể xem actor, channel, reason, before/after, decisions, revision và thời điểm

### 14.3. Definition of Done riêng

- [ ] Business Owner duyệt danh sách field được sửa theo Booking State.
- [ ] Chốt tra cứu Booking và quy tắc xử lý kết quả trùng.
- [ ] Chốt change policy trước/sau Offer/Assignment.
- [ ] M03 duyệt impact matrix và cơ chế thu hồi/phát lại Dispatch.
- [ ] M04 duyệt ranh giới Trip Start và Trip Change.
- [ ] M05 duyệt quy tắc requote/giữ giá.
- [ ] BKG-003/BKG-004 xác nhận contract validate địa điểm/Stops.
- [ ] X-01 xác nhận Booking State guards và Change Request lifecycle.
- [ ] X-02 xác nhận BookingRevision/ChangeRequest Entity.
- [ ] X-03 xác nhận API/Event/Saga/Outbox/Idempotency.
- [ ] X-04 xác nhận quyền, audit, PII/retention.
- [ ] X-05 xác nhận NFR/SLA/observability.
- [ ] QA hoàn thành Test Cases và evidence.
- [ ] Business UAT ký xác nhận.
- [ ] M01/M03/M04/M05 ký ownership và tích hợp.

---

## 15. Migration, cấu hình và triển khai

### 15.1. Cấu hình cần có

| Configuration | Ý nghĩa | Owner | Trạng thái |
|---|---|---|---|
| CHANGEABLE_FIELDS_BY_STATE | Field được sửa theo Booking State | Product/Operations | TBD |
| CHANGEABLE_FIELDS_BY_CHANNEL | Field được sửa theo channel | Product/Operations | TBD |
| CHANGE_CONFIRMATION_POLICY | Trường hợp cần khách xác nhận | Operations | TBD |
| DRIVER_ACK_POLICY | Trường hợp cần tài xế xác nhận | M03/Operations | TBD |
| CHANGE_REQUEST_EXPIRY | Thời hạn request/xác nhận | Operations | TBD |
| DISPATCH_CHANGE_POLICY | Quy tắc Offer/Assignment khi thay đổi | M03 | TBD |
| TRIP_CHANGE_BOUNDARY | Mốc chuyển sang M04 | M04/Operations | TBD |
| REQUOTE_POLICY | Trường hợp cần báo giá lại | M05 | TBD |
| CHANGE_REASON_CODES | Danh mục lý do thay đổi | Product/Operations | TBD |
| CHANGE_APPROVAL_MATRIX | Cấp duyệt theo mức tác động | Operations | TBD |

Không hard-code thời hạn xác nhận, số lần retry, SLA hoặc ngưỡng thời gian khi chưa có quyết định.

### 15.2. Triển khai

1. Phê duyệt whitelist field và state guard.
2. Hoàn thiện ChangeRequest/Revision model theo X-02.
3. Chốt API/Event/Saga với M03/M04/M05 theo X-03.
4. Kiểm thử concurrency, idempotency, timeout và partial propagation.
5. Chạy thử nội bộ với Call Center trước, sau đó App/Web/Chat theo kế hoạch.
6. Theo dõi Change Request pending, dispatch review latency, quote failure, propagation failure.
7. Bật theo feature flag; có thể tắt tạo request mới nhưng vẫn giữ lịch sử đã phát sinh.
8. Có quy trình reconciliation cho request APPLYING/partial failure.

---

## 16. Rủi ro, quyết định và TBD

| ID | Nội dung cần quyết định | Ảnh hưởng | Owner đề xuất |
|---|---|---|---|
| BKG005-DEC-01 | Danh sách field nào khách được tự sửa? | UX/Booking | Product/Operations |
| BKG005-DEC-02 | Danh sách field nào tổng đài được sửa thay khách? | Call Center/IAM | Operations |
| BKG005-DEC-03 | Field nào bị khóa sau khi Booking ACCEPTED? | Booking State | Operations |
| BKG005-DEC-04 | Có cho sửa Pickup sau khi tài xế nhận Assignment không? | Dispatch | M03/Operations |
| BKG005-DEC-05 | Đổi Pickup sau Assignment cần tài xế xác nhận hay M03 tự quyết định? | Dispatch/Driver | M03 |
| BKG005-DEC-06 | Nếu tài xế không xác nhận revision mới, xử lý ra sao? | Dispatch SLA | M03/Operations |
| BKG005-DEC-07 | Đổi Destination sau Assignment có cần báo khách/tài xế xác nhận? | Trip/Pricing | M04/M05 |
| BKG005-DEC-08 | Đổi Pickup/Time/Vehicle có buộc thu hồi Offer hiện tại? | Dispatch | M03 |
| BKG005-DEC-09 | Có cho đổi thời gian đón trong khoảng bao lâu trước giờ đón? | Scheduling | Operations |
| BKG005-DEC-10 | Có cho đổi loại xe sau khi đã có tài xế nhận? | Fleet/Dispatch | M02/M03 |
| BKG005-DEC-11 | Đổi số lượng xe xử lý bằng một Booking hay tách Booking? | Data/Dispatch | Product/M03 |
| BKG005-DEC-12 | Thay đổi nào cần khách xác nhận lại qua OTP/deep link/call? | Security/UX | Product/ATTT |
| BKG005-DEC-13 | Thay đổi nào cần Điều hành phê duyệt? | Operations | COO |
| BKG005-DEC-14 | Change Request có thời hạn xử lý bao lâu? | SLA | Operations |
| BKG005-DEC-15 | Request hết hạn thì giữ Booking cũ hay hủy? | Customer policy | Operations |
| BKG005-DEC-16 | Nếu M03 đã thu hồi Offer nhưng M01 apply thất bại thì compensation thế nào? | Saga | M01/M03 |
| BKG005-DEC-17 | Nếu M01 Revision đã commit nhưng M05 requote thất bại thì trạng thái nào? | Consistency | M01/M05 |
| BKG005-DEC-18 | Có cho khách sửa Booking sau khi Trip đã bắt đầu qua App không? | Trip Change | M04/Operations |
| BKG005-DEC-19 | Ai có quyền chấp nhận thay đổi trong Trip: khách, tài xế, Tổng đài? | Trip Governance | M04 |
| BKG005-DEC-20 | Quy tắc giá khi đổi Destination/Stops/Time/Vehicle? | Fare | M05/Finance |
| BKG005-DEC-21 | Có giữ giá cũ hay báo giá lại và yêu cầu khách xác nhận? | Fare UX | M05 |
| BKG005-DEC-22 | Có cho đổi contact phone sau khi tài xế nhận không? | Privacy/Driver | Operations/ATTT |
| BKG005-DEC-23 | Có cho sửa ghi chú sau khi Trip bắt đầu không? | Trip | M04 |
| BKG005-DEC-24 | Quy tắc thay đổi đối với Booking PARTNER khác DIRECT ra sao? | Contract | M02/Partner Ops |
| BKG005-DEC-25 | Có cho sửa Booking được tạo từ Partner API không, và bên nào là nguồn chuẩn? | Integration | M01/Partner |
| BKG005-DEC-26 | Mã lý do thay đổi và danh mục báo cáo quản trị? | Reporting | Operations/M10 |
| BKG005-DEC-27 | Tra cứu bằng phone/pickup cần xác minh danh tính ở mức nào? | Privacy/Fraud | CSKH/ATTT |
| BKG005-DEC-28 | Retention của Change Request, snapshot và bằng chứng xác nhận? | Compliance | Data Owner/ATTT |
| BKG005-DEC-29 | Có cho phép nhiều Change Request mở đồng thời trên một Booking không? | Concurrency/UX | Product/Architecture |
| BKG005-DEC-30 | Có cần giới hạn số lần sửa theo Booking hoặc theo field? | Operations/Fraud | Operations |
| BKG005-DEC-31 | Khi khách yêu cầu sửa nhưng thực chất muốn hủy, chuyển BKG-006 ở bước nào? | Customer Journey | Product |
| BKG005-DEC-32 | Cấp nào xử lý propagation failure và thời gian escalation? | Operations/SRE | M01/M03/M04/M05 |

---

## 17. Tiêu chí nghiệm thu Feature

- [ ] Tra cứu được Booking chính xác; xử lý nhiều kết quả mơ hồ an toàn.
- [ ] Có Change Request riêng, không trộn với Booking State.
- [ ] BookingRevision append-only và truy xuất được before/after.
- [ ] Field whitelist và state guard được phê duyệt.
- [ ] Change impact tới M03/M04/M05 được xác định rõ.
- [ ] BKG-003/BKG-004 được gọi khi thay đổi địa điểm/Stops.
- [ ] Trip đã bắt đầu chuyển M04, không sửa Booking gốc.
- [ ] M05 là nguồn duy nhất quyết định báo giá/cước.
- [ ] M03 là nguồn duy nhất quản lý Offer/Assignment/Dispatch.
- [ ] Có customer/driver confirmation theo policy.
- [ ] Có API/Event/Saga/Outbox/Idempotency theo X-03.
- [ ] Có concurrency control và reconciliation.
- [ ] Audit/PII được duyệt theo X-04.
- [ ] NFR/SLA/monitoring/runbook được duyệt theo X-05.
- [ ] QA hoàn tất Test Cases; Business UAT ký.
- [ ] M01/M03/M04/M05 ký xác nhận ownership.

### 17.1. Kết luận hiện tại

**Trạng thái:** DRAFT — Chưa đạt READY/DoR.

**Blocker chính:**
1. Chưa chốt field nào được sửa theo từng Booking State và channel.
2. Chưa chốt xử lý thay đổi khi Offer/Assignment đã tồn tại.
3. Chưa chốt xác nhận của tài xế và khách.
4. Chưa chốt ranh giới Booking Amendment và Trip Change.
5. Chưa chốt chính sách báo giá lại.
6. Chưa chốt Saga/compensation khi liên Module chỉ thành công một phần.

**Không triển khai cơ chế sửa trực tiếp Booking sau khi tài xế đã nhận chuyến hoặc Trip đã bắt đầu khi các chính sách và hợp đồng liên Module chưa được phê duyệt.**

---

**Quay lại:** [Feature Specifications](../README.md) · [M01 — Customer & Booking](../../../02-DOMAIN-MODULES/01-customer-and-booking/README.md)
