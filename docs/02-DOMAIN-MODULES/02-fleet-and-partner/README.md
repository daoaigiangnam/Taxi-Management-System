# M02 — FLEET & PARTNER

**Status:** DRAFT · **Owner:** Fleet + Partner Management (TBD)  
**Overview:** [Project Overview](../../../00-OVERVIEW.md) · [Operating Model](../../../01-FOUNDATION/02-operating-model-direct-partner.md)

## Purpose
Quản lý nguồn lực cung cấp dịch vụ và điều kiện hợp lệ để tham gia điều phối: Driver, Vehicle, mối liên hệ tài xế-xe, ca làm, đội xe, partner, contract và document compliance.

## Entities
Driver, Vehicle, DriverVehicleLink, Shift, FleetGroup, Partner, Contract, ContractVersion, DriverDocument, VehicleDocument, EligibilityProfile, AvailabilitySnapshot.

## Features

| ID | Feature | Priority | Detail |
|---|---|---|---|
| FLT-001 | Driver profile/onboarding | P0 | Planned |
| FLT-002 | Vehicle profile/onboarding | P0 | Planned |
| FLT-003 | Driver–Vehicle assignment validity | P0 | Planned |
| FLT-004 | Shift and availability | P0 | Planned |
| FLT-005 | Document/license expiry and compliance | P0 | Planned |
| FLT-006 | Fleet group/zone/service capability | P1 | Planned |
| PRT-001 | Partner profile and onboarding | P0 | Planned |
| PRT-002 | Contract and contract version | P0 | Planned |
| PRT-003 | Partner operational eligibility | P0 | Planned |
| PRT-004 | Partner suspension and reinstatement | P1 | Planned |

## Rules
- PARTNER resource must reference an active partner and effective contract version where policy requires.
- DIRECT/partner classification must be explicit on resource records.
- Eligibility checks precede ranking in Dispatch.
- Driver availability, vehicle availability, GPS freshness and connectivity are separate dimensions.
- Historical trip/settlement retains resource/contract snapshot references.

## Open decisions
Document requirements, validation source, shift rules, vehicle ownership/multi-partner rules, zones, insurance and legal compliance, partner approval/suspension and effective-date rules are TBD.
