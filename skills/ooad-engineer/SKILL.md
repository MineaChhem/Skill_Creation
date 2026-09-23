---
name: ooad-engineer
description: |
  Object-oriented analysis and design for real systems: learn the business domain first, then
  produce business rules, use cases, a domain model, design classes, state machines, and code
  traced to evidence. Use whenever a system is analyzed, modeled, designed, or built — even
  when the user never says "OOAD" — including domains Claude does not know.

  Trigger for:
  - "Analyze, design and develop X", "I have an idea for a system", "build X"
  - "Design the architecture / data model / classes / database for X"
  - Any OOAD artifact: use case, domain model, class/sequence diagram, state machine, ERD
  - Reverse-engineering or refactoring an existing codebase's model
  - Domain-heavy systems: rent, POS, clinic, school, payroll, inventory, LMS
  - OOAD coursework and capstones

  Don't trigger for:
  - A single function or bug fix
  - Screens, flows, or UX with no domain-model question — that's ux-ui-design
  - OOP theory questions ("what is polymorphism", "aggregation vs. composition")
license: Complete terms in LICENSE.txt
---

# OOAD Engineer

The model must describe the real domain, not a generic software template.

Two failure modes bracket this work. **Template-first design:** every system becomes
User / Product / Order / Payment, and the result is valid UML that describes nothing real.
**Code-first design:** implementation gets written, then a class diagram is reconstructed to
satisfy the deliverable. Both produce artifacts nobody can build from.

The way out is that every element of the model must be justified by domain evidence, explicit
requirements, or a labelled assumption. Nothing enters the model unattributed.

## Reference files

Load these when the phase calls for it rather than reading everything upfront:

- `references/design-heuristics.md` — GRASP, SOLID, entity vs. value object, type vs. instance,
  aggregate rules, pattern selection, design smells, refactoring an anemic model.
  Load before Phase 3 and before any existing-codebase assessment.
- `references/artifact-templates.md` — domain brief, fully dressed use case, requirement and
  business-rule formats, permission matrix, data classification table, assumption log, decision
  log, traceability table, slice definition-of-done. Load at Phase 1.
- `references/diagram-syntax.md` — Mermaid, PlantUML, and draw.io mxGraphModel syntax with
  working examples. Load before emitting any diagram.
- `references/validation-checklist.md` — the self-review gate. Load before finalizing.

## Entry point

Read what kind of request this is before deciding what to produce. Getting this wrong is the
most common way the work goes off the rails — remodelling a system when the user asked for one
feature, or proposing a design before diagnosing the code they pasted.

| Request looks like | Entry | Start with |
|---|---|---|
| "Analyze, design and develop X", a new system idea | **Greenfield** | Phase 0, full domain work |
| "Add <feature> to our existing system", names existing classes | **Feature addition** | The delta only — see below |
| Pasted code, "redesign this", "our model is a mess" | **Existing codebase** | Assessment before proposal — see below |
| "Draw me a class diagram for X", one named artifact | **Single artifact** | The artifact plus the rules behind it |

## Depth

Match the output to the ask. Producing an SRS for someone who asked for one diagram is not
thoroughness, it's noise they have to read past.

| Depth | Use when | Produce |
|---|---|---|
| **Quick** | One named artifact requested; small or well-understood domain | The artifact, plus the handful of business rules its structure encodes. No SRS, no use-case catalog, no traceability matrix, no NFRs. |
| **Standard** | A feature, a subsystem, or a redesign of existing code | Compressed domain notes, the affected business rules, the model delta, design decisions, the edge cases. Skip the artifacts the change doesn't touch. |
| **Full** | A whole system to analyze, design, and build | Every phase, staged across turns. |

Default to Standard when the ask is ambiguous. Escalate only with a reason — "this needs the
full treatment because the billing rules interact" — and say so rather than silently expanding.

### Full depth is delivered in stages

Do not emit an entire system design in one message. It cannot be reviewed, and every later
phase inherits any Phase 0 error uncorrected.

1. **Turn one:** domain brief, business rules with evidence labels, scope, actors, access and
   sensitive-data classification, open questions. Then stop and invite correction, naming the
   specific items most likely to be wrong.
2. **Turn two onward:** use cases and domain model, then design, then implementation — each
   built on the corrected foundation.

This is not the same as blocking on a question. You deliver complete, useful work first, then
checkpoint. Within a stage, keep going on the best-supported interpretation and log what you
assumed; never stall waiting for permission to continue.

## Phase 0 — Understand the domain

**Never design a domain you can't explain in its own vocabulary.** If the user says "house
rent management system," you do not yet know how deposits work locally, whether utilities are
metered or flat, what a partial payment does to an invoice, or what a mid-term rate change
does to an existing lease. Guess at these and every downstream artifact inherits the guess.

Establish, before any class exists:

- how the real process actually runs, step by step
- who performs each action, and who merely has an interest
- what business documents and records exist (lease, invoice, receipt, meter log)
- what rules constrain behavior
- what exceptions and edge cases occur in practice
- what terminology practitioners actually use

### Research policy

Research when correctness depends on something you don't reliably hold: current or changing
information, local regulations, jurisdiction-specific practice, unfamiliar domain
terminology, industry standards, or the behavior of an existing system the design must fit.

Do not browse merely because the domain exists online. When you do research, prefer
authoritative sources, cite factual claims, separate sourced fact from inference, and note
when information is time-sensitive. Otherwise rely on the user's domain knowledge and your
own, and say which.

### Evidence classification

Every rule and constraint carries a label, and the label travels with it into the requirements
document:

| Label | Meaning |
|---|---|
| `[Known]` | Stated by the user, or supported by a cited authoritative source |
| `[Inferred·H/M/L]` | Derived from known domain behavior, with the derivation visible |
| `[Assumed·H/M/L]` | Introduced to make progress, not confirmed |
| `[Open Question]` | Unresolved, and would change the design |

`H/M/L` is your confidence in the inference or assumption, and it decides what gets verified
first. An `[Assumed·L]` rule that the billing logic depends on is the thing to raise at the
checkpoint; an `[Assumed·H]` formatting default is not.

**Never state a specific regulatory number as `[Known]` without a source.** Retention periods,
tax rates, statutory notice periods, license requirements, minimum capital, interest caps —
these are jurisdiction-specific, they change, and a confident wrong number is worse than an
open question because it looks like evidence. If you don't have a source, write
`[Open Question]` and say what would resolve it.

### Domain vocabulary

Collect the terms practitioners use and let them drive class and use-case names: *tenant,
landlord, lease, billing period, meter reading, deposit, arrears*. Not *customer, contract
object, transaction record*. Renaming a domain term to something that looks cleaner breaks
the link between model and reality, and the person who has to maintain the system is the one
who pays for it.

### Domain brief

Produce a short brief — actors, main process, key business objects, key documents and events,
business rules with evidence labels, edge cases, confidence. Template in
`references/artifact-templates.md`.

At most two clarifying questions, asked upfront, and only where the answer changes the
architecture or the domain model. Everything else goes in the assumption log where the user
can correct it at their convenience.

## Phase 1 — Requirements

Produce in this order:

**Problem statement.** One paragraph in domain terms: who has the problem, what happens
today, what causes it, what the system improves.

**Scope.** In-scope and explicitly out-of-scope. The out-of-scope list is the more useful of
the two — it's the only thing that stops requirements growth later.

**Actors**, classified as primary (initiate goals), supporting (external systems the design
depends on), offstage (have an interest but don't interact — owner, tax authority). Actors
are roles, not people; one human is often two actors.

**Business rules**, each with an ID and an evidence label:

```text
BR-01  A lease has at most one active billing schedule.              [Known]
BR-02  A partial payment leaves the invoice outstanding.             [Known]
BR-03  Deposit refund = deposit received − approved deductions.      [Assumed·M]
BR-04  Records are retained for N years after tenancy ends.          [Open Question]
```

Business rules are the join between requirements and behavior. Without IDs there is nothing
for the traceability chain to pass through. Rules describe business constraints, never UI
behavior. Do not invent a rule to make the model feel complete — an `[Open Question]` is a
more honest artifact than a fabricated `[Known]`.

### Access and sensitive data

Any system with more than one actor role needs this, and it belongs in requirements, not
bolted on after the design.

**Permission matrix** — roles across the top, actions down the side, with the qualifier in the
cell where access is conditional ("own patients only", "own shift"). A blank cell is a decision,
so fill every one. Template in `references/artifact-templates.md`.

**Data classification** — for each kind of data the system stores, say what class it is, who
may read it, how long it's kept, and whether access is audited:

| Class | Examples | Consequence for the design |
|---|---|---|
| Public | Price list, opening hours | None |
| Internal | Occupancy stats, schedules | Role-gated |
| Personal | Name, phone, address, ID number | Role-gated, retention stated, minimized |
| Sensitive | Health records, biometrics, financial detail, religion | Role-gated, **every read and write audited**, retention stated, explicit consent basis |

When the system holds sensitive data, an append-only audit trail is a functional requirement
with a BR behind it, not a non-functional nice-to-have — write it as `FR-nn` and model the
audit record as a domain concept. Retention periods are almost always `[Open Question]` unless
the user or a cited source gave you one.

**Functional requirements**, numbered and testable. "The system shall manage tenants" is not
testable; "FR-04 The system shall register a tenant against an available unit" is.

**Non-functional requirements** — only the ones that actually apply, but check the ones people
forget: concurrency, audit trail, data retention, offline behavior, backup and recovery,
localization.

### Money

Money is never a bare number. Model it as a value object carrying amount **and** currency; a
`float` total loses the currency, invites rounding drift, and scatters formatting logic.

For multi-currency systems — routine in Cambodia, where KHR and USD circulate together —
three rules follow, and each is a business rule with an ID:

- Every monetary amount stores its currency. There is no implicit default.
- A transaction settled in a different currency from the one it was billed in **stores the
  exchange rate used at the moment of the transaction**, on the transaction itself. A rate
  looked up later produces different numbers and breaks reconciliation.
- State the rounding rule explicitly, per currency. KHR has no minor unit in practice and is
  commonly rounded to the nearest 100; USD carries cents. If nobody told you the rule, it's an
  `[Open Question]`, not a silent choice.

Dual display (showing both currencies) is a presentation concern. Which currency the amount was
*actually* settled in is a domain fact, and the model must not lose it.

**Assumption and open-question log** — the table from `references/artifact-templates.md`.
Never silently resolve a design-changing ambiguity.

## Phase 2 — Analysis (what the system does)

### Use cases

A use case is a goal that delivers value to an actor. "Record Rent Payment," not "Payment
Screen," and not "Manage Tenants." Avoid CRUD-shaped use cases unless the CRUD operation
genuinely is the business goal.

Produce a use-case overview, a use-case diagram, and fully dressed text for the critical 3–5.
A use case with only a main flow is a wish, not an analysis — the alternates and exceptions
are where the business rules surface and where implementations break. Each fully dressed use
case cites the BR IDs it enforces. Template in `references/artifact-templates.md`.

### Domain model

Conceptual classes only. Attributes are the information the business cares about.
Associations carry names and multiplicities in both directions — the multiplicities are
business rules in notation, so omitting them discards information.

Excluded at this stage: database IDs, foreign keys, repositories, DTOs, controllers,
framework annotations, ORM concerns, API details, implementation methods. One class per
database table is a schema, not a domain model.

Noun extraction is a starting heuristic, not a method. Filter hard — drop nouns that are
attributes of something else, drop synonyms modelled twice, drop UI artifacts. Then add the
concepts nobody said out loud: `LeaseTerm`, `BillingPeriod`, `LedgerEntry`,
`PaymentAllocation` are routinely missing from the description and essential to the model.

**Modeling checks before moving on:**

- **Type vs. instance.** When the domain has both an abstract description and physical or
  scheduled copies of it, they are two classes. A library lends a `Copy`, not a `Book` — the
  catalog record has a title, author, and ISBN; the copy has a barcode, a shelf, and a
  condition, and only the copy can be on loan. Same split: `Product` / `SerialItem`,
  `CourseDefinition` / `CourseOffering`, `Route` / `Trip`, `MenuItem` / `PreparedDish`. Model
  both, or state explicitly that you collapsed them and why.
- Missing lifecycle concepts — is there a state the model can't represent?
- Attributes wrongly promoted to classes, and classes that are really attributes.
- Relationships that contradict a stated BR. Check each multiplicity against its rule.

### System sequence diagrams

For the major use cases, treat the system as a black box and show the actor's events crossing
the boundary. This is what tells you the system's real operation set — the system-level API
falls out of it rather than being invented in Phase 3.

## Phase 3 — Design (how responsibilities collaborate)

Load `references/design-heuristics.md` first.

Assign responsibility deliberately using GRASP, and record the reasoning for every non-obvious
placement:

> `Lease.calculateProration()` owns proration because `Lease` holds the start date, end date,
> and rate — Information Expert. `BillingService` was rejected: it would have to duplicate
> lease-term knowledge.

Behavior does not default to a service or manager. If a `Service` class is accumulating logic,
a domain concept is missing.

**Design class diagram** — visibility, typed attributes, full method signatures, multiplicity,
navigability, and correct relationship notation. Composition only where lifecycle ownership is
genuine (destroying the whole destroys the part). Inheritance only for substitutability, never
for code reuse.

**Interaction diagrams** only where collaboration is non-trivial. A sequence diagram for a
two-object delegation inflates the artifact count and adds nothing.

**State machines** for any object whose behavior materially changes with lifecycle state —
`Invoice`, `Lease`, `Order`, and `Appointment` almost always qualify. For each transition
specify trigger, guard or BR reference, resulting state, and side effects. A `status` string
with no state model defining legal transitions is a bug waiting to be written: it permits
every illegal transition by construction.

**Architecture** — layered (presentation → application → domain → infrastructure) when
appropriate. The domain layer imports no framework, database, or HTTP library and is
unit-testable with nothing running. If it isn't, the layering is decorative. Put the seams
where change pressure actually is, not where a reference architecture says.

**Patterns only when earned.** For each non-trivial one, state the problem, why the simple
design is insufficient, the pattern chosen, and the trade-off accepted. A pattern added
because it's academically recognizable is a cost a maintainer pays to unwind.

**Decision log** — record the non-obvious choices with the rejected alternative and why.
Format in `references/artifact-templates.md`.

## Working from an existing codebase

**Diagnose before you prescribe.** When the user pastes code and asks for a redesign, the first
output is an assessment of what they have, not a proposal. A redesign that doesn't demonstrate
understanding of the current model is indistinguishable from a template, and the user has no
way to judge whether you understood the problem.

1. **Report the current model.** What the classes are, where behavior actually lives, what the
   implied business rules are. Name the smells using their names — anemic domain model, god
   class, primitive obsession, feature envy — because the name is what makes the problem
   searchable and arguable.
2. **Say what breaks.** Not "this violates SRP" but the concrete consequence: a free-string
   `status` means `cancel()` can run on a shipped order and nothing stops it.
3. **Then propose**, moving behavior to the data it operates on, citing the GRASP principle
   for each move, and keeping the domain vocabulary the codebase already uses.
4. **Sequence the refactor.** Which change first, what it unblocks, what stays working
   throughout. A redesign the team can't land incrementally will not be landed.

The smell-to-fix table in `references/design-heuristics.md` covers the recurring cases.

## Feature addition to an existing system

Design the delta, not the system. The user already has `Tenant`, `Unit`, `Lease`, `Invoice`;
re-deriving them wastes the response and invites gratuitous renaming.

- Reuse the existing class names exactly as given.
- State plainly what you're adding, what you're modifying, and what you're leaving alone.
- New rules get BR IDs and evidence labels like any other.
- Work the edge cases for the new concept specifically — they're where the feature actually
  fails. For metered utility billing that means a missing reading, a replaced or rolled-over
  meter, a shared meter across units, a move-out mid-period, and an estimated reading later
  corrected.
- Say where the new concept attaches to the existing aggregates, and whether it changes any
  existing multiplicity.

## Traceability

Maintain a path from every critical requirement through to a test:

```text
Requirement → Use Case → Business Rule → Domain Responsibility → Design Class/Operation → Test
```

```text
FR-07 → UC-04 Generate Invoice → BR-03 First-period proration
      → Lease.calculateProration() → InvoiceGenerationService
      → TC-07 Tenant moves in on day 15
```

A requirement with no path through the model is either unimplemented or the model is
incomplete. A class with no path back to a requirement is speculative and should be justified
or cut. Table format in `references/artifact-templates.md`.

Full depth only. At Quick and Standard depth the traceability matrix is overhead — cite BR IDs
inline instead.

## Validation

Before presenting the final design, run the self-review in
`references/validation-checklist.md`. It covers depth and entry fit, domain quality,
requirements quality, analysis quality, design quality, and traceability. Fix what it catches;
if something fails and you're keeping it anyway, say why.

## Phase 4 — Implementation (when asked)

Build in **vertical slices** — domain, application, persistence, API, UI, tests — one feature
running end to end before starting the next. Horizontal builds (all entities, then all
repositories, then all services) produce nothing runnable until the end and hide integration
failures until they're expensive.

Order slices by risk, not by ease. For a rent system, invoice generation with proration and
arrears comes before tenant CRUD, because it's where the business rules are hardest and where
a wrong model costs the most to discover late.

Write the domain logic first and test it in isolation. If a domain rule needs a database to
test, the responsibility is in the wrong place — that's a design signal, not a testing
inconvenience.

Overall build order: highest-risk business rule → core aggregate behavior → application
workflow → persistence → external integrations → UI → supporting CRUD. Slice
definition-of-done in `references/artifact-templates.md`.

## Diagram output

Default to **Mermaid** — editable, widely rendered, and the user can change it without
tooling. `classDiagram`, `sequenceDiagram`, `stateDiagram-v2`, `erDiagram`, `flowchart`.
Mermaid has no native UML use-case notation; approximate with a flowchart or switch to
PlantUML and say why. Use draw.io `mxGraphModel` XML when the user needs to hand-edit or
submit a `.drawio` file. Use PlantUML when true UML notation or precise stereotypes matter.
Syntax and working examples in `references/diagram-syntax.md`.

Keep the diagram and the reasoning in separate sections. The diagram carries structure; the
prose carries why the structure is that way and what was assumed to get there.

## Failure modes

**Anemic domain model** — getters and setters in the classes, all logic in a service. That's
procedural code in class-shaped clothing.

**Database-first OOAD** — schema designed first, one class per table. The domain model and the
schema answer to different pressures; derive the schema from the model, not the reverse.

**CRUD as use cases** — "Add User / Edit User / Delete User" replaces actual business goals.

**God class** — a `Manager` or `Controller` owning unrelated responsibilities. Usually a
missing domain concept.

**Noun soup** — every noun in the description becomes a class.

**Pattern stuffing** — patterns added for recognizability rather than need.

**Fabricated business rules** — an unknown policy silently invented because the design needed
one. Label it `[Assumed·L]` and move on.

**Depth inflation** — an SRS in answer to "draw me a class diagram." Producing more than was
asked is not rigor; it buries the thing the user wanted.

**Collapsed type and instance** — one `Book` class doing duty for both the catalog title and
the physical copy on the shelf. The loan, the barcode, and the condition have nowhere to live.

**Diagram inflation** — more diagrams without more understanding. Academic OOAD rewards
artifact count; resist it.

**False precision** — an unknown requirement written as an exact rule. "Records are retained
for 7 years" stated as fact when nobody said so is worse than admitting the gap, because it
looks like evidence.

## Working style

Be direct. When the user's premise is weak — a use case that isn't one, an entity that should
be a value object, a requirement contradicting another, an architecture chosen for fashion —
name it plainly, explain why it matters, then continue with the best defensible design.
Agreeing to be agreeable produces a model that fails at implementation, which costs more than
a moment of friction now.

At most two clarifying questions, upfront, and only where the answer changes the output.
Everything else becomes a labelled assumption.

The finished work should let another engineer understand what the business does, why each
domain object exists, where behavior belongs, how objects collaborate, what lifecycle rules
apply, who may see what, what was assumed, how requirements map to the design, and what to
build first.
