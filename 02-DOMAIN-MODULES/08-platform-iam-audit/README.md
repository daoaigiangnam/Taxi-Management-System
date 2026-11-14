# M08 — PLATFORM, IAM & AUDIT

**Status:** DRAFT · **Owner:** IT/Security/Architecture (TBD)  
**Overview:** [Project Overview](../../../00-OVERVIEW.md)

## Purpose
Cung cấp nền tảng dùng chung để mọi Module xác thực, phân quyền, giới hạn phạm vi dữ liệu, cấu hình và kiểm toán.

## Entities
User, Identity, Role, Permission, RoleBinding, OrgUnit, Branch, Zone, DataScope, PolicyConfig, AuditEvent, SystemSetting, SecretReference.

## Features
| ID | Feature | Priority | Detail |
|---|---|---|---|
| IAM-001 | Authentication/session/device | P0 | Planned |
| IAM-002 | RBAC and scoped authorization | P0 | Planned |
| IAM-003 | Organization/branch/zone/partner data scope | P0 | Planned |
| IAM-004 | MFA and privileged access | P1 | Planned |
| AUD-001 | Audit event and immutable history | P0 | Planned |
| CFG-001 | Versioned configuration/policy | P0 | Planned |
| OPS-001 | Health, logs, metrics and alerting foundation | P0 | Planned |

## Rules
Deny by default; every request checks actor, action, resource and scope. Privileged actions need stronger controls. Secrets are never stored in source code. Audit captures actor, action, resource, result, source, timestamp and correlation.

## Open decisions
Identity provider, SSO/MFA, branch hierarchy, delegation, break-glass, log retention, SIEM and segregation of duties are TBD.
