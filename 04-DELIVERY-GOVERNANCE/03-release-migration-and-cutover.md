# G-03 — RELEASE, MIGRATION VÀ CUTOVER
**Version:** 0.1 · **Status:** DRAFT · **Owner:** Release Manager/IT Operations
## Environment
Separate dev/test/UAT/staging/production with controlled secrets, masked data and promotion gates. Topology/tooling TBD.
## Release lifecycle
Scope freeze → migration rehearsal → deployment rehearsal → UAT → operational readiness → go/no-go → cutover → hypercare → closure.
## Migration
Source inventory; ownership; mapping/transformation; duplicate resolution; ID crosswalk; status mapping; PII controls; financial reconciliation; dry run; delta sync; freeze; cutover validation.
## Rollback/forward recovery
Define trigger, authority, write freeze, compatibility, restore point, DNS/integration switching, customer comms. Some financial/dispatch operations need forward recovery, not blind DB rollback.
## Readiness
Monitoring/alerts; backup/restore; DR runbook; capacity; security; vendor support; on-call; support scripts; reconciliation; feature flags; incident comms; rollback plan.
