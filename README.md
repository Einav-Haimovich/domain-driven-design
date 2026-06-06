# Domain-Driven Design: Deep Dive

This repo documents a full DDD journey from scratch — starting with domain discovery techniques and progressing through strategic design (event storming, sub-domains, bounded contexts, context mapping) into tactical implementation. The practical vehicle throughout is DomeGym, a gym session scheduling and reservation system. The repository holds both the strategic artifacts (diagrams and domain documents produced during the discovery and design phases) and four progressive code snapshots of the DomeGym application as it evolves from a flat domain model into a distributed bounded-context architecture.

---

## Course Sections

### Warm Up

This section orients the course within the full DDD lifecycle, starting from the "zero point" where nothing is known about a domain and tracing the path through strategic and tactical phases until well-defined aggregates are ready to implement. The instructor contextualizes DomeGym as the practical vehicle — a gym session reservation platform with participants, trainers, rooms, and scheduling constraints — and explains that domain discovery is the first act of strategic design, surveying techniques (event storming, domain storytelling, brainstorming, user story mapping) and the three inputs that feed them: domain experts, stakeholders, and the current state of the system. It also establishes the two living documents that will be refined throughout all subsequent sessions: an Invariants Draft and a Ubiquitous Language Draft, and previews the key milestones ahead.

**What I learned:**
- Knowing DDD patterns (aggregates, entities, value objects) is necessary but not sufficient — the hard, non-trivial part is discovering the domain structure from scratch, which is what strategic design is actually for.
- Domain discovery is the deliberate first act of the strategic phase: you gather context before making any architectural decisions, not after.
- Domain experts are not just subject-matter generalists; they are role-specific (gym owners, trainers, participants, marketing specialists), and each reveals a different blind spot in your understanding.
- Invariants and ubiquitous language exist at two levels of refinement: high-level (discovered early, across the whole domain) and low-level (precise, scoped to a specific bounded context or aggregate). Conflating the two stages leads to premature design decisions.
- The artifacts produced during event storming sessions — sticky-note boards, invariant drafts, ubiquitous language drafts — are not the goal; the shared understanding built while creating them is.
- Ubiquitous language must be actively decided and documented during domain discovery sessions, because those sessions are the only time all relevant people are in the same room; deferring the vocabulary to later means losing the opportunity.

---

### Big Picture Event Storming — Introduction

This section was delivered as video-only content with no written transcript or artifacts.

---

### Big Picture Event Storming

This section introduces Event Storming as a collaborative domain discovery technique and walks through a full Big Picture Event Storming session using the DomeGym scheduling system as the working example. The instructor covers the nine sticky note types (domain events, commands, actors, systems, aggregates, read models, policies, hotspots, and opportunities), the three varieties of Event Storming (Big Picture, Process Modeling, and Software Design), and then simulates a live session in five rounds: chaotic brainstorming of domain events, enforcing a chronological timeline and adding swim lanes, identifying actors and systems, filling gaps via walkthrough and reverse narrative, and prioritizing hotspots with arrow voting. Throughout the session, conflicts over terminology are resolved into a shared Ubiquitous Language, and recurring business rules are captured as invariants.

**What I learned:**
- Big Picture Event Storming deliberately casts a wide net by inviting every affected stakeholder — legal, sales, end users, developers — because each perspective surfaces domain events and edge cases that technical teams alone would miss.
- Domain events must be framed in past tense and judged by business relevance, not implementation detail; sticky notes that describe HTTP requests or UI state changes belong in code, not on the event storming board.
- Hotspots and opportunities are two sides of the same friction point: what one stakeholder sees as a conflict can be reframed as a monetization or feature opportunity by another.
- Ubiquitous Language is not defined upfront but emerges from conflict — inconsistency between "member" and "participant" is what forces the team to agree on a single term and commit it to a shared document.
- Invariants surface naturally during the session and should be recorded immediately, before the constraint is buried in implementation assumptions.
- The five rounds are loose guidelines, not a strict protocol; the only non-negotiable goal is a comprehensive, shared mental map of the problem space that everyone in the room can reference after the workshop ends.

---

### Process Modeling Event Storming

This section zooms into a specific business process identified during big-picture event storming and models it in fine-grained detail using a structured notation. Working through the DomeGym example, the section introduces three new elements not present in big-picture event storming: commands (what an actor intends to do), policies (automated reactions to domain events that trigger further commands), and read models (the data a user observes before deciding to act). The session is run as a goal-oriented game: one command anchors the left edge of the board, one or more domain events anchor the right edge, and the team works through a strict colour-coded sequence to connect them. Two rounds are demonstrated — framing the problem by establishing context events and the desired end state, then rushing to the goal by laying down key domain events and filling in the gaps.

**What I learned:**
- Process modeling event storming is scoped to one process, which means the participant list should shrink to only the people with direct knowledge of and authority over that process.
- The strict colour sequence — read model, actor, command, system, domain event, optional policy — forces the team to make implicit causality explicit rather than hand-wavy.
- Placing a few key domain events in the middle of the board first, then filling in from both ends, is often easier than narrating the entire flow from left to right in one pass.
- A policy gives a name to automated business reactions: it records not just that something happens automatically, but why and under what condition, which is knowledge that otherwise lives only in someone's head.
- Hotspots are a pressure-release valve that let the team acknowledge complexity without being derailed by it; the discipline is in deferring, not ignoring.
- The sticky-note diagram has a near-direct mapping to eventual code: each system block foreshadows a bounded context, each command foreshadows a use-case handler, and each domain event foreshadows a domain object method.
- Naming disputes during the session are productive, not wasteful — resolving them in front of the group is how the ubiquitous language gets agreed on and written down.

---

### Strategic Phase — Sub-Domains

This section transitions the course from domain discovery into the first layer of strategic thinking: decomposing the large gym session scheduling domain into smaller, more independently reasoned parts called sub-domains. The instructor first anchors the concept by distinguishing the general English meaning of "domain" from its DDD-specific meaning, then introduces five practical techniques for identifying sub-domains: observing semantic boundaries, tracing core flows, recognizing expertise boundaries, watching data flow, and examining organizational ownership. The section closes by classifying sub-domains as core (the competitive differentiator), supporting (necessary utility but not a selling point), or generic (a solved problem better handled by an off-the-shelf service).

**What I learned:**
- Sub-domain boundaries emerge from knowing the domain deeply, not from applying a formula — the five techniques are indicators, not rules.
- A shift in vocabulary between two parts of a system is one of the most reliable early signals that a conceptual boundary exists.
- Core, supporting, and generic classification directly informs resource allocation: the generic domain is the one you should most seriously consider delegating to a third-party service rather than building in-house.
- The same domain can be viewed at different granularities — a sub-domain is simultaneously a domain in its own right and a part of a larger domain, so the classification depends on where you place the frame.
- Identifying sub-domains is a problem-space activity; it produces insight for strategic decisions but does not yet determine how software will be structured.
- A sub-domain can migrate between classifications over time, so the model should be treated as a living artifact.

---

### Strategic Phase — Bounded Contexts

This section marks the pivot from problem space to solution space. Bounded contexts are defined as logical boundaries within which a specific domain model applies and has meaning — outside that boundary, the same term can carry a completely different shape and set of behaviors. The instructor distinguishes bounded contexts from sub-domains (sub-domains decompose the problem, bounded contexts decompose the solution) and demonstrates how DomeGym's eight identified sub-domains are condensed into four bounded contexts for the initial iteration. The factors that drive these decisions are practical and explicit: available expertise, technology stack constraints, budget, data ownership patterns, and organizational structure.

**What I learned:**
- The relationship between sub-domains and bounded contexts is many-to-many — multiple sub-domains can be folded into one bounded context; the mapping is a strategic choice, not a structural law.
- Merging two sub-domains into one bounded context collapses their domain models, which has direct consequences for code: the resulting model must carry the combined responsibilities of both, and this trade-off should be deliberate.
- A bounded context is a logical boundary, not an architectural one — it can be implemented as a module in a monolith, a microservice, or a cluster of microservices; the concept does not prescribe deployment topology.
- Budget constraints and team composition are first-class inputs to bounded context design, not afterthoughts — deferring or merging contexts is a legitimate strategic choice when resources are limited.
- Identifying that a sub-domain is thin (has little business logic) is a valid reason to absorb it into a neighboring context rather than maintaining it as a separate boundary.
- The process is explicitly iterative: initial bounded context decisions will be revised once domain modeling begins and contradictions surface.

---

### Strategic Phase — Context Mapping

This section covers how to visualize and reason about the relationships between bounded contexts using context maps and named patterns. The instructor works through three team relationship types (mutually dependent, upstream/downstream, and free) and eight context map patterns (open host service, conformist, anti-corruption layer, shared kernel, partnership, customer/supplier, published language, and separate ways). The patterns are then applied to DomeGym by tracing a single end-to-end flow — from a user purchasing a subscription through creating a gym, a room, and a session, to a participant reserving a spot — exposing which bounded context owns each decision and what pattern best represents each relationship.

**What I learned:**
- A context map should answer a specific question or model a specific flow rather than attempting to capture all integrations between all bounded contexts at once, because a single all-encompassing map becomes unreadable and hard to maintain.
- When a downstream context will evolve its domain model independently of the upstream, an anti-corruption layer is the right choice even when the conformist pattern would be simpler, because the two models will diverge in responsibility and data over time.
- Choosing conformist over anti-corruption layer is justified when the upstream genuinely owns the concept and the downstream has little reason to extend or reinterpret it — the trade-off is coupling for simplicity.
- A shared kernel introduces immediate coupling between both bounded contexts because any change to the shared model affects both; keeping the kernel as small as possible limits this blast radius.
- Integration events are the mechanism by which an upstream change propagates state into downstream bounded contexts without requiring synchronous coupling.
- Enforcing a cross-context invariant requires a single domain model to be the authoritative point of truth; embedding the constraint inside one aggregate and updating it via integration events keeps that authority clear.
- Strategic design is iterative: once a draft system design exists, stepping back to weigh it against business goals and budget constraints is a necessary checkpoint, not an optional refinement.

---

### Tactical Phase

The tactical phase is where strategic decisions made during event storming and context mapping are translated into concrete, implementable domain objects. This section provides a refresher on the seven core tactical building blocks — entities, value objects, aggregates, domain services, factories, repositories, and domain events — and frames what the tactical phase is actually trying to accomplish: arriving at a domain model that is detailed enough to write code from. Crucially, the tactical phase is not a one-way process; getting hands dirty with aggregate design frequently surfaces overlooked constraints, sending the team back to revisit bounded context boundaries and realign with stakeholders.

**What I learned:**
- The tactical phase is iterative, not sequential — designing aggregates will reveal gaps in the strategic model and require looping back to refine bounded contexts and the ubiquitous language.
- Each bounded context has its own slice of the ubiquitous language; the same term can legitimately mean different things in different contexts, and the tactical phase is when those distinctions become concrete.
- Domain events are the sanctioned mechanism for one aggregate affecting another — they preserve the one-aggregate-per-transaction rule while allowing the system to remain reactive across boundaries.
- The distinction between a domain service and application logic hinges on whether the logic represents a business decision; if it does, it belongs in the domain layer regardless of which object hosts it.

---

### Tactical Phase — Aggregate Design

This section presents ten rules for sound aggregate design and a four-step process for moving from a set of identified entities to a settled aggregate model. The rules cover consistency responsibility (the aggregate root owns invariant enforcement), referencing discipline (aggregates reference each other only by ID), size preference (smaller is better for both concurrency and cost), and access control (internals should be as private as possible). The four steps — identify entities, map relationships, mark all entities as aggregate roots, then selectively merge based on invariants or intolerable eventual consistency — give a repeatable, systematic path from concept to implementable boundaries, grounded in Vaughn Vernon's Effective Aggregate Design series.

**What I learned:**
- Start every design with the smallest possible aggregates (each entity as its own root) and only merge when a concrete invariant cannot be enforced otherwise — this prevents premature coupling.
- Merging two aggregates should be triggered by one of exactly two forces: an invariant that requires both to be written atomically, or an eventual-consistency window the business cannot tolerate.
- Large aggregates are a concurrency liability: every process that touches any part of the aggregate competes for the same lock, so aggregate size directly determines how often writes collide.
- An entity inside an aggregate needs only local uniqueness; only the aggregate root needs global identity — conflating these two leads to over-engineering nested objects.
- The rule "one aggregate per transaction" is also a design signal: if you consistently need to write two aggregates together, that is evidence they should be one aggregate, not that the transaction boundary should expand.

---

### Aggregate Design In Action

This section walks through the full four-step aggregate design process applied to DomeGym's three bounded contexts — user management, gym management, and session reservation — using the domain's invariant list as the forcing function at every decision point. The process exposes several non-obvious choices: whether to propagate subscription-type data into a child aggregate versus merging boundaries across contexts, where to introduce a missing domain concept (the Schedule entity) rather than loading arbitrarily large aggregates, and how the final design maps directly to C# aggregate root classes with private fields, invariant-enforcing methods, and references held only as IDs.

**What I learned:**
- When an aggregate cannot enforce an invariant with the data it currently holds, the first question is not "what data should we add?" but "is there a missing domain concept that belongs in the model?" — the Schedule entity emerged precisely this way.
- Propagating derived data (such as max-sessions-per-day) from a parent aggregate to a child aggregate enables the child to enforce limits independently, but creates an eventual-consistency window whenever the source changes — whether that window is acceptable is a business decision, not a technical one.
- Merging aggregates across bounded contexts is possible by duplicating a slimmed-down projection of one aggregate into the other context; the trade-off is that the context boundary now owns more, but concurrency contention and cross-context calls are eliminated.
- Once an aggregate design begins causing frequent write collisions at scale, splitting it back apart is a legitimate refactoring path — the design is never final and should be reassessed as usage patterns evolve.
- A well-designed aggregate diagram is a one-to-one map to code: each aggregate root becomes a class, nested entities become private fields, cross-aggregate references become stored IDs, and invariants become guard clauses on the mutation methods — there is no translation gap between the design artifact and the implementation.

---

## Code Snapshots

The `domain-model/`, `aggregates/`, `clean-architecture/`, and `bounded-contexts/` folders are four progressive implementations of DomeGym, each building directly on the previous one.

### Exploring a Complex Domain

Working entirely in the `domain-model/` folder, this snapshot walks through three foundational DDD activities before writing a single line of production code — defining a ubiquitous language, identifying invariants, and choosing where to enforce them. The author maps out domain objects (Subscription, Gym, Room, Session, Trainer, Participant, Admin) and their relationships, lists the business rules that must always hold (no overlapping sessions per room, per trainer, or per participant; subscription tier caps on gyms, rooms, and daily sessions), then implements each rule directly inside the domain objects using TDD. A result pattern library (ErrorOr) replaces raw exceptions so that callers can distinguish which specific invariant was violated.

**What I learned:**
- Ubiquitous language should be written down explicitly before coding and used as the authoritative source for naming methods, fields, and domain events — inconsistencies in the written language reveal inconsistencies in the model itself.
- Invariants are business rules that must always be true, and DDD pushes their enforcement inside the domain objects rather than in outer orchestration layers, so each object is responsible for keeping itself in a valid state.
- Domain objects should start with everything private and read-only, exposing members only when something external demonstrably needs them — this limits the surface area you have to reason about.
- Writing a unit test per invariant before implementing the enforcement logic (TDD) organically grows the domain model: the tests define the expected vocabulary and the objects fill in to meet it.
- The result pattern (returning a typed success/error value instead of throwing exceptions) lets callers identify exactly which invariant was violated, and centralising error definitions in per-aggregate `*Errors.cs` files keeps those identifiers in one place.
- A `TimeRange` value object encodes the no-overlap rule as a reusable concept, so the same logic that prevents a room from double-booking can enforce a trainer's or participant's schedule without duplicating the comparison logic.

---

### Tactical Design and Patterns

The `aggregates/` folder reorganizes the flat domain into proper aggregate roots, introducing entities, value objects, and base classes for equality and identity. Each aggregate (Admin, Gym, Participant, Room, Session, Subscription, Trainer) has its own folder under `DomeGym.Domain`, alongside shared base classes in `Common/`. An Application layer is added alongside the domain, with interfaces for repositories and domain services.

**What I learned:**
- An entity is defined by a unique ID, not by its data; two entity instances with the same ID represent the same logical object regardless of differing field values, which is enforced by overriding `Equals` and `GetHashCode` in a shared base class.
- A value object is defined entirely by its properties and is immutable; mutating it means returning a new instance, which eliminates a whole class of bugs where shared state changes invisibly.
- Primitive obsession — storing a time window as two raw `TimeOnly` values — is a design smell; wrapping them in a value object moves the validation responsibility into the type itself, so any instance in memory is guaranteed valid.
- An aggregate is a consistency boundary: one or more domain objects that must be read, validated, and written as an atomic unit. The aggregate root is the only externally reachable entity, and all invariant enforcement happens through it.
- Prefer putting business logic inside domain objects; a domain service is warranted only when the logic genuinely spans multiple aggregates — and even then, the need for a service is often a signal that a missing domain concept should be modeled instead.
- The repository pattern belongs as close to its consumers as possible: if only the application layer needs it, define the interface there rather than in the domain layer, keeping the domain free of infrastructure concerns.

---

### Domain Events

The `clean-architecture/` folder wraps the domain in a fully integrated ASP.NET Web API (controllers, CQRS handlers via MediatR, EF Core repositories). This snapshot introduces domain events as a mechanism for handling cross-aggregate side effects without coupling application-layer command handlers. The `AggregateRoot` base class gains a `PopDomainEvents()` method; `SaveChangesAsync` drains those events into an HTTP-scoped queue; and `EventualConsistencyMiddleware` publishes the queued events after the response is sent, committing or rolling back everything in a single database transaction.

**What I learned:**
- Domain events represent something that already happened in the domain — they are named in the past tense and carry only the data required by their handlers, keeping the domain model honest about causality.
- The choice between orchestration and domain events is a deliberate trade-off: orchestration is simpler but couples concerns and blocks the caller; domain events enable eventual consistency but require a resilient error-handling strategy.
- Eventual consistency means the caller receives a success response before all downstream side effects are guaranteed to complete; systems adopting this model must accept temporary inconsistency and invest in retry and error recovery mechanisms.
- Harvesting events inside `SaveChangesAsync` ensures events are only enqueued after data is durably written — preventing handlers from acting on changes that could still be rolled back.
- A single database transaction wrapping the entire event-processing queue (committed only after all handlers succeed) restores atomicity at the infrastructure level even though the application layer sees eventual consistency.
- Multiple handlers can subscribe to the same domain event, letting each bounded area of the system react independently — avoiding a single handler that knows too much about the rest of the system.

---

### Integration Events

The `bounded-contexts/` folder splits DomeGym into three independent microservices — GymManagement, SessionReservation, and UserManagement — each with its own domain model, persistence, and API. Integration events carry state changes across bounded context boundaries via a message broker. The full lifecycle is implemented: a domain event raised inside an aggregate, an outbox-writer handler that persists the event atomically, a background publisher service that reads and forwards messages, and consumer background services in downstream contexts that deserialize and dispatch the event to local handlers.

**What I learned:**
- Integration events are domain events scoped to the system level: domain events enforce consistency within a bounded context, while integration events enforce it across bounded contexts. Choosing the right boundary determines which pattern applies.
- Aggregates raise domain events internally; a separate handler within the same bounded context is responsible for translating a domain event into an integration event. This keeps aggregates free of messaging infrastructure concerns.
- The outbox pattern resolves the dual-write problem by writing the integration event to an outbox table in the same transaction as the aggregate change. A background service then publishes those events asynchronously, ensuring no event is silently dropped if the service crashes between a write and a publish.
- Fan-out exchanges on the message broker allow a single published integration event to be routed to all bounded contexts simultaneously; each context consumes from its own queue and ignores events it does not handle.
- Composing behavior through cascading events keeps complex workflows manageable: handling a `RoomRemoved` integration event calls `Cancel` on each session, which raises a `SessionCanceled` domain event whose handlers then update participant and trainer schedules.
- Eventual consistency across bounded contexts increases the number of intermediate states and therefore the number of failure modes. Each event handler should address its specific error scenarios rather than relying on a single catch-all mechanism.

---

## How to Run

**Prerequisites:** [.NET SDK](https://dotnet.microsoft.com/download) (version pinned in each folder's `global.json`)

```bash
# Build and test any snapshot
cd domain-model    # or: aggregates, clean-architecture
dotnet build
dotnet test

# Run the clean-architecture API
cd clean-architecture
dotnet run --project src/DomeGym.Api

# Run bounded-context services (each in a separate terminal)
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
warm-up/                             domain discovery artifacts: ubiquitous language, invariants, warmup diagram
big-picture-event-storming/          big picture event storming board and domain artifacts
process-modeling-event-storming/     process modeling board and accumulated domain artifacts
strategic-bounded-contexts/          bounded context analysis artifacts
strategic-context-mapping/           context mapping diagram and complete artifact set
aggregate-design-in-action/
  aggregate-design-figma.png         aggregate design diagram
  dome-gym/                          author's reference implementation at aggregate design stage
domain-model/                        code snapshot 1 — flat domain model with TDD
aggregates/                          code snapshot 2 — aggregate roots, entities, value objects
clean-architecture/
  src/
    DomeGym.Api/                     REST controllers
    DomeGym.Application/             CQRS commands, queries, domain event handlers (MediatR)
    DomeGym.Contracts/               request/response DTOs
    DomeGym.Domain/                  aggregate roots, entities, value objects, domain errors
    DomeGym.Infrastructure/          EF Core persistence, outbox pattern
  tests/
    DomeGym.Domain.UnitTests/
  Requests/                          .http files for manual endpoint testing
bounded-contexts/
  GymManagement/                     gyms, rooms, subscriptions
  SessionReservation/                sessions and reservations
  UserManagement/                    user profiles, admin/trainer registration
  SharedKernel/                      shared integration event contracts
certificate/
```

---

Thanks to Amichai Mantinband for the course — [Deep Dive: Domain Driven Design](https://dometrain.com/course/deep-dive-domain-driven-design-ddd/).

[Certificate of completion](certificate/Domain%20Driven%20Design%20-%20Deep%20Dive%20-%20Einav%20Haimovich.pdf)
