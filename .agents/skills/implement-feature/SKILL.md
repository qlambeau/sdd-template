---
name: implement-feature
description: >-
  Implement an approved feature packet test-first, following the constitutional
  RED -> GREEN -> REFACTOR -> TRACE engineering cycle.
---

# Implement Feature

## Purpose

This skill guides the implementation of a spec-ready feature packet. It strictly
enforces test-first development governed by the applicable constitution:
`specs/CONSTITUTION.md` for C#/.NET code, `specs/CONSTITUTION_FRONTEND.md` for
`Web.Spa` (React/TS) code.

## Invocation

Examples:

- `Implement feature specs/001-packet-dashboard/`
- `Drive the red-to-green implementation for US-001`

## Prerequisites

1. Verify that the feature packet passes the Spec-Ready Predicate (`user-story.md`,
   `scenarios.feature`, `requirements.md`, `design.md`, `tasks.md`, and all
   referenced ADRs/schemas are all `status: approved`). Validate via `promote-artifact`.
2. Read the complete applicable constitution(s) before writing or modifying code: `specs/CONSTITUTION.md` for C#/.NET, `specs/CONSTITUTION_FRONTEND.md` for `Web.Spa`.
3. Read the packet's `design.md`, `tasks.md`, and related ADRs/schemas.

## Execution Rules (The Constitutional Loop)

1. **RED Tests First:**
   - Author unit, integration, or scenario tests covering the planned tasks in `tasks.md`.
   - Execute tests and observe that they fail with the expected failure output.
   - Do NOT proceed to implementation before red test output is observed.
2. **GREEN Minimal Implementation:**
   - Author the minimal production code necessary to make the red tests pass.
   - Execute tests and observe that they pass.
3. **REFACTOR & TRACE:**
   - Clean up code, remove duplication, ensure idiomatic naming and error handling.
   - Attach traceability comments (`// Covers: REQ-NNN`) on public items and tests (CONSTITUTION R-SDD-02 / R-FE-SDD-02).
4. **Constitution Constraints:**
   - Do NOT add NuGet packages, project references, or layers without explicit current-session approval (R-AGT-02).
   - Do NOT write `unsafe` code (R-AGT-04).
   - Do NOT weaken tests or loosen analyzers (R-AGT-03).
5. **Specification Match:**
   - If implementation reveals a behavioral discrepancy, STOP and update the specification first.

## Handoff

When all ordered tasks are coded and passing locally, invoke `verify-feature` to execute
the quality gates defined by the applicable constitution (`dotnet format` /
`dotnet build -warnaserror` / `dotnet test` with coverage for .NET;
`npm run typecheck` / `lint` / `format:check` / `test --coverage` / `build` for
`Web.Spa`) and record completion evidence.
