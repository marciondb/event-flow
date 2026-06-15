# End-to-End Flow & Service Landscape (v1)

---

| Field            | Value |
|------------------|-------|
| **RFC**          | RFC-001 |
| **Author**       | Marcio Dias |
| **Contributors** | N/A |
| **Started at**   | 2026-01-13 |
| **Status**       | ACCEPTED |
| **Description**  | This RFC defines the end-to-end flow of EventFlow v1 and proposes an initial service landscape, establishing clear scope boundaries and a foundation for future evolution. |

---

## Context

EventFlow is an event-driven automation platform inspired by tools like Zapier, designed as a long-term learning and architectural exploration project.

Up to this point, EventFlow has:
- A defined product vision
- A system overview with synchronous and asynchronous communication
- Architectural decisions around hybrid communication and BFF usage
- Service-level design principles inspired by Diplomat architecture

What is still missing is a **clear end-to-end view of the system**, answering:

- What does EventFlow actually do in its first version?
- What is the minimal flow that delivers value?
- Which services make sense to exist from day one?
- Which services are intentionally postponed?

This RFC addresses those questions by defining the **EventFlow v1 scope** and mapping it to an initial service landscape.

---

## Goals

This RFC aims to:

- Define the functional scope of **EventFlow v1**
- Describe the **end-to-end user and system flow**
- Identify **initial service candidates** and responsibilities
- Establish a shared understanding of system boundaries
- Provide a foundation for future RFCs and milestones

---

## Non-Goals

This RFC does **not** aim to:

- Define detailed APIs or data models
- Finalize service contracts
- Optimize for performance or scale
- Lock service boundaries permanently
- Cover advanced automation features

Service definitions in this document are **directional and evolutionary**.

---

## EventFlow v1 — Functional Scope

EventFlow v1 focuses on a **minimal but complete automation loop**.

### In scope for v1

- Users can define a simple automation:
  - One triggering event
  - One or more actions
- Events can be:
  - Published into the system through the **Event Ingestion Service** (via simulated producers in v1)
- Automations are:
  - **Triggered by an event** (not by a synchronous call)
  - Executed asynchronously
- Workflow execution emits lifecycle events:
  - Started
  - Completed
  - Failed
- Execution state is observable via the UI

### Out of scope for v1

- Complex conditional logic
- Multi-step branching workflows
- External third-party integrations
- User-defined custom code
- Advanced retry or compensation logic

---

## End-to-End Flow (v1)

At a high level, the v1 flow is:

1. A user creates an automation via the frontend (synchronous: `Frontend → BFF → Gateway → Automation Management`)
2. Automation configuration is stored
3. An event is published into the system through the **Event Ingestion Service**, which validates it and emits a **trigger event** to the Event Backbone
4. The **Workflow Execution Service consumes the trigger event** and starts executing — the trigger is an event, never a direct call
5. Workflow execution emits lifecycle events (started / completed / failed) to the Event Backbone
6. The BFF consumes those events, updates its projections, and the frontend queries the BFF for status

This flow intentionally exercises:
- Synchronous APIs **only** for user-facing configuration and queries
- **Event-triggered**, asynchronous execution (the action path is event-driven end to end)
- Event propagation as the integration contract between services
- Observability through event-derived projections

---

## Proposed Service Landscape (v1)

The following services are proposed for EventFlow v1.

These are **initial candidates**, not final boundaries.

---

### 1. API Gateway

**Responsibility**
- Act as the single entry point for backend services
- Handle authentication, routing, and cross-cutting concerns

**Notes**
- Thin by design
- No domain logic
- Likely to evolve as the system grows

---

### 2. Automation Management Service

**Responsibility**
- Manage automation definitions
- Store triggers and actions
- Expose CRUD APIs for automations

**Why in v1**
- Defines what can be executed
- Anchors user intent

---

### 3. Event Ingestion Service

**Responsibility**
- Provide the synchronous HTTP entrypoint that turns user/system intent into events
- Validate incoming events and publish them to the Event Backbone as **trigger events**
- Decouple "something happened" from "a workflow runs"

**Why in v1**
- It is the component that makes the platform genuinely **event-driven**: workflows are triggered by events, not by direct service-to-service calls
- Even with simulated producers, it establishes the **event-as-trigger contract** from day one

---

### 4. Workflow Execution Service

**Responsibility**
- **React to trigger events** consumed from the Event Backbone
- Execute automations when triggered
- Coordinate workflow steps
- Emit execution lifecycle events

**Why in v1**
- Core domain of the system
- Consumes trigger events and produces lifecycle events
- Validates the event-driven architecture

---

### 5. Event Backbone (Streaming Platform)

**Responsibility**
- Transport events between services
- Decouple producers and consumers

**Notes**
- Infrastructure component
- Central to v1 execution flow

---

### 6. Backend For Frontend (BFF)

**Responsibility**
- Serve frontend-specific APIs
- Aggregate data from backend services
- Consume execution events
- Maintain UI-oriented projections

**Why in v1**
- Enables execution visibility
- Decouples UI from core services

---
## EventFlow v1 — End-to-End Architecture Diagram

## Purpose

This diagram illustrates the **end-to-end flow of EventFlow v1**, highlighting the main services involved and how synchronous and asynchronous communication coexist in the system.

It reflects the scope and service landscape defined in this RFC.

---

## High-Level System Diagram (v1)

```mermaid
flowchart LR
    FE[Frontend UI]

    BFF[Backend For Frontend]

    GW[API Gateway]

    AUTO[Automation Management Service]

    ING[Event Ingestion Service]

    EXEC[Workflow Execution Service]

    KAFKA[(Event Backbone)]

    %% Synchronous interactions (user-facing config and queries only)
    FE -->|HTTP| BFF
    BFF -->|HTTP| GW
    GW -->|HTTP: manage automations| AUTO
    GW -->|HTTP: publish trigger event| ING

    %% Asynchronous interactions (action + integration backbone)
    ING -->|trigger events| KAFKA
    KAFKA -->|triggers execution| EXEC
    EXEC -->|lifecycle events| KAFKA
    KAFKA -->|consume for projections| BFF
```

### Flow Description

- The Frontend interacts exclusively with the BFF

- The BFF delegates **only** user-facing requests to the API Gateway (configuration and queries)

- The API Gateway routes synchronous requests to:

- Automation Management Service (configuration)

- Event Ingestion Service (HTTP entrypoint that publishes a **trigger event**)

- The Workflow Execution Service **does not receive synchronous calls** — it is triggered by consuming a trigger event from the Event Backbone

- The Workflow Execution Service executes workflows asynchronously and publishes lifecycle events to the Event Backbone

- The BFF consumes execution events and maintains UI-specific projections

- The Frontend queries the BFF for real-time execution status

### Notes

- The API Gateway remains thin and stateless

- The BFF does not execute business logic

- The Event Backbone decouples execution from observation

- Execution visibility is eventually consistent

This diagram represents the minimal complete loop for EventFlow v1

### Evolution

- Future versions may introduce:

- External / third-party event producers feeding the Event Ingestion Service

- Notification services

- Advanced workflow orchestration

- These are intentionally excluded from v1.

---

## Services Explicitly Deferred

The following services are intentionally **not part of v1**, but are expected to appear later:

- External / third-party event producers (the v1 Event Ingestion Service uses simulated producers)
- Notification Service
- Advanced Rules Engine
- External Integration Connectors

These services depend on learnings from v1 execution behavior.

---

## Dependencies & Evolution

- The Workflow Execution Service is the **domain nucleus**
- Other services evolve around execution behavior
- Gateway and BFF are expected to evolve after domain stabilization
- Event schemas are expected to change during early iterations

This RFC establishes a **learning-first architecture**, not a frozen design.

---

## Consequences

### Positive

- Clear v1 scope and expectations
- Reduced architectural ambiguity
- Strong foundation for incremental evolution
- Natural milestones for future RFCs

### Negative

- Some services may be renamed or merged later
- Initial boundaries may shift
- Some features postponed intentionally

These trade-offs are accepted to preserve focus and learning value.

---

## Related

- Product Vision — EventFlow
- System Overview — EventFlow (v2)
- ADR-001 — Event-Driven Backbone
- ADR-002 — Backend For Frontend as an Event-Driven Projection Layer
- ADR-004 — Eventing Model & Delivery Semantics
- ADR-005 — Choreography over Orchestration
- RFC-002 — Workflow Execution Service
