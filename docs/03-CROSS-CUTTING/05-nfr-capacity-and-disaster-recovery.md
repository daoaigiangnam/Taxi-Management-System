# X-05 — NFR, NĂNG LỰC TẢI VÀ DISASTER RECOVERY
**Version:** 0.1 · **Status:** DRAFT · **Owner:** Architecture/IT Operations
## 1. NFR categories
Availability; latency; throughput; concurrent drivers/agents; GPS ingest; dispatch decision time; durability; consistency; recovery; security; privacy; accessibility; localization; observability; maintainability; interoperability.
## 2. SLO/SLA
Không giả định target số. Mỗi target cần service boundary, measurement window, percentile, exclusions, owner và report source. Target TBD sau khi có volume và criticality baseline.
## 3. Capacity model
Ước tính theo chi nhánh/khu vực/giờ cao điểm: bookings/sec; concurrent open bookings; online drivers/GPS updates/sec; dispatch runs/candidates/offer fanout; trip events/sec; payment callbacks/financial postings; map API quota; dashboard concurrency/query load; data/audit growth. Kiểm thử baseline, peak, stress, soak, failover và recovery.
## 4. Resilience
Define timeout, retry, circuit breaker, bulkhead, durable queue, backpressure, degraded mode, regional isolation and manual fallback. Never blindly retry non-idempotent financial/dispatch commands.
## 5. Backup/DR
Set RTO/RPO by service/data class; backup frequency, encryption, offsite/immutable copies, restore test, failover authority, DNS/network dependencies and runbook. Numeric RTO/RPO TBD.
## 6. Observability
Structured logs, metrics, traces, business events, correlation ID, freshness, SLO dashboards, alert ownership and incident review. Avoid PII/secrets in telemetry.
