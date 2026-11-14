# 02 — KHẢO SÁT HIỆN TRẠNG VÀ PHÂN TÍCH KHOẢNG CÁCH HỆ THỐNG TAXI MAI LINH

**Mã tài liệu:** TAXI-ASM-002  
**Phiên bản:** 0.1  
**Trạng thái:** MẪU KHẢO SÁT — CHƯA CÓ KẾT QUẢ XÁC NHẬN  
**Tài liệu liên quan:** [01 — Thiết kế nghiệp vụ và hệ thống toàn diện](./01-FULL-BUSINESS-SYSTEM-BLUEPRINT.md)

> **Lưu ý về độ tin cậy:** Tài liệu này là khung khảo sát và phương pháp phân tích khoảng cách. Tại thời điểm phát hành, dự án chưa có đủ bằng chứng khảo sát hệ thống đang vận hành của Mai Linh để kết luận tính năng nào đã có, đang hoạt động, thiếu hoặc cần thay thế. Vì vậy, mọi ô trạng thái khảo sát được để ở mức **Chưa khảo sát**. Không được hiểu nội dung đề xuất trong tài liệu này là hiện trạng đã được xác nhận.

## 1. Mục tiêu

1. Ghi nhận đúng hệ thống, quy trình và cách vận hành thực tế đang được Mai Linh sử dụng.
2. Đối chiếu hiện trạng với quy trình mục tiêu TO.P1–TO.P9 và các nhóm yêu cầu TO.A–TO.J trong tài liệu KPMG năm 2015.
3. Phân loại từng năng lực thành: giữ nguyên, nâng cấp, xây mới, thay thế, ngừng sử dụng hoặc chờ quyết định.
4. Xác định hệ thống đang có, hệ thống cần tích hợp, dữ liệu chủ, đơn vị chịu trách nhiệm và các rủi ro chuyển đổi.
5. Tạo đường cơ sở đã được nghiệp vụ xác nhận trước khi thiết kế chi tiết và giao DEV.

## 2. Nguyên tắc khảo sát

- Khảo sát theo bằng chứng: màn hình thực tế, tài liệu quy trình, cấu hình, báo cáo, API, dữ liệu mẫu đã ẩn thông tin nhạy cảm và phỏng vấn người trực tiếp vận hành.
- Phân biệt **có chức năng** với **đang sử dụng**, **đang sử dụng ổn định** và **đáp ứng đúng nghiệp vụ**.
- Không kết luận chỉ dựa trên lời mô tả hoặc tên chức năng.
- Ghi rõ phạm vi áp dụng: chi nhánh, khu vực, loại hình xe, DIRECT/PARTNER, kênh tiếp nhận và nhóm người dùng.
- Không đưa dữ liệu cá nhân, thông tin xác thực, khóa API hoặc dữ liệu khách hàng thật vào tài liệu này.
- Mọi khác biệt so với KPMG 2015 phải có người xác nhận và lý do.
- Không mặc định giải pháp hiện tại phải giữ lại; cũng không mặc định phải thay mới nếu chưa đánh giá chi phí, rủi ro và khả năng đáp ứng.

## 3. Quy ước trạng thái

### 3.1. Trạng thái khảo sát

| Mã | Trạng thái | Ý nghĩa |
|---|---|---|
| SUR-00 | Chưa khảo sát | Chưa thu thập đủ thông tin |
| SUR-01 | Đã ghi nhận | Có mô tả ban đầu nhưng chưa đủ bằng chứng |
| SUR-02 | Đã xác minh | Đã kiểm tra bằng chứng và người phụ trách xác nhận |
| SUR-03 | Có khác biệt | Thực tế khác tài liệu/quy trình mục tiêu |
| SUR-04 | Chưa thống nhất | Các bên cung cấp thông tin chưa thống nhất |
| SUR-05 | Không áp dụng | Có xác nhận lý do không áp dụng |

### 3.2. Kết luận khoảng cách và hướng xử lý

| Mã | Kết luận | Hướng xử lý |
|---|---|---|
| GAP-KEEP | Đáp ứng | Giữ nguyên, có thể cần chuẩn hóa tài liệu |
| GAP-IMPROVE | Đáp ứng một phần | Nâng cấp/cải tiến |
| GAP-BUILD | Chưa có | Xây mới |
| GAP-REPLACE | Có nhưng không còn phù hợp | Thay thế có kế hoạch |
| GAP-RETIRE | Không còn nhu cầu | Ngừng sử dụng sau phê duyệt |
| GAP-TBD | Chưa đủ căn cứ | Tiếp tục khảo sát hoặc xin quyết định |

**Không dùng một trạng thái duy nhất để thay thế cả trạng thái khảo sát và kết luận khoảng cách.** Ví dụ, một chức năng có thể đã xác minh đang hoạt động nhưng vẫn chỉ đáp ứng một phần yêu cầu mục tiêu.

## 4. Phạm vi khảo sát

### 4.1. Phạm vi tổ chức và vận hành

Cần xác nhận tối thiểu:
- Đơn vị/chi nhánh/khu vực tham gia khảo sát.
- Trung tâm điều hành, tổng đài, điều hành địa phương và điểm tiếp thị.
- Nhóm tài xế xe thuộc Mai Linh và tài xế/đơn vị đối tác.
- Bộ phận kinh doanh, hợp đồng, kế toán, công nợ, đối soát và chăm sóc khách hàng.
- Người sở hữu quy trình và người có thẩm quyền phê duyệt.

### 4.2. Phạm vi kênh dịch vụ

| Kênh | Nội dung cần xác minh | Trạng thái |
|---|---|---|
| Tổng đài điện thoại | Tiếp nhận, nhận diện khách, điều xe, ghi âm/tương tác, gọi lại | Chưa khảo sát |
| Ứng dụng khách hàng | Đặt xe, theo dõi xe, thay đổi/hủy, thanh toán, đánh giá | Chưa khảo sát |
| Ứng dụng tài xế | Trạng thái xe, nhận cuốc, đón/trả, chuyến, doanh thu | Chưa khảo sát |
| Website | Đặt xe, tra cứu, quản lý yêu cầu | Chưa khảo sát |
| SMS | Cú pháp, xác thực, phản hồi và xử lý yêu cầu | Chưa khảo sát |
| Chat/mạng xã hội | Kênh tiếp nhận, nhân viên xử lý, đồng bộ yêu cầu | Chưa khảo sát |
| Điểm tiếp thị | Tiếp nhận tại điểm, nhân viên, hàng đợi | Chưa khảo sát |
| API/đối tác | Kết nối nguồn đặt xe hoặc khách hàng doanh nghiệp | Chưa khảo sát |

### 4.3. Phạm vi mô hình kinh doanh

| Nội dung | Câu hỏi khảo sát | Trạng thái |
|---|---|---|
| DIRECT | Xe, tài xế, doanh thu và chi phí thuộc Mai Linh được quản lý thế nào? | Chưa khảo sát |
| PARTNER | Đối tác, xe, tài xế, hợp đồng và nghĩa vụ dịch vụ được quản lý thế nào? | Chưa khảo sát |
| Hợp đồng | Loại hợp đồng, thời hạn, phạm vi dịch vụ, bảng giá và phụ lục | Chưa khảo sát |
| Doanh thu | Doanh thu chuyến được ghi nhận cho chủ thể nào, theo nguyên tắc nào? | Chưa khảo sát |
| Công nợ | Đối tượng phải thu/phải trả, kỳ hạn và hạn mức | Chưa khảo sát |
| Đối soát/quyết toán | Chu kỳ, dữ liệu đối chiếu, chênh lệch và phê duyệt | Chưa khảo sát |

**Nguyên tắc kiến trúc đã thống nhất:** DIRECT và PARTNER dùng chung nền tảng điều phối, nhưng hợp đồng, tài chính, doanh thu, công nợ và quyết toán phải tách biệt theo đúng chủ thể và chính sách được duyệt.

## 5. Ma trận khảo sát theo Module

Danh mục khảo sát được chuẩn hóa theo tài liệu 03. Các dòng dưới đây là danh mục cần khảo sát, không phải xác nhận hệ thống hiện tại đã có hoặc chưa có chức năng.

| Mã | Module/năng lực | Hạng mục cần khảo sát | Trạng thái | Kết luận khoảng cách |
|---|---|---|---|---|
| M01 | Khách hàng và Đặt xe | Hồ sơ khách hàng, địa chỉ, tiếp nhận đa kênh, tạo/sửa/hủy Booking, đặt trước, mã yêu cầu, chống trùng | Chưa khảo sát | Chưa kết luận |
| M02 | Đội xe và Đối tác | Hồ sơ tài xế, xe, chủ xe, đơn vị đối tác, giấy tờ, trạng thái, điều kiện hoạt động và hợp đồng | Chưa khảo sát | Chưa kết luận |
| M03 | Điều phối và Hàng đợi | Tìm xe, Unicast/Broadcast, tự đăng ký, chỉ định thủ công, chống tranh chấp, FIFO, điểm tiếp thị và điều phối lại | Chưa khảo sát | Chưa kết luận |
| M04 | Ứng dụng Tài xế và Chuyến đi | Đến đón, xác nhận khách, bắt đầu/kết thúc Trip, GPS, hành trình, hủy và sự cố | Chưa khảo sát | Chưa kết luận |
| M05 | Giá cước và Thanh toán | Biểu giá, tính cước, đường dài, phụ trội, hóa đơn, tiền mặt/thẻ/voucher, hoàn tiền | Chưa khảo sát | Chưa kết luận |
| M06 | Tài chính và Quyết toán | Sổ cái, doanh thu, công nợ, nghĩa vụ đối tác, đối soát, chi trả và điều chỉnh | Chưa khảo sát | Chưa kết luận |
| M07 | Trung tâm Điều hành và Chăm sóc khách hàng | Bảng điều hành, bản đồ, hàng đợi giám sát, can thiệp, khiếu nại và SLA | Chưa khảo sát | Chưa kết luận |
| M08 | Nền tảng, Phân quyền và Kiểm toán | Tài khoản, phân quyền, phạm vi dữ liệu, nhật ký, cấu hình, giám sát, sao lưu và bảo mật | Chưa khảo sát | Chưa kết luận |
| M09 | Tích hợp và Thông báo | Bản đồ, SMS/Voice/Push/Email, cổng đối tác, thanh toán và tích hợp hệ thống doanh nghiệp | Chưa khảo sát | Chưa kết luận |
| M10 | Báo cáo và Phân tích | Báo cáo nhu cầu, doanh thu, vận hành, hiệu suất, SLA, KPI và phân tích | Chưa khảo sát | Chưa kết luận |


## 6. Ma trận đối chiếu quy trình TO.P1–TO.P9

| Mã KPMG | Quy trình cần khảo sát | Điểm cần xác minh tại hiện trường | Trạng thái |
|---|---|---|---|
| TO.P1 | Khách đặt Taxi qua Tổng đài | Từ cuộc gọi đến Booking, phân công, đón khách, Trip, thanh toán và phản hồi | Chưa khảo sát |
| TO.P2 | Khách sử dụng Ứng dụng | Luồng App, định vị, báo giá, chọn xe, theo dõi và thanh toán | Chưa khảo sát |
| TO.P3 | Khách tại Điểm tiếp thị có nhân viên | Nhận yêu cầu, điều hành điểm, liên hệ tổng đài, phân xe | Chưa khảo sát |
| TO.P4 | Khách tại Khu vực sắp tài không có nhân viên | Đăng ký hàng đợi, FIFO, phát yêu cầu, đến lượt và bỏ lượt | Chưa khảo sát |
| TO.P5 | Khách đặt xe qua SMS | Cú pháp, xác thực, phản hồi, mã đặt xe và xử lý sai cú pháp | Chưa khảo sát |
| TO.P6 | Khách đặt xe qua Chat | Kênh chat, nhân viên tiếp nhận, tạo yêu cầu và lịch sử hội thoại | Chưa khảo sát |
| TO.P7 | Khách đặt xe qua Web | Xác thực, tạo/sửa/hủy, xác nhận và theo dõi | Chưa khảo sát |
| TO.P8 | Khách đặt xe trước | Lịch đặt, xác nhận, thời điểm chuyển điều phối, thay đổi/hủy | Chưa khảo sát |
| TO.P9 | Cước phí đường dài | Điều kiện áp dụng, tuyến, loại xe, phụ trội, chiều về và hóa đơn | Chưa khảo sát |

## 7. Ma trận khảo sát dữ liệu và hệ thống tích hợp

| Nhóm dữ liệu/hệ thống | Cần xác minh | Đơn vị sở hữu/đầu mối | Trạng thái |
|---|---|---|---|
| Khách hàng và địa chỉ | Nguồn dữ liệu chuẩn, mã định danh, trùng lặp, lịch sử | Chưa xác định | Chưa khảo sát |
| Tài xế và nhân sự | Hệ thống nguồn, mã nhân viên, trạng thái và đồng bộ | Chưa xác định | Chưa khảo sát |
| Xe và đối tác | Mã xe, chủ sở hữu, loại xe, hợp đồng, tình trạng pháp lý | Chưa xác định | Chưa khảo sát |
| Booking/Dispatch/Trip | Hệ thống tạo dữ liệu, mã liên kết, lịch sử và lưu trữ | Chưa xác định | Chưa khảo sát |
| Giá cước | Nguồn biểu giá, phiên bản, thời gian hiệu lực và phê duyệt | Chưa xác định | Chưa khảo sát |
| Hóa đơn/thanh toán | Hệ thống kế toán, hóa đơn điện tử, cổng thanh toán, đối soát | Chưa xác định | Chưa khảo sát |
| Bản đồ/GPS | Nhà cung cấp, thiết bị, tần suất, độ chính xác, quyền sử dụng | Chưa xác định | Chưa khảo sát |
| Tổng đài/CRM | Nền tảng, API, ghi âm, lịch sử liên hệ và định danh | Chưa xác định | Chưa khảo sát |
| Báo cáo/Kho dữ liệu | Nguồn, lịch cập nhật, định nghĩa chỉ tiêu và người dùng | Chưa xác định | Chưa khảo sát |
| Hạ tầng CNTT | Máy chủ, mạng, cloud/on-premise, dự phòng, giám sát và sao lưu | Chưa xác định | Chưa khảo sát |

## 8. Phiếu ghi nhận chi tiết cho từng chức năng

Mỗi chức năng được khảo sát phải có một bản ghi riêng theo mẫu:

| Trường thông tin | Nội dung ghi nhận |
|---|---|
| Mã khảo sát | SUR-F-XXX |
| Module / chức năng | Tên Module và chức năng |
| Quy trình liên quan | TO.P1–TO.P9 hoặc quy trình nội bộ |
| Đơn vị sử dụng | Đơn vị/chi nhánh/nhóm vai trò |
| Mô tả cách làm hiện tại | Ghi đúng trình tự thực tế |
| Hệ thống đang sử dụng | Tên phần mềm/phiên bản/nhà cung cấp nếu xác minh được |
| Tần suất và quy mô | Số lượng giao dịch, người dùng, thời điểm cao điểm |
| Đầu vào / đầu ra | Dữ liệu nhận vào và kết quả tạo ra |
| Trạng thái và quy tắc | Trạng thái, điều kiện, thời hạn, ngoại lệ |
| Bằng chứng | Mã tài liệu, ảnh màn hình đã che dữ liệu nhạy cảm, báo cáo hoặc biên bản |
| Vấn đề hiện tại | Lỗi, thao tác thủ công, chậm, sai lệch, thiếu kiểm soát |
| Mong muốn nghiệp vụ | Kết quả cần đạt |
| Trạng thái khảo sát | SUR-00 đến SUR-05 |
| Kết luận khoảng cách | GAP-KEEP đến GAP-TBD |
| Hướng xử lý đề xuất | Giữ / Nâng cấp / Xây mới / Thay thế / Ngừng / Khảo sát thêm |
| Người xác nhận | Họ tên/chức danh theo biên bản nội bộ |
| Ngày xác nhận | Ngày ghi nhận |
| Quyết định/phê duyệt | Người có thẩm quyền và nội dung quyết định |

## 9. Danh mục vấn đề và quyết định cần làm rõ

| Mã | Vấn đề cần quyết định | Tác động nếu chưa chốt | Chủ trì đề xuất | Trạng thái |
|---|---|---|---|---|
| DEC-001 | Phạm vi triển khai và đơn vị thí điểm | Không xác định được phạm vi dữ liệu và nghiệm thu | Ban chỉ đạo/Chủ nghiệp vụ | Chưa quyết định |
| DEC-002 | Mô hình DIRECT/PARTNER và chủ thể tài chính | Sai lệch hợp đồng, doanh thu, công nợ, quyết toán | Kinh doanh/Kế toán/Vận hành | Chưa quyết định |
| DEC-003 | Kênh đặt xe giữ lại và kênh cần bổ sung | Ảnh hưởng M01/M07 và tích hợp | Vận hành/CNTT | Chưa quyết định |
| DEC-004 | Quy tắc điều phối, bán kính và Unicast/Broadcast | Ảnh hưởng trực tiếp khả năng phân xe | Điều hành | Chưa quyết định |
| DEC-005 | Trạng thái Booking/Dispatch/Trip hiện hành | Nguy cơ xung đột dữ liệu và luồng xử lý | Vận hành/CNTT | Chưa quyết định |
| DEC-006 | Biểu giá và chính sách đường dài/chiều về/đi ghép | Sai cước và doanh thu | Kinh doanh/Kế toán | Chưa quyết định |
| DEC-007 | Hệ thống nguồn cho khách hàng, tài xế, xe | Trùng lặp và sai lệch dữ liệu chủ | Chủ dữ liệu/CNTT | Chưa quyết định |
| DEC-008 | Nền tảng tích hợp và hệ thống cần giữ | Chi phí, rủi ro và phạm vi chuyển đổi | CNTT/Các đơn vị sở hữu | Chưa quyết định |
| DEC-009 | Năng lực tải và chỉ tiêu vận hành | Không đủ căn cứ chọn kiến trúc và hạ tầng | CNTT/Vận hành | Chưa quyết định |
| DEC-010 | Tiêu chí nghiệm thu và chuyển đổi | Không kiểm soát được chất lượng go-live | Ban chỉ đạo/Chủ nghiệp vụ | Chưa quyết định |

## 10. Phương pháp và kế hoạch thực hiện khảo sát

### Bước 1 — Chuẩn bị
- Chốt danh sách đơn vị, vai trò và đầu mối.
- Thu thập sơ đồ tổ chức, quy trình, tài liệu hệ thống và danh sách phần mềm.
- Lập lịch phỏng vấn và quan sát thực tế.

### Bước 2 — Khảo sát theo hành trình nghiệp vụ
- Theo dõi một yêu cầu thật đã được ẩn thông tin từ lúc tiếp nhận đến khi kết thúc và đối soát.
- Khảo sát riêng luồng Tổng đài, App, Web, SMS/Chat, điểm tiếp thị và đặt trước nếu đang áp dụng.
- Quan sát luồng điều phối, tài xế nhận cuốc, đón khách và xử lý ngoại lệ.

### Bước 3 — Xác minh hệ thống và dữ liệu
- Kiểm tra màn hình, quyền, cấu hình, báo cáo và nhật ký.
- Kiểm tra API/tích hợp bằng tài liệu kỹ thuật hoặc môi trường thử nghiệm được cấp phép.
- Lấy mẫu dữ liệu đã ẩn danh; xác minh mã định danh và quan hệ dữ liệu.

### Bước 4 — Phân tích khoảng cách
- Đối chiếu với yêu cầu KPMG và mục tiêu đã được Mai Linh xác nhận.
- Đánh giá mức đáp ứng, rủi ro, tác động và phụ thuộc.
- Đề xuất hướng xử lý, không tự ý quyết định thay chủ nghiệp vụ.

### Bước 5 — Xác nhận và khóa đường cơ sở
- Tổ chức phiên rà soát liên phòng ban.
- Ghi nhận ý kiến khác biệt và quyết định.
- Chủ nghiệp vụ ký xác nhận phạm vi, kết luận và các vấn đề còn mở.
- Chuyển kết quả đã duyệt thành đầu vào cho tài liệu kiến trúc và đặc tả chi tiết.

## 11. Tiêu chí hoàn thành tài liệu khảo sát

Tài liệu 02 chỉ được xem là đủ điều kiện làm đầu vào thiết kế khi:
1. Tất cả 10 Module đã có đầu mối nghiệp vụ và đầu mối hệ thống.
2. Các quy trình TO.P1–TO.P9 đã được xác nhận áp dụng, thay đổi hoặc không áp dụng.
3. Các chức năng trọng yếu có bằng chứng và trạng thái khảo sát.
4. Mô hình DIRECT/PARTNER và ranh giới tài chính được xác nhận.
5. Hệ thống nguồn, dữ liệu chủ và tích hợp đã được lập danh mục.
6. Các khoảng cách có hướng xử lý và người chịu trách nhiệm.
7. Các quyết định còn mở có người phụ trách và thời hạn.
8. Chủ nghiệp vụ và đại diện CNTT xác nhận biên bản khảo sát.

## 12. Đầu ra chuyển sang bước tiếp theo

Sau khi khảo sát và phê duyệt, dự án phải có:
- Ma trận hiện trạng và khoảng cách đã xác minh.
- Danh mục hệ thống/phần mềm và sơ đồ tích hợp hiện trạng.
- Danh mục dữ liệu chủ và chủ sở hữu dữ liệu.
- Danh mục quy trình được giữ/thay đổi/không áp dụng.
- Danh mục vấn đề, rủi ro và quyết định.
- Phạm vi ưu tiên triển khai và đơn vị thí điểm.

**Tài liệu tiếp theo:** 03 — ĐẶC TẢ QUY TRÌNH NGHIỆP VỤ ĐẦU CUỐI TO.P1–TO.P9. Chỉ bắt đầu chốt thiết kế chi tiết dựa trên những kết quả khảo sát đã được xác nhận; không tự điền giả định vào chỗ dữ liệu còn thiếu.

## 13. Lịch sử phiên bản

| Phiên bản | Ngày | Nội dung | Trạng thái |
|---|---|---|---|
| 0.1 | 2026-11-14 | Tạo khung khảo sát, ma trận Module, quy trình, dữ liệu và quyết định | Mẫu khảo sát |
