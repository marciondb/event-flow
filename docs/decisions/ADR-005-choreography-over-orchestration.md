# Choreography over Orchestration

| Field            | Value |
|------------------|-------|
| **ADR**          | ADR-005 |
| **Author**       | Marcio Dias |
| **Contributors** | N/A |
| **Started at**   | 2026-01-15 |
| **Status**       | ACCEPTED |
| **Description**  | This ADR documents the decision to coordinate EventFlow services through **choreography** (services react to events independently) rather than a central **orchestrator**, and defines when that decision should be revisited. |

---

## Context

EventFlow's action path is event-driven end to end (ADR-001): the Event Ingestion Service publishes a trigger event, the Workflow Execution Service reacts to it, and the BFF reacts to lifecycle events. With multiple services reacting to a shared event stream, there are two ways to coordinate a multi-service flow:

- **Orchestration** — a central component explicitly tells each service what to do next and tracks the overall progress (command-driven).
- **Choreography** — each service knows how to react to the events it cares about and emits its own events; the end-to-end flow **emerges** from those local reactions, with no central conductor.

The choice shapes coupling, failure modes, and how the system evolves. It must be decided explicitly, because drifting between the two produces the worst of both.

It is important to separate two levels:

- **Within a single service**, a Controller orchestrates its own steps (the "logic sandwich" in the Service Design Principles). That is *local* orchestration and is unchanged by this ADR.
- **Across services**, coordination is what this ADR decides.

---

## Decision

EventFlow coordinates **cross-service** flows through **choreography**:

- Services **react to events** and **emit events**; they do not send commands instructing other services what to do.
- There is **no central workflow orchestrator** across services in v1.
- The end-to-end automation flow (trigger → execution → projection) emerges from each service reacting to the relevant events on the backbone (ADR-004).
- Integration knowledge is **local**: a service only needs to know which events it consumes and which it produces — not the global topology.

Local, in-process orchestration **within** a service (a Controller sequencing its own steps) remains the norm and is not affected.

---

## Consequences

### Positive

- **Loose coupling**: services depend on event contracts, not on each other's APIs or presence
- **Independent evolution and deployment**: a new consumer can be added without touching producers
- **Fault isolation**: a slow or down consumer does not block producers or other consumers
- **Naturally event-driven**: the model reinforces events as the backbone rather than bolting events onto an RPC core

### Negative

- **No single place** shows the end-to-end flow — understanding it requires reading multiple services and the topic map
- **Distributed visibility**: cross-service progress must be reconstructed (correlation IDs + the BFF projection, per ADR-004)
- **Harder multi-step consistency**: flows needing coordinated rollback across services have no built-in coordinator (acceptable in v1, which has single-step executions and no cross-service compensation)

These trade-offs are accepted for v1's scope.

---

## When to Revisit

Choreography is the right default for the current scope, but this decision should be **reconsidered** if/when EventFlow needs:

- Multi-step workflows spanning several services with **compensation/rollback** (a Saga) — at which point an **orchestrated Saga** (or an explicit process manager) may be justified for those specific flows
- Strong, centralized **visibility/SLA tracking** of long multi-service flows
- Complex conditional branching that becomes hard to follow as emergent behavior

The expected evolution is **hybrid**: keep choreography as the default, and introduce a narrowly-scoped orchestrator/process manager only for the specific flows that genuinely need coordinated, compensatable steps — documented in a future ADR.

---

## Alternatives

### Central Orchestrator from day one

A dedicated service drives every automation, calling/commanding each step.

**Rejected because:**
- Reintroduces coupling: the orchestrator must know every service and step
- Becomes a single point of failure and a development bottleneck
- Over-engineered for v1's single-step, non-compensating executions

### Orchestration via commands on the bus

Events replaced by **command** messages addressed to specific services.

**Rejected because:**
- Turns the bus into RPC-over-Kafka, losing the fan-out and decoupling benefits
- Contradicts the "events are facts" decision (ADR-004)

---

## Related

- ADR-001 — Event-Driven Backbone
- ADR-002 — Backend For Frontend as an Event-Driven Projection Layer
- ADR-004 — Eventing Model & Delivery Semantics
- RFC-001 — End-to-End Flow & Service Landscape
- RFC-002 — Workflow Execution Service
