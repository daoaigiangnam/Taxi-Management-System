# M02 — FLEET & PARTNER
**Status:** DRAFT · **Owner:** Fleet + Partner Management (TBD) · [Back to Overview](../../../README.md)

## Purpose
Quản lý nguồn lực và điều kiện đủ tư cách khai thác: Driver, Vehicle, driver-vehicle link, shift, fleet, Partner, Contract và compliance documents.

## Entities
Driver, Vehicle, DriverVehicleLink, Shift, FleetGroup, Partner, Contract, ContractVersion, DriverDocument, VehicleDocument, EligibilityProfile, AvailabilitySnapshot.

## Feature Catalog
| ID | Feature | Priority |
|---|---|---|
| FLT-001 | Driver profile/onboarding | P0 |
| FLT-002 | Vehicle profile/onboarding | P0 |
| FLT-003 | Driver–Vehicle validity | P0 |
| FLT-004 | Shift and availability | P0 |
| FLT-005 | Document/license compliance | P0 |
| FLT-006 | Fleet group/zone/capability | P1 |
| PRT-001 | Partner profile/onboarding | P0 |
| PRT-002 | Contract/version | P0 |
| PRT-003 | Partner eligibility | P0 |
| PRT-004 | Suspension/reinstatement | P1 |

## Rules
PARTNER resource references active Partner and applicable ContractVersion. Fleet type is explicit. Eligibility precedes ranking. Availability, GPS freshness and connectivity are separate. Trip/settlement retains resource/contract snapshots.

## Open decisions
Required documents, validation source, shift rules, ownership, zones, insurance/legal compliance, partner approval and effective dates.
