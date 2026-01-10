# Service Design Principles

## Purpose

This document defines the **service-level architectural style** adopted by EventFlow services.

The architecture is inspired by concepts from Hexagonal Architecture and Clean Architecture, but follows a **more opinionated and explicit structure**, focused on:

- Clear separation of responsibilities
- Strict boundary between business logic and infrastructure
- High testability
- Predictable data flow
- Consistent service structure across the system

This document describes **roles and responsibilities**, not implementation details or language-specific patterns.

---

## Core Architectural Idea

Each service is structured around two clearly separated zones:

- **Domain (Core)** — business meaning and decision-making
- **Boundary (External)** — interaction with the outside world

The Domain is **protected and pure**.  

All side effects are **isolated in the Boundary**.

Dependencies always point **towards the Domain**.

---

## Domain Layer (Core)

The Domain layer represents the **business essence** of a service.  
It must be independent from transport, persistence, messaging, or infrastructure concerns.

### Domain Components
- Models: Pure data structures with strict schemas
- Logic: Pure functions implementing business rules
- Controllers: Orchestrate operations using logic sandwich pattern

---

### Model

**What it represents**

Models define the **internal data contracts** of the domain.

**Responsibilities**
- Describe domain data structures
- Express invariants and constraints
- Serve as the internal language of the business

**Rules**
- No behavior
- No dependencies
- Strict and explicit
- Independent from transport or storage formats

**Key principle**
> Models express *what the business understands*, not how data is received or stored.

---

### Logic

**What it represents**

Logic contains **pure business rules**.

**Responsibilities**
- Transform domain models into domain models
- Apply calculations, validations, and decisions
- Represent atomic business behaviors

**Rules**
- Knows only Models
- Has no side effects
- Deterministic and referentially transparent
- Independent from orchestration or infrastructure

**Key principle**
> Logic answers *“given this domain state, what is the correct outcome?”*

---

### Controller

**What it represents**

Controllers orchestrate **business workflows**.

**Responsibilities**
- Coordinate one use case or business flow
- Call Logic in the correct order
- Decide when external interactions are required
- Delegate side effects to the Boundary

**Rules**
- Knows Models and Logic
- Can depend on Boundary components
- Does not perform business calculations
- Focused on orchestration, not computation

**Key principle**
> Controllers coordinate *flows*; Logic performs *reasoning*.

---

## Boundary Layer (External)

The Boundary layer connects the Domain to the external world.

It absorbs all:
- I/O
- Side effects
- Infrastructure concerns
- External failures

### Boundary Components
- Wire
- Adapter
- Diplomat
- Consumers and Producers (event-based variants)

---

### Wire

**What it represents**

Wire defines **data contracts at the system boundary**.

**Responsibilities**
- Describe incoming and outgoing data shapes
- Represent transport-level formats (HTTP, events, persistence)

**Rules**
- Pure data definitions
- No dependencies
- Not reused inside the Domain
- Can be loose for input, strict for output

**Key principle**
> Wire represents *how the world speaks to the service*.

---

### Adapter

**What it represents**

Adapters translate between **external data (Wire)** and **internal data (Model)**.

**Responsibilities**
- Convert Wire → Model
- Convert Model → Wire
- Normalize and sanitize external data

**Rules**
- Knows Wire and Model
- Contains no business logic
- Side-effect free

**Key principle**
> Adapters protect the Domain from external complexity.

---

### Diplomat

**What it represents**

Diplomats orchestrate **external interactions**.

**Responsibilities**
- Handle HTTP requests and responses
- Interact with databases
- Call external services
- Publish or consume events
- Coordinate Adapters and Controllers

**Rules**
- Knows Wire, Adapter, and Controller
- Contains no business rules
- Owns all side effects

**Key principle**
> Diplomats speak to the world on behalf of the Domain.

---

### Consumers and Producers

Consumers and Producers are **event-based variants of Diplomats**.

**Consumers**
- Read messages from queues or streams
- Validate and adapt payloads into Models
- Trigger Controllers

**Producers**
- Emit events derived from domain decisions
- Translate Models into event payloads

The Domain never knows about messaging infrastructure.

---

## Dependency Rules (Strict)

| Component   | Can Depend On |
|------------|---------------|
| Model      | — |
| Logic      | Model |
| Controller | Model, Logic, Diplomat |
| Wire       | — |
| Adapter    | Wire, Model |
| Diplomat   | Wire, Adapter, Controller |

Any violation of these rules is considered **architectural drift**.

---

## Data Flow

A typical request follows this flow:
External Input
→ Wire (in)
→ Adapter
→ Model
→ Logic
→ Model
→ Adapter
→ Wire (out)
→ External Output


Data is validated and transformed **at the boundaries**, never inside the Domain.

---

## Error Handling Principles

- Domain errors are explicit and meaningful
- Technical errors are handled in the Boundary
- Validation happens as early as possible
- Business rules never depend on infrastructure failures

---

## Testing Strategy

- Logic is tested in isolation
- Adapters are tested as pure transformations
- Controllers are tested as orchestration units
- Diplomats are tested via integration tests
- External dependencies are mocked or simulated

---

## Common Pitfalls

Avoid:
- Business logic in Adapters or Diplomats
- Side effects inside Logic
- Transport-specific concerns in Models
- Leaking external naming into the Domain
- Direct infrastructure access from Controllers

---

## Summary

EventFlow services follow a **strict, opinionated service design style** where:

- The Domain is pure and stable
- The Boundary absorbs all external complexity
- Responsibilities are explicit and enforceable
- Data is translated at the edges
- Business logic remains isolated and testable

This approach enables scalable development, consistent service design, and long-term maintainability.


---
Directory Structure
```
src/
    models/
      customer.clj
    logic/
      customer.clj
    controllers/
      customer.clj
    diplomat/
      http_server.clj
      http_client.clj
      producer.clj
      consumer.clj
    adapters/
      customer.clj
    wire/
      in/
        customer.clj
      out/
        customer.clj
```