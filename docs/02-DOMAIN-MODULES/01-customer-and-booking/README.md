# M01 — CUSTOMER & BOOKING

**Status:** DRAFT · **Owner:** Product/Booking PO (TBD)  
**Overview:** [Project Overview](../../../00-OVERVIEW.md) · [E2E Flow](../../../01-FOUNDATION/03-end-to-end-business-processes.md)

## 1. Purpose

Quản lý khách hàng, thông tin liên hệ và yêu cầu đặt xe đa kênh; cung cấp Booking hợp lệ cho Dispatch. M01 không sở hữu Offer, Assignment hay Trip.

## 2. Scope
Customer profile/contact preferences; booking intake; immediate/scheduled requests; pickup/destination/stops; service/passenger/luggage instructions; validation; quote reference; revision; duplicate warning; cancel/change; customer communications trigger.

## 3. Ownership
Entities: Customer, ContactPoint, SavedPlace (if approved), Booking, BookingStop, BookingRevision. Owner of Booking state, version and public reference.

## 4. Core feature catalog

| Feature ID | Feature | Priority proposal | Detail spec |
|---|---|---|---|
| CUS-001 | Customer search/create/update | P1 | [CUS-001](../../../05-FEATURE-SPECS/CUS-001-customer-profile.md) |
| BKG-001 | Create immediate booking | P0 | [BKG-001](../../../05-FEATURE-SPECS/BKG-001-create-immediate-booking.md) |
| BKG-002 | Create scheduled booking | P1 | Planned |
| BKG-003 | Booking validation and serviceability | P0 | Planned |
| BKG-004 | Pickup, destination and ordered stops | P0 | Planned |
| BKG-005 | Booking revision/change request | P1 | Planned |
| BKG-006 | Cancel booking and cancellation policy | P1 | Planned |
| BKG-007 | Duplicate detection/warning | P2 | Planned |
| BKG-008 | Booking search, timeline and detail | P0 | Planned |
| BKG-009 | Quote request/reference integration | P1 | Planned |

## 5. Main process
Capture → validate → resolve customer → normalize location → save draft/request → confirmation/quote according to policy → ready for dispatch. Dispatch handoff occurs through a contract, not direct table access.

## 6. Rules
- Channel is independent of fleet type.
- A booking must have a stable ID, public reference, version and audit trail.
- Changes create revisions; do not overwrite history.
- Booking cancellation is state/policy/permission guarded.
- No candidate does not delete or automatically cancel the booking.

## 7. Dependencies
Platform/IAM; location/geocoding; service catalog; Pricing quote; Notification; Dispatch handoff.

## 8. Open decisions
Mandatory fields by channel, customer identity matching, guest booking policy, immediate/scheduled cutoffs, service zones, duplicate window, change/cancel rules and quote validity are TBD.
