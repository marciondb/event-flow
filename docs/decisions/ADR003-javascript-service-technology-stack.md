# JavaScript Service Technology Stack

---

| Field            | Value |
|------------------|-------|
| **ADR**          | ADR-0003 |
| **Author**       | Marcio Dias |
| **Contributors** | N/A |
| **Started at**   | 2026-01-14 |
| **Status**       | ACCEPTED |
| **Scope**        | JavaScript / TypeScript services only |
| **Description**  | This ADR documents the technology choices for backend services implemented in JavaScript/TypeScript within the EventFlow ecosystem. |

---

## Context

EventFlow backend services follow the **Service Design Principles** documented in `service-design-principles.md`, which define a strict separation between Domain (Core) and Boundary (External) components.

Services need:
- HTTP capabilities for synchronous APIs
- Event streaming integration (Kafka)
- Request/response validation
- Strong TypeScript support
- Transparency and control over implementation details

EventFlow is a **learning-oriented project**, prioritizing understanding over abstraction. The chosen technologies should expose how things work rather than hide complexity behind decorators or magic.

This decision applies **exclusively to JavaScript/TypeScript services**. Other language ecosystems (e.g., Python) will have their own technology stack decisions documented separately.

---

## Decision

EventFlow JavaScript/TypeScript services will use the following technology stack:

### HTTP Framework: Fastify

**Fastify** is adopted as the HTTP framework for services that expose synchronous APIs.

#### Rationale

- **Schema-first approach** aligns naturally with the Wire concept (input/output contracts)
- **Does not impose directory structure** — allows full implementation of Service Design Principles
- **Transparent** — minimal magic, explicit behavior
- **TypeScript-first** — excellent type inference and developer experience
- **Performance** — faster than Express with lower overhead
- **Plugin system** — organized but not mandatory
- **Mature and well-documented** — stable for production use

### Validation: Zod

**Zod** is adopted for schema definition and runtime validation.

#### Rationale

- **TypeScript-first** — types are inferred from schemas automatically
- **Runtime validation** — validates data at system boundaries
- **Composable** — schemas can be combined and reused
- **Integrates with Fastify** via `fastify-type-provider-zod`
- **Clear error messages** — useful for debugging and API consumers

### Integration: fastify-type-provider-zod

The **fastify-type-provider-zod** package connects Fastify and Zod, enabling:

- Automatic request validation (body, params, query, headers)
- Automatic error responses for invalid requests
- Type inference from Zod schemas to route handlers
- Response serialization validation

### Event Streaming: kafkajs

**kafkajs** is adopted for Kafka integration (already established in the service template).

---

## How This Maps to Service Design Principles

| Component | Technology |
|-----------|------------|
| `wire/in` | Zod schemas defining input contracts |
| `wire/out` | Zod schemas defining output contracts |
| `adapters` | Use `schema.parse()` to convert Wire ↔ Model |
| `diplomat/http` | Fastify routes with Zod validation |
| `diplomat/consumer` | kafkajs consumers |
| `diplomat/producer` | kafkajs producers |

The Wire layer becomes explicit Zod schemas. Validation happens automatically at the boundary, protecting the Domain from invalid data.

---

## Consequences

### Positive Consequences

- Clear mapping between architecture concepts and implementation
- Automatic validation reduces boilerplate in Adapters
- Strong type safety across the request lifecycle
- Schemas serve as documentation and contracts
- Learning-friendly — explicit behavior, no hidden magic

### Negative Consequences

- Additional dependency (`fastify-type-provider-zod`)
- Zod schemas require maintenance alongside TypeScript types
- Some duplication between Wire schemas and Model types (intentional separation)

These trade-offs are accepted as part of the project's learning objectives.

---

## Alternatives Considered

### NestJS

A full-featured, opinionated framework with decorators and dependency injection.

**Rejected because:**
- Imposes its own architectural patterns (modules, providers, controllers)
- Conflicts with Service Design Principles
- Hides behavior behind decorators
- Over-engineered for a learning-oriented project

### Express

Minimal, widely-adopted HTTP framework.

**Not chosen because:**
- Less modern than Fastify
- No native schema support
- Requires more boilerplate for validation
- Callback-based middleware (less clean than Fastify)

### No HTTP Framework (native http module)

Using Node.js native HTTP capabilities.

**Not chosen because:**
- Too low-level for services with multiple routes
- Significant boilerplate for common patterns
- Learning value diminishes after initial implementation

### Joi / Yup for Validation

Alternative schema validation libraries.

**Not chosen because:**
- Zod has better TypeScript integration
- Zod infers types automatically
- Zod has cleaner API for composition

---

## Scope Limitation

This ADR applies **only to JavaScript/TypeScript services**.

EventFlow may include services written in other languages in the future. Each language ecosystem will have its own technology stack decision documented in a separate ADR.

For example:
- **Python services** — separate ADR (planned)
- **Go services** — separate ADR (if applicable)

---

## Related

- Service Design Principles (`service-design-principles.md`)
- Service Bootstrap (`service-bootstrap.md`)
- ADR-0001 — Hybrid Communication Model
- ADR-0002 — Backend For Frontend as an Event-Driven Projection Layer
