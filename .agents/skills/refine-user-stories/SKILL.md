---
name: refine-user-stories
description: >-
  Refine one user story from an approved epic brief's candidate slices through
  an adaptive, one-question-at-a-time interview, then write the confirmed story
  into a numbered feature directory without generating Gherkin or task specs.
---

# Refine User Stories

## Purpose

This skill performs just-in-time story discovery below an approved project or
major-feature PRD. It produces a business-focused story file that can later be
formulated into Gherkin by the `user-story-to-gherkin` skill.

## Invocation

Examples:

- `Refine a user story for the packet-dashboard slice of EPIC-003.`
- `Refine one story from specs/prds/PRD-001-epics/EPIC-002.md.`
- `Refine three stories for EPIC-002, starting with the highest-value slice.`

Use the current working directory as the target project unless the user gives an
explicit project path. Accept an explicit epic brief path or candidate-slice
reference, and accept an optional story count or scope. If no count or scope is
supplied, default to one story.

## Prerequisites

1. Resolve the target root from the explicit path or current working directory.
2. Locate the approved parent epic brief at `specs/prds/PRD-NNN-epics/EPIC-NNN.md`,
   or use an explicitly selected epic brief path.
3. If no applicable epic brief exists, stop without writing files and tell the
   user to run `refine-epic` first (the PRD epic row alone is not sufficient —
   `specs/adr/ADR-003.md`).
4. Verify that the epic brief is in `status: approved`. If it is still in
   `status: draft`, stop and tell the user to run `promote-artifact`.
5. If no slice was supplied, list the candidate vertical slices from the epic
   brief and ask the user to choose one.
6. If the requested slice cannot be found, stop without writing files and ask
   the user to pick a listed slice or refine the epic brief.

Do not invent an epic, outcome, persona, business rule, or dependency to make
the skill continue.

Optional `specs/glossary.md`, `specs/product.md`, `specs/CONSTITUTION.md`,
`specs/tech.md`, and `specs/context.md` files may provide context when present,
but none is required by this skill. QMD is optional and must not be a runtime
dependency.

## Story Scope And Output

Produce one independently valuable, INVEST-shaped story per run by default. If
the user requests multiple stories, keep each story independently scoped and
preview each result before writing it.

For each approved story, allocate the next sequential directory number per
`specs/SDD_WORKFLOW.md` §Identifier Rules (scan active and archived
`specs/NNN-*/` directories). The default mapping is
`specs/001-feature-slug/user-story.md` with `id: US-001`. Keep the ID stable if
the title or slug changes. Allow an explicit ID only when the user is importing
an existing story and confirm collisions instead of reusing an ID.

The story file must contain:

- Story card: actor, goal, and value.
- Context and value.
- Business rules.
- Concrete examples.
- Concise, testable acceptance criteria as bullets or tables.
- In-scope and out-of-scope boundaries.
- Dependencies.
- Open and blocking questions.
- An INVEST check.

Set `parent` to the source PRD ID, record the source epic (`epic: EPIC-NNN`)
from the approved epic brief, and keep the story traceable:
story → epic brief → PRD. Use the canonical
`specs/templates/feature/user-story.md` template when available. Write with
`status: draft` (strict promote-only, `specs/adr/ADR-002.md`); approval happens
exclusively via `promote-artifact`.

Do not generate `scenarios.feature`, `requirements.md`, `design.md`, or
`tasks.md`. Do not update the PRD or create a central story index.

## Adaptive Interview

- Ask exactly one question at a time and wait for the answer.
- Start from the selected candidate slice of the approved epic brief and ask
  only for the highest-value missing
  information.
- Clarify the actor, user goal, business value, scope boundaries, rules,
  examples, error paths, dependencies, and measurable expectations as needed.
- Use concrete examples and edge cases to expose ambiguity.
- Keep the story free of database schemas, class or method names, UI styling,
  and other implementation prescriptions.
- Apply INVEST throughout the interview, especially independence, small size,
  value, and testability.
- Distinguish normal open questions from blocking questions.
- Stop before writing when a blocking product, domain, dependency, or behavior
  question remains.

## Confirmation And File Writes

1. Synthesize the proposed story in the chat.
2. Show the proposed ID, path, parent PRD, source epic, story card, rules,
   examples, scope, dependencies, and unresolved questions.
3. Ask for explicit approval or requested changes.
4. Only after approval, create the numbered feature directory and
   `user-story.md` with `status: draft`. Do not create empty downstream files.
5. Never advance the status beyond `draft`; tell the user to run
   `promote-artifact` for review and approval.

Never overwrite a story silently. If the target path or ID exists, read the
existing artifact, explain whether this is a revision or collision, and require
explicit confirmation before patching or replacing it. Preserve existing
decisions and open questions unless the user changes them.

## Completion Checklist

Before writing, verify:

- The story traces to an approved epic brief and its parent PRD.
- The actor, goal, and value are explicit.
- The story is independently valuable and small enough for roughly 1 to 3 days.
- Rules, examples, acceptance criteria, and boundaries are testable.
- Implementation details have not leaked into the story.
- Blocking questions are resolved.
- The user explicitly approved the synthesized story content.

Report the created path, story ID (`status: draft`), parent PRD, source epic,
any remaining non-blocking open questions, and the instruction to promote via
`promote-artifact` after writing.
