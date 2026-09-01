---
name: create-design
description: >-
  Create a technical feature design from an approved story, requirements
  contract, and Gherkin scenarios, recording consequential decisions as ADRs.
---

# Create Design

## Purpose

This skill explains how an approved feature will be realized. It converts
observable requirements into components, interfaces, state flow, data choices,
risks, alternatives, and verification. It must not silently change behavior.

## Invocation

Examples:

- `Create the design for specs/001-create-collection/`.
- `Design US-001 from its approved requirements and scenarios.`
- `Complete design.md and record any consequential decisions for this feature.`

Use the current working directory as the target project unless the user gives an
explicit project path. Resolve all paths relative to that root.

## Prerequisites

1. Resolve the target feature directory under `specs/NNN-feature-slug/`.
2. Read the complete approved `user-story.md`, `scenarios.feature`, and `requirements.md`.
3. Read the parent PRD, related ADRs, project context, and the complete `specs/CONSTITUTION.md` (for C#/.NET scope) and/or `specs/CONSTITUTION_FRONTEND.md` (for `Web.Spa` scope).
4. Inspect the current codebase when implementation code exists.
5. Inspect whether `design.md` already exists.

Stop without writing when requirements are not approved, behavior is ambiguous,
or a technical decision is required but no decision authority is available. Ask
the user rather than choosing a product behavior or silently adding an
architectural constraint.

## Design Rules

- Requirements define what; this document defines how.
- Keep the design bounded to the selected vertical slice.
- Define components by responsibility, not speculative abstractions.
- Define interfaces in domain terms and include inputs, outputs, and errors.
- Show data and state flow, including success, failure, rollback, and recovery paths.
- Record security, performance, operational, migration, and compatibility implications.
- Consider alternatives and explain why the selected approach fits the approved constraints.
- A consequential or durable decision requires an ADR under `specs/adr/` with the next `ADR-NNN` ID and explicit alternatives and consequences.
- Do not introduce a NuGet package, project reference, architectural layer, or dependency without the constitution's required human approval.
- Design for the dependency direction, port boundaries, typed errors, tests-first loop, and tooling gates in the applicable constitution: `specs/CONSTITUTION.md` for C#/.NET, `specs/CONSTITUTION_FRONTEND.md` for `Web.Spa`.
- If implementation reveals that the requirements are wrong, stop and revise the requirements before revising the design.

## Supporting Artifacts

When the design requires them, create the supporting artifact under its mapped
skill ownership:

- **Diagrams (`CHART-NNN`)** at `specs/charts/CHART-NNN.md` from
  `specs/templates/supporting/chart.md` — central Mermaid diagrams for
  architecture, design, or business processes. Keep Mermaid in fenced `mermaid`
  blocks.
- **Databases (`DB-NNN`)** at `specs/schema/DB-NNN.md` and **Tables
  (`TABLE-NNN`)** at `specs/schema/TABLE-NNN.md` from their templates when the
  approved behavior needs persisted state. Table frontmatter MUST carry
  `database: DB-NNN` and database frontmatter MUST list its tables — strict
  bidirectional references. Every table has a domain-independent primary key
  named `id` (CONSTITUTION R-DB-01).

All supporting artifacts use templates, allocate IDs per
`specs/SDD_WORKFLOW.md` §Identifier Rules, are written with `status: draft`,
and link back to `design.md`.

## Output

Write the sibling `design.md` using `specs/templates/feature/design.md` when
available. Preserve the existing design ID when revising; use the next
repository-wide `DES-NNN` value for a new artifact. Link `REQ-NNN` and all related
ADRs in front matter.

The file must contain:

- Context and constraints.
- Proposed design.
- Components and responsibilities.
- Interfaces and contracts.
- Data and state flow.
- Security, performance, and operations.
- Alternatives considered.
- Risks and open decisions.
- Verification approach.

Set `status: draft` while the design is under review. Do not create
`tasks.md` or implementation code as a side effect.

## ADR Handling

When a consequential decision is needed:

1. State the decision and the user-visible or architectural reason.
2. List credible alternatives and tradeoffs.
3. Ask for clarification when the decision is not already approved.
4. **ADR Immutability Rule:** If revising an existing approved/accepted ADR, do NOT edit it in-place. Create a new `ADR-NNN` with `supersedes: ADR-OLD` and mark `ADR-OLD` as `status: superseded` via `promote-artifact`. Only draft ADRs may be revised in-place.
5. Create or supersede the ADR only with explicit confirmation, writing `status: draft`; approval transitions go through `promote-artifact`.
6. Link the ADR from `design.md` and keep its status consistent with its review state.

Do not treat a template as an ADR, and never reuse an ADR ID. The independent
`ADR-NNN` sequence includes active, archived, and superseded records (ID rules:
`specs/SDD_WORKFLOW.md` §Identifier Rules).

## Review And File Writes

1. Synthesize the proposed technical approach, interfaces, state flow, risks, decisions, and any supporting artifacts in chat.
2. Identify any blocking decisions and stop if one remains unresolved.
3. Ask whether to write the design (and supporting artifacts) as a draft or revise it; approval transitions are owned by `promote-artifact`.
4. Never overwrite an existing `design.md`, ADR, or schema artifact silently; inspect it and require explicit confirmation before revising it.
5. After writing, report the design ID, ADR IDs, supporting artifact IDs, statuses (`draft`), deferred decisions, verification approach, and the instruction to promote via `promote-artifact`.

## Completion Checklist

- Story, scenarios, and requirements are approved and mutually consistent.
- The design is bounded to the selected slice.
- Interfaces, errors, state transitions, and recovery behavior are explicit.
- Every consequential decision is recorded or explicitly identified as deferred.
- Related ADRs are linked and their statuses are accurate.
- No new behavior or unapproved dependency has been introduced.
- .NET constraints from `specs/CONSTITUTION.md` and frontend constraints from `specs/CONSTITUTION_FRONTEND.md` are reflected where applicable.
- Required CHART/DB/TABLE artifacts exist with valid bidirectional references.
- Verification covers unit, integration, acceptance, and relevant non-functional checks.
- No tasks or code were created as a side effect.
