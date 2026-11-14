# M08 — PLATFORM, IAM & AUDIT
**Status:** DRAFT · **Owner:** IT/Security/Architecture (TBD) · [Back to Overview](../../../README.md)

## Purpose and entities
Nền tảng dùng chung cho authentication, authorization, data scope, configuration, audit và operations. Entities: User, Identity, Role, Permission, RoleBinding, OrgUnit, Branch, Zone, DataScope, PolicyConfig, AuditEvent, SystemSetting, SecretReference.

## Feature Catalog
| ID | Feature | Priority |
|---|---|---|
| IAM-001 | Authentication/session/device | P0 |
| IAM-002 | RBAC/scoped authorization | P0 |
| IAM-003 | Org/branch/zone/partner scope | P0 |
| IAM-004 | MFA/privileged access | P1 |
| AUD-001 | Audit immutable history | P0 |
| CFG-001 | Versioned configuration/policy | P0 |
| OPS-001 | Health/logs/metrics/alerts foundation | P0 |

## Rules
Deny by default. Every request checks actor/action/resource/scope. Privileged controls and audit. No secrets in source. Audit captures actor/action/resource/result/source/time/correlation.

## Open decisions
IdP, SSO/MFA, organization hierarchy, delegation, break-glass, retention, SIEM and segregation of duties.
