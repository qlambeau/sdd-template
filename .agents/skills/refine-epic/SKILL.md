---
name: refine-epic
description: >-
  Refine one epic from an approved PRD into an epic brief — outcome, capability
  boundaries, candidate vertical slices, and success criteria — through an
  adaptive, one-question-at-a-time interview. Does not author stories.
---

# Refine Epic

## Purpose

This skill performs just-in-time epic refinement between PRD approval and story
refinement (`specs/adr/ADR-003.md`). It turns a PRD epic row into an actionable
epic brief that gives story refinement a reviewed seed list of candidate
vertical slices. It produces exactly one brief per run and never authors
stories, scenarios, requirements, designs, or tasks.

## Invocation

Examples:

- `Refine EPIC-003 from specs/prds/PRD-001.md`
- `Refine the persistence epic for the current product PRD`
- `Refine the next eligible epic in PRD-001`

Use the current working directory as the target project unless the user gives an
explicit project path.

## Prerequisites

1. Resolve the target root from the explicit path or current working directory.
2. Locate the applicable PRD under `specs/prds/`, or use an explicitly selected
   PRD path.
3. If no applicable PRD exists, stop without writing files and tell the user to
   run `create-prd` first.
4. Verify that the selected PRD is in `status: approved`. If it is still in
   `status: draft`, stop and tell the user to run `promote-artifact`.
5. If no epic was supplied, list the epics from the applicable PRD and ask the
   user to choose one. Prefer epics whose dependencies are satisfied.
6. If the requested epic cannot be found, stop without writing files and ask
   the user to identify an existing epic or revise the PRD via `create-prd`.
7. Inspect whether `specs/prds/PRD-NNN-epics/EPIC-NNN.md` already exists; if it
   does, treat the run as a revision (see File Writes).

Do not invent a PRD, epic, outcome, business rule, or dependency to make the
skill continue. Optional `specs/glossary.md`, `specs/product.md`,
`specs/CONSTITUTION.md`, `specs/tech.md`, and `specs/context.md` files may
provide context when present, but none is required by this skill.

## Epic Scope And Output

Produce one epic brief per run at:

```text
specs/prds/PRD-NNN-epics/EPIC-NNN.md
```

The `EPIC-NNN` identifier is **per-PRD scoped** (`specs/adr/ADR-003.md`): reuse
the exact ID of the epic row in the parent PRD. Never allocate a new or global
epic number here. Create the `PRD-NNN-epics/` directory on first write.

The brief must contain:

- Refined outcome statement (capability level, no implementation).
- Capability boundaries: in scope and out of scope.
- Candidate vertical slices: independently valuable slice seeds for
  `refine-user-stories`, each with user value and notes/risks.
- Epic success criteria: observable, or explicitly marked `TBD`.
- Dependencies: epics and required artifacts this epic depends on.
- Open and blocking questions.

Use the canonical `specs/templates/supporting/epic.md` template when available.
Set `parent` to the source PRD ID and record epic dependencies in frontmatter
(`depends_on`). Write with `status: draft` (strict promote-only,
`specs/adr/ADR-002.md`); approval happens exclusively via `promote-artifact`.

Do not update the parent PRD and do not create any downstream artifacts.

## Adaptive Interview

- Ask exactly one question at a time and wait for the answer.
- Start from the PRD epic row and ask only for the highest-value missing
  information.
- Clarify the refined outcome, capability boundaries, what deliberately stays
  out of scope, candidate slices, success criteria, and epic dependencies as
  needed.
- Use concrete examples and edge cases to expose ambiguity about scope.
- Keep the brief free of database schemas, class or method names, UI styling,
  and other implementation prescriptions. Slice candidates describe user value,
  not architecture.
- Distinguish normal open questions from blocking questions.
- Stop before writing when a blocking product, domain, dependency, or behavior
  question remains.

## Confirmation And File Writes

1. Synthesize the proposed epic brief in the chat.
2. Show the path, parent PRD, epic ID, outcome statement, boundaries, candidate
   slices, success criteria, dependencies, and unresolved questions.
3. Ask for explicit approval or requested changes.
4. Only after approval, write the brief with `status: draft`. Do not create
   empty downstream files.
5. Never advance the status beyond `draft`; tell the user to run
   `promote-artifact` for review and approval.

Never overwrite a brief silently. If the target file exists, read it first,
explain whether this is a revision, and require explicit confirmation before
patching or replacing it. Preserve existing decisions and open questions unless
the user changes them.

## Completion Checklist

Before writing, verify:

- The brief traces to a real approved PRD and epic row.
- The outcome is stated without prescribing implementation.
- Boundaries exclude future epics and speculative capabilities.
- At least one independently valuable candidate slice is identified.
- Success criteria are observable or marked `TBD`.
- Blocking questions are resolved.
- The user explicitly approved the synthesized brief content.

Report the created path, epic ID, parent PRD (`status: draft`), remaining
non-blocking open questions, and the instruction to promote via
`promote-artifact` after writing.
