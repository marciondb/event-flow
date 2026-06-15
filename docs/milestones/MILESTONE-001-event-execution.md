# Milestone 001 — Event Execution Service (v1)

## Objective

Establish the foundational structure of the Event Execution Service and deliver a minimal, runnable v1 capable of publishing and consuming events using Kafka.

This milestone focuses on **infrastructure, architecture, and core execution flow**, with early emphasis on **testability and isolation**.

---

## Scope

### In Scope

- Service repository creation
- Development environment setup
- Base service structure following Service Design Principles
- Test environment configuration (unit and integration)
- Minimal executable service
- Kafka integration (producer and consumer)
- Execution lifecycle simulation
- Event emission and consumption

### Out of Scope

- Advanced workflow logic
- External service integrations
- Persistent storage
- Production hardening
- Advanced observability tooling

---

## Technology Choices (Service-Level)

- **Language**: JavaScript
- **Type System**: TypeScript
- **Test Framework**: Jest
- **Event Backbone**: Apache Kafka

Jest was chosen due to:
- Strong TypeScript support
- Large ecosystem adoption
- Simple configuration
- Clear separation between unit and integration tests

---

## Tasks

### Task 1 — Repository & Development Environment Setup

**Goal**  
Create the service repository and establish a consistent development environment.

**Includes**
- Create repository: `event-flow-workflow-execution`
- Initialize Node.js project
- Configure TypeScript
- Configure linting and formatting
- Define base project structure
- Add minimal README with service purpose

**Acceptance Criteria**
- Project builds successfully
- TypeScript compilation works
- Repository can be cloned and run locally

---

### Task 2 — Service Bootstrap (Hello World)

**Goal**  
Ensure the service can start, run, and shut down correctly.

**Includes**
- Application entry point
- Configuration loading
- Structured logging
- Graceful startup and shutdown
- Simple “service started” output

**Acceptance Criteria**
- Service starts locally
- No runtime errors
- Logs confirm successful startup

---

### Task 3 — Test Environment Setup

**Goal**  
Establish a robust test environment to support isolated development and safe refactoring.

**Includes**
- Configure Jest with TypeScript
- Create test directory structure:
  - `tests/unit`
  - `tests/integration`
- Add test scripts to project configuration
- Example unit test (pure logic)
- Example integration test (service bootstrap)

**Test Strategy**
- **Unit tests**
  - Test pure logic and adapters
  - No I/O or external dependencies
- **Integration tests**
  - Validate interaction between components
  - May involve Kafka (real or mocked)

**Acceptance Criteria**
- Unit tests run successfully
- Integration test suite executes independently
- Clear separation between unit and integration tests

---

### Task 4 — Execution Core Skeleton

**Goal**  
Establish the internal structure of the service aligned with Service Design Principles.

**Includes**
- Domain folders:
  - model
  - logic
  - controller
- Boundary folders:
  - wire
  - adapter
  - diplomat
- Placeholder execution controller
- Simulated execution flow (e.g. timeout-based)

**Acceptance Criteria**
- Execution flow can be triggered programmatically
- No external dependencies required yet
- Code structure matches documented architecture

---

### Task 5 — Kafka Local Environment Setup

**Goal**  
Enable local development and testing with Kafka.

**Includes**
- Local Kafka setup (e.g. Docker Compose)
- Minimal topic definitions
- Documentation for running Kafka locally
- Environment-based Kafka configuration

**Acceptance Criteria**
- Kafka can be started locally
- Service connects successfully
- Topics are available for publishing and consuming

---

### Task 6 — Kafka Producer Integration

**Goal**  
Enable the service to emit execution lifecycle events.

**Includes**
- Kafka producer configuration
- Event serialization
- Emission of lifecycle events:
  - `workflow_execution_started`
  - `workflow_execution_completed`
  - `workflow_execution_failed`

**Acceptance Criteria**
- Events are published to Kafka
- Payloads follow a consistent structure
- Events are observable via Kafka tooling

---

### Task 7 — Kafka Consumer Integration

**Goal**  
Enable the service to consume the **trigger event** that starts an execution (plus its own events for validation).

**Includes**
- Kafka consumer configuration
- Event deserialization and envelope validation
- Consume `automation_trigger_requested` → start an execution
- Idempotent handling using `event_id` (no duplicate execution on redelivery)
- Integration test covering consumption

**Acceptance Criteria**
- A trigger event consumed from Kafka starts an execution
- Unexpected payloads do not crash the service
- Redelivering the same `event_id` does not start a second execution
- Clear logging for consumed events

---

### Task 8 — End-to-End Execution Simulation

**Goal**  
Demonstrate a complete, **event-triggered** execution lifecycle using Kafka.

**Includes**
- Publish a `automation_trigger_requested` event (simulated producer) that the service consumes to start execution
- Simulated workflow step
- Emission of lifecycle events (`started` / `completed` / `failed`)
- `correlation_id` propagated from the trigger event through all lifecycle events

**Acceptance Criteria**
- Execution is started by an event, not by a direct call
- Execution lifecycle is observable end to end via Kafka
- `correlation_id` is the same across trigger and lifecycle events
- Service operates asynchronously

---

## Completion Criteria

Milestone 001 is complete when:

- The service runs locally
- Tests are in place and executable
- Kafka is integrated and functional
- Execution lifecycle events are published
- The service consumes events
- Architecture matches documented principles

---

## Notes

This milestone prioritizes **architectural correctness, testability, and learning** over feature completeness.

Refactoring is expected and encouraged as understanding evolves.
