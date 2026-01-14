# Service Bootstrap

## Purpose

This document describes how new backend services are bootstrapped in EventFlow.

Rather than treating service creation as an ad-hoc activity, EventFlow adopts a **standardized bootstrap process** to ensure architectural consistency, reduce setup overhead, and allow developers to focus on domain-specific concerns from day one.

This document explains **why** the bootstrap exists, **what it provides**, and **how it is intended to be used**.

---

## Motivation

As EventFlow grows, multiple services are expected to emerge, each responsible for a specific part of the system (e.g. execution, ingestion, projections).

Without a shared baseline, service creation tends to:
- Repeat the same setup work
- Drift architecturally over time
- Mix infrastructure concerns with domain logic
- Delay meaningful development

To avoid this, EventFlow introduces a **Service Template** that acts as the default starting point for all backend services.

---

## EventFlow Service Template

EventFlow backend services are expected to be bootstrapped from the **EventFlow Service Template**.

The template is a dedicated repository that encapsulates:
- Architectural conventions
- Tooling decisions
- Directory structure
- Test strategy
- Local development infrastructure

The template does **not** contain business logic and is not part of the runtime of any service.

---

## What the Template Provides

The Service Template provides:

- A predefined directory structure aligned with EventFlow Service Design Principles
- TypeScript configuration and build setup
- A test environment with clear separation between:
  - Unit tests
  - Integration tests
- Local Kafka setup for development and testing
- A minimal runnable service (bootstrap / hello world)
- Baseline configuration for logging and environment variables

This ensures every new service starts from a **working, testable baseline**.

---

## What the Template Does Not Provide

The Service Template intentionally does **not** include:

- Domain-specific logic
- Service-specific data models
- Production infrastructure configuration
- Runtime dependencies between services
- Framework-level abstractions

Each service remains fully independent and owns its own evolution.

---

## When to Use the Template

The Service Template should be used when:

- Creating a new backend service
- Prototyping a service defined in an RFC
- Exploring a new architectural component in isolation

Using the template is **strongly recommended** but not enforced by tooling.  
Deviation should be intentional and documented.

---

## When Not to Use the Template

The Service Template should not be used for:

- One-off scripts or experiments
- Frontend applications
- Infrastructure-only repositories
- Shared libraries

The template is specifically designed for **long-running backend services**.

---

## Relationship to Other Documentation

The Service Template is closely related to:

- **Service Design Principles**  
  The directory structure and boundaries in the template directly reflect the architectural rules defined there.

- **RFCs**  
  Service-level RFCs reference the template as the baseline for implementation.

- **Milestones**  
  Milestones assume the service starts from the template, allowing tasks to focus on behavior rather than setup.

---

## Evolution of the Template

The Service Template is expected to evolve over time.

Changes to the template:
- Are versioned
- Do not automatically propagate to existing services
- Should favor backward compatibility when possible

Existing services are not required to retroactively adopt template changes.

---

## Summary

EventFlow treats service creation as a **deliberate architectural activity**, not a mechanical setup task.

By adopting a standardized Service Template, EventFlow ensures:
- Consistent service structure
- Faster onboarding
- Reduced cognitive load
- Clear architectural boundaries
- Long-term maintainability

The Service Template is a foundational tool that enables scalable, disciplined service development across the system.
