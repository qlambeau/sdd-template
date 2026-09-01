# SDD Process Audit

**Date:** 2026-09-01
**Scope:** Full audit of the repository's SDD workflow, engineering constitutions,
templates, ADRs, lifecycle guide, and mapped workflow skills.
**Assessment:** Strong written policy, weak executable enforcement.

## Executive Summary

The process has clear intent and unusually detailed engineering guidance. Its
main weakness is that the central controls are not currently executable or
internally consistent. Lifecycle transitions, spec readiness, traceability,
RED/GREEN evidence, and release archival depend primarily on an agent following
Markdown instructions correctly.

The repository is a template rather than an application. It contains no
solution, frontend project, CI workflow, or validator, so the implementation
quality gates could not be executed during this audit.

## Findings

### Critical

#### C-01: `spec-ready` is an undefined lifecycle state

The normative lifecycle defines `draft`, `in-review`, `approved`,
`implemented`, `archived`, `released`, and `superseded`, but not `spec-ready`
(`specs/SDD_WORKFLOW.md:91-103`). The README, promotion skill, and
implementation skill nevertheless use `spec-ready` as a promotion target
(`README.md:27-33`; `.agents/skills/promote-artifact/SKILL.md:20-34,54-68`;
`.agents/skills/implement-feature/SKILL.md:26-29`).

This makes the implementation entry gate ambiguous and prevents a consistent
status validation.

**Recommendation:** Make `spec-ready` a non-persisted packet validation
predicate, or formally add it to the lifecycle with type-specific transitions.

#### C-02: Lifecycle enforcement is entirely prose-based

The workflow claims that `promote-artifact` is the sole state-control and
validation mechanism (`specs/SDD_WORKFLOW.md:196-213`; `.agents/skills/promote-artifact/SKILL.md:12-16`),
but the repository contains no executable promotion validator, frontmatter
schema validator, ID allocator, cross-reference checker, or CI workflow.

Manual status edits and malformed relationships cannot be detected reliably.

**Recommendation:** Add a repository-local validator that checks frontmatter,
IDs, transitions, prerequisites, cross-references, placeholders, and packet
readiness. Run it locally and in CI.

**Resolution Plan: Deterministic Python CLI Gatekeeper**

A small, non-AI Python CLI is an appropriate executable implementation of this
policy. It should be independent of the product runtime, deterministic, usable
offline, and called by both the workflow skills and CI.

Suggested command surface:

```text
sdd validate --all --strict
sdd validate specs/001-feature/
sdd check-ready specs/001-feature/
sdd promote <artifact-or-packet> --to <state> --actor <name>
sdd allocate-id <prefix>
```

The CLI should validate:

- Frontmatter schema, required fields, artifact types, and status values.
- ID format, uniqueness, allocation, and collisions across active, archived,
  and superseded artifacts.
- Parent, epic, dependency, and cross-reference integrity.
- Allowed transitions by artifact type, including rework, supersession,
  implementation, release, archival, and observation states.
- Placeholder values, unresolved blockers, and the complete Spec-Ready
  Predicate.
- Approval metadata, schema/table bidirectional links, and verification
  evidence.

`promote` should be the only supported status-mutating command. Authoring skills
should create drafts and invoke this command rather than editing lifecycle
fields. `spec-ready` should be implemented as the read-only result of
`check-ready`, not as a persisted status, resolving C-01.

The CLI alone cannot prevent a direct Markdown edit. Enforcement requires
`sdd validate --all --strict` to run in CI on every pull request, with GitHub
branch protection requiring that check to pass. Protect the validator and its
transition definitions with code ownership and review. This makes the CLI the
single policy authority while CI becomes the enforcement boundary.

The CLI cannot judge whether a product decision is semantically correct, whether
a human genuinely approved it, or whether a test is meaningful. Those remain
human-review and test-execution responsibilities. The CLI should validate their
required records and evidence, not pretend to replace them.

Suggested implementation layout:

```text
tools/sdd/
  pyproject.toml
  sdd/
    cli.py
    models.py
    parser.py
    schema.py
    state_machine.py
    validator.py
  tests/
    fixtures/
```

Acceptance criteria for resolving C-02:

- Invalid artifacts and invalid transitions return non-zero exit codes with
  actionable diagnostics.
- Valid packets pass `sdd check-ready` only when every readiness condition is
  satisfied.
- Promotions are atomic, collision-safe, and record approval metadata.
- Direct status edits that do not satisfy the transition rules fail CI.
- The CLI has fixture tests for every artifact type and transition.
- CI runs the same validator used locally, with no AI dependency.

### High

#### H-01: Approved artifacts have no valid revision path

Authoring skills explicitly support revising existing approved artifacts and
writing them as `draft`, while the lifecycle defines no `approved -> draft`
transition (`specs/SDD_WORKFLOW.md:91-103,159,237`; `create-prd/SKILL.md:31-35,104-107`).
Non-ADR artifacts also have no consistent successor-artifact rule.

This forces contributors to violate either the status discipline or the
revision requirement.

**Recommendation:** Define a promote-managed rework transition, or require a
new successor artifact with explicit supersession semantics.

#### H-02: The Spec-Ready Predicate is not fully validated

The workflow requires scenario coverage, complete requirements, explicit design
contracts, ordered RED/GREEN tasks, resolved blockers, approved dependencies,
and clean cross-references (`specs/SDD_WORKFLOW.md:176-190`). The promotion
skill mostly checks statuses and a limited set of links
(`.agents/skills/promote-artifact/SKILL.md:54-68`).

A packet can therefore appear ready while remaining materially incomplete.

**Recommendation:** Implement one packet validator whose checks map one-to-one
to every item in the Spec-Ready Predicate and emit a validation report.

#### H-03: End-to-end traceability is not addressable

Gherkin scenarios, story acceptance criteria, and epic candidate slices have no
stable identifiers (`specs/templates/feature/scenarios.feature:1-21`;
`specs/templates/feature/user-story.md:41-55`;
`specs/templates/supporting/epic.md:40-44`). Design and task templates also do
not map requirements to implementation tasks and tests
(`specs/templates/feature/design.md:30-64`;
`specs/templates/feature/tasks.md:28-47`).

The constitutions require every specified behavior to cite a specification ID,
but the chain does not provide stable IDs at every stage
(`specs/CONSTITUTION.md:54-60`; `specs/CONSTITUTION_FRONTEND.md:53-58`).

**Recommendation:** Add IDs for acceptance criteria and scenarios, then require
a Story -> Scenario -> Requirement -> Design -> Task -> Test/evidence matrix.

#### H-04: RED/GREEN evidence and specification matching are not durable

`implement-feature` asks the agent to observe RED and GREEN output
(`.agents/skills/implement-feature/SKILL.md:34-43`), but `verify-feature` only
requires general quality-gate output (`.agents/skills/verify-feature/SKILL.md:54-59`).
The workflow's implementation review and specification-match gate has no mapped
skill (`specs/SDD_WORKFLOW.md:81-84`).

Passing tests can coexist with specification drift, and there is no required
proof that tests failed before implementation.

**Recommendation:** Require test IDs, commands, observed RED/GREEN output,
commit or tree identity, and an explicit specification-match checklist in
`tasks.md`.

#### H-05: Verification skills diverge from constitutional gates

The .NET constitution requires a canonical wrapper, locked restore, coverage
threshold enforcement, and additional CI checks. The frontend constitution
requires `npm audit` and scheduled E2E checks
(`specs/CONSTITUTION.md:446-480`; `specs/CONSTITUTION_FRONTEND.md:267-295`).
`verify-feature` treats wrappers as optional and omits or weakens several of
these checks (`.agents/skills/verify-feature/SKILL.md:23-49`).

The .NET test command also differs from the normative command by omitting
`--no-build`, and no coverage configuration exists in this repository.

**Recommendation:** Require project bootstrap to create `build.ps1 ci` and
`npm run ci`, then make verification invoke those exact wrappers and record all
thresholds and CI-only checks.

#### H-06: Gherkin is called executable but has no execution contract

The workflow and task template require Gherkin scenarios to pass, but no runner,
step-binding convention, or verification command is defined
(`specs/SDD_WORKFLOW.md:128-130,230-235`;
`specs/templates/feature/tasks.md:36-46`). `verify-feature` runs only .NET and
frontend commands.

**Recommendation:** Select and document a Gherkin runner per supported stack,
or require every scenario ID to map to an ordinary automated test and verify that
mapping.

#### H-07: Release recording has a broken and unsafe sequence

The workflow orders release recording before archival
(`specs/SDD_WORKFLOW.md:85-87`). `record-release` moves packets during release
compilation and then hands off to promotion, while promotion only defines
`approved -> released` for release records
(`.agents/skills/record-release/SKILL.md:32-41`;
`.agents/skills/promote-artifact/SKILL.md:28-34`). A release starts as `draft`
and has no documented `draft -> in-review -> approved -> released` sequence.

Archival may occur before release approval and has no rollback procedure.

**Recommendation:** Define the exact release state machine, validate the release
before moving packets, and make archive moves reversible or transactional.

#### H-08: Observation triage bypasses the mandatory workflow

The triage skill instructs contributors to edit the PRD, change the observation
status, and refine a story directly (`.agents/skills/triage-observation/SKILL.md:34-39`).
This bypasses `create-prd`, `refine-epic`, and `promote-artifact`, contrary to the
workflow mapping (`specs/SDD_WORKFLOW.md:196-213`).

**Recommendation:** Let triage record and classify observations only. Route new
product scope through the normal PRD and epic workflow, with every status change
performed by promotion.

#### H-09: The PRD lifecycle guide contradicts epic refinement

The decision graph routes ordinary behavior directly to
`refine-user-stories` (`specs/prd_lifecycle_and_evolution_plan.md:20-29`), while
the normative workflow and ADR-003 require an approved epic brief first
(`specs/SDD_WORKFLOW.md:50-64`; `specs/adr/ADR-003.md:25-38`).

**Recommendation:** Route ordinary behavior through an existing approved epic,
or require `refine-epic` when the epic has not yet been refined.

### Medium

#### M-01: Lifecycle rules are not type-specific

The shared lifecycle presents the same states for every artifact, although
`implemented`, `released`, and `archived` apply differently to feature packets,
release records, ADRs, and supporting artifacts. The wildcard supersession rule
also conflicts with ADR immutability (`specs/SDD_WORKFLOW.md:91-103`;
`.agents/skills/promote-artifact/SKILL.md:28-52`).

**Recommendation:** Publish a transition matrix by artifact type and define
which artifacts may be implemented, released, archived, or superseded.

#### M-02: Approval metadata has no canonical schema

Most templates omit approval metadata, while promotion is instructed to add it
without defining required fields or reviewer semantics
(`.agents/skills/promote-artifact/SKILL.md:76-81`; feature templates' frontmatter).
Current status alone does not provide a durable approval audit trail.

**Recommendation:** Define required approval actor, date, decision, and evidence
fields, and validate them during promotion.

#### M-03: ID allocation rules are inconsistent

Epic numbering is described as per-PRD but omitted from the independent sequence
list. Story refinement also couples packet-directory numbering to `US-NNN`,
despite the workflow declaring those sequences independent
(`specs/SDD_WORKFLOW.md:140-165`; `.agents/skills/refine-user-stories/SKILL.md:59-64`).
`DEC-NNN` has no defined allocation or artifact process.

**Recommendation:** Provide one allocator that scans active, archived,
superseded, and historical IDs, and define epic and decision identifiers
explicitly.

#### M-04: Supporting artifact linkage is incomplete

`create-design` requires charts, databases, and tables to link back to
`design.md`, but the supporting templates do not provide canonical design
references. Charts are also absent from the artifact responsibility table
(`.agents/skills/create-design/SKILL.md:54-72`;
`specs/templates/supporting/chart.md:1-10`;
`specs/SDD_WORKFLOW.md:116-136`).

**Recommendation:** Add structured feature/design references and validate them
bidirectionally for every supporting artifact.

#### M-05: Applicability and project bootstrap are underspecified

The workflow refers to the “applicable” constitution but defines no formal stack
scope or `N/A` evidence. The constitutions also contain unconditional DoD items
for work that may not involve databases, multiple port implementations, or a
frontend. Project setup is deferred to later implementation without a bootstrap
skill or gate (`README.md:63-68`).

**Recommendation:** Add packet-level stack scope, explicit applicability rules,
and an `N/A` format with rationale. Add a project-bootstrap step before feature
implementation.

#### M-06: Frontend tokens and the .NET glossary lack governed ownership

Frontend rules make `design/epic-*/design-tokens.md` the styling source of truth,
while the design directory is described as optional and non-normative
(`specs/CONSTITUTION_FRONTEND.md:208-213`; `design/README.md:3-21`). The .NET
DoD requires a glossary update, but no canonical glossary artifact or mapped
skill exists (`specs/CONSTITUTION.md:504`).

**Recommendation:** Make these artifacts explicitly required and promotable, or
make their checks conditional with a defined fallback.

#### M-07: Release provenance is ambiguous

Release records require a commit hash, but verification can run on an
uncommitted tree and the process does not define whether the hash identifies the
implementation commit, release-record commit, or build revision
(`specs/templates/supporting/release.md:6-10`;
`.agents/skills/verify-feature/SKILL.md:54-59`).

**Recommendation:** Verify against an immutable CI revision and record the exact
tree or build identity used for evidence.

#### M-08: Constitutional rules contain internal gate inconsistencies

`R-ERR-02` broadly forbids catching `Exception` without mapping or rethrowing,
while `R-ERR-10` permits ignoring it with a comment
(`specs/CONSTITUTION.md:306-319`). Licence checks are required by dependency
rules but are not included in the canonical command set
(`specs/CONSTITUTION.md:446-480`). XML documentation is required but not wired
into the baseline build properties (`specs/CONSTITUTION.md:433-467`).

**Recommendation:** Resolve contradictory wording and make every MUST rule map
to an executable gate or an explicit review record.

#### M-09: The task template hardcodes CLI work

`TASK-NNN-3` requires CLI acceptance integration even for non-CLI features
(`specs/templates/feature/tasks.md:36-38`).

**Recommendation:** Replace it with generic scenario verification and make CLI
checks conditional on the approved contract.

#### M-10: Security assurance is not a concrete gate

Design guidance mentions security, performance, and operations, but the template
leaves them as free-form `TBD` fields and no threat-model, secret-handling,
logging-redaction, CSP/CSRF, or security-evidence gate is defined
(`.agents/skills/create-design/SKILL.md:46-51`;
`specs/templates/feature/design.md:46-50`).

**Recommendation:** Add conditional security requirements and verification checks
for features involving authentication, sensitive data, external input, or
privileged operations.

## Strengths

- Clear authority precedence between the workflow and engineering constitutions.
- Strong separation between product intent, behavior, contracts, design, ADRs,
  and implementation tasks.
- Explicit emphasis on vertical slices and just-in-time epic refinement.
- Strong test-first, dependency-approval, layering, type-safety, and observed
  evidence principles.
- Useful ADRs covering promote-only transitions, epic refinement, and flat
  feature packets.

## Execution Limits

- No `.sln`, `.csproj`, `package.json`, CI workflow, coverage configuration, or
  project gate wrapper exists in this repository.
- The repository now has an initial Git commit, but no release history or
  application implementation exists, so release commit evidence and archival
  behavior could not be exercised.
- No active PRD or feature packet exists, so end-to-end promotion and traceability
  remain untested.

## Recommended Remediation Order

1. Normalize lifecycle states, revision, supersession, observation, and release
   transitions.
2. Build an executable validator for frontmatter, IDs, prerequisites, statuses,
   placeholders, and cross-references.
3. Add stable traceability IDs and scenario-to-test mapping.
4. Establish canonical CI wrappers and enforce coverage and security thresholds.
5. Add bootstrap, applicability, approval-provenance, and security evidence
   rules.

## Overall Assessment

The SDD process is a good policy foundation, but it is not yet an auditable or
reliably executable delivery control. The highest priority is to turn the
promote, readiness, traceability, and verification rules into machine-checkable
contracts before relying on the process for production feature delivery.
