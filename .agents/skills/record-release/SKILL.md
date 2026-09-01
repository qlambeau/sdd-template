---
name: record-release
description: >-
  Compile a durable release record (specs/releases/REL-NNN.md) documenting included
  features, closed observations, quality gate evidence, and eval baselines.
---

# Record Release

## Purpose

This skill documents release milestones in a versioned artifact, ensuring traceability
between code commits, delivered feature packets, resolved observations, and quality metrics.

## Invocation

Examples:

- `Record release REL-001 for v0.1.0 baseline`
- `Compile release record for v0.2.0`

## Release Record Contents

Each `specs/releases/REL-NNN.md` file records:
1. **Frontmatter:** `id: REL-NNN`, `version: vX.Y.Z`, `status: draft`, `commit: <hash>`, `date: YYYY-MM-DD`. Promote the record to `released` with `promote-artifact` after validation.
2. **Included Features:** Table of all `US-NNN` / feature packets included in the release.
3. **Closed Observations:** Table of all `OBS-NNN` resolved in this milestone.
4. **Verification Evidence:** Recorded test summary and any domain evaluation metrics defined by approved ADRs.
5. **Known Open Observations:** Unresolved items deferred to future milestones.
6. **Rollback & Migration:** Database migration instructions and rollback steps.

## Archival Procedure

As part of compiling the release record:

1. Move each delivered packet directory from `specs/NNN-slug/` to
   `specs/archive/NNN-slug/`, preserving file contents and IDs.
2. Hand off to `promote-artifact` to transition each moved packet to
   `status: archived` and the release record to `status: released` (strict
   promote-only; relocation or creation alone is not a status transition).
3. Record the archive moves in the release record's Included Features table.

Allocate `REL-NNN` IDs per `specs/SDD_WORKFLOW.md` §Identifier Rules; never
reuse an ID, including after archiving or supersession.
