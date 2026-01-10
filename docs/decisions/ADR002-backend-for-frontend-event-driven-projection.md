# Backend For Frontend as an Event-Driven Projection Layer

---

| Field            | Value |
|------------------|-------|
| **ADR**          | ADR-0002 |
| **Author**       | Marcio Dias |
| **Contributors** | N/A |
| **Started at**   | 2026-01-10 |
| **Status**       | DRAFT |
| **Description**  | This ADR documents the decision to introduce a Backend For Frontend (BFF) responsible for frontend-specific APIs and for maintaining workflow execution state projections derived from events. |

---

## Context

EventFlow provides a web-based user interface where users create automations and monitor workflow executions.

The system includes:
- Multiple backend services responsible for workflow execution
- An API Gateway acting as a thin entry point
- An event streaming platform used to propagate workflow lifecycle events

The frontend requires:
- UI-specific APIs
- Aggregated views of workflow execution state
- Near real-time visibility into workflow progress and failures

Exposing core backend services directly to the frontend would:
- Leak domain-specific APIs into the UI
- Increase coupling between frontend and execution services
- Require synchronous polling or complex orchestration for execution status

Additionally, placing workflow state logic inside the API Gateway would violate its responsibility as a stateless routing and cross-cutting concerns layer.

---

## Decision

EventFlow introduces a **Backend For Frontend (BFF)** with the following responsibilities:

- Expose APIs tailored specifically to frontend use cases
- Aggregate data from multiple backend services
- Consume workflow execution events from the event streaming platform
- Maintain **UI-oriented projections** of workflow execution state

The BFF:
- Does **not** own business logic or workflow execution decisions
- Does **not** act as the authoritative source of truth
- Derives its state exclusively from events published by core backend services

The API Gateway remains a thin, stateless component and does not consume events or maintain workflow state.

---

## Consequences

### Positive Consequences

- Clear separation between UI concerns and core domain logic
- Reduced coupling between frontend and backend services
- Improved frontend performance through aggregated and optimized APIs
- Near real-time workflow visibility without synchronous polling
- Strong alignment with the event-driven communication model

### Negative Consequences

- Additional service to maintain and operate
- Eventual consistency between execution services and UI projections
- Need to manage projection rebuilding and replay scenarios
- Increased architectural complexity compared to a direct frontend-to-gateway model

These trade-offs are accepted as part of the system’s architectural goals and learning objectives.

---

## Alternatives

### Frontend Calling Backend Services Directly

The frontend communicates directly with backend services via the API Gateway.

**Rejected because:**
- High coupling between frontend and domain APIs
- Increased complexity in frontend logic
- Poor separation of concerns

---

### API Gateway Handling Workflow State

The API Gateway consumes events and maintains workflow execution state.

**Rejected because:**
- Violates the gateway’s responsibility as a stateless component
- Introduces domain logic into an infrastructure layer
- Reduces architectural clarity

---

### Backend Services Exposing UI-Specific APIs

Backend services provide APIs tailored for frontend needs.

**Rejected because:**
- Mixes domain logic with presentation concerns
- Leads to API fragmentation
- Makes service evolution more difficult

---

## Related

- Product Vision — EventFlow
- System Overview — EventFlow (v2)
- ADR-0001 — Hybrid Communication Model (Synchronous APIs + Event-Driven Workflows)
