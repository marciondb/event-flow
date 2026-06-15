# EventFlow — Architecture & Documentation Hub

This repository serves as the **central documentation hub** for the EventFlow initiative.

EventFlow is a long-term, personal engineering project focused on exploring **event-driven architectures, microservices, and distributed system design** through the evolution of a realistic automation platform.

This repository intentionally contains **no production code**.

Instead, it captures:
- Product thinking
- Architectural decisions
- Technical assessments
- Design trade-offs
- System evolution over time

---

## ⚡ Architectural Thesis

EventFlow is designed as a **genuinely event-driven** system — not a synchronous system with events bolted on:

- **Events are the backbone** for action and integration between services.
- **Workflows are triggered by events**, never by direct synchronous calls between services.
- **Synchronous APIs are the exception**, used only for user-facing configuration and queries.
- Services are coordinated by **choreography** (each reacts to events), not by a central orchestrator.

The hard parts of event-driven systems — delivery semantics, idempotency, ordering, dead-lettering, and projection replay — are addressed **explicitly** in the decision records, not deferred.

Start here: [ADR-001 — Event-Driven Backbone](docs/decisions/ADR-001-event-driven-backbone.md) · [ADR-004 — Eventing Model & Delivery Semantics](docs/decisions/ADR-004-eventing-model-and-delivery-semantics.md) · [ADR-005 — Choreography over Orchestration](docs/decisions/ADR-005-choreography-over-orchestration.md)

---

## 📌 Purpose of This Repository

The goal of this repository is to document **how and why** architectural decisions are made before and during implementation.

It acts as:
- A living architecture notebook
- A decision log (ADRs)
- A product and system design reference
- The source of truth for all EventFlow-related design artifacts

---

## 🧠 Project Philosophy

Most software projects showcase *what* was built.

EventFlow focuses on:
- **Why** certain decisions were made
- **What alternatives were considered**
- **How trade-offs were evaluated**

This mirrors the way complex systems are designed and evolved in mature engineering organizations.

---

## 🗂 Repository Structure

This repository is organized as a **documentation and architecture hub**, containing:

- Product vision and requirements
- Architecture diagrams and system overviews
- Architecture Decision Records (ADRs)
- Request for Comments (RFCs)
- Milestones and evolution notes

Artifacts follow a consistent naming convention: `PREFIX-NNN-title` (e.g. `ADR-001`, `RFC-002`, `MILESTONE-001`).

```text
event-flow/
├── README.md
└── docs/
    ├── vision/
    │   └── product-vision.md
    ├── architecture/
    │   ├── bff-diagram.md
    │   ├── service-bootstrap.md
    │   ├── service-design-principles.md
    │   ├── system-diagram.md
    │   └── system-overview.md
    ├── decisions/
    │   ├── ADR-001-event-driven-backbone.md
    │   ├── ADR-002-backend-for-frontend-event-driven-projection.md
    │   ├── ADR-003-javascript-service-technology-stack.md
    │   ├── ADR-004-eventing-model-and-delivery-semantics.md
    │   └── ADR-005-choreography-over-orchestration.md
    ├── rfcs/
    │   ├── RFC-001-end-to-end-flow-and-service-landscape.md
    │   └── RFC-002-workflow-execution-service.md
    └── milestones/
        └── MILESTONE-001-event-execution.md
```

---

## 🔗 Related Repositories

As the system evolves, implementation repositories will be created, each focused on a specific bounded context or service, for example:

- `event-flow-event-ingestion`
- `event-flow-automation-management`
- `event-flow-workflow-execution`
- `event-flow-bff`

Each implementation repository references this hub as its architectural source of truth.

---

## 🚧 Status

EventFlow is an **ongoing personal learning project**.

Documentation is expected to evolve alongside the system as new insights, trade-offs, and constraints emerge.

---

## 📎 Disclaimer

EventFlow is a personal educational initiative and is not affiliated with any commercial automation platforms.
