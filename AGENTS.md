# Project Agent Instructions

## Repository Purpose

This repository uses a specification-driven development workflow to turn
product intent into independently valuable, implementation-ready feature
packets. The workflow is defined by `specs/SDD_WORKFLOW.md`.

## Read First

- Read `specs/SDD_WORKFLOW.md` before any workflow action.
- Read `specs/CONSTITUTION.md` before the first C#/.NET edit.
- Read `specs/CONSTITUTION_FRONTEND.md` before the first `Web.Spa` edit.
- Read the applicable PRD before refining an epic or implementing a feature.
- Read the complete feature packet before generating code.
- Read related ADRs before making consequential technical decisions.
- Treat `specs/templates/` as source templates, never as active specifications.

The workflow document is authoritative for lifecycle and artifact rules. The
applicable constitution is authoritative for engineering form. Conflicts must
be raised rather than silently resolved.

## Operating Rules

Work on one independently valuable vertical slice at a time:

1. Create or revise product intent with `create-prd`.
2. Refine one approved epic with `refine-epic`.
3. Refine one candidate slice with `refine-user-stories`.
4. Formulate behavior with `user-story-to-gherkin`.
5. Define the observable contract with `create-requirements`.
6. Define the technical approach with `create-design`.
7. Define ordered implementation work with `create-tasks`.
8. Promote artifacts only through `promote-artifact`.
9. Implement approved packets with `implement-feature`.
10. Verify and record evidence with `verify-feature`.
11. Record releases with `record-release`.
12. Triage post-release observations with `triage-observation`.

Authoring skills create drafts only. Never edit lifecycle status by hand.
Stop and ask when a product, domain, dependency, behavior, or technical
question blocks progress. Do not invent missing requirements.

## Engineering Summary

- .NET work follows `specs/CONSTITUTION.md`.
- React/TypeScript work follows `specs/CONSTITUTION_FRONTEND.md`.
- Tests are written before implementation.
- Every specified behavior is traceable to a test.
- New dependencies, project references, and architectural layers require
  explicit human approval in the current session.
- Completion requires observed quality-gate output, not intent.

QMD is optional research support and must never be a runtime dependency.
