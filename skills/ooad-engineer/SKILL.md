---
name: ooad-engineer
description: |
  Use this skill when the user hands over a system to build, model, or analyze and expects
  proper object-oriented analysis and design before code — including domains Claude does not
  already know well.

  Trigger for:
  - "Analyze, design and develop X" / "I have an idea for a system"
  - Requests naming OOAD artifacts: use case diagram, domain model, class diagram,
    sequence diagram, state machine, ERD, activity diagram, SRS
  - Domain-specific systems where the real-world process must be learned first:
    rent management, POS, clinic booking, school registry, payroll, inventory, LMS
  - "Design the architecture / data model / classes for X"
  - Academic OOAD assignments and capstone projects

  Don't trigger for:
  - A single function, bug fix, or isolated code question
  - Pure UI/visual design with no system model behind it
  - Questions about OOP theory itself ("what is polymorphism") — that's teaching, not designing
license: Complete terms in LICENSE.txt
---

# OOAD Engineer

Two failure modes bracket this work. One is designing from a stock template: every system
becomes User / Product / Order / Payment, and the diagram is technically valid UML that
describes nothing real. The other is jumping to code, then reverse-engineering a class
diagram to satisfy the deliverable. Both produce artifacts nobody can build from.

The way out is that the model must come from the domain, not from your priors about what
systems look like. So the first phase is not requirements — it's learning how the actual
thing works.

## Phase 0 — Learn the domain first

**Never design a domain you can't explain in its own vocabulary.** If the user says "house
rent management system," you do not yet know: how deposits work locally, whether utilities
are metered or flat, what happens on partial payment, what a lease amendment does to an
existing contract, what the eviction process requires. Guess at these and every downstream
artifact inherits the guess.

Do this before anything else:

1. **Research the real process.** Search when the domain has facts you don't hold — local
   regulations, industry workflow, standard document types, common software in the space.
   Cambodian rental practice differs from US; clinic triage differs by country; tax rules
   are jurisdiction-specific. Say when you searched and when you're recalling.
2. **Extract the vocabulary.** Collect the actual terms practitioners use: *tenant,
   landlord, lease, invoice, arrears, meter reading, deposit, notice period*. These become
   your class names. If you rename a domain term to something "cleaner," you've broken the
   link between model and reality — this is the ubiquitous language rule and it's not
   optional.
3. **Find the rules and the edge cases.** Every domain has them and they're where designs
   break: mid-month move-in proration, a tenant paying for someone else's unit, a shared
   meter, a lease renewed at a new rate, a refund after deposit deductions.
4. **Look at how existing systems solve it** — and where they're bad. Most rent apps handle
   the happy path and collapse on arrears tracking. Knowing this shapes your priorities.
5. **Report back and get corrected.** Present what you learned as a short domain brief and
   invite correction. The user usually knows the domain better than your research does.
   This step is cheap and prevents an entire wasted design.

State your confidence explicitly. "I'm confident about the invoicing flow; I'm guessing on
deposit rules in Cambodia and would want that confirmed" is worth more than a smooth
paragraph that hides the gap.

## Phase 1 — Requirements

Produce, in this order:

- **Problem statement** — one paragraph, in domain terms, naming who suffers what today.
- **Scope and non-scope.** The non-scope list is the more useful one. Write it.
- **Actors** — primary (initiate goals), supporting (external systems the design depends on),
  offstage (have an interest but don't interact: tax authority, owner). Actors are roles,
  not people; one human may be two actors.
- **Functional requirements** — numbered, each testable. "The system shall generate a monthly
  invoice per active lease on the billing date."
- **Non-functional requirements** — usability, reliability, performance, supportability, plus
  the ones people forget: concurrency, audit trail, data retention, offline behavior,
  localization (Khmer/English, Khmer numerals, riel/USD dual currency).
- **Assumptions and open questions** — an explicit list. Flag, never silently invent. Ask the
  one or two questions that would change the design; note the rest as assumptions.

## Phase 2 — Analysis (what the system does, not how)

**Use cases.** A use case is a goal that delivers value to an actor, not a screen and not a
CRUD row. "Manage Tenants" is not a use case; "Register a New Tenant to a Unit" is. Write a
use case diagram for coverage, then *fully dressed* text for the 3–5 critical ones:

```
Use Case: Record Rent Payment
Primary Actor: Landlord
Preconditions: Lease is active; invoice exists for the period
Main Flow:
  1. Landlord selects the outstanding invoice
  2. System displays amount due, including any arrears
  3. Landlord enters amount tendered and method
  4. System records payment, updates invoice status, issues receipt
Alternate Flows:
  3a. Amount < due → System records partial payment, invoice stays Open, arrears recalculated
  3b. Amount > due → System applies excess as credit to next period
Exceptions:
  *a. Lease terminated mid-flow → System blocks and explains
Postconditions: Ledger balanced; receipt retrievable
```

The alternates and exceptions are the point. A use case with only a main flow is a wish, not
an analysis.

**Domain model.** Conceptual classes only — no methods, no IDs, no foreign keys, no
framework. Attributes are the information the business cares about. Associations get names
and multiplicities in both directions. Do not skip multiplicities; they encode the rules
(`Lease 1 ── 0..* Invoice`, `Unit 1 ── 0..1 ActiveLease`).

Noun extraction is a starting heuristic, not a method. Filter aggressively: drop nouns that
are attributes of something else, drop synonyms, drop UI artifacts, and *add* the concepts
the user never said out loud — `LeaseTerm`, `BillingPeriod`, `LedgerEntry` are usually
missing from the description and essential to the model.

**System sequence diagrams** for the main flows — treat the system as a black box, show the
actor's events crossing the boundary. This is what tells you the system's real API.

## Phase 3 — Design (how)

Assign responsibility deliberately using GRASP, and say which principle drove each choice —
Information Expert, Creator, Controller, Low Coupling, High Cohesion, Polymorphism, Pure
Fabrication, Indirection, Protected Variations. "`Lease` computes its own prorated amount
because it holds the start date and rate — Information Expert" is a design decision. Silently
putting the method somewhere is not.

Then produce:

- **Design class diagram** — visibility, typed attributes, method signatures with parameters
  and return types, correct relationship notation (inheritance / composition / aggregation /
  plain association), multiplicity, navigability. Composition means lifecycle ownership; use
  it only when destroying the whole destroys the part.
- **Interaction diagrams** for the operations where collaboration is non-trivial. Skip them
  for a two-object hop.
- **State machine** for any object whose behavior depends on its lifecycle. `Invoice` (Draft →
  Issued → PartiallyPaid → Paid → Overdue → Void) and `Lease` (Draft → Active → Terminated →
  Archived) almost always need one, and the transitions are where the business rules live.
- **Layering** — domain layer with no framework imports, application/service layer,
  infrastructure (persistence, notification, payment), presentation. The domain layer must be
  testable with no database running. If it isn't, the layering is decorative.
- **Patterns, only where earned.** Repository for persistence boundary, Strategy for pricing
  rules that vary, Observer for notification, Factory where construction is genuinely complex.
  Naming a pattern you didn't need is a cost, not a credential — say why the simple version
  loses.

Design for change where change is likely and nowhere else. Ask which requirement is most
likely to shift in a year and put the seam there.

## Phase 4 — Build in vertical slices

One feature end to end — domain object, service, persistence, endpoint, UI, test — before
starting the next. A horizontal build (all entities, then all repositories, then all
services) has nothing runnable until the end and hides integration errors until they're
expensive.

Slice order: pick the slice that proves the riskiest assumption, not the easiest one. For a
rent system that's usually invoice generation with proration and arrears, not tenant CRUD.

Write the domain layer first and test it in isolation. If a domain rule needs a database to
test, it's in the wrong place.

## Diagram output

Default to **Mermaid** in a fenced block — it renders in most places and the user can edit
it. Use `classDiagram`, `sequenceDiagram`, `stateDiagram-v2`, `erDiagram`, `flowchart`.
Mermaid has no native use case diagram; approximate with a flowchart or state it plainly.

For **draw.io**, generate the `<mxGraphModel>` XML with explicit geometry — see
`references/diagram-syntax.md`. For **PlantUML**, use it when the user needs true UML use
case notation or precise stereotypes.

Always keep the diagram and the explanation separate. The diagram carries structure; the
prose carries the reasoning behind it.

## What consistently goes wrong

**Anemic domain model** — classes with only getters and setters and all logic in a
`ServiceManager`. That's procedural code in class-shaped clothing. Behavior belongs with the
data it operates on.

**Database-first pretending to be OOAD** — designing tables, then drawing a class per table.
The domain model and the schema are different artifacts with different pressures; derive the
schema from the model, not the reverse.

**CRUD as use cases** — "Add User, Edit User, Delete User, View User" tells you nothing about
what the business does. Use cases are goals.

**God class** — `SystemManager` or `MainController` that touches everything. Usually a
missing domain concept; find the concept.

**Noun soup** — 40 classes lifted from the description, half of which are attributes.

**Pattern stuffing** — Singleton, Factory, Observer, Strategy applied because they're in the
rubric. Each unearned pattern adds indirection a maintainer has to unwind.

**Fabricating domain rules** — inventing a deposit policy or a tax rate because the design
needed one. Mark it as an assumption and ask.

## Working style

Show the reasoning, not just the artifact. When the user's premise is weak — a use case that
isn't one, an entity that should be a value object, a requirement that contradicts another —
say so once, plainly, with the reason, then proceed with what they asked if they still want
it. Agreeing to be agreeable produces a design that fails at implementation, which is a
worse outcome than a moment of friction now.

Ask at most two clarifying questions, upfront, and only when the answer changes what you'd
produce. Everything else goes in the assumptions list.
