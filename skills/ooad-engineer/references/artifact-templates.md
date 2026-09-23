# Artifact templates

Load at Phase 1. Copy the shape, fill with domain content. Every template that carries claims
also carries evidence labels: `[Known]`, `[Inferred]`, `[Assumed]`, `[Open Question]`.

---

## Domain brief (Phase 0)

Short. One page. Its job is to be corrected cheaply, before the design is built on it.

```markdown
# Domain brief — <system>

## What this business does
<2–4 sentences in the practitioners' own words.>

## Actors
| Actor | Type | Goal |
|---|---|---|
| Landlord | Primary | Collect rent on time, track arrears |
| Tenant | Primary | Pay rent, get a receipt |
| Property owner | Offstage | Monthly income report |

## Main process
1. ...
2. ...

## Key business objects
Lease, Unit, Invoice, Payment, Deposit, MeterReading

## Key documents and events
Lease agreement (signed, physical) · Monthly invoice · Receipt · Move-out inspection

## Business rules
| ID | Rule | Evidence |
|---|---|---|
| BR-01 | A unit has at most one active lease at a time | [Known] |
| BR-02 | A partial payment leaves the invoice outstanding | [Known] |
| BR-03 | Deposit refund = deposit − approved deductions | [Assumed·M] |

## Edge cases seen in practice
- Mid-month move-in → first invoice prorated
- Tenant pays for another tenant's unit
- Shared water meter across two units
- Rate change mid-lease

## Confidence
Confident: invoicing and payment flow.
Uncertain: deposit deduction rules, notice periods.
Would change the design: whether utilities are billed per-meter or flat.
```

---

## Business rule

```text
BR-<nn>  <Statement of the constraint, in domain terms, one sentence.>   [Evidence]
         Source: <user statement / cited source / the derivation, spelled out>
```

Evidence is one of `[Known]`, `[Inferred·H/M/L]`, `[Assumed·H/M/L]`, `[Open Question]`. The
confidence grade decides what gets verified first — a low-confidence assumption the billing
logic depends on is what you raise at the checkpoint.

Never grade a specific regulatory number as `[Known]` without a citation. Retention periods,
tax rates, statutory notice periods and licence thresholds are jurisdiction-specific and they
change; write `[Open Question]` and say what would resolve it.

Rules constrain the business, not the interface. "The deposit cannot exceed two months' rent"
is a rule; "the deposit field is disabled until a lease is selected" is UI behavior.

---

## Functional requirement

```text
FR-<nn>  The system shall <action> <object> [<condition>].
         Actor: <who>
         Enforces: BR-<nn>, BR-<nn>
         Acceptance: <observable outcome that makes this pass>
```

Testable or it isn't a requirement. "Manage tenants" fails; "register a tenant against an
available unit" passes.

---

## Permission matrix

Roles across, actions down. Fill every cell — a blank is an undecided access rule, and
undecided access rules become production incidents. Use a qualifier where access is
conditional rather than a bare tick.

```markdown
| Action | Receptionist | Doctor | Cashier | Clinic admin |
|---|---|---|---|---|
| Book / reschedule appointment | Yes | Own slots | No | Yes |
| View patient contact details | Yes | Own patients | Name only | Yes |
| View or write clinical note | No | Own patients | No | No |
| Order lab test | No | Own patients | No | No |
| View lab result | No | Own patients | No | No |
| Record payment | No | No | Yes | Yes |
| Issue refund | No | No | Needs approval | Yes |
| Change price list | No | No | No | Yes |
| Export patient data | No | No | No | Yes, audited |
```

"Own patients" and "needs approval" are business rules — give them BR IDs and reference them
from the use cases that enforce them.

---

## Data classification

One row per kind of data the system stores. Retention is an `[Open Question]` unless the user
or a cited source gave you the period — never fill it in from memory.

```markdown
| Data | Class | Who may read | Retention | Audited |
|---|---|---|---|---|
| Patient name, phone | Personal | Reception, treating doctor, admin | [Open Question] | Writes |
| Clinical note, diagnosis | Sensitive (health) | Treating doctor only | [Open Question] | Reads and writes |
| Lab result | Sensitive (health) | Treating doctor, ordering doctor | [Open Question] | Reads and writes |
| Invoice, payment record | Personal + financial | Cashier, admin | [Open Question] — tax law | Writes |
| Price list | Public | Everyone | n/a | No |
```

When a row is Sensitive, the audit trail stops being a non-functional aspiration and becomes a
functional requirement with a domain object behind it:

```text
FR-21  The system shall record an immutable audit entry for every read of a clinical note,
       capturing actor, patient, timestamp, and access reason.
       Enforces: BR-14
       Acceptance: A doctor opening a note produces exactly one audit entry; the entry
       cannot be edited or deleted through any application path.
```

Model it as an append-only `AuditEntry`, not a mutable log table — "immutable" is a design
constraint the model has to express, not a convention to hope for.

---

## Existing-model assessment

Produce this before proposing anything, when the user hands you code.

```markdown
## What you have now

**Structure:** `Order` holds `id, status: string, total: number, items: any[]` with accessors
only. `OrderService` holds `placeOrder, cancelOrder, applyDiscount, markShipped,
calculateTotal`.

**Diagnosis:**
| Smell | Where | Concrete consequence |
|---|---|---|
| Anemic domain model | `Order` / `OrderService` | Order invariants live nowhere; any caller can set any field to anything |
| Primitive obsession | `status: string`, `total: number` | `cancel()` runs on a shipped order; totals lose currency and drift on rounding |
| Untyped collection | `items: any[]` | No line-level invariant; quantity and price are unchecked |

**Implied business rules** (recovered from the code, to be confirmed):
| ID | Rule | Evidence |
|---|---|---|
| BR-01 | An order can be cancelled only before shipping | [Inferred·M] — implied by method set, not enforced anywhere |

## What I propose
<the redesign, each move citing its GRASP principle>

## Refactor order
1. ... (what lands first, what it unblocks, what keeps working)
```

---

## Fully dressed use case

```markdown
## UC-04 — Generate Monthly Invoice

**Scope:** Rent management system
**Level:** User goal
**Primary actor:** Landlord
**Stakeholders and interests:**
- Landlord — every active lease billed, correct amount, on time
- Tenant — accurate charge, arrears visible
- Owner — revenue recorded in the period it belongs to

**Preconditions:** At least one active lease; billing period not already invoiced.
**Success guarantee:** One issued invoice per active lease; ledger entries written.
**Trigger:** Billing date reached, or landlord runs billing manually.

**Main success scenario**
1. Landlord initiates billing for a period.
2. System lists active leases with no invoice for that period.
3. System computes the amount due per lease: base rent + metered utilities + arrears
   carried forward. (BR-02, BR-05)
4. System issues each invoice and writes the ledger entries.
5. System presents the batch result.

**Extensions**
- 3a. Lease started mid-period → System prorates by occupied days. (BR-03)
- 3b. Lease ends mid-period → System prorates and marks it the final invoice.
- 3c. Meter reading missing → System issues the invoice without the utility line and flags
  it for adjustment. (BR-07)
- 3d. Rate changed mid-period → System bills each segment at its own rate. (BR-09)
- 4a. Ledger write fails → System rolls the batch back; no partial issuance.

**Special requirements**
- Batch of 200 leases completes within 30 s.
- Amounts render in KHR and USD.

**Frequency:** Monthly, plus ad-hoc reruns.
**Open questions:** Rounding rule on prorated amounts — nearest 100 KHR? [Open Question]
```

---

## Assumption and open-question log

```markdown
| ID | Statement | Type | Impact if wrong | Resolves by |
|---|---|---|---|---|
| A-01 | Deposit is refunded within 14 days of move-out | Assumed·L | Low — a config value | Ask landlord; no source for the period |
| A-02 | Utilities are metered per unit, not shared | Assumed·M | **High — changes the domain model** | Ask user; inspect a bill |
| Q-01 | Can one tenant hold two concurrent leases? | Open | **High — changes Lease multiplicity** | Ask user |
```

Impact is the column that matters. High-impact open questions are the only legitimate reason
to stop and ask.

---

## Decision log (ADR-lite)

```markdown
### DD-03 — Invoice owns payment allocation

**Context:** A payment can cover part of an invoice, several invoices, or arrive before an
invoice exists.
**Decision:** `Invoice` is the aggregate root over `PaymentAllocation`. `Payment` records the
money received; allocation to invoices happens inside the `Invoice` aggregate.
**Rejected:** `Payment` holding a list of invoices — it would have to enforce arrears
invariants it has no data for.
**Consequence:** Unallocated payments need an explicit `CreditBalance` concept.
**Revisit if:** Cross-lease payment settlement becomes a requirement.
```

Log the decisions a reviewer would otherwise have to reverse-engineer. Skip the trivia.

---

## Traceability table

```markdown
| FR | Use case | BR | Domain responsibility | Design class / operation | Test |
|---|---|---|---|---|---|
| FR-07 | UC-04 Generate Invoice | BR-03 | Prorate first period | `Lease.calculateProration()` | TC-07 move-in day 15 |
| FR-08 | UC-05 Record Payment | BR-02 | Partial payment keeps invoice open | `Invoice.applyPayment()` | TC-11 pays 60% |
| FR-09 | UC-05 Record Payment | BR-06 | Overpayment becomes credit | `Invoice.applyPayment()` → `CreditBalance` | TC-12 pays 120% |
```

Run it both ways: every FR must reach a test, and every design class must reach an FR.

---

## Slice definition of done

A vertical slice is done when all of these hold:

- [ ] Domain logic implemented and unit-tested with no database, no HTTP, no framework
- [ ] Every business rule the slice enforces has a test named after the rule
- [ ] Application/service layer wires the use case end to end
- [ ] Persistence implemented behind the domain-defined interface
- [ ] Entry point exists (API endpoint, CLI command, or screen) and works against real storage
- [ ] Edge cases from the use case's extensions are covered, not just the main flow
- [ ] Traceability row updated
- [ ] Assumptions the slice exposed as wrong are corrected in the model, not patched in code
