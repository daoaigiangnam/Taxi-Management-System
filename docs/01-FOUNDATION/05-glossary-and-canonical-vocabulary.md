# FND-05 — THUẬT NGỮ VÀ TỪ VỰNG CHUẨN

**Version:** 0.1 · **Status:** DRAFT

| Term | Canonical meaning | Avoid confusing with |
|---|---|---|
| Booking | Customer's service request | Assignment, Trip |
| Booking Channel | Source of request | Fleet type |
| Fleet Type | DIRECT or PARTNER source classification | Booking channel |
| DispatchRun | One execution/attempt of dispatch search | Booking |
| Candidate | Resource that passed/awaits eligibility evaluation | Assigned driver |
| Offer | Time-bound invitation to a driver | Assignment |
| Assignment | Effective driver/vehicle allocation | Offer accepted event alone |
| Trip | Actual service execution | Booking |
| TripEvent | Immutable event in trip timeline | Current Trip state |
| FareQuote | Estimate/quote with validity and inputs | Final fare |
| FareResult | Reproducible final fare calculation | Payment |
| PaymentTransaction | Collection/refund transaction | Trip completion |
| LedgerEntry | Financial posting | Payment provider callback |
| SettlementBatch | Grouped partner reconciliation/payment basis | Ledger |
| IncidentCase | Safety/operational case | Trip state alone |
| Eligibility | Hard constraints a resource must pass | Ranking score |
| Ranking | Ordering of eligible candidates | Eligibility |
| System of Record | Authoritative owner of data | Read-model/cache |

Naming convention: canonical domain terms in English identifiers; Vietnamese explanations in business documents. State/API enum changes require decision approval and compatibility/migration analysis.
