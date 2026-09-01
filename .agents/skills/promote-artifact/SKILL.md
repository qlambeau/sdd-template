---
name: promote-artifact
description: >-
  Validate prerequisite checklists and advance the lifecycle status of a specification
  or feature packet (e.g. draft -> in-review -> approved -> implemented -> archived).
---

# Promote Artifact

## Purpose

This skill is the explicit and **sole** state-control mechanism for the SDD
workflow (strict promote-only, `specs/adr/ADR-002.md`). It prevents "ghost drafts"
and unverified status claims by evaluating formal prerequisite checklists before
transitioning an artifact or packet's `status` frontmatter. No other skill may
advance a status beyond `draft`; authoring skills hand off here.

## Invocation

Examples:

- `Promote specs/014-unify-query-semantics/requirements.md to approved`
- `Promote specs/015-embedding-dimensions/ to spec-ready`
- `Promote specs/001-create-collection/ to archived`

## State Transitions

Allowed artifact states:
- `draft -> in-review`: When initial authoring is complete.
- `in-review -> approved`: When human reviewer explicitly confirms the specification.
- `approved -> implemented`: When `verify-feature` has verified passing red/green tests and quality gates.
- `implemented -> archived`: When the feature is part of a closed release milestone and moved to `specs/archive/`.
- `approved -> released`: For a release record after its included features and evidence are validated.
- `* -> superseded`: When replaced by a successor artifact (`supersedes: ID`).

## Checklist Validation Rules

Before advancing an epic brief to `approved`:
1. **Parent Approval**: Parent PRD is `status: approved`.
2. **ID Match**: Brief ID matches the parent PRD's epic row (`specs/adr/ADR-003.md`).
3. **Completeness**: Refined outcome, capability boundaries, at least one candidate slice; no unresolved `TBD` in normative sections.
4. **Blockers**: No open blockers (`blockers: []` or all marked `resolved`).
5. **Dependencies**: `depends_on` epics are declared and approved or implemented.

Before advancing an artifact to `approved`:
1. **Parent Approval**: Ensure parent PRD/story is `status: approved`.
2. **Completeness**: No unresolved `TBD` or placeholder values in normative sections.
3. **Traceability**: Requirements trace to story acceptance criteria and named scenarios.
4. **Blockers**: No open blockers (`blockers: []` or all marked `resolved`).
5. **Bidirectional Schema Links**: Database and table references match (`database:` in table frontmatter, `tables:` in database frontmatter).
6. **Scenario Metadata**: For `scenarios.feature`, validate the `# parent: US-NNN` and `# status:` comment headers; allowed statuses match the repository lifecycle.
7. **Epic Linkage** (`specs/adr/ADR-004.md`): For `user-story.md`, the `epic: EPIC-NNN` frontmatter MUST reference an existing epic brief under the parent PRD's `PRD-NNN-epics/` directory that is `status: approved`; packets stay flat at `specs/NNN-feature-slug/` regardless of ownership.

Before advancing a packet to `spec-ready` (ready for implementation):
- Parent PRD: `status: approved`
- Parent epic brief (`specs/prds/PRD-NNN-epics/EPIC-NNN.md`): `status: approved`
- `user-story.md`: `status: approved`
- `scenarios.feature`: `# status: approved`
- `requirements.md`: `status: approved`
- `design.md`: `status: approved`
- `tasks.md`: `status: approved`
- All referenced ADRs and schemas (CHART/DB/TABLE): `status: approved`
- All `depends_on` dependencies: `status: implemented`

Before advancing to `implemented`: require recorded `verify-feature` evidence
(observed gate output) in `tasks.md`. Before advancing to `archived`: require
the packet directory to have been relocated to `specs/archive/` by
`record-release`.

Before advancing a release record to `released`:
1. Every included feature is `implemented` or `archived`.
2. Verification evidence and release commit are recorded.
3. Migration and rollback information is present when persistence is affected.
4. Known open observations are explicitly listed.

## Review And Write

1. Read the target artifact and its related dependencies.
2. Check all items against the transition checklist.
3. Report any validation failures or missing upstream approvals.
4. If validation passes, update `status: <target_state>` and set `updated: YYYY-MM-DD` and `approval` metadata. For Gherkin files, update the `# status:` comment header.
