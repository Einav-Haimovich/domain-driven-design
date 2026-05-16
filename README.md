# Domain Driven Design — DomeGym

A gym management application built progressively through the concepts of Domain Driven Design. Each folder is a snapshot of the codebase at the end of a major course section, showing how the same system evolves from a flat domain model into a distributed bounded-context architecture.

---

**[domain-model](./domain-model)**

Models the gym domain as a flat set of entities and value objects with no layering. Entities include Gym, Room, Session, Subscription, Participant, Trainer, and Admin. Value objects (Schedule, TimeRange) encapsulate domain rules. Each entity owns its invariants as domain errors.

When it's useful: starting a new domain — capturing what the business *is* before worrying about infrastructure.

_Learned: domain logic lives in the entities themselves. Invariants (e.g. "a room cannot have overlapping sessions") are enforced inside the model, not in service layers, so they can never be bypassed._

---

**[aggregates](./aggregates)**

Reorganizes the flat domain into proper aggregates. Each entity type gets its own aggregate folder. Common base classes (AggregateRoot, Entity, ValueObject) are extracted to enforce identity-by-id equality and make the aggregate boundaries explicit. The Application layer introduces repository interfaces to decouple domain from persistence.

When it's useful: once the domain is understood — grouping entities that must change together into a single transactional boundary.

_Learned: the aggregate root is the only entry point for mutations. Grouping by aggregate also reveals which entities truly belong together vs. which are just loosely related._

---

**[clean-architecture](./clean-architecture)**

A full working API built on top of the aggregate model. Layers: Api (controllers), Application (CQRS commands and queries with handlers), Contracts (DTOs), Domain, and Infrastructure (persistence). HTTP request files are included for manual endpoint testing.

When it's useful: when you need to ship a deployable service that respects the domain model — the layers keep infrastructure concerns from leaking into domain logic.

_Learned: CQRS naturally maps to DDD — commands go through the domain model and enforce invariants, queries bypass it and go straight to the data store. This separation keeps both sides simple._

---

**[bounded-contexts](./bounded-contexts)**

Splits the monolith into four independent services: GymManagement, SessionReservation, UserManagement, and SharedKernel. Services communicate through integration events published via an outbox pattern. SharedKernel holds only the event contracts shared across boundaries.

When it's useful: when different parts of the system need to evolve, scale, or be owned independently.

_Learned: a bounded context is not just a microservice — it's a consistency boundary with its own language. Integration events (not direct calls) keep services decoupled; the outbox pattern ensures events are never lost even if the broker is temporarily unavailable._

---

## How to Run

Each section is a self-contained .NET solution. To build and test any section:

```bash
cd domain-model          # or aggregates, clean-architecture, bounded-contexts/<service>
dotnet build
dotnet test
```

For `clean-architecture` and each service under `bounded-contexts`, run the API with:

```bash
dotnet run --project src/<ProjectName>.Api
```

HTTP request files (`.http`) in the `Requests/` or `requests/` folders can be executed with the VS Code REST Client extension.

---

## Folder Structure

```
domain-model/           flat domain entities and value objects
aggregates/             domain reorganized into aggregate roots with base classes
clean-architecture/     full API: Api, Application, Contracts, Domain, Infrastructure
bounded-contexts/
  GymManagement/        gyms, rooms, subscriptions service
  SessionReservation/   sessions and reservations service
  UserManagement/       user profiles service
  SharedKernel/         shared integration event contracts
certificate/            course completion certificate
```

---

Thanks to Amichai Mantinband for the course — [Getting Started: Domain Driven Design](https://dometrain.com/course/getting-started-domain-driven-design-ddd/).

[Certificate of completion](certificate/Domain%20Driven%20Design%20-%20Einav%20Haimovich.pdf)
