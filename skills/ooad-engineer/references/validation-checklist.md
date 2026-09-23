# Validation checklist

Run this before presenting a final design. It is a self-review gate, not a formality — each
item corresponds to a failure that shows up at implementation time.

Fix what it catches. If an item fails and you're keeping the design anyway, say so explicitly
and give the reason.

---

## Depth and entry fit

- [ ] The entry point was identified: greenfield, feature addition, existing codebase, or a
      single named artifact
- [ ] Output depth matches the ask — no SRS in answer to "draw me a class diagram"
- [ ] Full depth was staged, not dumped: Phase 0–1 delivered and checkpointed before the
      design
- [ ] Feature addition reuses the existing class names and designs only the delta
- [ ] Existing-codebase work diagnosed the current model, named the smells, and stated the
      concrete consequences before proposing a replacement
- [ ] Any escalation beyond the requested depth is stated with a reason, not done silently

## Domain quality

- [ ] Class and use-case names use practitioners' vocabulary, not generic software nouns
- [ ] No `User` / `Product` / `Order` / `Item` unless the domain genuinely uses those words
- [ ] The main real-world process is represented end to end, not just the happy path
- [ ] Every business document and record in the domain maps to something in the model
- [ ] Edge cases from the domain brief are handled somewhere, not silently dropped
- [ ] Every business rule carries an evidence label, with H/M/L on inferred and assumed rules
- [ ] No `[Assumed]` is presented as fact
- [ ] No regulatory number — retention period, tax rate, statutory notice, licence threshold —
      is stated as `[Known]` without a citation
- [ ] Nothing was researched-and-stated without a source, and nothing time-sensitive is
      asserted from memory

## Requirements quality

- [ ] Every functional requirement is testable and has an acceptance condition
- [ ] Out-of-scope list exists and is specific
- [ ] Actors are classified (primary / supporting / offstage) and are roles, not people
- [ ] Business rules have IDs and are referenced by the use cases that enforce them
- [ ] No requirement contradicts another
- [ ] Non-functional requirements are the ones that actually apply — concurrency, audit trail,
      retention, offline, localization, currency were considered
- [ ] High-impact open questions are flagged, not resolved by quiet assumption
- [ ] A role × action permission matrix exists when the system has more than one role, with
      every cell filled and conditional access qualified
- [ ] Every kind of stored data is classified; sensitive data has an audit requirement written
      as an FR with a BR behind it, and an immutable audit concept in the model
- [ ] Monetary amounts carry a currency; multi-currency transactions store the exchange rate
      used at transaction time, and the rounding rule is stated per currency

## Analysis quality

- [ ] Use cases are actor goals, not screens and not CRUD rows
- [ ] The critical 3–5 are fully dressed with extensions and exceptions
- [ ] Each fully dressed use case cites the BR IDs it enforces
- [ ] Domain model is conceptual: no IDs, FKs, repositories, DTOs, controllers, ORM
- [ ] Every association has a name and multiplicities on both ends
- [ ] Multiplicities match the stated business rules — check each one against its BR
- [ ] Concepts that were implicit in the description were added (billing period, ledger entry,
      allocation, term)
- [ ] No class is really an attribute of another class
- [ ] Type and instance are separated where the business tracks copies individually
      (`Book`/`Copy`, `Product`/`SerialItem`, `CourseDefinition`/`CourseOffering`), or the
      collapse is stated as a deliberate decision
- [ ] Synonyms are not modelled twice
- [ ] System sequence diagrams exist for the major use cases and the system operations fall
      out of them

## Design quality

- [ ] Every non-obvious responsibility placement names the GRASP principle behind it
- [ ] No class holds only getters and setters while a service holds its logic
- [ ] No `Manager` / `Helper` / `Util` class is accumulating unrelated responsibilities
- [ ] Value objects are used where identity doesn't matter — money, date ranges, addresses,
      readings
- [ ] Aggregate roots identified; cross-aggregate references are by ID
- [ ] Composition used only where destroying the whole destroys the part
- [ ] Inheritance used only for substitutability, and every subclass satisfies LSP
- [ ] Method signatures are complete: parameters and return types, not bare names
- [ ] Objects with meaningful lifecycles have state machines, with triggers, guards, and side
      effects per transition — no free-string `status`
- [ ] Domain layer imports no framework, database, or HTTP library
- [ ] Every pattern used states the problem, why the simple version loses, and the trade-off
- [ ] Decision log covers the choices a reviewer would otherwise have to reverse-engineer

## Traceability

- [ ] Full depth only — at Quick and Standard, BR IDs cited inline are enough
- [ ] Every critical FR has a complete path: FR → use case → BR → responsibility → class /
      operation → test
- [ ] Every design class traces back to at least one requirement
- [ ] Classes with no requirement behind them are justified in the decision log or removed
- [ ] Assumption log is current — anything resolved during design is updated, not left stale

## Output discipline

- [ ] Diagram count is driven by understanding, not by artifact quota
- [ ] Diagrams and reasoning are in separate sections
- [ ] Diagram syntax is valid and renders (check against `references/diagram-syntax.md`)
- [ ] Build order is stated, highest-risk slice first, with the reason
- [ ] Confidence is stated where it's low, and the gap is named specifically
