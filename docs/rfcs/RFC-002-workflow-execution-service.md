# Event Execution Service (v1)

---

| Field            | Value |
|------------------|-------|
| **RFC**          | RFC-002 |
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

- **Consuming a trigger event** to start a workflow execution (it is not invoked through a synchronous command)
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

## Event Contracts

Events are the **public contract** of this service, so they are defined explicitly.
Delivery semantics (at-least-once, idempotency, ordering, DLQ) are defined in
**ADR-004 — Eventing Model & Delivery Semantics**; this section defines the **shapes**.

### Common Envelope

Every event — trigger or lifecycle — shares the same envelope. Domain data lives in `payload`.

```json
{
  "event_id": "b3f1c2a4-1e2d-4c3b-8a9f-1d2e3f4a5b6c",
  "type": "workflow_execution_started",
  "version": 1,
  "occurred_at": "2026-01-14T12:00:00.000Z",
  "correlation_id": "9a8b7c6d-5e4f-3a2b-1c0d-9e8f7a6b5c4d",
  "partition_key": "exec_7f3a9b",
  "payload": {}
}
```

| Field | Type | Purpose |
|---|---|---|
| `event_id` | UUID | Unique id of this event; used by consumers for **idempotent dedup** |
| `type` | string | Event type (snake_case) |
| `version` | integer | Schema major version of the `payload` |
| `occurred_at` | ISO-8601 UTC | When the fact happened (not when it was published) |
| `correlation_id` | UUID | Carried across the whole flow (ingestion → execution → projection) for tracing |
| `partition_key` | string | Kafka partition key — guarantees ordering per execution (see ADR-004) |
| `payload` | object | Type-specific domain data |

### Trigger Event (consumed)

Published by the Event Ingestion Service; consumed by this service to start an execution.

```json
{
  "event_id": "0f1c8d2e-aa11-4bb2-9cc3-44dd55ee66ff",
  "type": "automation_trigger_requested",
  "version": 1,
  "occurred_at": "2026-01-14T12:00:00.000Z",
  "correlation_id": "9a8b7c6d-5e4f-3a2b-1c0d-9e8f7a6b5c4d",
  "partition_key": "auto_123",
  "payload": {
    "automation_id": "auto_123",
    "trigger": { "source": "simulated-producer", "event_name": "order.created" },
    "context": { "order_id": "ord_987" }
  }
}
```

### Lifecycle Events (produced)

`workflow_execution_started`

```json
{
  "event_id": "b3f1c2a4-1e2d-4c3b-8a9f-1d2e3f4a5b6c",
  "type": "workflow_execution_started",
  "version": 1,
  "occurred_at": "2026-01-14T12:00:00.050Z",
  "correlation_id": "9a8b7c6d-5e4f-3a2b-1c0d-9e8f7a6b5c4d",
  "partition_key": "exec_7f3a9b",
  "payload": {
    "execution_id": "exec_7f3a9b",
    "automation_id": "auto_123",
    "step_count": 2
  }
}
```

`workflow_execution_completed`

```json
{
  "event_id": "c4e2d3b5-2f3e-4d4c-9b0a-2e3f4a5b6c7d",
  "type": "workflow_execution_completed",
  "version": 1,
  "occurred_at": "2026-01-14T12:00:00.420Z",
  "correlation_id": "9a8b7c6d-5e4f-3a2b-1c0d-9e8f7a6b5c4d",
  "partition_key": "exec_7f3a9b",
  "payload": {
    "execution_id": "exec_7f3a9b",
    "automation_id": "auto_123",
    "duration_ms": 370,
    "steps": [
      { "name": "step-1", "status": "succeeded" },
      { "name": "step-2", "status": "succeeded" }
    ]
  }
}
```

`workflow_execution_failed`

```json
{
  "event_id": "d5f3e4c6-3a4f-4e5d-8c1b-3f4a5b6c7d8e",
  "type": "workflow_execution_failed",
  "version": 1,
  "occurred_at": "2026-01-14T12:00:00.310Z",
  "correlation_id": "9a8b7c6d-5e4f-3a2b-1c0d-9e8f7a6b5c4d",
  "partition_key": "exec_7f3a9b",
  "payload": {
    "execution_id": "exec_7f3a9b",
    "automation_id": "auto_123",
    "failed_step": "step-2",
    "error": { "code": "STEP_EXECUTION_ERROR", "message": "Action timed out" }
  }
}
```

### JSON Schema (envelope + `workflow_execution_started`)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://eventflow/events/workflow_execution_started/v1.json",
  "type": "object",
  "required": ["event_id", "type", "version", "occurred_at", "correlation_id", "partition_key", "payload"],
  "additionalProperties": false,
  "properties": {
    "event_id": { "type": "string", "format": "uuid" },
    "type": { "const": "workflow_execution_started" },
    "version": { "type": "integer", "const": 1 },
    "occurred_at": { "type": "string", "format": "date-time" },
    "correlation_id": { "type": "string", "format": "uuid" },
    "partition_key": { "type": "string", "minLength": 1 },
    "payload": {
      "type": "object",
      "required": ["execution_id", "automation_id", "step_count"],
      "additionalProperties": false,
      "properties": {
        "execution_id": { "type": "string" },
        "automation_id": { "type": "string" },
        "step_count": { "type": "integer", "minimum": 0 }
      }
    }
  }
}
```

### Versioning & Compatibility

- **Additive, backward-compatible changes** (new optional fields) keep the same `version`.
- **Breaking changes** (renaming/removing fields, changing types) require bumping `version` and publishing the new schema; old and new may coexist during migration.
- Consumers **must ignore unknown fields** (forward compatibility) and validate against the schema at the boundary (Adapter + Wire, per ADR-003).
- Schemas are intended to live in a **schema registry** (one subject per event type), enabling compatibility checks in CI before producers ship a change.

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

## Technology Choices

The following technology choices apply specifically to the Event Execution Service.

### Programming Language

- **Language**: JavaScript
- **Type System**: TypeScript

#### Rationale

- Strong ecosystem support for event-driven systems
- Mature Kafka client libraries
- Excellent developer experience for rapid iteration
- Type safety to support evolving event schemas
- Alignment with frontend and BFF technologies

These choices prioritize learning velocity and clarity over maximum performance.

---

### Event Backbone

- **Event Streaming Platform**: Apache Kafka

#### Rationale

- Explicit support for asynchronous, event-driven architectures
- Clear separation between producers and consumers
- Durable event storage
- Industry-standard tooling and mental model

Kafka is treated as a **core dependency** of the service, not an implementation detail.

---
## Implementation Guidelines

Implementation of this service should be bootstrapped from the **EventFlow Service Template**.

The template provides:
- A standardized directory structure
- Preconfigured TypeScript and test environment
- Kafka local setup for development
- A minimal runnable service (hello world)
- Baseline configuration aligned with Service Design Principles

Using the template ensures architectural consistency across services while allowing each service to evolve independently.

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

- RFC-001 — End-to-End Flow & Service Landscape
- ADR-001 — Event-Driven Backbone
- ADR-002 — Backend For Frontend as an Event-Driven Projection Layer
- ADR-004 — Eventing Model & Delivery Semantics
- ADR-005 — Choreography over Orchestration
