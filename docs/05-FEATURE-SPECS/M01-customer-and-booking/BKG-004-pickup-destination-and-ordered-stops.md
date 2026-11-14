# BKG-004 — QUẢN LÝ ĐIỂM ĐÓN, ĐIỂM ĐẾN VÀ CÁC ĐIỂM DỪNG THEO THỨ TỰ

**Mã tài liệu:** TAXI-BKG-004  
**Module sở hữu:** M01 — Khách hàng và Đặt xe  
**Epic:** Tiếp nhận và quản lý Booking đa kênh  
**Phiên bản:** 0.1  
**Trạng thái:** DRAFT — CHỜ NGHIỆP VỤ, M03, M04, M05 VÀ KIẾN TRÚC REVIEW  
**Priority theo Feature Catalog:** P0  
**Business Owner:** TBD  
**Technical Owner:** M01 Owner — TBD  
**Tài liệu liên quan:** [M01 Feature Catalog](../../../02-DOMAIN-MODULES/01-customer-and-booking/README.md) · [BKG-001 — Tạo Booking tức thời](./BKG-001-create-immediate-booking.md) · [BKG-003 — Kiểm tra hợp lệ và vùng phục vụ](./BKG-003-validation-and-serviceability.md) · [X-01 — State Machine](../../../03-CROSS-CUTTING/01-state-machines-and-transition-rules.md) · [X-02 — Data Architecture](../../../03-CROSS-CUTTING/02-data-architecture-and-erd.md) · [X-03 — API/Event](../../../03-CROSS-CUTTING/03-api-and-event-contracts.md)

> **Quy ước nguồn:** KPMG 2015 mô tả điểm đón và điểm đến tùy chọn trong TO.P2/TO.P7; dữ liệu địa chỉ, địa danh; ước tính lộ trình/cước phí; và các nhánh đường dài một chiều/hai chiều trong TO.P9. KPMG không đặc tả một mô hình tổng quát cho nhiều điểm dừng trung gian tùy ý. Chức năng Ordered Stops trong tài liệu này là đề xuất mở rộng thiết kế, phải được Mai Linh xác nhận trước khi phát triển.

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
| 0.1 | Đặc tả điểm đón, điểm đến, điểm dừng trung gian và thứ tự hành trình; đánh dấu phần cần xác nhận |

---

## 1. Bối cảnh và mục tiêu

### 1.1. Vấn đề nghiệp vụ

Một Booking cần biểu diễn rõ nơi khách muốn được đón, nơi khách muốn đến và (nếu chính sách cho phép) các điểm dừng trong hành trình. Dữ liệu này được sử dụng bởi nhiều Module:
- M01 lưu yêu cầu gốc và lịch sử thay đổi.
- BKG-003 kiểm tra định vị và khả năng phục vụ cho từng địa điểm.
- M03 sử dụng điểm đón để tìm xe và điều phối.
- M04 sử dụng danh sách điểm đã xác nhận để theo dõi chuyến đi.
- M05 sử dụng hành trình và loại dịch vụ để báo giá/tính cước.
- M08 ghi nhận audit và truy vết.

Nếu điểm đón, điểm đến và điểm dừng bị lưu thành một chuỗi địa chỉ tự do hoặc bị sửa trực tiếp, hệ thống sẽ khó kiểm soát thứ tự, tính cước, điều phối, lịch sử và đối soát khi khách thay đổi hành trình.

### 1.2. Mục tiêu

1. Chuẩn hóa cách biểu diễn Pickup, Destination và Intermediate Stop.
2. Hỗ trợ địa chỉ cụ thể, địa danh và vị trí đã được resolve theo BKG-003.
3. Quản lý thứ tự điểm dừng một cách rõ ràng, không phụ thuộc thứ tự hiển thị ngẫu nhiên.
4. Phân biệt điểm khách yêu cầu với điểm thực tế tài xế đã đi qua.
5. Quản lý thay đổi danh sách điểm theo Booking Revision và trạng thái Booking.
6. Cung cấp snapshot hành trình nhất quán cho M03/M04/M05.
7. Hỗ trợ các chặng đường dài/chiều về mà không gộp nhầm hai khái niệm “điểm dừng” và “chặng Trip”.
8. Bảo đảm mỗi điểm có thể được kiểm tra vị trí và Serviceability độc lập khi policy yêu cầu.

### 1.3. Không đặt mục tiêu

- Không tính tuyến đường tối ưu hoặc route distance: thuộc dịch vụ bản đồ/định tuyến.
- Không tính giá cước: M05.
- Không quyết định xe nào nhận Booking: M03.
- Không ghi nhận GPS thực tế hoặc tiến độ đến từng điểm: M04.
- Không tự động thay đổi thứ tự điểm để tối ưu đường đi nếu chưa có xác nhận khách.
- Không suy diễn hành trình nhiều điểm dừng là chính sách hiện hành chỉ từ KPMG 2015.

---

## 2. Nguồn nghiệp vụ và mức độ căn cứ

### 2.1. Nội dung được KPMG 2015 hỗ trợ

| Nguồn | Nội dung nguồn mô tả | Kết luận được phép dùng |
|---|---|---|
| TO.P2.1 | Khách nhập điểm đón bằng địa chỉ cụ thể hoặc địa danh; App có thể dò vị trí hiện tại làm mặc định và khách được thay đổi; điểm đến là tùy chọn | Có Pickup; có Destination tùy chọn; hỗ trợ address/landmark/GPS suggestion |
| TO.P2.2–P2.3 | Nếu có điểm đến, hệ thống ước tính khoảng cách hành trình và cước phí dự kiến | Destination có thể được dùng làm đầu vào tính route/fare estimate |
| TO.P2.4–P2.5 | Kiểm tra điều kiện cước đường dài và đề xuất cước cố định theo khoảng cách/khung cước | Không được hard-code ngưỡng lịch sử; M05 sở hữu chính sách cước |
| TO.P2.10 | Thông tin gửi tài xế gồm điểm đón, điểm đến (nếu có), thông tin đường dài và dữ liệu liên quan | Driver-facing trip request có thể cần các điểm đã xác nhận |
| TO.P2.20 | Hành trình chở khách được hệ thống quản lý hành trình trên xe lưu vết | Phân biệt yêu cầu hành trình với hành trình thực tế |
| TO.P7.1 | Web Booking có điểm đón; điểm đến tùy chọn; nếu nhập điểm đến thì hệ thống ước tính lộ trình/cước phí | Web có cùng mô hình Pickup + Destination tùy chọn |
| TO.P4.10–P4.12 | Khách thông báo điểm đến cho tài xế; tài xế nhập điểm đến vào ứng dụng để chỉ đường và có thể ước tính tiền | Điểm đến có thể được bổ sung trong vận hành; thời điểm và quyền cập nhật cần chính sách |
| TO.P9.3, P9.9 | Thông tin đường dài gồm địa chỉ đón/đến; chặng về có thể cập nhật địa chỉ đích, km và giá | Đường dài có dữ liệu một chiều/hai chiều và chặng về riêng |
| TO.P9.11–P9.16 | Thực hiện chặng về, hủy chặng về, tìm khách chiều về và kết thúc hành trình | Return Leg cần mô hình nghiệp vụ riêng, không chỉ là một Intermediate Stop |

### 2.2. Phần KPMG chưa đặc tả

KPMG 2015 không cung cấp đủ căn cứ để kết luận:
- Một Booking thông thường được phép có bao nhiêu điểm dừng trung gian.
- Có được thêm/xóa/sắp xếp lại điểm dừng sau khi đã điều phối hay không.
- Có được đổi thứ tự điểm sau khi Trip bắt đầu hay không.
- Có loại điểm dừng chờ, đón thêm khách, trả khách, giao nhận hay dừng kỹ thuật không.
- Thời gian chờ tối đa tại một điểm và cách tính phí chờ.
- Điểm dừng có bắt buộc phải nằm trong Service Area hay chỉ cần hợp lệ về địa chỉ.
- Có cho phép nhiều điểm đón trong cùng một Booking hay phải tạo nhiều Booking.
- Chặng về TO.P9 được mô hình hóa trong dữ liệu Trip/Booking hiện tại như thế nào.

Các nội dung này phải là DEC/TBD, không tự coi là nghiệp vụ đã được duyệt.

### 2.3. Phân biệt ba lớp dữ liệu

| Lớp | Ý nghĩa | Owner |
|---|---|---|
| Requested Stops | Các điểm khách yêu cầu, thứ tự mong muốn và chỉ dẫn | M01 |
| Confirmed Trip Plan | Kế hoạch điểm/chặng đã được xác nhận để thực hiện | M04/M01 theo ranh giới được duyệt |
| Actual Trip Track | GPS, thời điểm đến/rời, điểm thực tế và sai lệch | M04 |

Không dùng dữ liệu GPS thực tế để ghi đè Requested Stops. Không thay đổi yêu cầu gốc chỉ vì tài xế đi đường khác.

---

## 3. Phạm vi

### 3.1. Trong phạm vi

1. Một điểm đón bắt buộc cho Booking theo mô hình cơ bản.
2. Một điểm đến tùy chọn theo mô hình KPMG TO.P2/TO.P7.
3. Danh sách Intermediate Stops có thứ tự theo đề xuất mở rộng, chờ Mai Linh phê duyệt.
4. Mỗi Stop có loại, thứ tự, địa chỉ/location reference, chỉ dẫn và metadata cần thiết.
5. Resolve và kiểm tra từng điểm qua BKG-003.
6. Thêm/sửa/xóa/sắp xếp điểm theo quyền và Booking State.
7. Lưu lịch sử thay đổi trong BookingRevision.
8. Xuất bản snapshot hành trình cho các Module tiêu thụ.
9. Phân biệt Return Leg của dịch vụ đường dài với Intermediate Stop.
10. Kiểm tra dữ liệu đầu vào và các ràng buộc nhất quán.

### 3.2. Ngoài phạm vi

- Tính route geometry, khoảng cách đường bộ, ETA và chỉ đường.
- Tối ưu thứ tự điểm tự động.
- Tính cước theo từng đoạn, phí chờ hoặc phụ phí: M05.
- Thực thi tiến độ đến/rời điểm và GPS: M04.
- Quyết định Dispatch Candidate/Assignment: M03.
- Chính sách đường dài và mã dịch vụ đường dài: TO.P9/M05/M03 theo ownership được duyệt.
- Quản lý danh mục địa chỉ/địa danh và geometry nền: Location/GIS service.
- Quy trình nhiều Booking cho đoàn xe hoặc khách hàng đặt nhiều xe: Feature khác.

### 3.3. Định nghĩa Stop Type đề xuất

| Stop Type | Ý nghĩa | Căn cứ |
|---|---|---|
| PICKUP | Điểm đón khách | KPMG TO.P2/TO.P7 |
| DESTINATION | Điểm trả khách cuối hành trình | KPMG TO.P2/TO.P7 |
| VIA | Điểm dừng trung gian | Đề xuất mở rộng, chưa có đặc tả rõ trong KPMG |
| RETURN_ORIGIN | Điểm bắt đầu chặng về | Mô hình hóa từ TO.P9, cần xác nhận |
| RETURN_DESTINATION | Điểm kết thúc chặng về | Mô hình hóa từ TO.P9, cần xác nhận |

**Lưu ý:** RETURN_ORIGIN/RETURN_DESTINATION là cách biểu diễn logic đề xuất; không có nghĩa KPMG dùng các tên Stop Type này. Có thể mô hình Trip Leg riêng thay vì thêm hai loại Stop, tùy thiết kế M04/M05 được phê duyệt.

---

## 4. Actor, quyền và Data Scope

| Actor | Hành động | Permission đề xuất | Điều kiện |
|---|---|---|---|
| Khách App/Web | Nhập Pickup/Destination | `booking.stop.create.self` | Booking thuộc khách |
| Khách App/Web | Thêm/sửa/xóa VIA | `booking.stop.manage.self` | Chỉ khi policy cho phép |
| Tổng đài viên | Nhập/chỉnh địa điểm theo yêu cầu khách | `booking.stop.manage.on_behalf` | Có lý do và scope |
| Nhân viên điểm tiếp thị | Ghi nhận điểm đón | `booking.pickup.manage.marketing_point` | Theo điểm/ca |
| M01 Service | Validate, persist, revision | Internal permission | Theo Booking aggregate |
| M03 Dispatch | Đọc Pickup và snapshot cần thiết | `booking.route.read.dispatch` | Không được tự sửa Requested Stops |
| M04 Trip | Đọc Confirmed Trip Plan; ghi Actual Trip Progress | Permission theo API riêng | Không sửa Booking gốc |
| M05 Pricing | Đọc hành trình yêu cầu để quote/calculate | `booking.route.read.pricing` | Không sửa Requested Stops |
| Auditor | Đọc lịch sử thay đổi | `booking.route.audit.read` | Theo data scope |

Quy tắc:
- Client không được tự gửi `sequence_no` tùy ý để ghi đè thứ tự mà không qua validation.
- Chỉ M01 được cập nhật Requested Stops.
- M03/M04/M05 chỉ đọc snapshot hoặc gọi command theo hợp đồng; không ghi trực tiếp bảng dữ liệu M01.
- Thay đổi do tổng đài viên thực hiện phải ghi actor, reason và nguồn yêu cầu.
- Không hiển thị thông tin liên hệ/địa chỉ ngoài phạm vi cần thiết.

---

## 5. Mô hình dữ liệu và quy tắc cấu trúc

### 5.1. Mô hình logic BookingStop

Theo X-02, M01 có Entity BookingStop với các thuộc tính logic như `booking_stop_id`, `booking_id`, `sequence_no`, `stop_type`, `location_ref`, `instruction`.

BKG-004 đề xuất làm rõ thêm các thuộc tính sau; đây chưa phải DDL đã duyệt:

| Field logic | Ý nghĩa | Bắt buộc |
|---|---|---|
| booking_stop_id | Định danh duy nhất của Stop | Có |
| booking_id | Booking sở hữu Stop | Có |
| sequence_no | Thứ tự trong hành trình yêu cầu | Có |
| stop_type | PICKUP / DESTINATION / VIA hoặc loại được duyệt | Có |
| location_ref | Tham chiếu Resolved Location | Có sau khi resolve |
| raw_location_text | Nội dung khách/nhân viên nhập ban đầu | Có khi nhập text |
| normalized_address_snapshot | Địa chỉ chuẩn hóa tại thời điểm xác nhận | Có sau resolve |
| location_source | USER_ENTERED / GPS_SUGGESTED / SAVED_PLACE / AGENT_SELECTED / MAP_PIN | Có |
| instruction | Hướng dẫn đón/trả/dừng | Không |
| contact_reference | Tham chiếu thông tin liên hệ riêng nếu được phép | Không |
| requested_arrival_at | Thời gian mong muốn tại điểm, nếu loại dịch vụ hỗ trợ | TBD |
| dwell_time_request | Thời gian dừng mong muốn | TBD |
| serviceability_decision_ref | Kết quả BKG-003 | Theo policy |
| created_at / updated_at | Thời điểm tạo/cập nhật | Có |
| created_by / updated_by | Actor tạo/cập nhật | Có |

### 5.2. Quy tắc thứ tự

Đề xuất:
- `sequence_no` là thứ tự nghiệp vụ ổn định, không phụ thuộc thứ tự bản ghi trong DB.
- Pickup phải đứng đầu trong hành trình một chiều thông thường.
- Destination là điểm kết thúc của hành trình một chiều thông thường.
- VIA nằm giữa Pickup và Destination.
- Mỗi Stop có một sequence duy nhất trong cùng Booking.
- Không được có khoảng trống hoặc trùng sequence sau khi commit; cách đánh số nội bộ có thể là 1..N.
- Khi reorder, thực hiện trong một transaction để tránh trạng thái trùng sequence tạm thời bị lộ ra ngoài.
- Nếu không có Destination, không được tự suy ra một điểm kết thúc từ địa chỉ hiện tại của xe/tài xế.
- Quy tắc cho chặng về phải dùng mô hình Return Leg đã được duyệt; không nhét Return Destination vào giữa các VIA.

Đây là quy tắc thiết kế đề xuất, chờ Business Owner phê duyệt.

### 5.3. Ràng buộc số lượng

| Ràng buộc | Giá trị hiện tại |
|---|---|
| Số Pickup | Đề xuất đúng 1 trong một Booking thông thường |
| Số Destination | Đề xuất 0 hoặc 1 trong một leg |
| Số VIA | Chưa chốt |
| Tổng số Stops | Chưa chốt |
| Nhiều Pickup trong cùng Booking | Chưa chốt; không mặc định cho phép |
| Nhiều Destination | Chưa chốt; mô hình leg có thể phù hợp hơn |

Không đặt cứng giới hạn số Stop cho đến khi Mai Linh xác nhận nghiệp vụ và đánh giá hiệu năng/UX.

### 5.4. Địa điểm và BKG-003

Mỗi Stop có một Location Input và một Resolved Location reference:
1. Giữ raw input.
2. Resolve qua BKG-003.
3. Người dùng/nhân viên chọn candidate nếu mơ hồ.
4. Lưu location reference và snapshot.
5. Lưu serviceability decision reference nếu policy yêu cầu.
6. Khi địa chỉ thay đổi, tạo revision/đánh giá lại; không sửa tọa độ âm thầm.

Serviceability của Pickup, Destination và VIA là các decision riêng. Policy cho từng loại điểm có thể khác nhau; BKG-004 không tự suy diễn tất cả phải nằm trong cùng Service Area.

### 5.5. Requested Stops và Trip Leg

Không đồng nhất:
- Stop: một địa điểm trong kế hoạch hành trình.
- Leg: một chặng di chuyển giữa hai điểm/nhóm điểm.
- Trip: một phiên thực hiện dịch vụ.
- Return Leg: chặng quay về trong nghiệp vụ đường dài.

TO.P9 mô tả một chiều/hai chiều, thêm hoặc hủy chặng về, và tìm khách chiều về. Vì vậy, kiến trúc cần xác nhận Return Leg thuộc Booking, Trip hay Long-distance Service entity; BKG-004 không quyết định thay M04/M05.

---

## 6. Quy tắc nghiệp vụ (Business Rules)

| Rule ID | Quy tắc | Nguồn | Trạng thái |
|---|---|---|---|
| BKG004-R01 | Booking thông thường phải có một Pickup đã được xác định | KPMG TO.P2.1/TO.P7.1 | Draft |
| BKG004-R02 | Destination có thể không được cung cấp khi tạo Booking | KPMG TO.P2.1/TO.P7.1 | Draft |
| BKG004-R03 | Khi Destination được cung cấp, có thể dùng làm đầu vào ước tính route/fare theo quy trình | KPMG TO.P2.3/TO.P7.1 | Draft |
| BKG004-R04 | Địa chỉ có thể là địa chỉ cụ thể hoặc địa danh | KPMG TO.P2.1/TO.P7.1 | Draft |
| BKG004-R05 | GPS là vị trí gợi ý ban đầu; khách có thể thay đổi | KPMG TO.P2.1 | Draft |
| BKG004-R06 | Mỗi Stop phải có thứ tự ổn định và duy nhất trong Booking | Đề xuất thiết kế | Draft |
| BKG004-R07 | Pickup đứng đầu trong hành trình một chiều thông thường | Đề xuất thiết kế | Draft |
| BKG004-R08 | Destination là điểm kết thúc của hành trình một chiều khi có khai báo | Đề xuất thiết kế | Draft |
| BKG004-R09 | VIA chỉ được sử dụng nếu chính sách nhiều điểm dừng được Mai Linh phê duyệt | TBD | Draft |
| BKG004-R10 | Không cho phép hai Stop cùng sequence trong cùng Booking | Đề xuất dữ liệu | Draft |
| BKG004-R11 | Thay đổi Stop phải tạo BookingRevision theo X-01/X-02 | Kiến trúc hiện tại | Draft |
| BKG004-R12 | Không sửa Requested Stops trực tiếp sau khi Trip đã bắt đầu; xử lý qua Trip Change/Incident policy | Đề xuất; cần M04/Operations duyệt | Draft |
| BKG004-R13 | Thay đổi Pickup sau khi đã có Offer/Assignment phải gọi cơ chế impact/re-dispatch của M03 | Ranh giới Module | Draft |
| BKG004-R14 | Thay đổi Destination/VIA có thể ảnh hưởng báo giá; cần M05 đánh giá lại theo policy | Ranh giới Module | Draft |
| BKG004-R15 | Không tự tối ưu/reorder hành trình nếu chưa có xác nhận khách hoặc policy được duyệt | Đề xuất | Draft |
| BKG004-R16 | Điểm thực tế GPS không được ghi đè Requested Stops | M01/M04 boundary | Draft |
| BKG004-R17 | Return Leg TO.P9 không được tự biểu diễn như một VIA thông thường | KPMG TO.P9 + đề xuất mô hình | Draft |
| BKG004-R18 | Điểm đón/đến/điểm dừng cần được resolve qua BKG-003 theo policy | BKG-003 | Draft |
| BKG004-R19 | Mỗi Stop phải có source và audit khi được thêm/sửa/xóa | Đề xuất audit | Draft |
| BKG004-R20 | Chỉ M01 sở hữu Requested Stops; M03/M04/M05 không được cập nhật trực tiếp | X-02 ownership | Draft |
| BKG004-R21 | Nhiều Pickup, nhiều Destination, giới hạn VIA và loại Stop phải được Business Owner chốt | TBD | Draft |
| BKG004-R22 | Không có Destination không được hiểu là địa chỉ đích tự động hoặc cho phép kết thúc Trip tùy ý | Đề xuất an toàn dữ liệu | Draft |
| BKG004-R23 | Mọi thay đổi thứ tự phải đảm bảo sequence nhất quán sau transaction | Đề xuất kỹ thuật | Draft |
| BKG004-R24 | Snapshot hành trình gửi sang Module khác phải có version/revision tham chiếu | Đề xuất tích hợp | Draft |

---

## 7. Luồng nghiệp vụ

### 7.1. Tiền điều kiện

- Booking đang ở trạng thái cho phép nhập/sửa điểm theo X-01.
- Actor có permission trên Booking.
- Có Pickup input; Destination/VIA tùy loại dịch vụ và policy.
- BKG-003 sẵn sàng resolve/check địa điểm hoặc trả trạng thái review/unknown rõ ràng.
- Có expected_version để kiểm soát cập nhật đồng thời.

### 7.2. Luồng chính — Tạo hành trình một chiều

| Bước | Actor | Hành động | Hệ thống xử lý | Kết quả |
|---|---|---|---|---|
| 1 | Khách/nhân viên | Nhập Pickup | M01 lưu raw input | Pickup Input |
| 2 | M01 | Gọi BKG-003 | Resolve và kiểm tra theo policy | Pickup Location |
| 3 | Khách/nhân viên | Có thể nhập Destination | M01 lưu raw input | Destination Input hoặc null |
| 4 | M01 | Gọi BKG-003 cho Destination nếu có | Resolve/check riêng | Destination Location |
| 5 | Khách/nhân viên | Thêm VIA nếu tính năng được bật | Kiểm tra permission và policy | Danh sách Stop dự kiến |
| 6 | M01 | Kiểm tra cấu trúc/thứ tự | Validate type, sequence, duplicate, required fields | Stop list hợp lệ |
| 7 | M01 | Lưu BookingStop | Transaction và tạo revision | Requested Stops version mới |
| 8 | M01 | Trả hành trình | Trả danh sách có sequence và location reference | Khách/nhân viên xác nhận |

### 7.3. Luồng thay thế

| Mã | Điều kiện | Xử lý | Kết quả |
|---|---|---|---|
| ALT-01 | Không nhập Destination | Lưu Pickup và destination=null | Booking hợp lệ theo policy |
| ALT-02 | App có GPS | Đề xuất GPS làm Pickup; khách có thể sửa | Lưu source và xác nhận |
| ALT-03 | Nhập địa danh | Resolve candidate qua BKG-003 | Location reference |
| ALT-04 | Địa chỉ đã lưu | Dùng SavedPlace làm input và kiểm tra theo policy | Stop mới có snapshot hiện tại |
| ALT-05 | Thêm nhiều VIA | Kiểm tra giới hạn đã được phê duyệt và sequence | Danh sách ordered stops |
| ALT-06 | Reorder VIA | Client gửi danh sách ID và expected_version; M01 kiểm tra toàn bộ thứ tự mới | Revision mới |
| ALT-07 | Xóa VIA | Kiểm tra quyền/state; đánh số lại trong transaction | Revision mới |
| ALT-08 | Đổi Destination | Resolve lại, kiểm tra serviceability, đánh giá ảnh hưởng báo giá | Revision mới |
| ALT-09 | Đổi Pickup trước Dispatch | Resolve/check lại; M03 chưa có hoặc được yêu cầu đánh giá lại theo policy | Booking revision |
| ALT-10 | Đổi Pickup sau Offer/Assignment | Gửi change-impact command tới M03; không tự sửa như trường hợp chưa điều phối | Chờ kết quả M03 |
| ALT-11 | Đường dài hai chiều | Chuyển sang mô hình Return Leg/TO.P9 đã được phê duyệt | Không coi là VIA |
| ALT-12 | Khách thêm điểm dừng khi Trip đang chạy | Chuyển sang Trip Change flow của M04/Operations | Không cập nhật Requested Stops trực tiếp |

### 7.4. Luồng ngoại lệ và phục hồi

| Mã | Tình huống | Phản hồi | Dữ liệu/trạng thái | Phục hồi |
|---|---|---|---|---|
| EX-01 | Thiếu Pickup | PICKUP_REQUIRED | Không commit Stop list | Yêu cầu nhập |
| EX-02 | Pickup chưa resolve | LOCATION_NOT_RESOLVED | Không chuyển trạng thái đủ điều kiện | BKG-003 retry/manual |
| EX-03 | Destination mơ hồ | LOCATION_AMBIGUOUS | Chưa xác nhận Destination | Chọn candidate |
| EX-04 | VIA chưa được bật theo policy | INTERMEDIATE_STOPS_NOT_ALLOWED | Không thêm VIA | Thông báo/đề nghị luồng khác |
| EX-05 | Vượt giới hạn số Stop đã cấu hình | STOP_LIMIT_EXCEEDED | Không commit thay đổi | Điều chỉnh danh sách |
| EX-06 | Stop trùng sequence | STOP_SEQUENCE_CONFLICT | Transaction rollback | Gửi thứ tự mới |
| EX-07 | Stop ID không thuộc Booking | STOP_NOT_IN_BOOKING | Không thay đổi dữ liệu | Kiểm tra request |
| EX-08 | Reorder thiếu/mất Stop hiện có | STOP_SET_MISMATCH | Không commit | Tải lại danh sách mới nhất |
| EX-09 | Expected version cũ | STATE_VERSION_CONFLICT | Không ghi đè thay đổi mới | Reload và merge |
| EX-10 | M03 đang có Assignment hiệu lực | ROUTE_CHANGE_REQUIRES_DISPATCH_REVIEW | Không tự cập nhật Pickup | M03 đánh giá |
| EX-11 | M05 không thể tái báo giá | QUOTE_REEVALUATION_REQUIRED | Revision ở trạng thái chờ theo policy | Retry/agent review |
| EX-12 | Provider định vị lỗi | LOCATION_PROVIDER_UNAVAILABLE | Không kết luận location invalid | Retry/fallback |
| EX-13 | Actor không có quyền sửa Stop | PERMISSION_DENIED | Không thay đổi | Kiểm tra scope |
| EX-14 | Trip đã bắt đầu | TRIP_CHANGE_NOT_ALLOWED_IN_BOOKING | Requested Stops giữ nguyên | M04 xử lý Trip Change |
| EX-15 | Khách đổi thứ tự sau khi tài xế nhận chuyến | ROUTE_CHANGE_REQUIRES_RECONFIRMATION | Không tự phát lệnh mới | M03/M04 và khách xác nhận |
| EX-16 | Hai yêu cầu cập nhật Stop đồng thời | STATE_VERSION_CONFLICT | Chỉ một transaction thắng | Tải lại và gửi lại |

### 7.5. Hậu điều kiện

**Thành công**
- Stop list được lưu nguyên tử và có thứ tự nhất quán.
- BookingRevision ghi lại thay đổi.
- Mỗi Stop có location reference/snapshot và kết quả BKG-003 cần thiết.
- Các Module tiêu thụ nhận được version hành trình rõ ràng.

**Thất bại**
- Không lưu một phần danh sách Stop.
- Không làm mất thứ tự trước đó.
- Không tự thay đổi Booking State nếu command thất bại.
- Không gửi Event hành trình mới khi transaction chưa commit.

---

## 8. State, Version và Change Impact

### 8.1. Stop lifecycle đề xuất

Stop không nhất thiết cần một State Machine độc lập đầy đủ. Bản đầu có thể dùng các trạng thái dữ liệu:
- DRAFT: đang nhập, chưa xác nhận.
- VALIDATED: địa điểm đã resolve/check theo policy.
- CONFIRMED: thuộc hành trình được khách xác nhận.
- REMOVED: đã bị loại khỏi revision hiện hành nhưng còn lịch sử.

Đây là trạng thái logic đề xuất; phải đồng bộ với X-01 và tránh tạo enum dư thừa nếu chỉ cần revision/snapshot.

### 8.2. Quy tắc sửa đổi theo Booking State

| Booking State | Sửa Stop? | Điều kiện đề xuất |
|---|---|---|
| DRAFT | Có | Theo quyền |
| SUBMITTED | Có | Trước khi validation hoàn tất, tạo revision |
| VALIDATING | Có điều kiện | Kiểm soát version/concurrency |
| ACCEPTED | Có điều kiện | Theo policy; có thể cần xác nhận lại |
| SCHEDULED | Có điều kiện | Phải đánh giá ảnh hưởng lịch/giá/điều phối |
| READY_FOR_DISPATCH | Có điều kiện | Thay đổi Pickup phải đánh giá lại BKG-003/M03 |
| FULFILLING | Không qua M01 thông thường | Dùng Trip Change của M04 |
| COMPLETED | Không | Chỉ xem lịch sử |
| CANCELLED/EXPIRED/REJECTED | Không | Không sửa Booking đã kết thúc |

Ánh xạ cuối cùng phải theo X-01 và policy vận hành được phê duyệt.

### 8.3. Change Impact Matrix

| Thay đổi | BKG-003 | M03 Dispatch | M04 Trip | M05 Pricing |
|---|---|---|---|---|
| Thêm/sửa Pickup | Resolve + Serviceability | Đánh giá lại Offer/Candidate/Assignment | Cập nhật plan nếu chưa bắt đầu | Đánh giá lại quote nếu policy cần |
| Sửa Destination | Resolve + policy destination | Thông báo nếu ảnh hưởng điều kiện điều phối | Cập nhật plan nếu chưa bắt đầu | Đánh giá lại route/fare |
| Thêm/sửa VIA | Resolve + policy stop | Có thể ảnh hưởng dispatch policy/ETA | Cập nhật plan theo quy trình | Đánh giá lại quote |
| Xóa VIA | Kiểm tra lại danh sách | Có thể ảnh hưởng | Cập nhật plan | Đánh giá lại quote |
| Reorder VIA | Không nhất thiết resolve lại nếu location không đổi; kiểm tra policy nếu cần | Có thể ảnh hưởng | Cập nhật thứ tự plan | Đánh giá lại route/fare |
| Sửa Return Leg | Theo điểm của leg | Theo hợp đồng M03 | Trip leg update | Đường dài/giá phải tính lại |

Không phải mọi thay đổi đều bắt buộc gọi lại mọi Module; quy tắc chi tiết phụ thuộc policy và trạng thái hiện tại.

---

## 9. UI/UX

### 9.1. Thành phần giao diện

- Một trường Pickup bắt buộc.
- Một trường Destination tùy chọn.
- Khu vực danh sách VIA chỉ hiển thị nếu Business Owner bật tính năng.
- Mỗi điểm có ô tìm địa chỉ/địa danh, bản đồ/pin nếu kênh hỗ trợ.
- Hiển thị thứ tự rõ ràng.
- Hỗ trợ thêm/xóa/sắp xếp bằng thao tác có thể tiếp cận.
- Chỉ dẫn tại điểm: cổng vào, sảnh, tầng, vị trí đón, lưu ý trả khách.
- Hiển thị kết quả resolve và Serviceability của từng điểm theo BKG-003.
- Hiển thị cảnh báo nếu thay đổi có thể làm thay đổi báo giá hoặc yêu cầu điều phối lại.

### 9.2. Bố cục logic

1. **Điểm đón:** địa chỉ/địa danh, vị trí bản đồ, hướng dẫn đón.
2. **Điểm dừng trung gian:** danh sách có số thứ tự (nếu được phép).
3. **Điểm đến:** địa chỉ/địa danh, hướng dẫn trả khách.
4. **Thông tin hành trình:** tổng quan điểm đã nhập, cảnh báo chính sách.
5. **Xác nhận:** người dùng xác nhận hành trình trước khi gửi Booking.

### 9.3. Hành vi reorder

- Cho phép kéo/thả hoặc nút lên/xuống.
- Không cho reorder Pickup xuống sau VIA/Destination trong hành trình một chiều thông thường.
- Không cho Destination đứng giữa danh sách nếu policy không định nghĩa multi-leg.
- Reorder phải gửi toàn bộ danh sách ID theo thứ tự mới và expected_version.
- Nếu có thay đổi đồng thời, hiển thị dữ liệu mới nhất và yêu cầu xác nhận lại.
- Không tự reorder theo tuyến đường ngắn nhất.

### 9.4. Message Code đề xuất

| Code | Điều kiện | Nội dung gợi ý |
|---|---|---|
| STOP-001 | Thiếu Pickup | Vui lòng nhập điểm đón |
| STOP-002 | Destination tùy chọn | Bạn có thể bổ sung điểm đến để tham khảo lộ trình/cước phí nếu dịch vụ hỗ trợ |
| STOP-003 | Địa chỉ chưa rõ | Vui lòng chọn đúng địa điểm |
| STOP-004 | VIA chưa được bật | Dịch vụ hiện chưa hỗ trợ thêm điểm dừng theo yêu cầu này |
| STOP-005 | Thay đổi ảnh hưởng điều phối | Thay đổi điểm đón có thể cần hệ thống điều phối xác nhận lại |
| STOP-006 | Thay đổi ảnh hưởng báo giá | Hành trình thay đổi; cần cập nhật báo giá |
| STOP-007 | Xung đột cập nhật | Thông tin hành trình đã thay đổi. Vui lòng kiểm tra lại |
| STOP-008 | Trip đã bắt đầu | Vui lòng liên hệ tài xế/Tổng đài để xử lý thay đổi trong chuyến đi |

Nội dung cuối cùng cần Business Owner phê duyệt.

---

## 10. API và Event Contract

> Endpoint/payload là đề xuất, chưa phải hợp đồng production. Đồng bộ X-03 và BKG-001.

### 10.1. Add Stop

**Command:** `AddBookingStop`  
**Endpoint tham khảo:** `POST /api/v1/bookings/{booking_id}/stops`

Request minh họa:
```json
{
  "stop_type": "VIA",
  "location_input": {
    "input_text": "Địa điểm khách yêu cầu",
    "source": "USER_ENTERED"
  },
  "instruction": "Đón thêm người tại cổng chính",
  "expected_version": 3
}
```

### 10.2. Reorder Stops

**Command:** `ReorderBookingStops`  
**Endpoint tham khảo:** `PUT /api/v1/bookings/{booking_id}/stops/order`

Request minh họa:
```json
{
  "ordered_stop_ids": [
    "pickup-stop-id",
    "via-stop-id-1",
    "via-stop-id-2",
    "destination-stop-id"
  ],
  "expected_version": 4,
  "change_reason": "Khách yêu cầu thay đổi thứ tự điểm dừng"
}
```

Server phải xác thực:
- Tất cả Stop ID thuộc Booking.
- Danh sách có đủ các Stop còn hiệu lực cần sắp xếp.
- Không có ID trùng.
- Thứ tự đáp ứng rule của Stop Type.
- Booking State cho phép thay đổi.
- Version không bị stale.

### 10.3. Update/Remove Stop

Đề xuất:
- `PATCH /api/v1/bookings/{booking_id}/stops/{stop_id}`
- `DELETE /api/v1/bookings/{booking_id}/stops/{stop_id}`

Delete là command có kiểm soát, không xóa vật lý lịch sử đã được tham chiếu; phải tạo revision/audit.

### 10.4. Response minh họa

```json
{
  "booking_id": "booking-id",
  "booking_revision": 5,
  "version": 5,
  "stops": [
    {
      "stop_id": "pickup-stop-id",
      "sequence_no": 1,
      "stop_type": "PICKUP",
      "location_reference": "location-ref"
    },
    {
      "stop_id": "destination-stop-id",
      "sequence_no": 2,
      "stop_type": "DESTINATION",
      "location_reference": "location-ref-2"
    }
  ]
}
```

### 10.5. Error Codes đề xuất

| Code | Ý nghĩa |
|---|---|
| PICKUP_REQUIRED | Thiếu điểm đón |
| STOP_NOT_FOUND | Không tìm thấy Stop |
| STOP_NOT_IN_BOOKING | Stop không thuộc Booking |
| STOP_TYPE_NOT_ALLOWED | Stop Type không được phép |
| INTERMEDIATE_STOPS_NOT_ALLOWED | Chính sách chưa cho phép VIA |
| STOP_LIMIT_EXCEEDED | Vượt giới hạn Stop |
| STOP_SEQUENCE_CONFLICT | Thứ tự trùng/xung đột |
| STOP_SET_MISMATCH | Danh sách reorder thiếu/thừa Stop |
| LOCATION_NOT_RESOLVED | Địa điểm chưa resolve |
| SERVICEABILITY_REVIEW_REQUIRED | Địa điểm cần kiểm tra |
| ROUTE_CHANGE_REQUIRES_DISPATCH_REVIEW | Thay đổi cần M03 đánh giá |
| QUOTE_REEVALUATION_REQUIRED | Cần đánh giá lại báo giá |
| TRIP_CHANGE_NOT_ALLOWED_IN_BOOKING | Trip đã bắt đầu; không sửa Requested Stops |
| STATE_VERSION_CONFLICT | Version không còn mới |
| INVALID_STATE_TRANSITION | Booking State không cho phép |

### 10.6. Event đề xuất

- `BookingStopsChanged`: phát sau khi Stop list/revision commit thành công.
- `BookingPickupChanged`: có thể là event chuyên biệt nếu consumer cần.
- `BookingDestinationChanged`: có thể là event chuyên biệt nếu consumer cần.
- `BookingRouteOrderChanged`: có thể là event chuyên biệt nếu consumer cần.

Không phát cả event tổng quát và nhiều event chuyên biệt nếu không có consumer/contract rõ ràng. Event phải có:
- event_id, event_type, occurred_at.
- booking_id, booking_revision, aggregate_version.
- actor/source và change_reason theo mức cần thiết.
- danh sách Stop snapshot/reference cần cho consumer.
- correlation_id/causation_id.

Dùng Transactional Outbox theo X-03; consumer phải idempotent.

---

## 11. Bảo mật, Audit và dữ liệu cá nhân

### 11.1. Audit bắt buộc

| Audit Action | Khi nào |
|---|---|
| BOOKING_STOP_ADDED | Thêm Stop |
| BOOKING_STOP_UPDATED | Sửa địa điểm/chỉ dẫn |
| BOOKING_STOP_REMOVED | Xóa/loại Stop |
| BOOKING_STOPS_REORDERED | Thay đổi thứ tự |
| BOOKING_PICKUP_CHANGED | Thay Pickup |
| BOOKING_DESTINATION_CHANGED | Thay Destination |
| BOOKING_ROUTE_CHANGE_REVIEWED | M03/M04/M05 review change impact |
| BOOKING_ROUTE_CHANGE_REJECTED | Từ chối thay đổi |
| BOOKING_ROUTE_CHANGE_CONFIRMED | Khách/nhân viên xác nhận thay đổi |

Audit lưu actor, channel, timestamp, reason, before/after snapshot hoặc reference, BookingRevision, correlation ID. Không lưu token/bí mật.

### 11.2. Quyền riêng tư

- Không hiển thị contact reference của điểm dừng cho vai trò không cần.
- Không ghi raw location/GPS vào log kỹ thuật không được bảo vệ.
- Áp dụng masking và retention theo X-04.
- Hạn chế truy cập lịch sử địa điểm theo Booking scope.
- Dữ liệu điểm đón/đến có thể nhạy cảm theo ngữ cảnh; chỉ sử dụng đúng mục đích vận hành.

---

## 12. NFR và vận hành

| NFR | Yêu cầu |
|---|---|
| Data Integrity | Stop list phải commit nguyên tử, sequence duy nhất |
| Consistency | Snapshot hành trình có revision/version |
| Concurrency | Expected version/ETag cho mọi thay đổi |
| Idempotency | Command thêm/sửa quan trọng có idempotency theo X-03 |
| Performance | Thời gian tải danh sách Stop và reorder cần chốt trong X-05 |
| Availability | Lỗi map/provider không làm mất raw input đã nhận |
| Observability | Correlation ID xuyên M01/BKG-003/M03/M04/M05 |
| Auditability | Có thể tái hiện hành trình yêu cầu tại một revision |
| Security | RBAC, data scope, PII masking |
| Compatibility | Consumer xử lý được revision/event version mới |
| Resilience | Retry an toàn, không tạo Stop trùng |

Metrics:
- booking_stop_count distribution.
- booking_stop_validation_failure_total theo reason.
- booking_route_revision_total.
- booking_route_reorder_total.
- booking_route_change_after_dispatch_total.
- booking_route_change_after_trip_start_attempt_total.
- booking_route_conflict_total.
- route_change_dispatch_review_latency.
- quote_reevaluation_failure_total.

---

## 13. Kiểm thử và Acceptance Criteria

### 13.1. Test Cases

| Test ID | Tình huống | Kết quả mong đợi |
|---|---|---|
| BKG004-TC-001 | Tạo Booking có Pickup, không có Destination | Lưu Pickup, Destination null |
| BKG004-TC-002 | Tạo Booking có Pickup và Destination | Hai Stop đúng type/thứ tự |
| BKG004-TC-003 | Pickup là địa chỉ cụ thể | Resolve qua BKG-003 |
| BKG004-TC-004 | Pickup là địa danh | Resolve candidate; chọn nếu mơ hồ |
| BKG004-TC-005 | Pickup GPS suggestion | Lưu source; khách có thể sửa/xác nhận |
| BKG004-TC-006 | Thiếu Pickup | Từ chối commit |
| BKG004-TC-007 | Destination chưa resolve | Không xác nhận Destination |
| BKG004-TC-008 | Thêm VIA khi policy chưa bật | Từ chối INTERMEDIATE_STOPS_NOT_ALLOWED |
| BKG004-TC-009 | Thêm VIA khi policy bật | Lưu Stop và sequence đúng |
| BKG004-TC-010 | Thêm VIA vượt giới hạn cấu hình | STOP_LIMIT_EXCEEDED |
| BKG004-TC-011 | Hai Stop cùng sequence | Transaction rollback |
| BKG004-TC-012 | Reorder danh sách hợp lệ | Lưu thứ tự mới và revision |
| BKG004-TC-013 | Reorder thiếu một Stop hiện hữu | STOP_SET_MISMATCH |
| BKG004-TC-014 | Reorder Stop ID thuộc Booking khác | STOP_NOT_IN_BOOKING |
| BKG004-TC-015 | Reorder làm Destination đứng trước Pickup | Từ chối theo rule |
| BKG004-TC-016 | Xóa VIA | Revision mới; không mất audit |
| BKG004-TC-017 | Xóa Pickup | Từ chối vì Pickup bắt buộc |
| BKG004-TC-018 | Đổi Destination | Resolve lại và đánh giá impact |
| BKG004-TC-019 | Đổi Pickup trước dispatch | Resolve/check lại theo BKG-003 |
| BKG004-TC-020 | Đổi Pickup sau Assignment | Yêu cầu M03 review; không tự cập nhật |
| BKG004-TC-021 | Đổi Destination ảnh hưởng giá | Yêu cầu M05 reevaluate |
| BKG004-TC-022 | Trip đã bắt đầu, sửa Stop qua M01 | Từ chối; hướng sang M04 Trip Change |
| BKG004-TC-023 | Hai actor cập nhật cùng version | Một lệnh thành công; lệnh còn lại conflict |
| BKG004-TC-024 | Retry Add Stop với cùng idempotency key | Không tạo Stop trùng |
| BKG004-TC-025 | Provider lỗi khi thêm VIA | Giữ input theo policy; không giả resolve |
| BKG004-TC-026 | Stop list commit thất bại giữa chừng | Không lưu một phần |
| BKG004-TC-027 | Event phát sau commit | Consumer nhận revision/version đúng |
| BKG004-TC-028 | Consumer nhận Event lặp | Xử lý idempotent |
| BKG004-TC-029 | Actual GPS khác Requested Stop | Không ghi đè yêu cầu gốc |
| BKG004-TC-030 | TO.P9 hai chiều | Return Leg không bị lưu nhầm thành VIA |
| BKG004-TC-031 | Nhân viên sửa Stop không có scope | PERMISSION_DENIED |
| BKG004-TC-032 | Đọc lịch sử revision | Tái hiện đúng thứ tự và địa điểm ở revision cũ |
| BKG004-TC-033 | Điểm đến ngoài Service Area nhưng pickup hợp lệ | Decision độc lập theo BKG-003; không tự từ chối pickup |
| BKG004-TC-034 | Booking có nhiều VIA và báo giá | M05 nhận đúng thứ tự/revision |
| BKG004-TC-035 | Thay đổi danh sách trong khi M03 đang đánh giá | Version conflict/review lại; không dùng snapshot cũ như mới |

### 13.2. Acceptance Criteria

**AC-01 — Pickup bắt buộc**
- Given một Booking thông thường
- When người dùng gửi hành trình không có Pickup
- Then hệ thống từ chối commit với PICKUP_REQUIRED
- And không chuyển Booking sang trạng thái đủ điều kiện điều phối

**AC-02 — Destination tùy chọn**
- Given Booking có Pickup hợp lệ
- When người dùng không nhập Destination
- Then hệ thống lưu Destination là null
- And không tự suy ra địa chỉ đích

**AC-03 — Địa điểm được resolve**
- Given Stop có input địa chỉ/địa danh
- When BKG-003 resolve thành công
- Then Stop lưu location reference và snapshot cần thiết
- And giữ raw input/source để truy vết

**AC-04 — Ordered Stops**
- Given tính năng VIA đã được Mai Linh phê duyệt và bật
- When khách thêm/sắp xếp điểm dừng
- Then hệ thống lưu sequence duy nhất và ổn định
- And tạo BookingRevision tương ứng

**AC-05 — Reorder an toàn**
- Given danh sách Stop và expected_version hiện tại
- When client gửi reorder hợp lệ
- Then toàn bộ thứ tự được commit trong một transaction
- And không để lộ danh sách có sequence trùng/thiếu

**AC-06 — Change impact**
- Given Pickup/Destination/Stop thay đổi sau khi có Offer/Assignment
- When M01 nhận command thay đổi
- Then gọi cơ chế review của Module liên quan
- And không tự sửa dữ liệu của M03/M04/M05

**AC-07 — Không sửa Booking gốc sau khi Trip bắt đầu**
- Given Trip đã bắt đầu
- When client cố cập nhật Requested Stops qua M01
- Then từ chối theo state guard
- And hướng sang Trip Change flow được phê duyệt của M04

**AC-08 — Return Leg**
- Given Booking đường dài hai chiều theo TO.P9
- When hệ thống biểu diễn hành trình
- Then Return Leg được phân biệt với VIA
- And có reference/version riêng theo mô hình được duyệt

**AC-09 — Event/version**
- Given Stop list thay đổi thành công
- When transaction commit
- Then Event có booking_revision/aggregate_version chính xác được phát qua Outbox
- And consumer có thể xử lý lặp an toàn

### 13.3. Definition of Done riêng

- [ ] Business Owner xác nhận có/không hỗ trợ VIA và giới hạn số điểm.
- [ ] Chốt có/không hỗ trợ nhiều Pickup/multiple destinations.
- [ ] Chốt mô hình Return Leg TO.P9 với M04/M05.
- [ ] Chốt quy tắc sửa điểm theo Booking State.
- [ ] Chốt change impact tới M03/M04/M05.
- [ ] BKG-003 xác nhận contract resolve/serviceability cho từng Stop.
- [ ] X-02 xác nhận Entity/relationship/version.
- [ ] X-03 xác nhận API/Event.
- [ ] X-04/X-05 xác nhận security/NFR.
- [ ] QA hoàn thành test cases và Business UAT.
- [ ] M01/M03/M04/M05 ký xác nhận ownership.

---

## 14. Migration, cấu hình và triển khai

### 14.1. Cấu hình

| Configuration | Ý nghĩa | Owner | Trạng thái |
|---|---|---|---|
| MULTI_STOP_ENABLED | Bật/tắt VIA | Business Owner | TBD |
| MAX_INTERMEDIATE_STOPS | Số VIA tối đa | Business Owner | TBD |
| MULTI_PICKUP_ENABLED | Có cho phép nhiều Pickup | Business Owner | TBD |
| MULTI_DESTINATION_ENABLED | Có cho phép nhiều Destination trong một leg | Business Owner | TBD |
| STOP_TYPES_ENABLED | Danh sách Stop Type | Product/Operations | TBD |
| STOP_CHANGE_POLICY_BY_STATE | Quy tắc sửa theo Booking State | Operations | TBD |
| STOP_SERVICEABILITY_POLICY | Policy kiểm tra từng Stop | Operations/BKG-003 | TBD |
| RETURN_LEG_MODEL | Cách mô hình hóa chặng về | M04/M05/Operations | TBD |
| ROUTE_CHANGE_REQUOTE_POLICY | Khi nào phải báo giá lại | M05 | TBD |

Không đặt giá trị mặc định cho giới hạn số Stop khi chưa được xác nhận.

### 14.2. Triển khai

1. Chốt Business Decision trước khi bật VIA.
2. Hoàn tất schema/contract và migration tương thích.
3. Bật tính năng qua feature flag theo kênh/địa bàn nếu được phê duyệt.
4. Kiểm thử dữ liệu thứ tự, concurrency, revision, Event và consumer.
5. Chạy thử với nhóm nội bộ/địa bàn pilot.
6. Theo dõi lỗi resolve, reorder, change impact và quote reevaluation.
7. Có rollback feature flag; không xóa lịch sử Stop/Revision.

---

## 15. Rủi ro, quyết định và TBD

| ID | Nội dung cần quyết định | Ảnh hưởng | Owner đề xuất |
|---|---|---|---|
| BKG004-DEC-01 | Có cho phép nhiều điểm dừng trung gian trong Booking Taxi thông thường không? | Phạm vi sản phẩm | COO/Operations |
| BKG004-DEC-02 | Số VIA tối đa là bao nhiêu? | UX/dispatch/pricing | Operations |
| BKG004-DEC-03 | Có cho phép nhiều Pickup trong một Booking không? | Dispatch/Trip | Operations |
| BKG004-DEC-04 | Có cho phép nhiều Destination trong một Booking không? | Trip/Pricing | Operations |
| BKG004-DEC-05 | Loại VIA nào được hỗ trợ: đón thêm khách, trả khách, chờ, giao nhận hay loại khác? | Product/finance | Operations |
| BKG004-DEC-06 | Có tính phí dừng/chờ không và M05 tính theo quy tắc nào? | Pricing/finance | M05/Finance |
| BKG004-DEC-07 | Thời gian chờ tại mỗi điểm có được nhập không? | UX/Trip | Operations |
| BKG004-DEC-08 | Có cho khách reorder điểm sau khi đã gửi Booking không? | Booking/Dispatch | Operations |
| BKG004-DEC-09 | Sau khi có Offer, ai được phép thay Pickup/Destination/VIA? | Dispatch control | M03/Operations |
| BKG004-DEC-10 | Sau khi Assignment đã được tài xế chấp nhận, thay đổi cần khách xác nhận lại thế nào? | Driver/Customer | M03/M04 |
| BKG004-DEC-11 | Sau khi Trip bắt đầu, thêm điểm dừng qua App hay chỉ qua tài xế/Tổng đài? | Trip operations | M04/Operations |
| BKG004-DEC-12 | Return Leg TO.P9 thuộc Booking, Trip hay Long-distance Service entity? | Data architecture | M04/M05 |
| BKG004-DEC-13 | Đường dài hai chiều có cấu trúc điểm/chặng cụ thể ra sao? | Trip/Fare | Operations/M05 |
| BKG004-DEC-14 | Có cho phép điểm đến ngoài Service Area không? | Service policy | Operations/BKG-003 |
| BKG004-DEC-15 | VIA có bắt buộc nằm trong Service Area không? | Service policy | Operations/BKG-003 |
| BKG004-DEC-16 | Địa chỉ/địa danh nào được ưu tiên khi GPS và text khác nhau? | Location UX | Product |
| BKG004-DEC-17 | Có cho phép tài xế tự chỉnh điểm đến khi khách lên xe như TO.P4.11 không? | Booking vs Trip | M04/Operations |
| BKG004-DEC-18 | Nếu tài xế đề xuất đổi tuyến/điểm, khách xác nhận ở đâu và lưu thế nào? | Trip audit | M04 |
| BKG004-DEC-19 | Thay đổi điểm nào bắt buộc M05 báo giá lại? | Fare/quote | M05 |
| BKG004-DEC-20 | Thay đổi điểm nào buộc M03 phát lại Offer hoặc Assignment? | Dispatch | M03 |
| BKG004-DEC-21 | Có cần lưu thời gian mong muốn đến từng điểm không? | Scheduled/Trip | Operations |
| BKG004-DEC-22 | Có cần lưu contact riêng cho từng điểm không? | Privacy/UX | Operations/ATTT |
| BKG004-DEC-23 | Có cần hỗ trợ điểm dừng lặp lại/cùng tọa độ không? | Data/route | Operations |
| BKG004-DEC-24 | Có cho phép Booking nhiều xe dùng chung một danh sách điểm hay mỗi xe có route riêng? | Multi-vehicle | Operations |
| BKG004-DEC-25 | Thứ tự do khách chọn có được hệ thống gợi ý tối ưu nhưng không tự đổi không? | UX/routing | Product |
| BKG004-DEC-26 | Quy tắc lưu snapshot địa chỉ và retention | Privacy/audit | Data Owner/ATTT |

---

## 16. Tiêu chí nghiệm thu Feature

- [ ] Pickup bắt buộc và Destination tùy chọn được Business Owner xác nhận.
- [ ] VIA và giới hạn số lượng được phê duyệt rõ ràng trước khi bật.
- [ ] Return Leg TO.P9 có mô hình riêng được M04/M05 xác nhận.
- [ ] Mỗi Stop có ID, type, sequence và location reference.
- [ ] Resolve/serviceability dùng chung BKG-003.
- [ ] Requested Stops tách biệt Actual Trip Track.
- [ ] Thay đổi Stop tạo BookingRevision và audit.
- [ ] Có guard theo Booking State và change impact tới M03/M04/M05.
- [ ] Reorder có transaction, concurrency control và idempotency.
- [ ] API/Event contract được duyệt theo X-03.
- [ ] PII/retention được duyệt theo X-04.
- [ ] NFR/monitoring/runbook được duyệt theo X-05.
- [ ] QA hoàn tất test evidence; Business UAT ký.
- [ ] M01/M03/M04/M05 ký xác nhận ranh giới trách nhiệm.

### 16.1. Kết luận hiện tại

**Trạng thái:** DRAFT — Chưa đạt READY/DoR.

**Blocker chính:**
1. Chưa xác nhận chính sách nhiều điểm dừng trung gian.
2. Chưa xác nhận giới hạn số điểm và các loại Stop.
3. Chưa chốt cách biểu diễn Return Leg TO.P9.
4. Chưa chốt chính sách thay đổi hành trình sau Dispatch/Assignment/Trip Start.
5. Chưa chốt ảnh hưởng thay đổi hành trình đến báo giá và điều phối.

**Không bật tính năng VIA hoặc cho phép thay đổi hành trình sau khi tài xế nhận chuyến trước khi các quyết định liên quan được phê duyệt.**

---

**Quay lại:** [Feature Specifications](../README.md) · [M01 — Customer & Booking](../../../02-DOMAIN-MODULES/01-customer-and-booking/README.md)
