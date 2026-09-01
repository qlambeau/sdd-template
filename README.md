# SDD Workflow Template

This directory is a reusable spec-driven development kit for .NET backends and
React/TypeScript frontends. It contains the workflow authority, engineering
constitutions, artifact templates, and project-local agent skills. It does not
contain a product PRD, application code, database implementation, or example
feature packet.

## Start Here

1. Read `AGENTS.md`.
2. Read `specs/SDD_WORKFLOW.md`.
3. Read the applicable constitution before editing code.
4. Invoke `create-prd` to begin the first product PRD.
5. Use `promote-artifact` for every lifecycle transition.

## Workflow

```text
create-prd
  -> refine-epic
  -> refine-user-stories
  -> user-story-to-gherkin
  -> create-requirements
  -> create-design
  -> create-tasks
  -> promote-artifact: spec-ready
  -> implement-feature
  -> verify-feature
  -> promote-artifact: implemented
  -> record-release
  -> promote-artifact: archived
```

Work on one independently valuable vertical slice at a time. The complete
workflow, artifact boundaries, lifecycle, readiness predicate, identifiers,
and gates are defined in `specs/SDD_WORKFLOW.md`.

## Repository Layout

```text
.agents/skills/              Workflow skills loaded by the agent runtime
specs/SDD_WORKFLOW.md        Normative SDD lifecycle authority
specs/CONSTITUTION.md        C#/.NET engineering authority
specs/CONSTITUTION_FRONTEND.md React/TypeScript engineering authority
specs/prds/                  Product PRDs and epic briefs
specs/NNN-feature-slug/      Flat feature packets
specs/adr/                   Architecture decision records
specs/charts/                Central Mermaid diagrams
specs/schema/                Database and table specifications
specs/releases/              Release records
specs/archive/               Delivered feature packets
specs/templates/             Source templates, not active artifacts
design/                      Non-normative visual references and tokens
```

## Status Discipline

Authoring skills write `draft`. The only lifecycle transition authority is
`promote-artifact`. Human confirmation is required for semantic approval.
Tests and verification evidence do not replace approval.

## Project Setup

This kit does not scaffold application projects. After the first approved
design, use `create-tasks` and `implement-feature` to add the target project's
.NET solution, React application, tests, CI commands, and deployment files.
Adapt the constitutions' example paths and gate commands to that project.

## Optional Research

The `qmd` skill can be used when local Markdown research is explicitly needed.
It requires an installed QMD CLI and is not required for normal SDD operation.
