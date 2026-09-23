# Design heuristics

Load before Phase 3. This is the decision material for placing behavior, choosing structure,
and knowing when a pattern has been earned.

---

## GRASP — where does this responsibility go?

Nine principles. In practice the first four answer most questions and the last five resolve
the hard cases.

| Principle | Question it answers | Applies when |
|---|---|---|
| **Information Expert** | Who has the data needed to do this? | Default. Behavior goes with the data it reads. |
| **Creator** | Who should instantiate this object? | The class that aggregates, contains, records, or holds the initializing data for it. |
| **Controller** | Who receives a system event from the UI? | One use-case controller or a facade — never the domain object, never the UI. |
| **Low Coupling** | Which placement creates fewer dependencies? | Tie-breaker between two otherwise valid placements. |
| **High Cohesion** | Does this class still do one thing? | Tie-breaker; the counterweight to Low Coupling. |
| **Polymorphism** | Behavior varies by type? | Replace a type-switch with overriding. |
| **Pure Fabrication** | No domain class fits without damaging cohesion? | Invent a non-domain class (`InvoiceRepository`, `ProrationCalculator`). Use sparingly — an over-fabricated design is an anemic one. |
| **Indirection** | How to decouple two things that shouldn't know each other? | Introduce an intermediary (repository, adapter, event bus). |
| **Protected Variations** | What will change? | Put a stable interface at the predicted variation point — and only there. |

**Worked example.** Who computes a first-month prorated rent?

- `Lease` holds `startDate`, `monthlyRate`, `billingDayOfMonth` → Information Expert says
  `Lease.calculateProration(period)`.
- `BillingService.prorate(lease, period)` would need to read all three off the lease, which is
  feature envy and duplicated lease-term knowledge.
- Decision: `Lease`. `InvoiceGenerationService` orchestrates (which leases, which period, what
  to do with the result) but does not compute.

**The rule that catches the most mistakes:** if a method reads mostly another object's data,
it belongs on that object.

---

## SOLID, stated as things to check

- **SRP** — one reason to change. `InvoiceService` that formats PDFs, emails tenants, and
  computes arrears has three.
- **OCP** — new variants added without editing existing code. Only worth engineering where
  variants genuinely keep arriving.
- **LSP** — a subclass must be usable wherever the parent is. If the subclass throws on an
  inherited operation, or narrows what the parent accepts, the hierarchy is wrong. Classic
  break: `Square extends Rectangle`.
- **ISP** — clients shouldn't depend on operations they don't call. A 20-method
  `IRepository` forces every implementer to care about all 20.
- **DIP** — the domain layer defines the interface; infrastructure implements it. If
  `domain/` imports the ORM, the dependency runs the wrong way.

---

## Entity vs. value object

| | Entity | Value object |
|---|---|---|
| Identity | Has one, independent of attributes | Defined entirely by its attributes |
| Equality | Same ID = same thing | Same values = same thing |
| Mutability | Changes over time, keeps identity | Immutable; change means replace |
| Examples | `Tenant`, `Lease`, `Invoice`, `Unit` | `Money`, `Address`, `DateRange`, `MeterReading`, `PhoneNumber` |

Test: "If two of these have identical attributes, are they the same thing?" Two tenants named
Sok with the same phone number are still two tenants — entity. Two `Money(50, USD)` are
interchangeable — value object.

Most models under-use value objects. `Money` as a bare `float` is the standard mistake: it
loses currency, invites rounding bugs, and scatters formatting logic. Same for a date range
modelled as two loose fields — `DateRange` can enforce `start ≤ end` once, in one place.

---

## Type vs. instance

When the domain has both an abstract description and the physical or scheduled things that
realize it, those are two classes. Collapsing them is one of the most common modeling errors,
and it shows up immediately: the attributes that belong to one copy have nowhere to live.

| Abstract type | Realized instance | Only the instance has |
|---|---|---|
| `Book` (title, author, ISBN) | `Copy` (barcode, shelf, condition) | Loan status, acquisition date, damage |
| `Product` | `SerialItem` / `StockItem` | Serial number, warehouse bin, expiry |
| `CourseDefinition` | `CourseOffering` | Term, instructor, enrolled students, room |
| `Route` | `Trip` | Departure time, vehicle, passengers |
| `MenuItem` | `PreparedDish` | Ticket, cook time, table |
| `ServiceType` (consultation) | `Appointment` | Doctor, slot, patient, outcome |

The test: **can two of them exist at once, and does the business track them separately?** A
library with three copies of the same title lends one specific copy, and the loan attaches to
that copy, not to the title. If the answer is yes, split.

It is legitimate to collapse them for a genuinely single-instance domain — say so explicitly
and record it as a decision, rather than leaving the reader to wonder whether you noticed.

---

## Refactoring an anemic model

The recurring rescue job. Symptoms: entities are field bags with accessors, a `*Service` holds
every operation, state is a free string, money is a number.

Work it in this order — each step is independently shippable:

1. **Name the current state.** "Anemic domain model: `Order` has no behavior; `OrderService`
   holds `placeOrder`, `cancelOrder`, `applyDiscount`, `markShipped`, `calculateTotal`."
   Naming it is not pedantry — it tells the reader the problem is known and bounded.
2. **Replace primitives with value objects first.** `total: number` becomes `Money`;
   `items: any[]` becomes `OrderLine[]`. This is low-risk, mechanical, and it makes the next
   steps possible because the types now carry meaning.
3. **Model the state machine.** A free-string `status` permits every illegal transition by
   construction: nothing stops `cancel()` on a shipped order, or a second `markShipped()`.
   Define the states and the legal transitions, and make the object reject the rest.
4. **Move behavior to the data.** `calculateTotal()` reads only the order's own lines →
   Information Expert puts it on `Order`. `applyDiscount()` mutates order state → `Order`.
   Each move cites its principle.
5. **Leave the service as an orchestrator.** What remains is genuinely application-layer:
   loading the aggregate, calling one domain method, persisting, publishing an event. If the
   service still holds a calculation after this, a domain concept is still missing.

What the service keeps vs. what the entity takes:

| Belongs on the entity | Belongs in the application service |
|---|---|
| Invariants and state transitions | Transaction boundary |
| Calculations over its own data | Loading and saving aggregates |
| Rejecting illegal operations | Calling external systems |
| Producing domain events | Mapping to and from DTOs |

---

## Aggregates

An aggregate is a cluster of objects treated as one unit for changes and invariants.

Rules:

1. **One root.** External references point at the root only; inner objects are reached through
   it.
2. **Invariants enforced inside.** If a rule must hold across two objects at all times, they
   belong in the same aggregate.
3. **Reference other aggregates by ID**, not by object pointer.
4. **One aggregate per transaction**, as a default. Cross-aggregate consistency becomes an
   event or a scheduled reconciliation.
5. **Keep them small.** A large aggregate is a contention point and usually a sign that two
   concepts got merged.

Rent-domain example: `Invoice` (root) owns its `LineItem`s and `PaymentAllocation`s — arrears
math is an invariant across them. `Lease` is a separate aggregate, referenced as `leaseId`,
because a lease's lifecycle is independent of any single invoice's.

---

## Pattern selection

State the problem before naming the pattern. If the plain version isn't clearly insufficient,
don't.

| Problem | Pattern | Not worth it when |
|---|---|---|
| Domain must not know about the database | Repository | The app is a single script against one table |
| A calculation varies by policy, and policies keep arriving | Strategy | There are two variants and always will be — use a conditional |
| Something must react when state changes, without the source knowing who | Observer / domain events | One known listener called directly |
| Construction is genuinely complex or conditional | Factory | A constructor with three arguments |
| Two incompatible interfaces must meet | Adapter | You control both sides and can just change one |
| An operation needs to be queued, logged, or undone | Command | It's a direct method call |
| A multi-step process with varying steps | Template Method / Strategy | The steps never vary |

Singleton is almost never the right answer in a domain model; it's global state with a
credential. Prefer one instance injected at composition.

---

## Design smells and what they usually mean

| Smell | Likely cause | Fix |
|---|---|---|
| Class is all getters/setters | Anemic model | Move the logic that mutates state onto the class |
| `XManager`, `XHelper`, `XUtil` growing | Missing domain concept | Name the concept and give it the behavior |
| `if (type == ...)` repeated | Missing polymorphism | Subtype or strategy |
| Long parameter list | Missing value object | Group the parameters into a type |
| Method reads another object's fields | Feature envy | Move the method there |
| Two classes always change together | Wrong boundary | Merge, or move the shared invariant |
| `status` is a free string | Missing state machine | Model states and legal transitions |
| Deep inheritance for reuse | Inheritance misuse | Compose instead |
| A class has no path back to a requirement | Speculative design | Justify or cut |
