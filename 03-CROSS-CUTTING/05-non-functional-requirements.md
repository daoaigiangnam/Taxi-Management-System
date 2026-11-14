# X-05 — YÊU CẦU PHI CHỨC NĂNG (NON-FUNCTIONAL REQUIREMENTS)

**Mã tài liệu:** TAXI-X-05  
**Phiên bản:** 0.1  
**Trạng thái:** DỰ THẢO KIẾN TRÚC — CHỜ BUSINESS, VẬN HÀNH, CNTT, ATTT VÀ BAN ĐIỀU HÀNH PHÊ DUYỆT  
**Phạm vi:** Hiệu năng, khả năng mở rộng, tính sẵn sàng, độ tin cậy, phục hồi thảm họa, khả năng quan sát, khả năng bảo trì, tương thích, dữ liệu và vận hành  
**Tài liệu liên quan:** [00 — Tổng quan](../00-OVERVIEW.md) · [03 — Kiến trúc Module](../03-MODULE-ARCHITECTURE-AND-OWNERSHIP.md) · [04 — Quy trình TO.P1–TO.P9](../04-END-TO-END-WORKFLOWS-TO-P1-TO-P9.md) · [X-01 — State Machine](./01-state-machines-and-transition-rules.md) · [X-02 — Data Architecture & ERD](./02-data-architecture-and-erd.md) · [X-03 — API & Event Contracts](./03-api-and-event-contracts.md) · [X-04 — Security, IAM & Audit](./04-security-iam-and-audit.md)

> **Phân biệt nguồn và đề xuất:** Tài liệu KPMG Taxi năm 2015 mô tả quy trình, vai trò, nhu cầu điều hành và một số tham số nghiệp vụ lịch sử; không đưa ra SLO/SLA kỹ thuật toàn hệ thống, công suất tải, RTO/RPO, kiến trúc HA hoặc chỉ tiêu giám sát. Vì vậy, tài liệu này là khung NFR đề xuất cho hệ thống mới. Mọi con số mục tiêu phải được đo, mô hình hóa và phê duyệt trước khi trở thành cam kết nghiệm thu.

---

## 1. Mục tiêu và nguyên tắc

NFR xác định chất lượng và giới hạn vận hành mà toàn hệ thống phải đáp ứng, độc lập với cách từng Feature thực hiện nghiệp vụ.

### 1.1. Mục tiêu

- Hệ thống phục vụ đặt xe, điều phối, tài xế và chuyến đi trong điều kiện vận hành liên tục.
- Duy trì tính nhất quán của Booking, Offer, Assignment, Trip, Fare và giao dịch tài chính khi tải cao hoặc lỗi thành phần.
- Có khả năng mở rộng theo số lượng khách, tài xế, chi nhánh, đối tác, chuyến đi và kênh tích hợp.
- Phát hiện nhanh, khoanh vùng và khôi phục sự cố.
- Có số liệu đo lường được để nghiệm thu, không dùng các nhận định cảm tính như “nhanh”, “ổn định”, “chịu tải tốt”.
- Đảm bảo DIRECT và PARTNER dùng chung nền tảng vận hành nhưng không phá vỡ ranh giới dữ liệu và tài chính.

### 1.2. Nguyên tắc

1. Mỗi NFR phải có ID, mô tả, phương pháp đo, ngưỡng mục tiêu, phạm vi áp dụng, owner và bằng chứng nghiệm thu.
2. Phân biệt **SLO** (mục tiêu chất lượng dịch vụ), **SLI** (chỉ số đo) và **SLA** (cam kết dịch vụ nếu có).
3. Đo từ góc nhìn người dùng và từ từng thành phần kỹ thuật.
4. Dùng percentile (p50/p95/p99) cho latency; không chỉ dùng trung bình.
5. Đo tải theo giờ cao điểm, đột biến, mùa vụ và kịch bản lỗi.
6. Không đánh đổi tính đúng đắn của trạng thái nghiệp vụ và tài chính để lấy tốc độ.
7. NFR phải được kiểm tra từ sớm và xuyên suốt, không đợi đến cuối dự án.
8. Chỉ tiêu TBD phải được xác định từ khảo sát và load test; không tự đặt làm cam kết.

---

## 2. Mô hình phân loại yêu cầu

| Nhóm | Nội dung | Ví dụ chỉ số |
|---|---|---|
| Performance | Thời gian phản hồi và xử lý | API latency p95/p99 |
| Capacity | Năng lực xử lý tải | Booking/giây, concurrent sessions |
| Scalability | Mở rộng theo tải | Tăng năng lực khi tăng node |
| Availability | Khả năng sẵn sàng | Uptime, error budget |
| Reliability | Độ tin cậy và nhất quán | Tỷ lệ lỗi, duplicate/lost event |
| Resilience | Chịu lỗi và suy giảm có kiểm soát | Failover, retry, circuit breaker |
| DR/BCP | Phục hồi thảm họa | RTO, RPO |
| Observability | Khả năng giám sát | Metrics, logs, traces, alert |
| Security & Privacy | Bảo mật và dữ liệu cá nhân | Kiểm soát truy cập, audit |
| Maintainability | Khả năng bảo trì | MTTR, deployment rollback |
| Compatibility | Tương thích thiết bị/kênh | OS, browser, API version |
| Data Lifecycle | Lưu trữ, truy vấn, lưu giữ | Retention, archive, restore |
| Usability & Accessibility | Dễ sử dụng, dễ tiếp cận | Task completion, usability test |

---

## 3. NFR theo luồng nghiệp vụ trọng yếu

### 3.1. Booking

- Tiếp nhận Booking từ App, Web, Tổng đài, SMS, Chat, điểm tiếp thị và đặt trước.
- Không tạo Booking trùng khi client retry hoặc gửi lại cùng một yêu cầu có idempotency key.
- Phản hồi phải phân biệt: đã tiếp nhận, đang xử lý, bị từ chối và chưa xác nhận.
- Khi dịch vụ bản đồ/định vị lỗi, hệ thống phải có cách báo trạng thái rõ ràng và quy trình dự phòng được nghiệp vụ phê duyệt.
- Sửa/hủy Booking phải đồng bộ đúng với trạng thái Dispatch/Assignment liên quan, không để lời mời hoặc phân công cũ tiếp tục có hiệu lực.

### 3.2. Dispatch

- Tải vị trí tài xế và yêu cầu điều phối có thể tăng đột biến vào giờ cao điểm.
- Candidate eligibility phải được áp dụng trước ranking; không trả xe không đủ điều kiện chỉ để cải thiện tốc độ.
- Một Booking không được có nhiều Assignment hiệu lực đồng thời trái quy tắc.
- Xử lý cạnh tranh khi nhiều điều phối viên hoặc worker cùng thao tác phải có concurrency control.
- Khi bản đồ/route provider lỗi, phải phân biệt dữ liệu route cũ, dữ liệu không khả dụng và fallback; không giả định khoảng cách đường chim bay tương đương quãng đường thực tế.
- Retry không được tạo Offer/Assignment trùng; timeout phải có trạng thái có thể truy vết.

### 3.3. Driver App và Trip

- Cập nhật vị trí phải chịu được mạng yếu, mất kết nối tạm thời và đồng bộ trở lại.
- Sự kiện offline phải có sequence/time metadata và quy tắc xử lý dữ liệu đến muộn.
- Không được để cập nhật GPS làm ghi đè trạng thái Trip hợp lệ.
- SOS/Incident cần luồng ưu tiên, giám sát và cảnh báo riêng theo mức độ nghiêm trọng.
- Ứng dụng phải thông báo rõ khi GPS, mạng hoặc quyền thiết bị không khả dụng.

### 3.4. Fare, Payment và Finance

- Tính cước phải có khả năng tái lập từ dữ liệu chuyến, biểu giá và phiên bản quy tắc đã áp dụng.
- Payment/Refund/Settlement phải idempotent và chống ghi nhận trùng.
- Không được mất dấu giao dịch khi timeout giữa hệ thống Taxi và cổng thanh toán.
- Dữ liệu đã ghi sổ phải điều chỉnh bằng giao dịch bù/điều chỉnh có Audit, không sửa âm thầm lịch sử.
- Batch quyết toán phải có kiểm tra toàn vẹn, khả năng chạy lại an toàn và đối soát kết quả.

---

## 4. Performance — Hiệu năng

### 4.1. Chỉ số cần đo

| ID | Chỉ số | Cách đo | Mục tiêu |
|---|---|---|---|
| NFR-PERF-01 | API latency | p50/p95/p99 theo endpoint và tải | TBD |
| NFR-PERF-02 | Booking acceptance time | Từ lúc gửi yêu cầu đến xác nhận tiếp nhận | TBD |
| NFR-PERF-03 | Dispatch decision time | Từ lúc yêu cầu đủ điều kiện điều phối đến kết quả candidate/offer | TBD |
| NFR-PERF-04 | Driver location freshness | Tuổi dữ liệu GPS tại thời điểm hiển thị/ra quyết định | TBD |
| NFR-PERF-05 | Notification delivery latency | Thời gian phát lệnh đến provider và trạng thái provider trả về | TBD |
| NFR-PERF-06 | Trip/Fare completion | Thời gian hoàn tất tác vụ và tính cước | TBD |
| NFR-PERF-07 | Report query/export | Thời gian truy vấn và xuất theo khối lượng dữ liệu | TBD |
| NFR-PERF-08 | Database query latency | p95/p99 theo nhóm truy vấn | TBD |

### 4.2. Quy tắc đo

- Đo riêng thời gian xử lý nội bộ và thời gian phụ thuộc nhà cung cấp bên ngoài.
- Báo cáo p50, p95, p99 và tỷ lệ timeout/error.
- Phân nhóm theo API, Module, kênh, chi nhánh, DIRECT/PARTNER và giờ cao điểm.
- Không gộp API đọc nhanh với tác vụ báo cáo nặng thành một số trung bình.
- Ghi nhận tải nền, số lượng worker, tài nguyên và trạng thái dependency khi benchmark.

### 4.3. Tối ưu có kiểm soát

- Phân trang và giới hạn kích thước response.
- Index theo truy vấn thực tế, kiểm tra execution plan.
- Cache dữ liệu tham chiếu phù hợp; không cache trạng thái nhạy cảm quá thời hạn nghiệp vụ.
- Tách truy vấn báo cáo nặng khỏi giao dịch vận hành khi cần.
- Dùng queue cho tác vụ bất đồng bộ; hiển thị trạng thái xử lý.
- Không cache hoặc tối ưu theo cách làm sai lệch quyền truy cập/Data Scope.

---

## 5. Capacity Planning — Năng lực và tải

### 5.1. Dữ liệu đầu vào bắt buộc phải khảo sát

| Nhóm | Dữ liệu cần thu thập |
|---|---|
| Nhu cầu khách | Booking/ngày, giờ cao điểm, tỷ lệ theo App/Web/Call/SMS/Chat |
| Tài xế | Tổng tài xế, online đồng thời, số lần cập nhật GPS/phút |
| Điều phối | Số Booking chờ, số Offer đồng thời, số Dispatcher hoạt động |
| Chuyến đi | Trip/ngày, thời lượng trung bình, lịch sử GPS mỗi Trip |
| Tài chính | Payment/Refund/Settlement mỗi ngày, khối lượng batch |
| Báo cáo | Người dùng đồng thời, truy vấn và export lớn nhất |
| Tích hợp | Request rate, quota, latency và giới hạn của provider |
| Tăng trưởng | Tăng trưởng theo tháng/năm, mở rộng địa bàn và đối tác |

### 5.2. Kịch bản tải kiểm thử

1. **Baseline:** tải vận hành bình thường được đo từ dữ liệu hiện trạng.
2. **Peak:** tải cao điểm thực tế hoặc dự báo đã được phê duyệt.
3. **Burst:** Booking/GPS/notification tăng nhanh trong khoảng thời gian ngắn.
4. **Stress:** tăng tải đến khi hệ thống suy giảm; xác định giới hạn và hành vi.
5. **Soak:** duy trì tải dài giờ để phát hiện memory leak, queue backlog và connection leak.
6. **Failover under load:** mất một node hoặc dependency trong lúc tải cao.
7. **Recovery surge:** lượng request dồn lại sau khi hệ thống phục hồi.

### 5.3. Chỉ tiêu Capacity cần chốt

- Số Booking đồng thời và Booking/giây.
- Số tài xế online và GPS update/giây.
- Số Offer/DispatchRun đồng thời.
- Số Trip hoạt động đồng thời.
- Concurrent user theo từng nhóm.
- Queue throughput và backlog tối đa chấp nhận được.
- Dung lượng DB, object storage, log và GPS theo ngày/tháng.
- Dự báo tăng trưởng 12/24/36 tháng.

Tất cả hiện là TBD đến khi có baseline và mô hình tăng trưởng.

---

## 6. Scalability — Khả năng mở rộng

- Có thể mở rộng ngang các thành phần stateless khi tải tăng.
- Tách workload theo đặc tính: API giao dịch, dispatch engine, GPS ingestion, notification, reporting và batch finance.
- Worker có thể tăng/giảm số lượng mà không xử lý trùng hoặc làm mất message.
- Queue phải có back-pressure, retry policy, dead-letter handling và cơ chế replay có kiểm soát.
- Partition/sharding dữ liệu chỉ quyết định sau khi có số liệu và thiết kế truy vấn.
- Cache phải có chiến lược invalidation và giới hạn stale data.
- Mở rộng chi nhánh/đối tác không được yêu cầu nhân bản toàn bộ hệ thống nếu mô hình multi-tenant/shared platform đã được chọn.
- DIRECT/PARTNER phải giữ cách ly dữ liệu và tài chính khi scale.

---

## 7. Availability & Reliability — Sẵn sàng và tin cậy

### 7.1. Phân loại mức độ quan trọng

| Cấp | Nhóm dịch vụ | Ảnh hưởng khi gián đoạn |
|---|---|---|
| Critical | Booking intake, Dispatch, Driver assignment, Trip state | Ngừng hoặc ảnh hưởng trực tiếp hoạt động điều xe |
| High | GPS ingestion, notification, customer service | Giảm chất lượng giám sát/liên lạc |
| High | Fare, Payment capture, finance event | Ảnh hưởng thu tiền và đối soát |
| Medium/High | Reporting và analytics | Có thể chấp nhận trễ theo kỳ, tùy báo cáo |
| Supporting | Admin/configuration, catalog | Ảnh hưởng quản trị, không nhất thiết dừng toàn bộ vận hành |

Phân loại trên là đề xuất sơ bộ. Business Impact Analysis (BIA) phải xác nhận mức độ và phụ thuộc.

### 7.2. Chỉ số

- Availability theo từng dịch vụ, không chỉ toàn hệ thống.
- Error rate và tỷ lệ request thành công.
- Mean Time To Detect (MTTD).
- Mean Time To Recover/Restore (MTTR).
- Queue backlog age và message loss/duplicate.
- Database replication lag và failover time.
- Tỷ lệ giao dịch nghiệp vụ không thể hoàn tất.

### 7.3. Nguyên tắc độ tin cậy

- Health check phân biệt liveness và readiness.
- Có graceful degradation khi dependency không thiết yếu bị lỗi.
- Timeout, retry có giới hạn và exponential backoff có jitter.
- Circuit breaker với dependency không ổn định.
- Không retry mù quáng các lệnh không idempotent.
- Có reconciliation job cho trạng thái lệch giữa các hệ thống.
- Có quy trình vận hành thủ công dự phòng được phê duyệt cho tình huống mất hệ thống.

---

## 8. High Availability — HA và dự phòng

### 8.1. Thành phần cần phân tích HA

- Load balancer / API entry.
- Ứng dụng Booking, Dispatch, Trip.
- Database và cơ chế replication/failover.
- Message broker/queue.
- Cache/session store.
- GPS ingestion và location store.
- Notification gateway/worker.
- Object/file storage.
- Monitoring, logging và backup.

### 8.2. Nguyên tắc thiết kế

- Loại bỏ single point of failure ở thành phần được phân loại Critical/High.
- Có health check và cơ chế loại node lỗi khỏi traffic.
- Kiểm tra split-brain, replication lag và consistency khi failover DB.
- Có fencing/leader election phù hợp cho worker hoặc scheduler chỉ được chạy một lần.
- Có khả năng khôi phục queue và xử lý message tồn.
- Cấu hình, secret, certificate và backup cũng phải có dự phòng.
- HA không thay thế Backup/DR; replication có thể nhân bản cả lỗi hoặc xóa nhầm.

Không chốt công nghệ HA cụ thể (cluster, managed service, multi-zone hay multi-region) trước khi có yêu cầu RTO/RPO, hạ tầng, ngân sách và năng lực vận hành.

---

## 9. Backup, Disaster Recovery và Business Continuity

### 9.1. Định nghĩa

- **RPO:** lượng dữ liệu tối đa có thể mất tính theo thời gian.
- **RTO:** thời gian mục tiêu để khôi phục dịch vụ.
- **DR:** khôi phục sau thảm họa lớn.
- **BCP:** duy trì hoạt động kinh doanh ở mức chấp nhận được khi hệ thống chính gián đoạn.

### 9.2. Bảng mục tiêu cần phê duyệt

| Nhóm dữ liệu/dịch vụ | RPO mục tiêu | RTO mục tiêu | Owner |
|---|---:|---:|---|
| Booking/Dispatch/Assignment | TBD | TBD | Vận hành/CNTT |
| Trip/GPS | TBD | TBD | Vận hành/CNTT |
| Fare/Payment | TBD | TBD | Tài chính/CNTT |
| Ledger/Settlement | TBD | TBD | Tài chính/CNTT |
| Customer/Driver/Partner master | TBD | TBD | Chủ dữ liệu |
| Audit/Security logs | TBD | TBD | ATTT |
| Reporting/Analytics | TBD | TBD | Chủ báo cáo |

### 9.3. Yêu cầu Backup

- Xác định backup full/incremental/log theo công nghệ được chọn.
- Mã hóa backup và kiểm soát quyền truy cập.
- Có bản sao tách biệt khỏi môi trường production; cân nhắc immutable/offline copy.
- Kiểm tra tính toàn vẹn và khả năng restore.
- Có runbook khôi phục DB, object, cấu hình, secret và tích hợp.
- Ghi nhận thời điểm, phạm vi, kết quả và người thực hiện.
- Thử nghiệm restore định kỳ; backup thành công không đồng nghĩa restore thành công.

### 9.4. Kịch bản DR/BCP tối thiểu

1. Mất một máy chủ/VM/node.
2. Mất một vùng hạ tầng hoặc trung tâm dữ liệu.
3. DB lỗi, corruption hoặc thao tác xóa nhầm.
4. Ransomware hoặc credential bị chiếm.
5. Mất kết nối Internet/đường truyền tại trung tâm điều hành.
6. Nhà cung cấp bản đồ/GPS/SMS/Payment gián đoạn.
7. Mất điện hoặc hệ thống tòa nhà ảnh hưởng phòng máy.
8. Queue backlog hoặc message broker hỏng.
9. Nhân sự vận hành chủ chốt không sẵn sàng.

---

## 10. Network Degradation và Offline Operation

Taxi có tài xế di chuyển liên tục nên mạng di động không ổn định là điều kiện vận hành bình thường cần thiết kế trước.

- Driver App phải phân biệt Online, Degraded/Offline và Reconnecting.
- Dữ liệu được phép lưu offline phải giới hạn và bảo vệ.
- Mỗi command/event có ID duy nhất, timestamp, sequence/version và idempotency strategy.
- Đồng bộ lại phải xử lý duplicate, out-of-order và stale event.
- Quy tắc ưu tiên dữ liệu khi xung đột phải được định nghĩa theo từng loại dữ liệu; không dùng một chính sách chung cho mọi state.
- Không cho phép offline client tự xác nhận các trạng thái cần kiểm chứng server-side nếu chưa có policy.
- Hiển thị rõ cho tài xế và điều hành dữ liệu vị trí cuối cùng cùng tuổi dữ liệu.
- Có hướng dẫn tác nghiệp dự phòng khi mất mạng kéo dài.

---

## 11. Observability — Giám sát, Logging, Tracing và Alerting

### 11.1. Ba trụ cột

- **Metrics:** latency, throughput, error rate, resource, queue lag, GPS freshness, DB lag.
- **Logs:** sự kiện ứng dụng, lỗi, thay đổi cấu hình, audit/security; có correlation_id.
- **Traces:** theo dõi một giao dịch xuyên Booking → Dispatch → Trip → Fare → Payment.

### 11.2. Dashboard tối thiểu

| Dashboard | Chỉ số chính |
|---|---|
| Business Operations | Booking mới/chờ/hủy, Assignment, Trip đang chạy, hoàn tất |
| Dispatch Health | Candidate count, dispatch latency, Offer timeout, queue backlog |
| Driver/GPS | Driver online, GPS freshness, thiết bị offline, vùng mất tín hiệu |
| Platform Health | CPU, RAM, disk, network, process, container/node |
| Database | Connections, slow query, replication lag, lock/deadlock, storage |
| Integration | API success/error, provider latency, webhook failures, quota |
| Finance Pipeline | Payment pending/failed, reconciliation mismatch, settlement batch |
| Security | Failed login, privilege changes, suspicious access, audit pipeline |
| DR/Backup | Backup status, age, restore test, replication health |

### 11.3. Quy tắc cảnh báo

- Cảnh báo phải có mức độ, owner, điều kiện kích hoạt, thời gian duy trì và hướng xử lý.
- Phân biệt cảnh báo kỹ thuật và cảnh báo nghiệp vụ.
- Không gửi alert lặp vô hạn; có deduplication, grouping và escalation.
- Critical alert phải có cơ chế xác nhận tiếp nhận.
- Có runbook liên kết từng loại cảnh báo.
- Tránh đưa PII, token hoặc dữ liệu nhạy cảm vào alert.

---

## 12. Data Volume, Retention và Reporting

- Ước tính riêng dung lượng Booking, Trip, GPS point, event, audit, recording, invoice và attachment.
- GPS có thể tạo dữ liệu lớn; cần xác định tần suất thu thập, độ chính xác, retention, archive và khả năng truy vấn.
- Tách workload phân tích/báo cáo khỏi giao dịch vận hành nếu có nguy cơ tranh chấp tài nguyên.
- Báo cáo phải nêu thời điểm dữ liệu cập nhật gần nhất và phạm vi dữ liệu.
- Export lớn phải chạy nền, có trạng thái, giới hạn và quyền truy cập.
- Chính sách retention/xóa/ẩn danh theo loại dữ liệu phải được Pháp chế, ATTT và Data Owner phê duyệt.
- Không tự đặt thời hạn lưu GPS, ghi âm, hóa đơn hoặc Audit khi chưa có căn cứ pháp lý/nghiệp vụ.

---

## 13. Maintainability, Deployability và Change Management

- Module có ranh giới sở hữu dữ liệu và hợp đồng API/Event rõ ràng.
- Có automated test cho business rules, state transition, authorization và integration contract.
- Có CI/CD, code review, dependency scanning và quản lý phiên bản.
- Cấu hình môi trường tách khỏi mã nguồn; secret quản lý riêng.
- Migration DB có kế hoạch backward compatibility, backup và rollback/forward-fix.
- Release có version, changelog, người phê duyệt và kế hoạch rollback.
- Feature flag chỉ dùng khi có owner, ngày hết hạn và kiểm soát.
- Có runbook vận hành, sơ đồ phụ thuộc, tài liệu khôi phục và hướng dẫn hỗ trợ.
- Có khả năng truy vết từ yêu cầu NFR đến test và bằng chứng nghiệm thu.

---

## 14. Compatibility và Accessibility

- Xác định danh sách thiết bị, phiên bản Android/iOS và trình duyệt được hỗ trợ.
- Xác định cấu hình tối thiểu cho máy tính tổng đài/điều hành.
- Kiểm thử trên thiết bị di động có màn hình nhỏ, cấu hình thấp và mạng yếu.
- Giao diện phải thể hiện rõ trạng thái, lỗi và hành động tiếp theo.
- Màu sắc không phải phương tiện duy nhất để phân biệt trạng thái.
- Hỗ trợ ngôn ngữ và định dạng ngày/giờ/tiền tệ theo thị trường triển khai.
- Các tiêu chí accessibility cụ thể cần xác định theo nhóm người dùng và nền tảng.

---

## 15. NFR Traceability và Test Strategy

### 15.1. Loại kiểm thử

| Loại test | Mục tiêu |
|---|---|
| Load Test | Đo tải dự kiến và giờ cao điểm |
| Stress Test | Tìm giới hạn và cách suy giảm |
| Spike Test | Đánh giá tăng tải đột ngột |
| Soak Test | Phát hiện lỗi tích lũy dài hạn |
| Failover Test | Xác minh chuyển đổi khi node/dependency lỗi |
| DR Test | Xác minh RTO/RPO và khả năng khôi phục |
| Chaos/Resilience Test | Kiểm tra hành vi khi dependency lỗi có kiểm soát |
| Security Test | Xác minh quyền, lỗ hổng và bảo vệ dữ liệu |
| Recovery/Replay Test | Xử lý message tồn, duplicate và event đến muộn |
| Usability Test | Kiểm tra tác vụ thực tế của tổng đài/điều hành/tài xế |

### 15.2. Bằng chứng nghiệm thu

Mỗi NFR cần:
- Test ID và môi trường test.
- Phiên bản phần mềm/cấu hình.
- Dataset và tải mô phỏng.
- Kết quả thô, biểu đồ và percentile.
- Error rate, resource utilization và dependency status.
- So sánh với ngưỡng đã phê duyệt.
- Defect, ngoại lệ và chấp thuận rủi ro.
- Người thực hiện, người review và người phê duyệt.

### 15.3. Ma trận truy vết mẫu

| NFR ID | Module/luồng | Test | Bằng chứng |
|---|---|---|---|
| NFR-PERF-02 | M01 Booking | Load/latency test | Báo cáo p95/p99 |
| NFR-PERF-03 | M03 Dispatch | Peak + concurrency test | Dispatch latency, conflict rate |
| NFR-REL-01 | M01/M03/M04 | Retry/idempotency test | Không tạo giao dịch trùng |
| NFR-AVAIL-01 | M01/M03/M04 | Node failure test | Thời gian gián đoạn |
| NFR-DR-01 | Data Platform | Restore/DR exercise | RTO/RPO thực tế |
| NFR-OBS-01 | Toàn hệ thống | Alert simulation | Alert và runbook evidence |
| NFR-SEC-01 | X-04 IAM | Authorization test | Không truy cập chéo scope |
| NFR-DATA-01 | GPS/Trip/Audit | Retention/archive test | Báo cáo lưu trữ và truy xuất |

---

## 16. Danh mục quyết định cần xác nhận

| ID | Nội dung | Owner đề xuất |
|---|---|---|
| NFR-01 | Số Booking/ngày và giờ cao điểm theo từng kênh | Vận hành |
| NFR-02 | Số tài xế online đồng thời và GPS update rate | Vận hành |
| NFR-03 | Số chi nhánh, khu vực, đối tác và tốc độ mở rộng | Kinh doanh/Vận hành |
| NFR-04 | SLO latency cho Booking, Dispatch, Trip, Payment | Business/CNTT |
| NFR-05 | Mức availability theo Critical/High/Medium service | Ban điều hành/CNTT |
| NFR-06 | RTO/RPO theo nhóm dữ liệu và dịch vụ | BCP/Business/CNTT |
| NFR-07 | Hạ tầng hiện tại, DC/Cloud, network và ngân sách | CNTT |
| NFR-08 | Mức dự phòng HA: node, zone, site hay region | CNTT |
| NFR-09 | Tần suất và retention GPS | Vận hành/Pháp chế |
| NFR-10 | Retention ghi âm, Audit, Booking, Trip và tài chính | Pháp chế/Data Owner |
| NFR-11 | Khối lượng báo cáo và thời gian chấp nhận được | Business/BI |
| NFR-12 | Nhà cung cấp bản đồ, SMS, thoại, Payment và quota | CNTT/Kinh doanh |
| NFR-13 | Kịch bản offline và tác nghiệp thủ công | Vận hành |
| NFR-14 | Thiết bị/OS/browser hỗ trợ | Sản phẩm/CNTT |
| NFR-15 | Mức tăng trưởng dự báo 12/24/36 tháng | Ban điều hành/Kinh doanh |
| NFR-16 | Thời gian bảo trì và cửa sổ triển khai | Vận hành/CNTT |
| NFR-17 | Mục tiêu phục hồi sau ransomware và mất DC | ATTT/BCP |
| NFR-18 | Ngưỡng backlog, retry, dead-letter và replay | Kiến trúc/CNTT |

---

## 17. Tiêu chí nghiệm thu NFR

- [ ] Mỗi dịch vụ được phân loại Critical/High/Medium và có Business Owner.
- [ ] Có baseline tải hiện trạng và dự báo tăng trưởng.
- [ ] Có SLI/SLO và ngưỡng p95/p99 được phê duyệt.
- [ ] Có kết quả Load, Stress, Spike và Soak Test.
- [ ] Có đánh giá Capacity cho Booking, Dispatch, GPS, Trip, Finance và Reporting.
- [ ] Có HA design và kết quả failover test cho thành phần trọng yếu.
- [ ] Có RTO/RPO được phê duyệt và kết quả DR/Restore Test.
- [ ] Có phương án offline/degraded mode và diễn tập.
- [ ] Có Dashboard, Alert, Logs, Traces và Runbook.
- [ ] Có kiểm soát Data Retention, Archive và Restore.
- [ ] Có kiểm thử bảo mật và phân quyền liên kết X-04.
- [ ] Có quy trình release, rollback, migration và change management.
- [ ] Các quyết định NFR-01 đến NFR-18 có owner, approver và trạng thái.

---

## 18. Lộ trình thực hiện

1. Thu thập số liệu vận hành thực tế: Booking, tài xế online, GPS, Trip, giao dịch và báo cáo.
2. Khảo sát hạ tầng hiện tại, network, DC/Cloud, database, tích hợp và năng lực đội vận hành.
3. Thực hiện BIA để phân loại Critical/High/Medium.
4. Xác định SLI/SLO, RTO/RPO và capacity target.
5. Thiết kế HA, Backup/DR, Monitoring và Offline Operation.
6. Xây dựng Performance/Resilience Test Plan.
7. Chạy PoC và benchmark các thành phần rủi ro cao, đặc biệt Dispatch, GPS và dữ liệu thời gian thực.
8. Chốt NFR baseline trước khi phê duyệt thiết kế triển khai và hạ tầng production.

**Nguyên tắc chốt:** Không tuyên bố hệ thống “đủ tải”, “high availability” hoặc “có DR” chỉ vì đã cài cluster hay backup. Chỉ được công nhận khi có mục tiêu được phê duyệt, kịch bản kiểm thử và bằng chứng phục hồi/đo tải thực tế.

---
**Quay lại:** [README — Taxi Management System](../README.md)
