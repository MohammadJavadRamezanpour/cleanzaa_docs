# ADR 0003: Phase 1 launch area and service inputs

**Status:** Approved product direction, 2026-10-18. The launch catalogue is
deliberately small and may change through a new version.

## Decision

### Launch coverage

Phase 1 offers Home Cleaning, Car Wash, and Pet Care only within the **Comune di
Milano, Italy**. The single selectable service area has stable code
`IT-MI-MILANO` and display name `Milano`. The city and service-area code are
controlled selections, not arbitrary customer text. A booking address must
include street, house number, city, and five-digit CAP. The initial coarse CAP
gate accepts `20121` through `20162` with city `Milano`; all other city/CAP
combinations are unsupported. The backend validates this at quote and order
submission. The CAP gate does not prove that a particular street address exists
or that its building lies inside the municipal boundary; operations must
resolve an invalid or disputed address before dispatch. The launch restriction
applies to all three services and to workers' selected service areas.

The area code, city label, and CAP rule belong in one versioned service-area
catalogue shared by quote, order, worker-profile, and eligibility policies.
Historical quotes and orders retain the catalogue version used. Adding a city
or changing a postcode rule creates a new version rather than reinterpreting
past orders.

### Booking input shared by all services

- The customer selects one or more of the three services, a service date and
  start time in `Europe/Rome`, the Milano service area, and a service address.
- The customer states whether the service is for themselves or someone else.
  For someone else, first name, last name, and phone number of that person are
  required. That contact is used only for the service and is not a customer
  account.
- Each selected service has a requested duration in **whole hours**, from
  **1 through 8** inclusive. Combined services use the sum of their selected
  hours to reserve a contiguous time window. Overtime and buffer/travel rules
  remain separate open decisions.
- Each service asks separately whether the worker must bring equipment and
  whether the worker must bring supplies. The answer is required even if both
  are false.

### Service-specific booking inputs and worker eligibility

| Service | Required customer input | Worker qualification and matching |
| --- | --- | --- |
| Home Cleaning | Beginner or professional worker preference; selected parts/rooms to clean; positive room count. | Worker offers Home Cleaning in Milano, declares beginner or professional level, and is manually approved for that level. Professional requests require an approved professional worker. No certificate is required for a beginner or professional home worker by this decision. |
| Car Wash | Car make/model as text. | Worker offers Car Wash in Milano and has certificate evidence approved by a reviewer for Car Wash. |
| Pet Care | Pet species (`dog` or `cat`) and breed as text. | Worker offers Pet Care in Milano and has certificate evidence approved by a reviewer for Pet Care. |

Every worker provides a short work-experience description for review and
declares whether they can bring equipment and supplies. A request that needs
either item matches only workers approved for the service who can bring it.
If a booking combines services, the assigned worker must meet every selected
service's qualification and equipment requirements. Manual identity and
qualification approval in [ADR 0002](0002-phase-1-personal-data.md) remains
mandatory before a worker receives or accepts work.

## Scope and implementation boundary

The supported location and required fields above are approved for the Phase 1
catalogue. Pricing, payment, cancellation, overtime, travel buffers, and the
exact quote calculation remain separately gated decisions in the technical
architecture. A quote must still provide the scheduled duration and immutable
catalogue/rule version before an order can enter matching. No order may be
submitted without an accepted price quote once that feature exists.

This ADR does not change identity-evidence retention or production collection
gates in ADR 0002. Only synthetic worker identity data may be used in controlled
development and staging until those reviews are complete.

The initial Milano city CAP interval follows Poste Italiane's published city
range (`20121`–`20162`); municipal coverage is the product boundary, and the
CAP check is a deliberately simple input gate, not a geospatial claim.

## References

- [Phase 1 product scope](../phase_1_product_scope.md)
- [Product description](../product_description.md)
- [Technical architecture](../technical_architecture.md)
- [Poste Italiane city CAP ranges](https://business.cert.poste.it/business/files/1473806071814/posta1pro-posta4pro-elenco-cap-province-afferenti.pdf)
