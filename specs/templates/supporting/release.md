---
id: REL-NNN
title: "Release Milestone vX.Y.Z"
type: release-record
status: draft
version: "vX.Y.Z"
commit: "<git-hash>"
date: YYYY-MM-DD
owner: TBD
related: []
---

# Release Record

## Release Overview

Summary of release intent, version milestone, and major capabilities delivered.

## Included Features

| Feature ID | Title | User Story | Status |
| --- | --- | --- | --- |
| NNN | Feature slug | `US-NNN` | implemented |

## Closed Observations & Fixes

| Observation ID | Title | Resolution |
| --- | --- | --- |
| `OBS-001` | Observation title | Promoted to EPIC-008 & implemented |

## Verification Evidence

- `dotnet format --verify-no-changes`: observed clean output
- `dotnet build -warnaserror`: observed green output
- `dotnet test` with coverage: observed passing, coverage >= thresholds (R-TST-16)
- `dotnet list package --vulnerable`: no known vulnerabilities
- `npm run typecheck`, `npm run lint`, `npm run format:check`, `npm test -- --coverage`, and `npm run build`: observed passing when `Web.Spa` is in scope
- `npm audit`: no known vulnerabilities when `Web.Spa` is in scope
- Domain evaluation gates (when defined by an approved ADR): TBD

## Known Open Observations

- `OBS-NNN`: Deferred to next milestone.

## Migration & Rollback

- Schema version: `N`
- Rollback procedure: TBD
