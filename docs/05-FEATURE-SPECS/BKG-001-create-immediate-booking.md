# BKG-001 — CREATE IMMEDIATE BOOKING
**Module:** M01 Customer & Booking · **Version:** 0.1 · **Status:** DRAFT · **Priority:** P0 proposal  
**Owner:** Booking Product Owner (TBD) · [M01](../02-DOMAIN-MODULES/01-customer-and-booking/README.md) · [Overview](../../README.md)

## 1. Objective and user story
As a Call Center Agent or authorized channel, create a valid immediate taxi request and hand it to Dispatch with traceable customer, location, service and timing data.

## 2. Actors and permissions
Call Center Agent creates within assigned organization scope; Customer App/Web creates own request; Partner/API client only within authorized partner scope if enabled; Supervisor handles exception under approved policy. Exact role codes TBD.

## 3. Scope
Immediate booking capture, validation, customer association, pickup/destination/stops, service/vehicle request, passenger/luggage instructions, confirmation, public reference and dispatch handoff. Scheduled bookings, tariff administration and candidate selection are separate features.

## 4. Preconditions
Actor authenticated; channel authorized; service catalog and serviceability configuration available; customer/contact policy satisfied; pickup resolved sufficiently; required fields by channel configured.

## 5. Main flow
1. Open create booking.
2. Search/select customer or create minimal profile if policy permits.
3. Select pickup via address search/map pin and confirm landmark/instructions.
4. Enter destination and ordered stops where enabled.
5. Select service category, capacity and special assistance.
6. Validate formats, service area, zone, schedule and required data.
7. Show quote/estimate only if Pricing policy/provider is available; store quote reference/version.
8. Review summary and confirm.
9. Server creates Booking with stable ID, public reference, channel, state, version, requested time, customer, stops, actor/source and audit.
10. Publish BookingReadyForDispatch only after commit and readiness guards.
11. UI shows accepted reference/current state; never claim vehicle assigned before Assignment exists.

## 6. Alternate and exception flows
- Customer unknown: create minimal profile if policy permits; otherwise request details.
- Address ambiguous: require pin/landmark confirmation.
- Outside service area: reject or manual review per policy; do not silently dispatch.
- Duplicate candidate: warn; authorized actor decides; do not auto-delete.
- Quote unavailable: allow/block per business policy (TBD).
- Dispatch unavailable: persist valid booking and show queued/pending if durable handoff is supported; never silently lose accepted request.
- Double submit: idempotency prevents duplicate Booking.
- Concurrent edit: optimistic version conflict and reload/merge.
- Customer cancellation during submission: resolve command order against current server state.

## 7. Field and validation matrix
| Field | Required | Validation |
|---|---|---|
| channel | Yes | Server-controlled enum/channel identity |
| customer_id | Policy | Existing scoped customer or approved guest flow |
| contact_name/phone | Policy | Normalize, mask and validate |
| pickup address/coordinates | Yes | Address/pin consistency and serviceability |
| destination | Policy | Unknown destination only if approved |
| stops[] | Optional | Ordered, unique sequence and service rules |
| service_type | Yes | Active service catalog/capacity compatible |
| passenger_count | Yes | Positive and within capacity |
| luggage/special needs | Optional | Supported capability |
| requested_at | Yes | Immediate semantics and server timestamp |

## 8. Business rules
- booking_channel does not determine fleet_type.
- Booking creation does not assign Driver/Vehicle.
- Only Booking owner changes Booking fields/state.
- Post-submit edit creates revision/audit and may invalidate quote/dispatch context.
- No candidate is not a reason to delete Booking.
- Server timestamps authoritative; device time is metadata.
- Fare policy and quote mandatory/optional are TBD.
- Fleet selection belongs to Dispatch after eligibility.

## 9. State and handoff
Proposed: DRAFT → REQUESTED/CONFIRMED → READY_FOR_DISPATCH → DISPATCHING (or derived searching) → ASSIGNED → IN_SERVICE → COMPLETED. Exact canonical enum pending approval; see [State Catalogue](../03-CROSS-CUTTING/01-state-machines-and-transition-rules.md).

## 10. Logical API contract
Command: CreateImmediateBooking. Request: channel context, customer/contact reference, pickup, destination, stops, service type, passenger/luggage requirements, requested_at, quote reference if applicable, idempotency_key, correlation_id.
Response: booking_id, public_reference, state, version, created_at, dispatch handoff status, correlation_id.
Errors: validation_error, unauthorized, forbidden_scope, service_unavailable, duplicate_request, conflict, dependency_unavailable. Paths/schema TBD.

## 11. Data, audit and transactional behavior
Persist Booking and BookingStop in one transaction; add BookingRevision/AuditEvent. Publish outbox event after commit. Unique public reference and stop sequence; idempotent request key. Provider response is not authoritative quote snapshot unless normalized/versioned.

## 12. UI/UX states
Service catalog loading; customer search idle/loading/empty/error; geocoding suggestions/ambiguous; map pin selected/unconfirmed/confirmed; form invalid; submitting; accepted; queued for dispatch; conflict; unauthorized; dependency outage; duplicate warning. Distinguish “booking accepted” from “vehicle assigned”.

## 13. Security and privacy
Field-level masking; role/data scope; anti-enumeration; audit create/change/cancel; consent/legal basis/retention TBD; avoid PII in URL/logs.

## 14. Tests and acceptance
- Valid hotline booking creates exactly one Booking/reference.
- APP/HOTLINE channel correctly recorded; fleet_type remains unset until source assignment.
- Invalid pickup/serviceability cannot enter dispatch.
- Same idempotency key returns same result.
- Concurrent submit does not duplicate booking.
- Dispatch outage does not silently lose accepted booking.
- UI shows accepted/queued, not falsely assigned.
- Unauthorized partner/caller cannot create outside scope.
- Audit/outbox is consistent with committed Booking.

## 15. Dependencies and open decisions
Customer; IAM; service catalog; geocoding; pricing quote; durable outbox; notification. Required fields by channel, guest/customer match, serviceability, quote mandatory/optional, stop count, immediate scheduling semantics and dispatch handoff SLA TBD.
