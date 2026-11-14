# M01 — CUSTOMER & BOOKING
**Status:** DRAFT · **Owner:** Product/Booking PO (TBD) · [Back to Overview](../../../README.md)

## Purpose
Quản lý khách hàng và yêu cầu đặt xe đa kênh; cung cấp Booking hợp lệ cho Dispatch. Không sở hữu Offer, Assignment hoặc Trip.

## Entities
Customer, ContactPoint, Booking, BookingStop, BookingRevision, QuoteReference.

## Feature Catalog
| ID | Feature | Priority | Spec |
|---|---|---|---|
| CUS-001 | Customer profile/search | P1 | Planned |
| [BKG-001](../../docs/05-FEATURE-SPECS/M01-customer-and-booking/BKG-001-create-immediate-booking.md) | Create immediate booking | P0 | Draft v0.1 |
| BKG-002 | Create scheduled booking | P1 | Planned |
| [BKG-003](../../docs/05-FEATURE-SPECS/M01-customer-and-booking/BKG-003-validation-and-serviceability.md) | Validation/serviceability | P0 | Draft v0.1 |
| BKG-004 | Pickup/destination/ordered stops | P0 | Planned |
| BKG-005 | Booking revision/change request | P1 | Planned |
| BKG-006 | Cancel booking/policy | P1 | Planned |
| BKG-007 | Duplicate detection/warning | P2 | Planned |
| BKG-008 | Search/detail/timeline | P0 | Planned |
| BKG-009 | Quote request/reference | P1 | Planned |

## Workflow
Capture → validate → resolve customer → normalize location → save → quote/confirm per policy → dispatch readiness. Handoff via API/event contract.

## Rules
Channel independent of fleet type; stable ID/reference/version; edits create revisions; cancel is guarded; no candidate does not delete Booking.

## Dependencies
IAM; location/geocoding; service catalog; Pricing; Notification; Dispatch.

## Open decisions
Channel-specific fields; guest/customer matching; immediate/scheduled cutoffs; service zones; duplicate window; revise/cancel policy; quote validity.
