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
| BR-03 | Deposit refund = deposit − approved deductions | [Assumed] |

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
         Source: <user statement / source / derivation>
```

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
| A-01 | Deposit is refunded within 14 days of move-out | Assumed | Low — a config value | Ask landlord |
| A-02 | Utilities are metered per unit, not shared | Assumed | **High — changes the domain model** | Ask user; inspect a bill |
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
