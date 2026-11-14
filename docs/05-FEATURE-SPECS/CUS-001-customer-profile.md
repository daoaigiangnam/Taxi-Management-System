# CUS-001 — CUSTOMER PROFILE / SEARCH
**Module:** M01 Customer & Booking · **Version:** 0.1 · **Status:** DRAFT · **Priority:** P1 proposal  
**Owner:** Customer Product Owner (TBD) · [M01](../02-DOMAIN-MODULES/01-customer-and-booking/README.md) · [Overview](../../README.md)

## 1. Objective
Cung cấp hồ sơ khách hàng tối thiểu, tìm kiếm/nhận diện khi đặt xe, giảm nhập lặp và truy xuất lịch sử theo quyền.

## 2. Actors and permissions
Call Center Agent, Customer self-service, authorized CSKH, system integration. Role and data scope must be explicit; deny by default.

## 3. Scope
Create/search/view/update customer profile and contact points; duplicate candidate warnings; consent/preferences where applicable. Merge, delete/anonymize and identity verification rules are TBD.

## 4. Main flow
Search by allowed identifier → display scoped matches → verify minimum details → select existing profile or create new → validate contact → save with audit → return customer_id to Booking.

## 5. Validation and business rules
Normalize phone/email; avoid exposing full PII in search results. Dedup uses configurable matching and human review; never auto-merge from weak match. Customer visibility must respect legal entity/branch/partner scope.

## 6. Data
Customer(id, status, display_name, created_at, updated_at, version); ContactPoint(id, customer_id, type, normalized_value, verification_status, is_primary). Consent/Preference model if legally required. Physical schema TBD.

## 7. API (logical)
Search customers; create customer; get detail with field masking; update profile/contact using expected_version. Apply authorization, validation, idempotency for create, audit and correlation envelope. Exact OpenAPI paths TBD.

## 8. UI states
Search: idle/loading/results/empty/invalid/error/denied. Create/edit: pristine/dirty/validating/saving/saved/conflict. Mask sensitive values and prevent cross-scope disclosure.

## 9. Tests and acceptance
- Authorized agent finds customer by permitted identifier.
- Unauthorized branch/partner cannot enumerate another scope.
- Duplicate candidate warns but does not auto-merge.
- Invalid normalized contact rejected with field errors.
- Concurrent edits cause version conflict, not silent overwrite.
- Audit has actor/action/result/correlation; PII/secrets not leaked to logs.

## 10. Dependencies and TBD
IAM/data scope; channel-specific identifiers; privacy basis/retention; verification; dedup; profile merge; customer ID/reference display policy.
