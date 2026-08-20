
---
title:  Architectural Principles within Cloud Environment
type: doc
source_ref: chatgpt-￥conversation://6a76a1c8-2c0c-83ea-bdc0-f82ccb4775a7
captured_at: 2026-08-09
---
# ARCHITECTURAL_PRINCIPLES.md

## Goal

Design software that can be developed primarily in the cloud while
isolating runtime-specific complexity to local integration.

The objective is architectural stability, not implementation
convenience.

------------------------------------------------------------------------

## Layer Architecture

``` text
Skill
    │
    ▼
Domain Service
    │
    ▼
Validate
    │
    ▼
Adapter
    │
    ▼
Runtime Manager
    │
    ▼
External Runtime
(MCP / Browser / SSH / Docker / API)
```

### Skill

-   AI reasoning
-   Workflow
-   Prompt
-   Planning

### Domain Service

-   Stable public Contract
-   Normalized domain objects
-   No AI reasoning

### Validate

-   Schema validation
-   Fact validation
-   Duplicate detection
-   Never infer meaning

### Adapter

-   Translate external models into domain models
-   Hide platform differences

### Runtime Manager

Treat every external runtime as a managed state machine.

Responsibilities:

-   lifecycle
-   connection
-   authentication
-   browser/session
-   retry
-   timeout
-   recovery
-   fallback
-   health monitoring

The Runtime Manager absorbs runtime complexity.

Upper layers must never observe runtime state transitions.

------------------------------------------------------------------------

## Runtime Principle

If an external dependency has lifecycle or state, introduce Runtime
Management.

Do not expose infrastructure failures to business logic.

Recover locally whenever possible.

Return structured runtime errors only when recovery is impossible.

------------------------------------------------------------------------

## Cloud-First Development

Cloud:

-   Contract
-   Domain Service
-   Validate
-   Adapter
-   Mock Runtime
-   Unit Tests
-   Skill

Local:

-   MCP
-   Browser
-   Chrome Login
-   Playwright/CDP
-   Integration Tests

Only integration requires the local environment.

------------------------------------------------------------------------

## Architectural Rules

1.  Contract first.
2.  Capability before abstraction.
3.  One responsibility per layer.
4.  Runtime complexity terminates in Runtime Manager.
5.  Public APIs remain stable.
6.  Add Adapters instead of changing Contracts.
7.  Extend Runtime instead of leaking infrastructure upward.
8.  Test contracts before integrations.

------------------------------------------------------------------------

## Success

A good architecture has:

-   Stable Contracts
-   Replaceable Runtime
-   Platform-independent domain logic
-   Cloud-first development
-   Local-only integration testing
-   Predictable runtime recovery
