# Domain Driven Design

This repo follows the construction of DomeGym, a gym session scheduling and booking system, as a vehicle for learning Domain-Driven Design. The codebase is organized as four progressive snapshots — from a flat domain model through aggregates, clean architecture, and finally a distributed system of bounded contexts — each building directly on the previous one. The course also covers strategic and conceptual DDD topics (strategic design, bounded contexts, key modeling rules) that have no corresponding code snapshot but are documented in the Concepts Reference section below.

---

## DDD Foundations

### Introduction to Domain-Driven Design

This section establishes the foundational concepts of Domain-Driven Design before any code is written. It defines what DDD is — an approach for designing software so that the codebase closely mirrors the real-world problem space it addresses — and explains why that alignment matters for maintainability and team communication. The section clarifies the meaning of "domain" in the context of software (the sphere of knowledge and activity around which application logic revolves), illustrates how large domains decompose into progressively more specific sub-domains, and motivates the practice by pointing to the pain of codebases that drift away from the problem they model. It then introduces the two pillars of DDD: strategic design, which is the high-level activity of mapping and decomposing the problem space, and tactical design, which is the lower-level activity of translating a single domain into concrete classes and relationships.

**What I learned:**
- Domain-Driven Design is fundamentally about closing the gap between the language and structure of the code and the language and structure of the business problem it solves.
- A domain is not a fixed boundary; it is a nested hierarchy of increasingly specific spheres of knowledge, and understanding where your system sits within that hierarchy shapes every design decision.
- Business rules are domain decisions, not implementation details, and they belong in the model — not in service layers where they can be bypassed.
- Strategic design addresses the "what" — identifying sub-domains, their importance, and their relationships — before any code is written, and this phase is often neglected despite being the most valuable part of DDD.
- Tactical design addresses the "how" — defining classes, their states, and their invariants within a single bounded domain once strategic decisions are in place.

---

## Code Progression

### Exploring a Complex Domain

This section introduces the domain the course builds throughout: DomeGym, a gym session scheduling and booking system. Working entirely in the `domain-model/` folder in the repo, the section walks through three foundational DDD activities before writing a single line of production code — defining a ubiquitous language, identifying invariants, and choosing where to enforce them. The author maps out domain objects (Subscription, Gym, Room, Session, Trainer, Participant, Admin) and their relationships, lists the business rules that must always hold (no overlapping sessions per room, per trainer, or per participant; subscription tier caps on gyms, rooms, and daily sessions), then implements each rule directly inside the domain objects using TDD. A result pattern library (ErrorOr) replaces raw exceptions so that callers can distinguish which specific invariant was violated.

**What I learned:**
- Ubiquitous language should be written down explicitly before coding and used as the authoritative source for naming methods, fields, and domain events — inconsistencies in the written language reveal inconsistencies in the model itself.
- Invariants are business rules that must always be true, and DDD pushes their enforcement inside the domain objects rather than in outer orchestration layers, so each object is responsible for keeping itself in a valid state.
- Domain objects should start with everything private and read-only, exposing members only when something external demonstrably needs them — this limits the surface area you have to reason about.
- Writing a unit test per invariant before implementing the enforcement logic (TDD) organically grows the domain model: the tests define the expected vocabulary and the objects fill in to meet it.
- The result pattern (returning a typed success/error value instead of throwing exceptions) lets callers identify exactly which invariant was violated, and centralising error definitions in per-aggregate `*Errors.cs` files keeps those identifiers in one place.
- A `TimeRange` value object encodes the no-overlap rule as a reusable concept, so the same logic that prevents a room from double-booking can enforce a trainer's or participant's schedule without duplicating the comparison logic.

---

### Tactical Design and Patterns

This section covers the core building blocks used to model a domain in code, collectively known as tactical patterns. Working from the DomeGym system as a concrete example, the section walks through entities (objects with a stable identity), value objects (objects defined purely by their attributes), and aggregates (clusters of domain objects that must stay consistent as a whole). It explains how these patterns are structured in code using base classes for equality and identity, how aggregate roots serve as the sole entry point into a consistency boundary, and how supporting patterns — domain services, factories, and repositories — fill the gaps where domain object logic alone is insufficient. All of this is implemented in the `aggregates/` folder in the repo, where each aggregate (Admin, Gym, Participant, Room, Session, Subscription, Trainer) has its own folder under `DomeGym.Domain`, alongside shared base classes in `Common/`.

**What I learned:**
- An entity is defined by a unique ID, not by its data; two entity instances with the same ID represent the same logical object regardless of differing field values, which is enforced by overriding `Equals` and `GetHashCode` in a shared base class.
- A value object is defined entirely by its properties and is immutable; mutating it means returning a new instance, which eliminates a whole class of bugs where shared state changes invisibly.
- Primitive obsession — storing a time window as two raw `TimeOnly` values — is a design smell; wrapping them in a value object moves the validation responsibility into the type itself, so any instance in memory is guaranteed valid.
- An aggregate is a consistency boundary: one or more domain objects that must be read, validated, and written as an atomic unit. The aggregate root is the only externally reachable entity, and all invariant enforcement happens through it.
- Prefer putting business logic inside domain objects; a domain service is warranted only when the logic genuinely spans multiple aggregates — and even then, the need for a service is often a signal that a missing domain concept should be modeled instead.
- The repository pattern belongs as close to its consumers as possible: if only the application layer needs it, define the interface there rather than in the domain layer, keeping the domain free of infrastructure concerns.

---

### Domain Events

This section introduces domain events as a mechanism for handling cross-aggregate side effects without tightly coupling application-layer command handlers. Working in the `clean-architecture/` folder in the repo, the section first walks through a fully integrated ASP.NET Web API wrapping the domain layer (controllers, application handlers, infrastructure repositories), then demonstrates a concrete failure: creating a subscription stores the admin's reference but never persists the subscription itself. That bug motivates the contrast between orchestration — where a single handler chains every side effect — and the domain events pattern, where an aggregate raises a notification at the moment something meaningful occurs, and decoupled handlers process it afterward. The `AggregateRoot` base class gains a `PopDomainEvents()` method; `SaveChangesAsync` drains those events into an HTTP-scoped queue; and `EventualConsistencyMiddleware` publishes the queued events after the response is sent, committing or rolling back everything in a single database transaction.

**What I learned:**
- Domain events represent something that already happened in the domain — they are named in the past tense and carry only the data required by their handlers, keeping the domain model honest about causality.
- The choice between orchestration and domain events is a deliberate trade-off: orchestration is simpler but couples concerns and blocks the caller; domain events enable eventual consistency but require a resilient error-handling strategy.
- Eventual consistency means the caller receives a success response before all downstream side effects are guaranteed to complete; systems adopting this model must accept temporary inconsistency and invest in retry and error recovery mechanisms.
- Harvesting events inside `SaveChangesAsync` ensures events are only enqueued after data is durably written — preventing handlers from acting on changes that could still be rolled back.
- A single database transaction wrapping the entire event-processing queue (committed only after all handlers succeed) restores atomicity at the infrastructure level even though the application layer sees eventual consistency.
- Multiple handlers can subscribe to the same domain event, letting each bounded area of the system react independently — avoiding a single handler that knows too much about the rest of the system.

---

### Integration Events

This section introduces integration events as the mechanism for propagating state changes across bounded context boundaries in the `bounded-contexts/` folder of the repo. Where domain events handle eventual consistency within a single bounded context, integration events extend that pattern to the system level: when something meaningful occurs in one bounded context that another needs to react to, an integration event carries that notification across the boundary via a message broker. The course walks through the full lifecycle — from a domain event raised inside an aggregate, through an outbox-writer handler that persists the event atomically, to a background publisher service that reads and forwards messages, and finally to consumer background services in downstream contexts that deserialize and dispatch the event to local handlers.

**What I learned:**
- Integration events are domain events scoped to the system level: domain events enforce consistency within a bounded context, while integration events enforce it across bounded contexts. Choosing the right boundary determines which pattern applies.
- Aggregates raise domain events internally; a separate handler within the same bounded context is responsible for translating a domain event into an integration event. This keeps aggregates free of messaging infrastructure concerns.
- The outbox pattern resolves the dual-write problem by writing the integration event to an outbox table in the same transaction as the aggregate change. A background service then publishes those events asynchronously, ensuring no event is silently dropped if the service crashes between a write and a publish.
- Fan-out exchanges on the message broker allow a single published integration event to be routed to all bounded contexts simultaneously; each context consumes from its own queue and ignores events it does not handle. This decouples producers from consumers structurally.
- Composing behavior through cascading events keeps complex workflows manageable: handling a `RoomRemoved` integration event calls `Cancel` on each session, which raises a `SessionCanceled` domain event whose handlers then update participant and trainer schedules.
- Eventual consistency across bounded contexts increases the number of intermediate states and therefore the number of failure modes. Each event handler should address its specific error scenarios rather than relying on a single catch-all mechanism.

---

## Concepts Reference

### Key Domain Modeling Rules

This section addresses the practical rules that govern how domain objects relate to each other, how they are stored and deleted, and how they reference one another across aggregate boundaries. The author frames domain modeling as an inherently iterative process where perfect design is unattainable upfront, then works through four concrete rules: identity uniqueness scoping, cross-aggregate deletion via events, one aggregate per transaction, and reference-by-ID rather than direct object references. Together these rules prevent tight coupling between aggregates and make the domain easier to evolve and relocate across bounded contexts over time.

**What I learned:**
- An aggregate's identifier must be unique across the entire system, whereas an entity's identifier only needs to be unique within its parent aggregate. To uniquely identify a nested entity, you need the combination of its own ID and its parent aggregate's ID.
- Aggregates should carry no database-level foreign key relationships to other aggregates. When deleting an aggregate, side effects on other aggregates must propagate through domain events, not cascading database constraints — this is what makes aggregates safely portable between bounded contexts.
- Prefer one aggregate per transaction. Changes that affect multiple aggregates should be driven by domain events handled eventually, keeping the initial transaction small and avoiding timeouts or race conditions from large blocking operations.
- Smaller aggregates are preferable to larger ones because every change requires reading and writing the whole aggregate; a larger aggregate increases data transfer costs and raises the probability of concurrent modification conflicts.
- Aggregates reference other aggregates only by ID, never by holding a direct object reference. Strongly typed ID value objects are a common pattern to make this explicit and prevent accidental coupling.
- Never reference a nested entity from outside its owning aggregate. If another aggregate needs to interact with what is currently an entity, promote that entity to an aggregate root and reference it by ID.

---

### Key Concepts

This section covers several foundational DDD concepts that shape how you model, structure, and reason about a domain. It draws a clear line between domain logic and application logic, explains two consistency models for coordinating changes across aggregates, contrasts anemic and rich domain models, discusses whether a domain object should ever exist in an invalid state, and introduces persistence ignorance as a design discipline.

**What I learned:**
- Domain logic is logic that reflects business rules and is often state-dependent; if removing it would degrade the user's experience of the product itself, it belongs in the domain, not the application layer.
- Transactional consistency (all-or-nothing within a single operation) is simpler to reason about but forces callers to wait, while eventual consistency trades immediate coherence for performance and the freedom to apply robust retry strategies without a client waiting online.
- A practical rule of thumb from Vaughn Vernon: keep changes within a single aggregate transactionally consistent, and propagate all cross-aggregate side effects through eventual consistency, because the aggregate already contains everything needed to make one specific business decision.
- Anemic domain models expose public getters and setters and push business logic into application services, so no part of the system can trust that an object's data is valid; rich domain models invert this by hiding internal state and exposing only intentional behavior, guaranteeing validity inside the object itself.
- An always-valid domain model ensures that any property you access on a constructed object is valid at all times; the preferred approach combines always-valid constraints with a factory or static factory method to collect and surface construction errors.
- Persistence ignorance means modeling the domain purely around business rules, without letting the choice of database technology influence object design; the repository pattern supports this by presenting an in-memory illusion of how domain objects are accessed and stored.

---

### Strategic Design

Strategic design is the practice of zooming out from individual domain models to map the large-scale structure of an entire system. Rather than asking how to model a single concept, it asks how to carve a complex business domain into meaningful sub-domains and understand the relationships between them. The section covers three tools: identifying sub-domains and classifying them by business importance, using context mapping to visualize how different parts of the system relate to one another, and applying named patterns (conformist, open host service, anti-corruption layer, customer-supplier) to describe those relationships precisely. It also establishes that strategic and tactical design are not sequential phases but an ongoing back-and-forth, and that domain experts and practices like event storming are essential sources of knowledge when exploring an unfamiliar domain.

**What I learned:**
- Strategic design is about breaking a large domain into smaller sub-domains before modeling any of them in code, which means the structure of the system is a design decision, not an emergent accident.
- Sub-domains have three types with different business weight: the core sub-domain (the money-maker that deserves the most investment), the supporting sub-domain (necessary but not differentiating), and the generic sub-domain (a solved problem that should be bought or outsourced rather than built).
- The same concept can carry a different meaning in different contexts — a "room" in gym management and a "room" in session reservation are distinct models that share a name, and keeping that distinction explicit prevents confusion.
- Context map patterns give teams a shared vocabulary for the relationships between bounded contexts, distinguishing a conformist (one context fully adopts another's model) from a customer-supplier (the upstream has responsibility toward its downstream consumer).
- Team structure and context structure mirror each other: the degree of coordination a team needs reflects the coupling between the bounded contexts they own, and mapping those relationships clarifies who bears responsibility for breaking changes.
- Strategic and tactical design are iterative, not linear — discoveries made during implementation feed back into strategic decisions, making domain exploration a continuous process rather than a one-time upfront exercise.

---

### Bounded Contexts

This section introduces bounded contexts as a core concept in DDD's strategic design phase. A bounded context is a logical boundary within which a domain model has a specific, consistent meaning. Using the DomeGym application as a running example, the section walks through the distinction between problem space and solution space — subdomains belong to the problem space (how we understand the challenge), while bounded contexts belong to the solution space (how we choose to solve it). The instructor then maps DomeGym's three bounded contexts — user management, gym management, and session reservation — and shows how the same term (such as "user" or "room") can exist in multiple contexts with entirely different properties and responsibilities. The section closes by introducing context maps to visualize upstream/downstream relationships, and how an anti-corruption layer protects each context from being shaped by another context's domain model.

**What I learned:**
- Subdomains and bounded contexts are not the same thing: subdomains live in the problem space, while bounded contexts live in the solution space, and there is no mandatory one-to-one mapping between them.
- The same term can legitimately mean different things in different bounded contexts — a "user" in user management holds profile IDs and permissions, while a "user" in billing holds payment details — and this is a feature, not a flaw.
- Bounded contexts are a design decision, not a discovery: you choose where to draw logical boundaries based on how tightly domain models need to be coupled, and that choice can change as the system grows.
- A bounded context does not require its own deployment unit; it can be a namespace or module within a monolith, though in practice the industry tends to align each bounded context with a microservice or modular-monolith module.
- Upstream/downstream relationships between bounded contexts reveal integration obligations: the upstream publishes events and the downstream must consume them, and mapping these relationships early exposes which contexts are independent and which carry infrastructure responsibilities.
- An anti-corruption layer in the downstream bounded context translates an incoming domain model into the downstream's own terms, shielding it from changes in the upstream's representation.

---

## How to Run

### Prerequisites
- [.NET SDK](https://dotnet.microsoft.com/download) (version pinned in each folder's `global.json`)

### Build and test any snapshot
```bash
cd domain-model    # or: aggregates, clean-architecture
dotnet build
dotnet test
```

### Run the clean-architecture API
```bash
cd clean-architecture
dotnet run --project src/DomeGym.Api
```

### Run a bounded-context service
```bash
cd bounded-contexts/GymManagement
dotnet run --project src/GymManagement.Api

cd bounded-contexts/SessionReservation
dotnet run --project src/SessionReservation.Api

cd bounded-contexts/UserManagement
dotnet run --project src/UserManagement.Api
```

HTTP `.http` files in the `Requests/` folders can be run with the VS Code [REST Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) extension.

---

## Folder Structure

```
domain-model/                   flat domain: entities, value objects, no layering
aggregates/                     domain reorganized into aggregate roots + Application layer
clean-architecture/
  src/
    DomeGym.Api/                REST controllers
    DomeGym.Application/        CQRS commands, queries, domain event handlers (MediatR)
    DomeGym.Contracts/          request/response DTOs
    DomeGym.Domain/             aggregate roots, entities, value objects, domain errors
    DomeGym.Infrastructure/     EF Core persistence, outbox pattern
  tests/
    DomeGym.Domain.UnitTests/
  Requests/                     .http files for manual endpoint testing
bounded-contexts/
  GymManagement/                gyms, rooms, subscriptions
  SessionReservation/           sessions and reservations
  UserManagement/               user profiles, admin/trainer registration
  SharedKernel/                 shared integration event contracts
certificate/
  Domain Driven Design - Einav Haimovich.pdf
```

---

Thanks to Amichai Mantinband for the course — [Getting Started: Domain Driven Design](https://dometrain.com/course/getting-started-domain-driven-design-ddd/).

[Certificate of completion](certificate/Domain%20Driven%20Design%20-%20Einav%20Haimovich.pdf)
