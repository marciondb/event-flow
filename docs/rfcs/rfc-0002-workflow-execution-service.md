# Title
Event Execution Service (v1)

---

| Field            | Value |
|------------------|-------|
| **RFC**          | RFC-0002 |
| **Author**       | Marcio Dias |
| **Contributors** | N/A |
| **Started at**   | 2026-01-14 |
| **Status**       | ACCEPTED |
| **Service Name** | event-flow-workflow-execution |
| **Description**  | This RFC defines the Event Execution Service, its responsibilities, scope, emitted events, architectural constraints, and the initial milestone for its standalone delivery. |

---

## Context

EventFlow is an event-driven automation platform where workflows react to events and execute actions asynchronously.

At the core of this system is a service responsible for **executing workflows and emitting execution lifecycle events**. This service acts as the **domain nucleus** of EventFlow v1, as other services (BFF, event ingestion, notifications) depend on its behavior and emitted events.

This RFC defines the first concrete backend service to be implemented in EventFlow, designed to be developed and validated **in isolation**, with other system components mocked or simulated as needed.

---

## Service Naming

The service is named **`event-flow-workflow-execution`**.

### Rationale

- **EventFlow**: ties the service clearly to the overall platform
- **event-execution**: emphasizes execution responsibility without implying orchestration of external systems
- Avoids generic terms like “engine” or “orchestrator” at this stage
- Leaves room for future services (e.g. event-ingestion, execution-history)

The name reflects **what the service does**, not how it is implemented.

---

## Responsibilities

The Event Execution Service is responsible for:

- Receiving a command to execute a workflow
- Executing workflow steps asynchronously
- Managing the execution lifecycle
- Emitting execution lifecycle events
- Reporting execution failures via events

The service is **not responsible** for:

- Persisting automation definitions long-term
- Serving frontend-specific queries
- Aggregating execution state for UI
- Ingesting external events
- Managing retries, compensation, or scheduling

---

## Scope (v1)

### In Scope

- Execution of simple workflows
- One trigger → one or more sequential steps
- Asynchronous execution model
- Emission of execution lifecycle events
- Ability to run standalone with mocked dependencies

### Out of Scope

- Conditional branching
- Parallel execution
- Retry policies
- Idempotency guarantees
- External system integrations
- Advanced observability or metrics

These concerns are intentionally deferred.

---

## Execution Lifecycle

Each workflow execution follows a minimal lifecycle:

1. Execution requested
2. Execution started
3. Steps executed
4. Execution completed **or** execution failed

Each lifecycle transition produces an immutable event.

---

## Emitted Events

The service emits **events as facts**, not commands.

### Initial Event Types

- `workflow_execution_started`
- `workflow_execution_completed`
- `workflow_execution_failed`

### Event Characteristics

- Events are append-only
- Events represent state transitions that already occurred
- Events are the primary integration contract with other services
- Schemas are expected to evolve during early iterations

---

## Architecture & Design Principles

The Event Execution Service follows the **Service Design Principles** defined for EventFlow:

- Domain logic is isolated and pure
- Side effects are handled in boundary components
- Execution logic is deterministic
- External interactions are abstracted
- Events are emitted at clear domain boundaries

Internally, the service adopts a **Diplomat-inspired structure**, ensuring:
- Clear separation between domain logic and infrastructure
- Explicit orchestration boundaries
- High testability

---

## Assumptions & Mocked Dependencies

For v1 development, the following assumptions apply:

- Workflow definitions may be:
  - Hardcoded
  - In-memory
  - Loaded from static configuration
- Event streaming infrastructure is assumed to exist
- No guarantees are made about event consumers
- Failure handling is intentionally simple

These assumptions allow independent delivery and learning.

---

## Milestone — Event Execution Service v1

### Objective

Deliver a standalone service capable of executing a simple workflow asynchronously and emitting lifecycle events.

---

### Deliverables

- Executable service (`event-flow-workflow-execution`)
- One or more mock workflow definitions
- Asynchronous execution mechanism
- Event emission to a message broker (or simulated equivalent)
- Basic logging for execution lifecycle

---

### Success Criteria

This milestone is considered complete when:

- A workflow execution can be triggered programmatically
- Execution runs asynchronously
- Execution lifecycle events are emitted
- Both success and failure paths are observable
- The service runs independently of other EventFlow services

---

## Consequences

### Positive

- Establishes the execution core of EventFlow
- Validates the event-driven model early
- Enables parallel development of other services
- Provides concrete contracts for downstream consumers

### Negative

- Early event schemas may change
- Mocked dependencies may diverge from future implementations
- Refactoring is expected as scope expands

These trade-offs are accepted.

---

## Related

- RFC-0001 — End-to-End Flow & Service Landscape
- ADR-0001 — Hybrid Communication Model
- ADR-0002 — Backend For Frontend as an Event-Driven Projection Layer
