# X-04 — BẢO MẬT, QUYỀN RIÊNG TƯ VÀ KIỂM TOÁN
**Version:** 0.1 · **Status:** DRAFT · **Owner:** Security/Legal/IT
## 1. Nguyên tắc
Least privilege, deny by default, separation of duties, explicit data scope, encryption in transit/at rest, secure secrets, defense in depth, auditable administration.
## 2. Identity/access
Authentication/session; RBAC by module/action plus ABAC/data scope where needed. Scope dimensions: legal entity, branch, zone, fleet type, partner, resource owner. Privileged/MFA/break-glass requires time-bound control and audit. Maker-checker for sensitive finance. Service identities separate from human users.
## 3. Phân loại dữ liệu
Public, Internal, Confidential, Restricted là phân loại đề xuất. PII, precise location, identity documents, call recordings, payment references và incident evidence cần kiểm soát theo phân loại và quy định áp dụng.
## 4. Audit
Capture actor/service, action, target, old/new state where permitted, result, reason, source/device, timestamps, IP/session and correlation_id. Protect from ordinary update/delete. Avoid secrets and unnecessary PII in logs.
## 5. Privacy lifecycle
Purpose limitation, minimization, consent/legal basis, retention, access/correction/deletion, masking/anonymization, backup expiry and legal hold. Legal controls and retention periods require Legal approval.
## 6. Threats and acceptance
IDOR/BOLA, privilege escalation, session/token theft, injection, XSS/CSRF/SSRF, insecure upload, webhook replay, rate abuse, location leakage, insider risk, supply chain and secret exposure. Require threat model, access matrix, security tests, audit verification, restore and incident response.
