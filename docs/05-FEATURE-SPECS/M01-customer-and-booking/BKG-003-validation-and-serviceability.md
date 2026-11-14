# BKG-003 — KIỂM TRA HỢP LỆ VÀ KHẢ NĂNG PHỤC VỤ

**Mã tài liệu:** TAXI-BKG-003  
**Module sở hữu:** M01 — Khách hàng và Đặt xe  
**Epic:** Tiếp nhận và quản lý Booking đa kênh  
**Phiên bản:** 0.1  
**Trạng thái:** DRAFT — CHỜ NGHIỆP VỤ, M03 VÀ KIẾN TRÚC REVIEW  
**Priority theo Feature Catalog:** P0  
**Business Owner:** TBD  
**Technical Owner:** M01 Owner — TBD  
**Tài liệu liên quan:** [M01 Feature Catalog](../../../02-DOMAIN-MODULES/01-customer-and-booking/README.md) · [BKG-001 — Tạo Booking tức thời](./BKG-001-create-immediate-booking.md) · [X-01 — State Machine](../../../03-CROSS-CUTTING/01-state-machines-and-transition-rules.md) · [X-02 — Data Architecture](../../../03-CROSS-CUTTING/02-data-architecture-and-erd.md) · [X-03 — API/Event](../../../03-CROSS-CUTTING/03-api-and-event-contracts.md) · [X-04 — Security/IAM/Audit](../../../03-CROSS-CUTTING/04-security-iam-and-audit.md) · [X-05 — NFR](../../../03-CROSS-CUTTING/05-non-functional-requirements.md) · [X-06 — Delivery Governance](../../../03-CROSS-CUTTING/06-delivery-governance.md)

> **Quy ước nguồn:** KPMG 2015 mô tả một số thao tác về địa chỉ, điểm đón, khu vực sắp tài và vùng phát yêu cầu xe. Tài liệu không định nghĩa đầy đủ một chính sách “vùng phục vụ khách” thống nhất cho mọi kênh. Vì vậy, các quy tắc serviceability trong tài liệu này là thiết kế đề xuất cần Mai Linh phê duyệt, không được xem là chính sách đang áp dụng.

---

## 0. Quản lý phiên bản và phê duyệt

| Vai trò | Người duyệt | Trạng thái |
|---|---|---|
| Business Owner — Điều hành/Vận hành | TBD | Pending |
| Product Owner M01 | TBD | Pending |
| Technical Owner M01 | TBD | Pending |
| Chủ Module M03 — Điều phối | TBD | Pending |
| Chủ dữ liệu bản đồ/địa lý | TBD | Pending |
| QA Lead | TBD | Pending |
| Security/Data Owner (nếu cần) | TBD | Pending |

| Phiên bản | Nội dung |
|---|---|
| 0.1 | Đặc tả bản đầu, tách validation địa chỉ, serviceability, staging area và dispatch coverage |

---

## 1. Bối cảnh và mục tiêu

### 1.1. Vấn đề cần giải quyết

Booking chỉ có thể được tiếp nhận đúng khi hệ thống hiểu được địa điểm khách yêu cầu và xác định được kết quả kiểm tra theo chính sách. Một chuỗi địa chỉ nhập tự do chưa đủ để kết luận vị trí chính xác; một tọa độ GPS cũng chưa chắc là điểm đón khách đã được xác nhận.

Ngoài ra, hệ thống Taxi có nhiều khái niệm địa lý phục vụ các mục đích khác nhau:
- Địa điểm khách yêu cầu đón/đến.
- Khu vực mà Mai Linh có chính sách cung cấp dịch vụ.
- Khu vực sắp tài/điểm tập kết xe.
- Vùng địa lý dùng để tìm và phát yêu cầu cho tài xế (Unicast/Broadcast).
- Khu vực địa lý phục vụ báo cáo, tổ chức hoặc vận hành.

Nếu dùng chung một polygon hoặc một cờ “trong vùng” cho tất cả các mục đích trên, hệ thống có thể từ chối khách sai, điều phối sai hoặc đưa xe đến khu vực không phù hợp.

### 1.2. Mục tiêu

BKG-003 cung cấp năng lực kiểm tra có thể tái sử dụng cho các Feature M01:
1. Kiểm tra dữ liệu địa điểm có đủ để định vị hay không.
2. Chuẩn hóa địa chỉ/địa danh thành một địa điểm có thể tham chiếu.
3. Xác định mức độ tin cậy và tình trạng mơ hồ của kết quả định vị.
4. Kiểm tra điều kiện phục vụ dựa trên bộ chính sách Service Area được Mai Linh phê duyệt.
5. Trả kết quả và lý do rõ ràng cho BKG-001/BKG-002 và các kênh.
6. Lưu phiên bản chính sách, nguồn dữ liệu và kết quả kiểm tra để truy vết.
7. Tách hoàn toàn Service Area khỏi Staging Area và Dispatch Coverage.

### 1.3. Không đặt mục tiêu

- Không tìm xe, xếp hạng xe hoặc phát Unicast/Broadcast.
- Không xác nhận có xe thực tế tại khu vực.
- Không cam kết thời gian xe đến (ETA) hoặc khả năng hoàn thành chuyến.
- Không tính giá cước.
- Không tự quyết định chính sách kinh doanh khi dữ liệu địa lý thiếu hoặc mâu thuẫn.
- Không tự sửa địa chỉ khách nhập mà không lưu dấu vết.

---

## 2. Nguồn nghiệp vụ và giới hạn diễn giải

### 2.1. Căn cứ KPMG 2015

| Mã nguồn | Nội dung thực tế trong tài liệu | Cách sử dụng trong BKG-003 |
|---|---|---|
| TO.P1.7 | Tổng đài nhập số nhà, đường, phường, quận; hệ thống có thể gợi ý địa chỉ theo số điện thoại bàn hoặc ba địa chỉ yêu cầu gần nhất của số di động | Căn cứ cho chuẩn hóa/tra cứu địa chỉ và SavedPlace; không đủ để xác định chính sách vùng phục vụ |
| TO.P2.1 | Điểm đón có thể là địa chỉ cụ thể hoặc địa danh; ứng dụng dò vị trí thiết bị làm điểm đón mặc định và khách có thể thay đổi | Căn cứ cho GPS suggestion, địa chỉ/địa danh và xác nhận điểm đón |
| TO.P7.1 | Điểm đón có thể là địa chỉ cụ thể hoặc địa danh cụ thể | Căn cứ hỗ trợ nhiều dạng dữ liệu đầu vào |
| TO.P4 — phần mô tả quy trình | Khu vực sắp tài do Mai Linh định nghĩa trên hệ thống | Đây là Staging Area, không phải mặc nhiên Service Area |
| TO.P4.2 | Kiểm tra điểm đón có nằm trong khu vực sắp tài hay không để đi vào nhánh quy trình TO.P4 | Kiểm tra loại khu vực sắp tài trong ngữ cảnh quy trình TO.P4; không áp dụng thành điều kiện từ chối Booking toàn hệ thống |
| TO.P4.5 | Tài xế trong bán kính [100]m mới được đăng ký sắp tài; tham số có thể điều chỉnh | Quy tắc đăng ký Queue/Staging Area lịch sử; không phải bán kính phục vụ khách |
| TO.P1.12 và TO.P3/P4 | Unicast phát yêu cầu tới xe trống trong bán kính [xxx] km; Broadcast phát rộng trong điều kiện đặc biệt, tham số có thể điều chỉnh | Đây là Dispatch Coverage/Dispatch Policy; không phải ranh giới Service Area |
| TO.B.1 | Unicast mặc định theo khu vực xác định (ví dụ quận 1, quận 3); Broadcast theo phạm vi toàn hệ thống và được Tổng đài kiểm soát | Căn cứ cho vùng phát yêu cầu M03; tham số và chính sách hiện hành phải xác nhận |

### 2.2. Ba khái niệm không được nhập làm một

| Khái niệm | Câu hỏi nghiệp vụ | Module/Owner đề xuất | Kết quả |
|---|---|---|---|
| Service Area — Khu vực phục vụ khách | Mai Linh có tiếp nhận cung cấp dịch vụ tại điểm đón này không? | Policy Owner nghiệp vụ; M01 thực thi kiểm tra | SERVICABLE / NOT_SERVICABLE / REVIEW_REQUIRED |
| Staging Area — Khu vực sắp tài | Điểm đón có thuộc khu vực sắp tài có quy trình hàng đợi riêng không? | M03 — Dispatch & Queue | STAGING_AREA_MATCH / NO_MATCH |
| Dispatch Coverage — Vùng phát xe | Có thể tìm/phát yêu cầu cho xe trong phạm vi địa lý nào? | M03 — Dispatch Policy | Candidate search scope / Unicast / Broadcast |

**Quy tắc kiến trúc:** Một địa điểm có thể đồng thời nằm trong một Service Area, một Staging Area và một Dispatch Coverage; ba kết quả được tính độc lập theo mục đích, phiên bản và thời điểm hiệu lực riêng.

### 2.3. Điều chưa có trong nguồn

KPMG 2015 không cung cấp đủ căn cứ để kết luận:
- Danh sách Service Area hiện hành của Mai Linh.
- Cách xử lý khách ngoài Service Area.
- Ranh giới theo polygon, quận/huyện, bán kính hay danh sách địa chỉ.
- Có vùng phục vụ khác nhau theo loại xe, thời gian, kênh hoặc DIRECT/PARTNER hay không.
- Quy tắc phục vụ chuyến liên tỉnh, đường dài, chiều về hoặc nhiều điểm dừng.
- Tần suất cập nhật bản đồ, địa giới hành chính và vùng chính sách.
- SLA/độ chính xác định vị tối thiểu.

Các nội dung này được quản lý trong mục DEC/TBD, không tự điền bằng giả định.

---

## 3. Phạm vi

### 3.1. Trong phạm vi

1. Nhận đầu vào địa điểm từ M01/kênh.
2. Kiểm tra trường địa chỉ và dữ liệu tọa độ.
3. Chuẩn hóa địa chỉ/địa danh qua Location Provider hoặc dữ liệu nội bộ.
4. Quản lý nhiều kết quả định vị và mức độ mơ hồ.
5. Xác định vị trí theo nguồn: người dùng nhập, GPS gợi ý, SavedPlace, nhân viên chọn hoặc geocoding.
6. Kiểm tra điểm đón theo Service Area Policy đã được duyệt.
7. Trả kết quả serviceability và reason code.
8. Ghi lại policy version, location source, kết quả và thời điểm kiểm tra.
9. Hỗ trợ kiểm tra lại khi địa chỉ hoặc policy thay đổi.
10. Cung cấp hợp đồng kiểm tra cho BKG-001/BKG-002 và các Feature khác thuộc M01.

### 3.2. Ngoài phạm vi

- Tạo/sửa Booking: BKG-001/BKG-002/BKG-005.
- Quản lý cấu trúc điểm đón, điểm đến và danh sách nhiều điểm dừng: BKG-004.
- Phát hiện Booking trùng: BKG-007.
- Tính giá: BKG-009/M05.
- Xác định xe trống, xe đủ điều kiện, route distance và candidate ranking: M03.
- Quản lý Queue/Staging Area và thứ tự FIFO: M03.
- Quản lý danh mục polygon/địa giới địa lý cấp nền tảng nếu được giao cho một GIS/Platform service riêng: TBD.
- Chính sách chấp nhận Booking theo hợp đồng DIRECT/PARTNER nếu chưa được cung cấp qua policy service có thẩm quyền: TBD.

### 3.3. Đối tượng địa điểm

| Đối tượng | BKG-003 kiểm tra? | Ghi chú |
|---|---|---|
| Điểm đón Booking | Có | Đối tượng chính; phải được resolve và kiểm tra policy |
| Điểm đến | Có thể kiểm tra vị trí; serviceability không mặc nhiên bắt buộc | Điểm đến có thể ngoài vùng phục vụ chiều đón; cần policy riêng |
| Điểm dừng trung gian | Theo yêu cầu BKG-004 | BKG-003 cung cấp serviceability check tái sử dụng |
| Điểm đón tại Marketing Point | Có | Không được nhầm với việc điểm đó thuộc Staging Area |
| Điểm đón tại Staging Area | Có thể kiểm tra Service Area độc lập | Sau đó M03 kiểm tra Staging Area |
| Điểm đón do GPS gợi ý | Có | Phải phân biệt GPS suggestion với vị trí khách xác nhận |

---

## 4. Actor, quyền và Data Scope

| Actor | Hành động | Permission đề xuất | Scope |
|---|---|---|---|
| Khách App/Web | Gửi địa chỉ để kiểm tra | `location.validate.self` | Địa điểm của yêu cầu hiện tại |
| Tổng đài viên | Kiểm tra/chọn địa chỉ thay khách | `location.validate.on_behalf` | Đơn vị/chi nhánh được cấp |
| Nhân viên điểm tiếp thị | Kiểm tra điểm đón tại điểm | `location.validate.marketing_point` | Điểm/ca được phân công |
| BKG-001/BKG-002 | Gọi serviceability nội bộ | `serviceability.evaluate.internal` | Context của Booking |
| M03 | Query Staging/Dispatch Coverage | Permission nội bộ riêng của M03 | Dispatch policy scope |
| Policy Administrator | Quản lý Service Area Policy | `service_area.policy.manage` | Đơn vị/địa bàn được ủy quyền |
| Auditor | Xem lịch sử thay đổi policy | `service_area.audit.read` | Scope kiểm toán |
| Location Adapter | Gọi provider định vị | Service credential | Không được tự quyết định nghiệp vụ |

Nguyên tắc:
- Không cho khách hàng hoặc client tự truyền `service_area_id`, `policy_version` để ép kết quả.
- Không cho M01 cập nhật trực tiếp Staging Area/Dispatch Coverage của M03.
- Quyền sửa policy tách khỏi quyền sử dụng policy.
- Mọi thao tác override kết quả serviceability phải có quyền, lý do và audit.
- Data Scope của người dùng nội bộ phải được kiểm tra trước khi hiển thị địa chỉ/PII liên quan.

---

## 5. Mô hình dữ liệu và thuật ngữ

### 5.1. Các khái niệm logic

| Khái niệm | Mô tả | Ownership đề xuất |
|---|---|---|
| Location Input | Dữ liệu địa điểm ban đầu do khách/nhân viên cung cấp | M01 lưu trong ngữ cảnh Booking |
| Resolved Location | Kết quả chuẩn hóa địa chỉ/địa danh và tọa độ | Location Service/Platform; M01 giữ reference/snapshot |
| Location Candidate | Một kết quả gợi ý trong nhiều kết quả định vị | Location Service |
| Service Area | Vùng địa lý có policy cung cấp dịch vụ | Business Policy Owner; technical owner TBD |
| Service Area Policy | Điều kiện, phiên bản, hiệu lực và phạm vi áp dụng của Service Area | Policy owner được phân quyền |
| Serviceability Decision | Kết quả kiểm tra một địa điểm theo policy tại thời điểm cụ thể | M01 hoặc Serviceability component theo kiến trúc được duyệt |
| Staging Area | Vùng sắp tài có quy trình Queue | M03 |
| Dispatch Coverage | Vùng tìm/phát yêu cầu xe | M03 |
| Location Validation Audit | Lịch sử resolve/check/override | M08 Audit, domain data owner TBD |

### 5.2. Dữ liệu địa điểm logic

| Field logic | Ý nghĩa | Bắt buộc |
|---|---|---|
| location_input_text | Chuỗi địa chỉ/địa danh gốc | Có |
| location_source | USER_ENTERED / GPS_SUGGESTED / SAVED_PLACE / AGENT_SELECTED / PROVIDER_RESOLVED | Có |
| provider_reference | Mã tham chiếu provider nếu có | Không |
| normalized_address | Địa chỉ chuẩn hóa | Có nếu resolve thành công |
| latitude / longitude | Tọa độ đã resolve | Có nếu kết quả định vị dùng tọa độ |
| confidence / precision | Mức tin cậy hoặc loại độ chính xác do provider trả | Theo provider |
| candidate_count | Số kết quả ứng viên | Có trong trường hợp nhiều kết quả |
| selected_candidate | Ứng viên được người dùng/nhân viên chọn | Khi cần lựa chọn |
| administrative_area | Tỉnh/thành, quận/huyện, phường/xã theo dữ liệu địa lý | Theo mức dữ liệu có |
| resolved_at | Thời điểm resolve | Có |
| provider_name/version | Nhà cung cấp và phiên bản dữ liệu nếu có | Có khi dùng provider |

Không mặc định mọi provider có cùng định nghĩa confidence score. Phải có adapter chuẩn hóa hoặc dùng precision category đã thống nhất.

### 5.3. Service Area Policy — mô hình logic đề xuất

| Field logic | Ý nghĩa |
|---|---|
| service_area_id | Định danh vùng |
| service_area_code/name | Mã và tên vùng |
| geometry_reference | Tham chiếu polygon/geometry hoặc mô hình ranh giới được duyệt |
| policy_version | Phiên bản chính sách |
| effective_from / effective_to | Thời gian hiệu lực |
| status | DRAFT / ACTIVE / RETIRED (đề xuất) |
| applicable_service_types | Loại dịch vụ áp dụng, nếu policy có phân biệt |
| applicable_vehicle_types | Loại xe áp dụng, nếu policy có phân biệt |
| applicable_fleet_types | DIRECT/PARTNER, nếu policy có phân biệt và được phê duyệt |
| applicable_channels | Kênh áp dụng, nếu policy có phân biệt và được phê duyệt |
| decision_mode | ALLOW / DENY / REVIEW hoặc mô hình được duyệt |
| priority | Thứ tự áp dụng khi có nhiều policy chồng lấn |
| created_by/approved_by | Người lập và phê duyệt |
| change_reason | Lý do thay đổi |

**Lưu ý:** Đây là logical model đề xuất. Chưa quyết định lưu polygon trong M01, GIS service, PostGIS hay một nhà cung cấp bản đồ cụ thể.

### 5.4. Kết quả Serviceability Decision

| Field logic | Ý nghĩa |
|---|---|
| decision_id | ID kết quả |
| subject_location_ref | Địa điểm được kiểm tra |
| subject_type | PICKUP / DESTINATION / INTERMEDIATE_STOP |
| decision | SERVICABLE / NOT_SERVICABLE / REVIEW_REQUIRED / UNKNOWN |
| reason_code | Mã lý do |
| service_area_id | Vùng policy khớp, nếu có |
| policy_version | Phiên bản policy đã áp dụng |
| evaluated_at | Thời điểm kiểm tra |
| evaluator_version | Phiên bản logic đánh giá |
| override_reference | Tham chiếu override nếu có |
| correlation_id | Truy vết xuyên hệ thống |

### 5.5. Ownership và ranh giới dữ liệu

- M01 sở hữu Booking, BookingStop và các reference/snapshot cần thiết để tái hiện điểm khách yêu cầu.
- M03 sở hữu Staging Area, Queue và Dispatch Coverage/Dispatch Policy theo X-02.
- Dữ liệu nền địa lý và Service Area Policy cần một owner được Mai Linh chỉ định; không mặc định M01 là chủ sở hữu polygon.
- Không copy toàn bộ geometry vào mỗi Booking; chỉ lưu reference, version và snapshot tối thiểu cần truy vết.
- Serviceability Decision không thay thế Booking State và cũng không phải DispatchRun State.

---

## 6. Quy tắc nghiệp vụ (Business Rules)

| Rule ID | Quy tắc | Nguồn | Trạng thái |
|---|---|---|---|
| BKG003-R01 | Địa điểm đầu vào phải được kiểm tra trước khi dùng làm căn cứ tạo Booking đủ điều kiện điều phối | Đề xuất kiến trúc | Draft |
| BKG003-R02 | Giữ nguyên input text; không ghi đè âm thầm bằng normalized address | Đề xuất dữ liệu | Draft |
| BKG003-R03 | GPS của thiết bị là nguồn gợi ý, không mặc nhiên là pickup khách đã xác nhận | TO.P2.1 cho phép khách đổi vị trí; diễn giải thiết kế | Draft |
| BKG003-R04 | Nếu có nhiều kết quả định vị và không thể phân biệt chắc chắn, yêu cầu người dùng/nhân viên chọn | Đề xuất UX | Draft |
| BKG003-R05 | Không tự tạo tọa độ từ địa chỉ mơ hồ hoặc không có kết quả tin cậy | Đề xuất an toàn dữ liệu | Draft |
| BKG003-R06 | Serviceability chỉ được đánh giá theo Service Area Policy ACTIVE và có hiệu lực tại thời điểm đánh giá | Đề xuất policy governance | Draft |
| BKG003-R07 | Nếu không có policy áp dụng hoặc policy chưa rõ, kết quả phải là UNKNOWN/REVIEW_REQUIRED, không tự kết luận ALLOW hoặc DENY | Đề xuất | Draft |
| BKG003-R08 | Service Area không đồng nhất với Staging Area | KPMG TO.P4 + kiến trúc phân ranh | Draft |
| BKG003-R09 | Service Area không đồng nhất với Dispatch Coverage/Unicast radius | KPMG TO.P1.12, TO.P3/P4, TO.B.1 | Draft |
| BKG003-R10 | Vị trí nằm trong Service Area không có nghĩa là có xe trống hoặc chắc chắn điều phối được xe | Ranh giới M01/M03 | Draft |
| BKG003-R11 | Điểm đến tùy chọn không được dùng để từ chối Booking chỉ vì chưa có điểm đến, trừ khi loại dịch vụ yêu cầu | TO.P2.1/TO.P7.1 | Draft |
| BKG003-R12 | Nếu policy có điều kiện theo loại xe/dịch vụ/fleet/channel, chỉ áp dụng khi các chiều điều kiện này được Business Owner phê duyệt và có dữ liệu tin cậy | TBD | Draft |
| BKG003-R13 | Mỗi kết quả phải lưu policy version và thời điểm đánh giá | Đề xuất audit | Draft |
| BKG003-R14 | Thay đổi Service Area Policy không được âm thầm sửa lại kết quả đã dùng cho Booking cũ; cần đánh giá lại có kiểm soát | Đề xuất versioning | Draft |
| BKG003-R15 | Override chỉ dành cho vai trò được cấp quyền, phải có lý do và audit | X-04/đề xuất | Draft |
| BKG003-R16 | BKG-003 không phát Unicast/Broadcast và không quyết định thứ tự hàng đợi | M03 ownership | Draft |
| BKG003-R17 | Kết quả địa chỉ/Serviceability của điểm dừng phải tham chiếu đúng stop và thứ tự do BKG-004 quản lý | Feature boundary | Draft |
| BKG003-R18 | Mọi lỗi provider phải phân biệt lỗi kỹ thuật với kết quả nghiệp vụ NOT_SERVICABLE | X-03 | Draft |

---

## 7. Luồng nghiệp vụ

### 7.1. Tiền điều kiện

- Có location input từ BKG-001/BKG-002 hoặc Feature gọi.
- Actor/service có quyền thực hiện validation.
- Có cấu hình Location Provider hoặc nguồn dữ liệu địa lý nội bộ.
- Service Area Policy đã được nạp và có trạng thái/hiệu lực rõ ràng; nếu chưa có, hệ thống phải trả UNKNOWN/REVIEW_REQUIRED.
- Có correlation ID để truy vết.

### 7.2. Luồng chính

| Bước | Actor | Hành động | Hệ thống xử lý | Kết quả |
|---|---|---|---|---|
| 1 | BKG-001/BKG-002 | Gửi location input | M01 gọi Location Validation | Tạo context kiểm tra |
| 2 | BKG-003 | Kiểm tra cấu trúc đầu vào | Kiểm tra text, tọa độ, source và format | Input hợp lệ về cấu trúc |
| 3 | BKG-003/Location Adapter | Resolve địa chỉ/địa danh | Gọi provider hoặc dữ liệu nội bộ; lưu provider reference | Một hoặc nhiều candidate |
| 4 | BKG-003 | Đánh giá độ rõ ràng | Kiểm tra số candidate, precision, dữ liệu thiếu | RESOLVED / AMBIGUOUS / UNRESOLVED |
| 5 | Khách/nhân viên | Chọn candidate nếu cần | Gửi candidate đã chọn | Một resolved location được xác nhận |
| 6 | BKG-003 | Đọc Service Area Policy | Chọn policy ACTIVE theo thời điểm và điều kiện được duyệt | Policy context/version |
| 7 | BKG-003 | Đánh giá vị trí | Spatial/policy evaluation | SERVICABLE / NOT_SERVICABLE / REVIEW_REQUIRED / UNKNOWN |
| 8 | BKG-003 | Ghi decision | Lưu decision, reason, policy version, location reference | Có kết quả truy vết |
| 9 | BKG-003 → M01 | Trả kết quả | Trả normalized location và decision | M01 xử lý bước tiếp theo |
| 10 | M01 | Áp dụng kết quả | Chỉ cho phép chuyển tiếp theo guard của Booking State | Booking không bị đánh dấu READY sai |

### 7.3. Luồng thay thế

| Mã | Điều kiện | Xử lý | Kết quả |
|---|---|---|---|
| ALT-01 | App có GPS và người dùng cho phép | Dùng GPS làm candidate mặc định; người dùng có thể sửa/chọn | Lưu source GPS và thao tác xác nhận |
| ALT-02 | Khách nhập địa danh | Tìm candidate theo tên địa danh; nếu nhiều kết quả thì hiển thị lựa chọn | Resolved Location |
| ALT-03 | Tổng đài viên tìm địa chỉ | Cho phép tìm kiếm và chọn kết quả trên bản đồ/danh sách | Lưu actor và selected candidate |
| ALT-04 | Địa chỉ đã lưu | Resolve từ SavedPlace reference và kiểm tra lại nếu cần | Không mặc định SavedPlace còn chính xác tuyệt đối |
| ALT-05 | Địa điểm thuộc nhiều Service Area | Áp dụng priority/precedence đã được phê duyệt | Một decision có trace policy |
| ALT-06 | Pickup thuộc Service Area nhưng không thuộc Staging Area | BKG-003 trả serviceable; M03 tự đánh giá Staging/Dispatch | Không từ chối Booking vì thiếu Staging Area |
| ALT-07 | Pickup thuộc Service Area nhưng không có xe trống | BKG-003 vẫn trả serviceable; M03 xử lý no candidate | Không trộn serviceability với supply |
| ALT-08 | Điểm đến nằm ngoài Service Area | Trả kết quả destination riêng; áp dụng policy điểm đến nếu có | Không tự động từ chối pickup |
| ALT-09 | Có nhiều điểm dừng | BKG-004 gọi BKG-003 cho từng stop theo policy | Kết quả gắn đúng stop |

### 7.4. Luồng ngoại lệ và phục hồi

| Mã | Tình huống | Phản hồi | Kết quả dữ liệu | Phục hồi |
|---|---|---|---|---|
| EX-01 | Input rỗng/sai format | LOCATION_INPUT_INVALID | Không có decision serviceability | Yêu cầu nhập lại |
| EX-02 | Không tìm thấy địa chỉ | LOCATION_NOT_FOUND | Không tạo resolved location | Nhập địa chỉ/landmark khác hoặc chọn trên bản đồ |
| EX-03 | Nhiều candidate không phân biệt | LOCATION_AMBIGUOUS | Decision REVIEW_REQUIRED | Người dùng/nhân viên chọn |
| EX-04 | Tọa độ ngoài phạm vi dữ liệu địa lý hỗ trợ | LOCATION_OUT_OF_DATA_COVERAGE | UNKNOWN | Kiểm tra thủ công hoặc provider khác |
| EX-05 | Provider timeout/unavailable | LOCATION_PROVIDER_UNAVAILABLE | Không được chuyển thành NOT_SERVICABLE | Retry có kiểm soát/fallback |
| EX-06 | Không có Service Area Policy áp dụng | SERVICE_AREA_POLICY_NOT_FOUND | UNKNOWN/REVIEW_REQUIRED | Escalate Policy Owner |
| EX-07 | Policy hết hiệu lực hoặc có xung đột | SERVICE_AREA_POLICY_CONFLICT | REVIEW_REQUIRED | Policy Admin xử lý |
| EX-08 | Vị trí không thuộc Service Area theo policy ACTIVE | SERVICE_AREA_NOT_SUPPORTED | NOT_SERVICABLE | Thông báo theo policy; không tự ý override |
| EX-09 | Spatial engine lỗi | SERVICEABILITY_EVALUATION_FAILED | UNKNOWN | Retry/incident |
| EX-10 | Policy thay đổi trong lúc request đang xử lý | POLICY_VERSION_CONFLICT | Không dùng kết quả nửa cũ/nửa mới | Đánh giá lại với một version nhất quán |
| EX-11 | Người dùng gửi tọa độ và địa chỉ text mâu thuẫn | LOCATION_INPUT_CONFLICT | REVIEW_REQUIRED | Yêu cầu xác nhận nguồn đúng |
| EX-12 | Actor không có quyền xem/chọn địa chỉ | PERMISSION_DENIED | Không tiết lộ location ngoài scope | Kiểm tra quyền |
| EX-13 | Override không có quyền/lý do | SERVICEABILITY_OVERRIDE_FORBIDDEN | Không thay decision | Từ chối và audit |
| EX-14 | Booking đã READY_FOR_DISPATCH nhưng location bị phát hiện sai | LOCATION_REVALIDATION_REQUIRED | Không tự sửa Booking/Dispatch | Chuyển BKG-005/M03 theo change-impact policy |

### 7.5. Hậu điều kiện

**Thành công**
- Có resolved location reference/snapshot theo quy tắc dữ liệu.
- Có Serviceability Decision hoặc trạng thái UNKNOWN/REVIEW_REQUIRED được thể hiện rõ.
- Ghi policy version, source và timestamp.
- M01 nhận kết quả và áp dụng guard phù hợp.

**Thất bại**
- Không tự tạo kết luận NOT_SERVICABLE khi lỗi thực chất là lỗi provider/hạ tầng.
- Không chuyển Booking sang READY_FOR_DISPATCH khi pickup chưa đủ tin cậy hoặc decision bị policy yêu cầu chặn.
- Ghi error/trace phù hợp, không lộ PII.

---

## 8. State và decision model

### 8.1. Kết quả định vị (Location Resolution)

| Enum đề xuất | Tên hiển thị | Ý nghĩa |
|---|---|---|
| NOT_STARTED | Chưa kiểm tra | Chưa gửi yêu cầu resolve |
| RESOLVING | Đang định vị | Đang gọi provider/nguồn dữ liệu |
| RESOLVED | Đã định vị | Có một location candidate đủ rõ |
| AMBIGUOUS | Địa chỉ chưa rõ | Có nhiều candidate cần chọn |
| UNRESOLVED | Chưa định vị được | Không tìm thấy kết quả phù hợp |
| FAILED | Lỗi định vị | Provider/technical error |

### 8.2. Serviceability Decision

| Enum đề xuất | Tên hiển thị | Ý nghĩa |
|---|---|---|
| SERVICABLE | Có thể tiếp nhận theo policy | Vị trí thỏa policy áp dụng |
| NOT_SERVICABLE | Không thuộc phạm vi phục vụ | Policy hiện hành từ chối |
| REVIEW_REQUIRED | Cần kiểm tra | Có mơ hồ, xung đột hoặc cần quyết định thủ công |
| UNKNOWN | Chưa xác định | Thiếu policy hoặc lỗi kỹ thuật chưa thể kết luận |

Không nhập hai nhóm trạng thái này thành một trường `status`. Location Resolution mô tả độ tin cậy của vị trí; Serviceability Decision mô tả kết quả áp dụng chính sách.

### 8.3. Ánh xạ sang Booking

| Location Resolution | Serviceability Decision | Hành vi M01 đề xuất |
|---|---|---|
| RESOLVED | SERVICABLE | Có thể tiếp tục validation Booking |
| RESOLVED | NOT_SERVICABLE | Không chuyển READY_FOR_DISPATCH; xử lý theo policy |
| RESOLVED | REVIEW_REQUIRED | Chờ nhân viên/khách xác minh |
| RESOLVED | UNKNOWN | Chờ policy/fallback; không tự kết luận |
| AMBIGUOUS | Chưa đánh giá | Yêu cầu chọn địa điểm |
| UNRESOLVED | Chưa đánh giá | Yêu cầu nhập lại |
| FAILED | Chưa đánh giá | Báo lỗi kỹ thuật; retry/fallback |

Ánh xạ cuối cùng vào Booking State (VALIDATING, ACCEPTED, REJECTED, ON_HOLD, READY_FOR_DISPATCH) phải theo X-01 và policy được phê duyệt; BKG-003 không tự định nghĩa Booking State mới.

---

## 9. UI/UX

### 9.1. Thành phần dùng chung

- Ô tìm kiếm địa chỉ/địa danh.
- Gợi ý địa chỉ khi nhập.
- Bản đồ/pin vị trí nếu kênh hỗ trợ.
- Danh sách candidate khi kết quả mơ hồ.
- Hiển thị địa chỉ chuẩn hóa.
- Nguồn vị trí (nhập tay/GPS/địa chỉ đã lưu/nhân viên chọn).
- Thông báo Serviceability theo nội dung đã được duyệt.
- Nút xác nhận/chọn lại địa điểm.

### 9.2. Hành vi GPS

1. Xin quyền vị trí theo cơ chế của nền tảng.
2. Nếu người dùng không cấp quyền, cho nhập địa chỉ thủ công nếu policy cho phép.
3. Nếu GPS có sai số lớn hoặc vị trí không ổn định, hiển thị cảnh báo/không tự xác nhận.
4. GPS suggestion không tự thay thế vị trí khách đã chọn.
5. Khi người dùng kéo pin, lưu source là USER_SELECTED_MAP hoặc tương đương.
6. Không hiển thị độ chính xác giả nếu thiết bị/provider không cung cấp.

### 9.3. Trạng thái giao diện

| UI State | Nội dung |
|---|---|
| Empty | Nhập địa chỉ hoặc chọn vị trí trên bản đồ |
| Resolving | Đang tìm địa điểm |
| Resolved | Hiển thị địa chỉ chuẩn hóa và vị trí |
| Ambiguous | Hiển thị danh sách candidate để chọn |
| Not Found | Không tìm thấy, đề nghị nhập thêm thông tin |
| Serviceable | Có thể tiếp tục theo policy |
| Not Serviceable | Khu vực chưa được xác nhận phục vụ; hiển thị hướng dẫn theo policy |
| Review Required | Cần nhân viên kiểm tra |
| Provider Error | Tạm thời không kiểm tra được địa điểm; không hiển thị như bị từ chối phục vụ |

### 9.4. Message Code đề xuất

| Code | Điều kiện | Nội dung gợi ý |
|---|---|---|
| LOC-001 | Thiếu địa chỉ | Vui lòng nhập hoặc chọn điểm đón |
| LOC-002 | Đang resolve | Đang xác định vị trí |
| LOC-003 | Nhiều kết quả | Vui lòng chọn đúng địa chỉ/địa danh |
| LOC-004 | Không tìm thấy | Chưa tìm thấy địa điểm. Vui lòng nhập chi tiết hơn |
| LOC-005 | Có thể phục vụ | Điểm đón đáp ứng điều kiện khu vực theo chính sách hiện hành |
| LOC-006 | Không thuộc vùng phục vụ | Khu vực này hiện chưa được xác nhận phục vụ. Vui lòng liên hệ Tổng đài |
| LOC-007 | Cần kiểm tra | Điểm đón cần được nhân viên xác minh |
| LOC-008 | Provider lỗi | Hiện chưa kiểm tra được vị trí. Vui lòng thử lại |
| LOC-009 | GPS chưa xác nhận | Vui lòng kiểm tra và xác nhận điểm đón trên bản đồ |

Nội dung cuối cùng phải được Business Owner duyệt, tránh cam kết vượt quá kết quả kỹ thuật.

---

## 10. API và Event Contract

> Endpoint và payload dưới đây là đề xuất logic; chưa phải quyết định công nghệ hay hợp đồng production. Phải đồng bộ với X-03 và Data Dictionary.

### 10.1. Resolve Location

**Tên Command/Query:** `ResolveLocation`  
**Endpoint tham khảo:** `POST /api/v1/locations:resolve`

Request minh họa:
```json
{
  "input_text": "115 Nguyễn Huệ, phường Bến Nghé, Quận 1, TP.HCM",
  "coordinates": null,
  "source": "USER_ENTERED",
  "language": "vi",
  "context": {
    "booking_id": null,
    "subject_type": "PICKUP"
  }
}
```

Response minh họa:
```json
{
  "resolution_status": "RESOLVED",
  "candidates": [
    {
      "location_reference": "resolved-location-reference",
      "normalized_address": "Địa chỉ chuẩn hóa từ provider",
      "latitude": null,
      "longitude": null,
      "precision": "ROOFTOP_OR_PROVIDER_DEFINED",
      "provider_reference": "provider-place-reference"
    }
  ],
  "selected_candidate_required": false
}
```

Không dùng precision label cụ thể nếu provider không hỗ trợ; ví dụ trên chỉ là minh họa.

### 10.2. Evaluate Serviceability

**Tên Command/Query:** `EvaluateServiceability`  
**Endpoint tham khảo:** `POST /api/v1/serviceability:evaluate`

Request logic:
- location_reference hoặc tọa độ/geometry đã resolve.
- subject_type: PICKUP / DESTINATION / INTERMEDIATE_STOP.
- service_type nếu policy yêu cầu.
- vehicle_type nếu policy yêu cầu.
- fleet_type chỉ khi được xác định từ nguồn có thẩm quyền.
- booking_channel nếu policy được duyệt có phân biệt kênh.
- evaluation_time do server quản lý hoặc xác minh.
- correlation_id.

Response minh họa:
```json
{
  "decision": "SERVICABLE",
  "reason_code": "SERVICE_AREA_MATCH",
  "service_area_reference": "service-area-reference",
  "policy_version": "policy-version",
  "evaluated_at": "server-generated-timestamp",
  "review_required": false
}
```

### 10.3. Error Codes đề xuất

| Code | Ý nghĩa | Xử lý |
|---|---|---|
| LOCATION_INPUT_INVALID | Đầu vào địa điểm sai | Sửa input |
| LOCATION_NOT_FOUND | Không tìm được địa điểm | Nhập lại |
| LOCATION_AMBIGUOUS | Có nhiều kết quả | Người dùng chọn |
| LOCATION_PROVIDER_UNAVAILABLE | Provider lỗi/timeout | Retry/fallback |
| LOCATION_INPUT_CONFLICT | Text và tọa độ mâu thuẫn | Xác minh |
| SERVICE_AREA_POLICY_NOT_FOUND | Không có policy áp dụng | REVIEW_REQUIRED |
| SERVICE_AREA_POLICY_CONFLICT | Policy xung đột | Escalate |
| SERVICE_AREA_NOT_SUPPORTED | Policy từ chối vị trí | Xử lý theo business policy |
| SERVICEABILITY_EVALUATION_FAILED | Lỗi đánh giá | Retry/incident |
| SERVICEABILITY_OVERRIDE_FORBIDDEN | Không có quyền override | Từ chối |
| SERVICEABILITY_OVERRIDE_REASON_REQUIRED | Thiếu lý do override | Bổ sung lý do |
| PERMISSION_DENIED | Không có quyền/scope | Từ chối |

### 10.4. Event đề xuất

Nếu cần phát sự kiện:
- `LocationResolved`: phát khi có resolved location theo chính sách event.
- `ServiceabilityEvaluated`: phát khi có decision và cần consumer khác sử dụng.
- `ServiceAreaPolicyChanged`: do policy owner phát sau khi policy mới được phê duyệt/active.

Không phát Event cho từng lần người dùng gõ phím. Không đưa địa chỉ/PII đầy đủ vào Event nếu consumer không cần.

---

## 11. Bảo mật, Audit và quản trị Policy

### 11.1. Audit bắt buộc

| Audit Action | Khi nào |
|---|---|
| LOCATION_RESOLVE_REQUESTED | Nhận yêu cầu resolve có ý nghĩa nghiệp vụ |
| LOCATION_RESOLVED | Resolve thành công |
| LOCATION_CANDIDATE_SELECTED | Actor chọn candidate |
| SERVICEABILITY_EVALUATED | Tạo decision |
| SERVICE_AREA_POLICY_CREATED | Tạo policy |
| SERVICE_AREA_POLICY_UPDATED | Thay đổi policy |
| SERVICE_AREA_POLICY_ACTIVATED | Phê duyệt/kích hoạt policy |
| SERVICE_AREA_POLICY_RETIRED | Ngừng hiệu lực policy |
| SERVICEABILITY_OVERRIDDEN | Override decision |
| SERVICEABILITY_REEVALUATED | Đánh giá lại theo yêu cầu hợp lệ |

Audit phải lưu actor/service, thời điểm, policy version, lý do, correlation ID và before/after đối với thay đổi policy. Không lưu token/bí mật trong log.

### 11.2. Quản trị Service Area Policy

**Quy trình đề xuất:**
1. Người có quyền tạo policy ở trạng thái DRAFT.
2. Kiểm tra geometry, thời gian hiệu lực, chồng lấn và điều kiện áp dụng.
3. Người có thẩm quyền review.
4. Approver độc lập phê duyệt nếu áp dụng maker-checker.
5. Kích hoạt phiên bản mới với effective_from.
6. Phiên bản cũ được đóng hiệu lực/retire theo quy tắc; không xóa lịch sử.
7. Có khả năng rollback bằng cách kích hoạt phiên bản hợp lệ trước đó hoặc tạo phiên bản thay thế, không sửa ngược lịch sử đã dùng.

Quy trình maker-checker, cấp duyệt và người sở hữu cuối cùng cần X-04/Business Owner chốt.

### 11.3. PII và dữ liệu vị trí

- Chỉ thu thập vị trí cần thiết cho mục đích đặt xe.
- Hạn chế lưu raw GPS liên tục; BKG-003 chỉ cần dữ liệu định vị phục vụ yêu cầu cụ thể.
- Mask địa chỉ/số điện thoại trong log theo X-04.
- Retention của location input, resolved snapshot và audit cần được Pháp chế/ATTT xác nhận.
- Quyền đọc location lịch sử phải theo Booking scope và mục đích công việc.

---

## 12. NFR và vận hành

| NFR | Yêu cầu |
|---|---|
| Accuracy | Không tự khẳng định độ chính xác cao hơn dữ liệu provider |
| Latency | Mục tiêu p95/p99 cần chốt trong X-05 theo provider và loại request |
| Availability | Có fallback/REVIEW_REQUIRED khi provider không sẵn sàng |
| Consistency | Mỗi decision phải gắn đúng một policy version |
| Scalability | Hỗ trợ nhiều channel đồng thời và batch re-evaluation có kiểm soát |
| Security | Permission, scope, PII masking, policy administration audit |
| Observability | Correlation ID xuyên M01 → Location → Serviceability |
| Change Management | Policy versioning, effective time, approval, rollback |
| Resilience | Timeout, retry, circuit breaker/fallback theo quyết định kỹ thuật |
| Data Quality | Theo dõi địa chỉ không resolve, ambiguous, out-of-coverage, policy conflict |

Metrics tối thiểu:
- location_resolve_requests_total theo source/provider/result.
- location_resolve_latency p50/p95/p99.
- location_ambiguous_total.
- location_unresolved_total.
- location_provider_error_total.
- serviceability_decisions_total theo decision/reason/policy_version.
- service_area_policy_conflict_total.
- serviceability_override_total theo actor/reason.
- policy_version_distribution trong Booking mới.

---

## 13. Kiểm thử và Acceptance Criteria

### 13.1. Test Cases

| Test ID | Tình huống | Kết quả mong đợi |
|---|---|---|
| BKG003-TC-001 | Địa chỉ đầy đủ, resolve được một kết quả | Trả RESOLVED và normalized location |
| BKG003-TC-002 | Địa danh có nhiều candidate | Trả AMBIGUOUS, yêu cầu chọn |
| BKG003-TC-003 | Địa chỉ không tìm thấy | Trả LOCATION_NOT_FOUND |
| BKG003-TC-004 | GPS được cấp quyền | GPS là candidate; không tự thay vị trí khách đã xác nhận |
| BKG003-TC-005 | Không cấp quyền GPS | Cho nhập/chọn thủ công theo policy |
| BKG003-TC-006 | Text và tọa độ không tương ứng | REVIEW_REQUIRED/LOCATION_INPUT_CONFLICT |
| BKG003-TC-007 | Vị trí nằm trong Service Area ACTIVE | SERVICABLE với policy version |
| BKG003-TC-008 | Vị trí ngoài Service Area theo policy | NOT_SERVICABLE với reason code |
| BKG003-TC-009 | Không có policy áp dụng | UNKNOWN/REVIEW_REQUIRED, không tự ALLOW/DENY |
| BKG003-TC-010 | Policy hết hiệu lực | Không áp dụng policy hết hạn |
| BKG003-TC-011 | Hai policy chồng lấn, có precedence đã duyệt | Áp dụng đúng precedence và audit version |
| BKG003-TC-012 | Hai policy chồng lấn, chưa có precedence | REVIEW_REQUIRED, không đoán |
| BKG003-TC-013 | Location Provider timeout | Technical error; không trả NOT_SERVICABLE |
| BKG003-TC-014 | Serviceability engine lỗi | UNKNOWN/technical error; không tạo quyết định giả |
| BKG003-TC-015 | Điểm đón thuộc Service Area nhưng không thuộc Staging Area | Serviceability vẫn theo Service Area; M03 kiểm tra Staging riêng |
| BKG003-TC-016 | Điểm đón thuộc Service Area nhưng không có xe trống | BKG-003 vẫn SERVICABLE; M03 xử lý no candidate |
| BKG003-TC-017 | Điểm đến ngoài Service Area nhưng pickup hợp lệ | Trả decision riêng cho destination; không tự từ chối pickup |
| BKG003-TC-018 | Điểm dừng trung gian | Gắn đúng BookingStop/sequence theo BKG-004 |
| BKG003-TC-019 | Policy thay đổi trong lúc evaluation | Decision dùng một policy version nhất quán hoặc retry |
| BKG003-TC-020 | Actor không có quyền quản lý policy | Từ chối và audit |
| BKG003-TC-021 | Actor override không có permission | Từ chối |
| BKG003-TC-022 | Override có quyền nhưng thiếu reason | Yêu cầu reason |
| BKG003-TC-023 | Override hợp lệ | Lưu actor, reason, before/after và policy reference |
| BKG003-TC-024 | Policy mới được active | Booking mới dùng policy mới theo effective time; lịch sử cũ giữ version cũ |
| BKG003-TC-025 | Service Area bị retire | Không áp dụng cho request mới sau thời điểm hiệu lực |
| BKG003-TC-026 | Input có địa chỉ nhưng không có tọa độ và provider lỗi | Không tự tạo tọa độ; trả technical/review state |
| BKG003-TC-027 | Kết quả resolve từ SavedPlace cũ | Kiểm tra lại theo policy; không mặc định location còn đúng |
| BKG003-TC-028 | BKG-001 nhận decision NOT_SERVICABLE | Không chuyển Booking READY_FOR_DISPATCH |
| BKG003-TC-029 | BKG-001 nhận UNKNOWN do provider lỗi | Không biến thành REJECTED vì lý do ngoài vùng |
| BKG003-TC-030 | BKG-003 được gọi lặp cùng context | Không tạo side effect nghiệp vụ trùng; kết quả truy vết được |

### 13.2. Acceptance Criteria

**AC-01 — Resolve địa điểm rõ ràng**
- Given một location input có thể định vị
- When BKG-003 thực hiện ResolveLocation
- Then trả normalized location và nguồn/provider reference
- And giữ nguyên input text gốc để truy vết

**AC-02 — Địa điểm mơ hồ**
- Given provider trả nhiều candidate không đủ căn cứ tự chọn
- When BKG-003 đánh giá kết quả
- Then trả AMBIGUOUS
- And yêu cầu khách/nhân viên chọn
- And không tự tạo kết quả tọa độ đã xác nhận

**AC-03 — Serviceability theo policy**
- Given một Service Area Policy ACTIVE và có hiệu lực
- When vị trí được đánh giá theo policy
- Then trả decision, reason và policy version
- And lưu timestamp/correlation ID

**AC-04 — Không đồng nhất với Staging/Dispatch**
- Given điểm đón thuộc Service Area nhưng không thuộc Staging Area
- When BKG-003 kiểm tra
- Then kết quả Serviceability không bị thay đổi bởi Staging Area
- And M03 tự thực hiện kiểm tra Staging/Dispatch khi cần

**AC-05 — Không có policy**
- Given không tìm thấy policy áp dụng
- When BKG-003 đánh giá
- Then trả UNKNOWN hoặc REVIEW_REQUIRED theo policy vận hành
- And không tự kết luận NOT_SERVICABLE hoặc SERVICABLE

**AC-06 — Lỗi provider**
- Given Location Provider timeout/unavailable
- When BKG-003 xử lý
- Then trả lỗi kỹ thuật/UNKNOWN
- And không ghi nhận khách nằm ngoài vùng phục vụ

**AC-07 — Policy version**
- Given một policy version đã được dùng cho Booking
- When policy mới được kích hoạt
- Then Booking cũ vẫn truy vết được version đã áp dụng
- And Booking mới áp dụng version theo effective time

**AC-08 — Override có kiểm soát**
- Given actor có permission override và cung cấp lý do hợp lệ
- When actor thực hiện override
- Then decision được ghi nhận theo quy trình
- And lưu actor, reason, before/after, timestamp và audit
- And không sửa/xóa lịch sử decision gốc

**AC-09 — Không cam kết có xe**
- Given location được xác định SERVICABLE
- When M01 nhận kết quả
- Then chỉ kết luận địa điểm đáp ứng policy tiếp nhận
- And không khẳng định có xe hoặc có thể điều phối thành công

### 13.3. Definition of Done riêng

- [ ] Phân biệt Service Area, Staging Area và Dispatch Coverage được Business Owner/M03 phê duyệt.
- [ ] Có owner cho Service Area Policy và dữ liệu địa lý.
- [ ] Có danh sách vùng, geometry/source, policy version và effective time được xác nhận.
- [ ] Có quy tắc location precision/ambiguity được duyệt.
- [ ] Có quy tắc pickup, destination, stop và các loại dịch vụ.
- [ ] API contract và error codes được duyệt theo X-03.
- [ ] Data ownership/retention/PII được duyệt theo X-02/X-04.
- [ ] NFR/monitoring theo X-05.
- [ ] Test cases/AC đạt và Business UAT ký.
- [ ] M01 và M03 xác nhận không chồng lấn ownership.

---

## 14. Migration, cấu hình và triển khai

### 14.1. Cấu hình cần có

| Configuration | Ý nghĩa | Owner | Trạng thái |
|---|---|---|---|
| LOCATION_PROVIDER | Nhà cung cấp định vị | Platform/Architecture | TBD |
| LOCATION_PROVIDER_TIMEOUT | Timeout provider | Platform | TBD |
| LOCATION_RESOLUTION_POLICY | Quy tắc candidate/precision | Product/Operations | TBD |
| SERVICE_AREA_DATASET | Bộ dữ liệu vùng phục vụ | Business Policy Owner | TBD |
| SERVICE_AREA_POLICY_VERSION | Phiên bản policy active | Policy Admin | TBD |
| SERVICE_AREA_EFFECTIVE_TIME | Thời gian hiệu lực | Policy Admin | TBD |
| SERVICEABILITY_FALLBACK | Hành vi khi không đánh giá được | Operations | TBD |
| SERVICEABILITY_OVERRIDE_ENABLED | Có bật override hay không | Business/ATTT | TBD |

Không đặt default giả cho các vùng/phạm vi chưa được khảo sát.

### 14.2. Triển khai

- Nạp dữ liệu Service Area ở chế độ draft trước khi kích hoạt.
- Kiểm tra geometry, vùng chồng lấn, khoảng trống và policy precedence.
- Chạy shadow evaluation nếu có dữ liệu hiện trạng để so sánh, không tự động từ chối Booking trong giai đoạn shadow.
- Business Owner xác nhận các điểm biên/địa chỉ thực tế.
- Chỉ bật enforcement sau khi có phê duyệt và UAT.
- Có rollback/forward-fix policy; không xóa lịch sử decision.
- Có smoke test cho từng vùng, vùng biên, điểm nằm ngoài, địa chỉ mơ hồ và provider outage.

---

## 15. Rủi ro, quyết định và TBD

| ID | Nội dung cần quyết định | Ảnh hưởng | Owner đề xuất |
|---|---|---|---|
| BKG003-DEC-01 | Ai là Business Owner chính thức của Service Area Policy? | Governance/approval | COO/Operations |
| BKG003-DEC-02 | Danh sách Service Area hiện hành là gì? | Có tiếp nhận Booking hay không | Operations |
| BKG003-DEC-03 | Service Area được định nghĩa bằng polygon, địa giới hành chính, bán kính hay kết hợp? | GIS/data model | Architecture/Operations |
| BKG003-DEC-04 | Có vùng khác nhau theo DIRECT/PARTNER không? | Contract/dispatch/finance | Business/M02 |
| BKG003-DEC-05 | Có vùng khác nhau theo loại xe hoặc loại dịch vụ không? | Booking/dispatch | Operations |
| BKG003-DEC-06 | Có giới hạn theo kênh đặt xe không? | Channel policy | Customer/Operations |
| BKG003-DEC-07 | Quy tắc điểm đến ngoài Service Area? | One-way/long-distance/return | Operations/Pricing |
| BKG003-DEC-08 | Quy tắc điểm dừng trung gian? | BKG-004 | Operations |
| BKG003-DEC-09 | Độ chính xác định vị tối thiểu theo pickup? | Location provider/UX | Product/Operations |
| BKG003-DEC-10 | Khi địa chỉ mơ hồ, ai được chọn/override? | Call Center/ATTT | Operations |
| BKG003-DEC-11 | Provider bản đồ/geocoding nào được dùng? | Cost/latency/coverage | Architecture/Procurement |
| BKG003-DEC-12 | Có dữ liệu địa chỉ nội bộ/SavedPlace cần migration không? | Data migration | M01/Data Owner |
| BKG003-DEC-13 | Quy tắc ưu tiên khi nhiều Service Area chồng lấn? | Policy engine | Operations |
| BKG003-DEC-14 | Có cơ chế review thủ công/ngoại lệ ngoài vùng không? | Customer experience/risk | Operations |
| BKG003-DEC-15 | Maker-checker cho tạo/sửa/kích hoạt policy? | Security/governance | ATTT/Operations |
| BKG003-DEC-16 | Thời hạn lưu location input, GPS và decision? | Privacy/legal | Legal/ATTT |
| BKG003-DEC-17 | Có cần đánh giá lại Booking đang mở khi policy thay đổi? | Operational impact | Operations/M01 |
| BKG003-DEC-18 | Có fallback provider hoặc manual map pin khi provider lỗi? | Availability | Architecture |
| BKG003-DEC-19 | Ranh giới Staging Area hiện tại do ai quản lý, ở đâu? | M03 integration | Dispatch |
| BKG003-DEC-20 | Dispatch Coverage/Unicast policy hiện tại có nguồn dữ liệu riêng nào? | M03 integration | Dispatch |
| BKG003-DEC-21 | Cách xử lý địa chỉ ngoại tỉnh/liên tỉnh và đường dài? | TO.P9/BKG-009 | Operations/Pricing |
| BKG003-DEC-22 | Có cho phép Booking ở vùng mới trong thời gian thử nghiệm? | Rollout | Operations |
| BKG003-DEC-23 | SLA khi kết quả UNKNOWN/REVIEW_REQUIRED? | Call Center/UX | Operations |
| BKG003-DEC-24 | Bộ dữ liệu địa giới hành chính cập nhật theo nguồn nào và chu kỳ nào? | Data quality | Data Owner |

---

## 16. Tiêu chí nghiệm thu Feature

- [ ] Service Area, Staging Area và Dispatch Coverage được định nghĩa riêng, có owner.
- [ ] Nguồn dữ liệu vùng phục vụ và quy tắc policy được Business Owner phê duyệt.
- [ ] Location resolution có quy tắc input, candidate, ambiguity và precision.
- [ ] GPS suggestion không tự động thay vị trí khách xác nhận.
- [ ] Có kết quả SERVICABLE/NOT_SERVICABLE/REVIEW_REQUIRED/UNKNOWN và reason code.
- [ ] Lỗi provider không bị diễn giải thành từ chối phục vụ.
- [ ] Mọi decision lưu policy version và timestamp.
- [ ] Policy có version/effective time/approval/audit.
- [ ] Override có permission, reason và audit.
- [ ] M01/M03 xác nhận rõ data ownership và API boundary.
- [ ] BKG-001/BKG-002 dùng chung contract; không viết logic vùng riêng trong từng channel.
- [ ] BKG-004 có thể gọi kiểm tra cho từng stop theo cùng contract.
- [ ] NFR, security, test evidence và runbook hoàn tất.
- [ ] Business UAT và Architecture review được ký.

### 16.1. Kết luận hiện tại

**Trạng thái:** DRAFT — Chưa đạt READY/DoR.

**Các blocker chính:**
1. Chưa có danh sách Service Area hiện hành được xác nhận.
2. Chưa xác định Business Owner và Policy Administrator.
3. Chưa chốt nguồn dữ liệu địa lý/provider.
4. Chưa chốt quy tắc điểm đến/điểm dừng và các ngoại lệ.
5. Cần M03 xác nhận riêng Staging Area và Dispatch Coverage.

**Không bật cơ chế tự động từ chối Booking theo Service Area cho đến khi các policy này được Mai Linh phê duyệt và kiểm thử vùng biên.**

---

**Quay lại:** [Feature Specifications](../README.md) · [M01 — Customer & Booking](../../../02-DOMAIN-MODULES/01-customer-and-booking/README.md)
