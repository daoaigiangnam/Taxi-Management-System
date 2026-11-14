# G-02 — TESTING & ACCEPTANCE
**Version:** 0.1 · **Status:** DRAFT · **Owner:** QA + Business Owners
## Test levels
Unit; component; contract; integration; E2E; regression; security; performance/capacity; resilience/DR; accessibility/usability; UAT.
## Critical E2E
Multi-channel booking; DIRECT/PARTNER candidate policy; no candidate/reject/expiry/retry/manual queue; concurrent accepts; resource eligibility change; booking revision; offline/out-of-order trip events; duplicate trip completion; payment callback replay/unknown outcome/refund; partner contract snapshot/variance/lock/payout/adjustment; cross-partner unauthorized access; provider/queue/DB outage and recovery.
## Financial acceptance
Fare components = total; currency precision; cash/provider evidence; payment-to-ledger reconciliation; ledger balance; settlement matches contract snapshot; adjustment links original.
## Release gate
Critical suite passed; no unresolved critical/high security without formal exception; no duplicate assignment/payment posting; Finance reconciliation signed; UAT signed; restore/DR evidence; monitoring/support ready.
Feature acceptance: Given/When/Then, positive/negative/boundary/concurrency/offline/retry/access tests, evidence link, defect severity and business-owner sign-off.
