# System Overview — EventFlow (v2)

## Purpose

This document provides a high-level overview of the EventFlow system architecture.

It describes the main system components, their responsibilities, and how they interact, with a focus on **clarity of roles**, **separation of concerns**, and **support for both synchronous and asynchronous communication patterns**.

This document intentionally avoids implementation details and serves as a foundation for architecture decisions and future RFCs.

---

## Architectural Style

EventFlow adopts a **hybrid architectural model**, combining:

- Synchronous, request-response APIs for user-driven interactions
- Asynchronous, event-driven communication for automation execution and state propagation

This approach balances usability, scalability, and architectural realism.

---

## High-Level Architecture

At a high level, EventFlow consists of:

- A web-based frontend
- A Backend For Frontend (BFF)
- An API Gateway
- Domain-focused backend services
- An event streaming platform
- Supporting infrastructure services

Each component has a clearly defined responsibility and communicates with others using appropriate interaction patterns.

---

## Core Components

### Frontend Application

The frontend provides the primary user interface for EventFlow.

Responsibilities:
- User authentication and session handling
- Creation and management of automations
- Visualization of workflow execution state
- Inspection of execution history and logs

The frontend communicates exclusively with the BFF using synchronous APIs.

---

### Backend For Frontend (BFF)

The BFF is a frontend-oriented backend service designed to support UI-specific use cases.

Responsibilities:
- Expose APIs tailored to frontend needs
- Aggregate data from multiple backend services
- Consume workflow-related events
- Maintain UI-specific projections of workflow execution state

The BFF does **not** own business logic or execution decisions.  
It derives its state from events published by core services and serves as a **read-oriented projection layer** for the frontend.

---

### API Gateway

The API Gateway acts as a thin entry point for backend services.

Responsibilities:
- Request routing
- Authentication and authorization
- Rate limiting and throttling
- API exposure for external producers and consumers

The API Gateway is intentionally kept **stateless** and does **not** maintain domain-specific logic or consume events.

---

### Backend Services

Backend services are organized around clear domain boundaries.

Each service:
- Owns its data and persistence
- Exposes well-defined APIs
- Can be deployed independently
- May produce and/or consume events

Example service domains include:
- Event ingestion and validation
- Automation configuration and management
- Workflow execution
- Execution tracking and auditing

---

### Event Streaming Platform

An event streaming platform provides the backbone for asynchronous communication across the system.

Responsibilities:
- Transport events between producers and consumers
- Enable multiple independent consumers per event
- Decouple producers from downstream processing

Events published to the platform represent significant occurrences such as:
- Workflow execution started
- Step completed
- Execution failed
- Execution completed

---

## Communication Patterns

### Synchronous Communication

Synchronous APIs are used when:
- Immediate feedback is required by the user
- Managing configuration and state (e.g. creating automations)
- Orchestrating frontend-driven workflows

Typical synchronous flows include:
- Frontend → BFF
- BFF → API Gateway
- API Gateway → backend services

---

### Asynchronous Communication

Asynchronous communication is used when:
- Executing workflows
- Propagating execution state
- Performing background or long-running operations
- Isolating failures between components

Typical asynchronous flows include:
- Backend services publishing workflow events
- BFF consuming events to update UI projections
- Multiple services reacting independently to the same event

---

## Workflow Execution Visibility

Workflow execution visibility is achieved through **event propagation and projection**.

- Execution services publish workflow lifecycle events
- The BFF consumes these events
- UI-specific execution state is derived and stored
- The frontend queries the BFF for real-time status

This model avoids tight coupling between the frontend and execution services while providing near real-time feedback.

---

## Data Ownership & Boundaries

- Each backend service owns its data
- There is no shared database between services
- Data is exchanged explicitly via APIs or events
- The BFF maintains derived, non-authoritative state

The authoritative source of workflow state remains within core backend services.

---

## Observability

Observability is treated as a first-class concern.

The system is designed to provide visibility into:
- Event ingestion
- Workflow execution progress
- Success and failure states
- Execution history

This information is surfaced through the BFF for frontend consumption.

---

## Scalability & Evolution

The architecture supports:
- Independent scaling of backend services
- Addition of new event consumers without modifying producers
- Incremental expansion of workflow complexity

The system is designed to evolve gradually as new learning goals and architectural explorations emerge.

---

## Constraints & Trade-offs

The architecture intentionally prioritizes:
- Clear responsibilities over minimal component count
- Observability over aggressive optimization
- Architectural learning over production-grade performance tuning

These trade-offs are explicitly documented and revisited as the system evolves.

---

## Summary

EventFlow uses a **hybrid architecture** combining synchronous APIs and asynchronous, event-driven communication.

The introduction of a **Backend For Frontend** as an event-driven projection layer enables rich workflow visibility while keeping core services decoupled and focused on execution.

This structure provides a realistic and extensible foundation for exploring automation platforms and distributed system design.
