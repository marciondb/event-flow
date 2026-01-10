# Service Architecture Diagram

## Purpose

This document provides a **visual explanation** of the service-level architecture used in EventFlow services.

It illustrates:
- The separation between **Domain (Core)** and **Boundary (External)**
- The responsibilities of each component
- The direction of dependencies
- The flow of data through the system

This diagram complements the `service-design-principles.md` document.

---

## High-Level Architecture

The service is divided into two protected zones:

- **Domain (Core)**: business meaning and decision-making
- **Boundary (External)**: interaction with the outside world and side effects

```mermaid
flowchart LR
    subgraph Boundary["Boundary (External)"]
        WIRE[Wire]
        ADAPTER[Adapter]
        DIPLOMAT[Diplomat]
    end

    subgraph Domain["Domain (Core)"]
        CONTROLLER[Controller]
        LOGIC[Logic]
        MODEL[Model]
    end

    %% Data flow
    WIRE --> ADAPTER
    ADAPTER --> CONTROLLER
    CONTROLLER --> LOGIC
    LOGIC --> MODEL
    MODEL --> LOGIC
    LOGIC --> CONTROLLER

    %% Boundary interaction
    CONTROLLER --> DIPLOMAT
    DIPLOMAT --> CONTROLLER
```

## Request Flow (Step by Step)

The diagram below represents a typical synchronous request flow.

```mermaid
sequenceDiagram
    participant External
    participant Wire
    participant Adapter
    participant Diplomat
    participant Controller
    participant Logic
    participant Model

    External->>Wire: Incoming request
    Wire->>Adapter: Validate & parse
    Adapter->>Diplomat: Adapted input
    Diplomat->>Controller: Execute use case
    Controller->>Logic: Apply business rules
    Logic->>Model: Read / transform data
    Model-->>Logic: Domain data
    Logic-->>Controller: Result
    Controller-->>Diplomat: Domain outcome
    Diplomat-->>Adapter: Prepare response
    Adapter-->>Wire: Adapt output
    Wire-->>External: Response
```

## Event-Based Flow (Consumer / Producer)

Event-driven interactions follow the same architectural rules.

```mermaid
flowchart LR
    EVENT[Event Stream]

    subgraph Boundary["Boundary (External)"]
        CONSUMER[Consumer (Diplomat)]
        PRODUCER[Producer (Diplomat)]
        ADAPTER2[Adapter]
    end

    subgraph Domain["Domain (Core)"]
        CONTROLLER2[Controller]
        LOGIC2[Logic]
        MODEL2[Model]
    end

    EVENT --> CONSUMER
    CONSUMER --> ADAPTER2
    ADAPTER2 --> CONTROLLER2
    CONTROLLER2 --> LOGIC2
    LOGIC2 --> MODEL2

    CONTROLLER2 --> PRODUCER
    PRODUCER --> EVENT
```

## Dependency Direction

The following diagram highlights allowed dependencies only.

```mermaid
flowchart TB
    MODEL_ONLY[Model]
    LOGIC_ONLY[Logic]
    CONTROLLER_ONLY[Controller]
    WIRE_ONLY[Wire]
    ADAPTER_ONLY[Adapter]
    DIPLOMAT_ONLY[Diplomat]

    LOGIC_ONLY --> MODEL_ONLY
    CONTROLLER_ONLY --> LOGIC_ONLY
    CONTROLLER_ONLY --> MODEL_ONLY
    CONTROLLER_ONLY --> DIPLOMAT_ONLY
    ADAPTER_ONLY --> MODEL_ONLY
    ADAPTER_ONLY --> WIRE_ONLY
    DIPLOMAT_ONLY --> ADAPTER_ONLY
    DIPLOMAT_ONLY --> CONTROLLER_ONLY
```
## Key Takeaways

The Domain never depends on infrastructure

All side effects happen in the Boundary

Data is translated at the edges

Logic remains pure and testable

Controllers orchestrate, Logic decides

Diplomats interact with the external world

## Summary

This diagram represents the mental model behind EventFlow service design.

By enforcing strict boundaries and explicit responsibilities, services remain:

Easy to reason about

Easy to test

Easy to evolve

For detailed explanations of each component, refer to service-design-principles.md.