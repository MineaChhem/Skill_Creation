---
name: ooad-engineer
description: |
  Use this skill when the user hands over a system or business domain to analyze, model,
  design, or build, and expects object-oriented analysis and design before implementation —
  including domains Claude does not already know well.

  Trigger for:
  - "Analyze, design and develop X" / "I have an idea for a system"
  - "Design the architecture / data model / classes for X"
  - Requests naming OOAD artifacts: use case diagram, domain model, class diagram,
    sequence diagram, state machine, ERD, activity diagram, SRS
  - Domain-heavy systems where the real-world process must be learned first:
    rent management, POS, clinic booking, school registry, payroll, inventory, LMS
  - Academic OOAD assignments and capstone projects

  Don't trigger for:
  - A single function, bug fix, or isolated implementation question
  - Pure UI/visual design with no system or domain model behind it
  - OOP theory questions ("what is polymorphism") — that's teaching, not designing
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

- `references/design-heuristics.md` — GRASP, SOLID, entity vs. value object, aggregate rules,
  pattern selection, design smells. Load before Phase 3.
- `references/artifact-templates.md` — domain brief, fully dressed use case, requirement and
  business-rule formats, assumption log, decision log, traceability table, slice
  definition-of-done. Load at Phase 1.
- `references/diagram-syntax.md` — Mermaid, PlantUML, and draw.io mxGraphModel syntax with
  working examples. Load before emitting any diagram.
- `references/validation-checklist.md` — the self-review gate. Load before finalizing.

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

Every rule and constraint carries one of these labels, and the label travels with it into the
requirements document:

- **Known** — stated by the user, or supported by an authoritative source
- **Inferred** — logically derived from known domain behavior, with the derivation visible
- **Assumed** — introduced to make progress, not confirmed
- **Open Question** — unresolved, and would change the design

Never present an assumption as a fact. A design built on three unlabelled guesses looks
identical to one built on three confirmed rules, which is exactly why the labels matter.

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

Then keep going using the best-supported interpretation. Do not stop and wait for
confirmation unless a missing answer would materially change the architecture or the domain
model. At most two clarifying questions, asked upfront. Everything else goes in the
assumption log where the user can correct it at their convenience.

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
BR-03  Deposit refund = deposit received − approved deductions.      [Assumed]
```

Business rules are the join between requirements and behavior. Without IDs there is nothing
for the traceability chain to pass through. Rules describe business constraints, never UI
behavior. Do not invent a rule to make the model feel complete — an `[Open Question]` is a
more honest artifact than a fabricated `[Known]`.

**Functional requirements**, numbered and testable. "The system shall manage tenants" is not
testable; "FR-04 The system shall register a tenant against an available unit" is.

**Non-functional requirements** — only the ones that actually apply, but check the ones people
forget: concurrency, audit trail, data retention, offline behavior, backup and recovery,
localization. For systems in this region that means Khmer/English, KHR/USD dual currency, and
local date formats — treat these as design inputs, not polish.

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
Before moving on, check for missing lifecycle concepts, attributes wrongly promoted to
classes, and relationships that contradict a stated BR.

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
`Invoice` and `Lease` almost always qualify. For each transition specify trigger, guard or BR
reference, resulting state, and side effects. A `status` string with no state model defining
legal transitions is a bug waiting to be written.

**Architecture** — layered (presentation → application → domain → infrastructure) when
appropriate. The domain layer imports no framework, database, or HTTP library and is
unit-testable with nothing running. If it isn't, the layering is decorative. Put the seams
where change pressure actually is, not where a reference architecture says.

**Patterns only when earned.** For each non-trivial one, state the problem, why the simple
design is insufficient, the pattern chosen, and the trade-off accepted. A pattern added
because it's academically recognizable is a cost a maintainer pays to unwind.

**Decision log** — record the non-obvious choices with the rejected alternative and why.
Format in `references/artifact-templates.md`. Skip trivial implementation details; this is for
decisions a reviewer would otherwise have to reverse-engineer.

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

## Validation

Before presenting the final design, run the self-review in
`references/validation-checklist.md`. It covers domain quality, requirements quality, analysis
quality, design quality, and traceability. Fix what it catches; if something fails and you're
keeping it anyway, say why.

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
one. Label it `[Assumed]` and move on.

**Diagram inflation** — more diagrams without more understanding. Academic OOAD rewards
artifact count; resist it.

**False precision** — an unknown requirement written as an exact rule. "Deposits are refunded
within 14 days" stated as fact when nobody said so is worse than admitting the gap, because it
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
apply, what was assumed, how requirements map to the design, and what to build first.
