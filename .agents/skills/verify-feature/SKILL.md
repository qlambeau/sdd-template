---
name: verify-feature
description: >-
  Execute all repository quality gates, record verification output, and update
  feature packet completion evidence in tasks.md.
---

# Verify Feature

## Purpose

This skill executes the Definition of Done checks for a completed feature packet.
It runs all automated gates, verifies domain evaluation quality where specified,
and records concrete, observable evidence into `tasks.md`.

## Invocation

Examples:

- `Verify implementation of specs/001-packet-dashboard/`
- `Run verification gates and record evidence for US-001`

## Verification Gates

1. **Repository CI Suite (.NET)** — run every gate defined in `specs/CONSTITUTION.md`
   §13 / R-TOOL-04 with observed output when C#/.NET code is in scope:
   ```bash
    dotnet format --verify-no-changes
    dotnet build --no-restore -warnaserror
    dotnet test --collect:"XPlat Code Coverage" --settings coverlet.runsettings
    dotnet list package --vulnerable
   ```
   Coverage thresholds per R-TST-16 (≥85% line solution-wide, ≥95% in `src/Domain`).
    If a canonical single-command wrapper exists (e.g. `build.ps1 ci`), use it.
    Locked-mode restore (`dotnet restore --locked-mode`) is required when the
    repository commits `packages.lock.json` files.
2. **Repository CI Suite (Web.Spa)** — run every gate defined in
   `specs/CONSTITUTION_FRONTEND.md` §13 / R-FE-TOOL-04 with observed output when
   `Web.Spa` (React/TS) code is in scope:
   ```bash
   npm run typecheck
   npm run lint
   npm run format:check
   npm test -- --coverage
   npm run build
   ```
   Coverage thresholds per R-FE-TST-11 (≥80% of `Web.Spa/src` production code).
   If a canonical single-command wrapper exists (e.g. `npm run ci`), use it.
3. **Domain Evaluation Gates (when applicable):** run only gates defined by an
   approved ADR or PRD constraint for the feature; record observed metrics
   against their stated thresholds. Never cite thresholds from an ADR that does
   not exist in this repository.

## Evidence Recording

1. Record observed command output, date, and commit hash in `tasks.md` under `## Test And Verification Plan` and `## Definition Of Done`.
2. Check all completed task boxes `[x]`.
3. Hand off to `promote-artifact` to transition the packet to `status: implemented`
   (strict promote-only; this skill records evidence but does not transition status).
4. Report the observed verification summary.
