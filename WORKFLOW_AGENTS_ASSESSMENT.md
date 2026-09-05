# Workflow, Agents, and Skills Assessment

- Assessment date: 2026-09-06
- Repository baseline: `2e956f1ba27b466110ac048e55c70f69e99e12a6`
- Assessment type: static implementation and process review
- Disposition: informational report, not a specification approval or lifecycle transition

## Executive Assessment

**The repository is a well-structured specification-driven development kit, but its multi-agent implementation is not yet a reliably executable or strongly permission-isolated workflow.** The strongest parts are the authoring disciplines, artifact responsibilities, human semantic approval principle, and requirement for observed test evidence. The weakest parts are lifecycle transitions after initial authoring, orchestration across approval boundaries, evidence handoffs, and release recovery.

The implementation consists primarily of Markdown instructions and OpenCode agent permission declarations. All twelve workflow-mapped skills and all eight advertised agents exist. This is meaningful implementation, but it is not an executable state machine: many invariants depend on an agent reconciling distributed prose correctly. Some of that prose is contradictory.

This report identifies **26 consolidated findings: 12 high-priority and 14 medium-priority**. These are process and implementation priorities, not security vulnerability scores. No production incident, successful permission exploit, or live workflow failure was observed.

The most important conclusions are:

- **The advertised cluster sequence cannot run as described.** Skills require approvals inside clusters, but the topology emphasizes approvals after clusters. Final readiness review is also placed before the approvals it requires.
- **Promotion is insufficiently specified.** `spec-ready` is exposed as a target state even though it is a predicate; root/supporting artifact approval, reopening, and packet-wide transitions lack precise rules.
- **Approval is not tied to a stable baseline.** Upstream revisions can leave downstream approvals and verification evidence apparently valid without mandated revalidation.
- **The implementer/verifier split is internally inconsistent.** The implementer must write evidence into a location it cannot edit, and its skill invokes work assigned to another agent.
- **Permission claims exceed the actual boundaries.** Broad shell access bypasses native edit-path restrictions, and most stage agents can edit more specifications and load more skills than their role descriptions permit.
- **Release and observation handling are incomplete.** Archive moves lack a failure-safe protocol, archived dependencies fail the literal readiness predicate, and observation routing skips required stages.

**Recommended disposition:** preserve the overall artifact model and role separation. Resolve the high-priority lifecycle and handoff conflicts before relying on cluster autonomy. Continue treating the kit as a supervised workflow rather than an independently enforcing delivery system.

## Scope and Method

The review covered:

| Area | Coverage |
| --- | --- |
| Lifecycle authority | Complete `specs/SDD_WORKFLOW.md` |
| Agent topology | All eight files in `.opencode/agents/`, plus `AGENTS.md` |
| Workflow skills | All twelve mapped `SKILL.md` files |
| Optional research | Local QMD bootstrap skill; not the external CLI implementation |
| Templates | All thirteen source templates |
| Architecture decisions | ADR-002, ADR-003, ADR-004, and ADR-005 |
| Engineering authorities | Both constitutions, focusing on workflow interfaces, prerequisites, traceability, and gates |
| Supporting guidance | README, PRD lifecycle guide, visual-design guidance, and tracked repository inventory |
| Runtime packaging | Repository-local agent/configuration structure; no live merged-runtime validation |

Three independent read-only review passes examined workflow/artifact rules, agent orchestration, and skill internals. Findings were consolidated, with central conflicts checked directly against the relevant source files. The existing `SDD_PROCESS_AUDIT.md` was not used as evidence and was not modified.

The repository was clean at the recorded baseline. Source references below use repository-relative paths and line ranges from that baseline. They identify instructions on disk, not proof of historical compliance or noncompliance.

### Evidence Boundaries

- Contradictions and missing procedures are directly supported by repository text.
- Example failure paths are reasoned walkthroughs, not executions of the workflow.
- Runtime-related recommendations distinguish configured capabilities from behavioral promises. Global configuration, extensions, the installed OpenCode executable version, and effective merged permissions were not validated live.
- No application solution, frontend application, real product packet, or release history is supplied in this template checkout. Their absence is not itself a defect.
- No build, install, promotion, archival move, publication, or permission-bypass attempt was performed during the assessment.
- The review does not establish model reliability, performance, cost, security certification, or production readiness through empirical benchmarks.
- This report proposes changes; it does not approve new dependencies, architecture, lifecycle transitions, or implementation work.

### Priority Definitions

| Priority | Meaning |
| --- | --- |
| High | Can block a normal conforming workflow, misrepresent readiness/completion, or materially undermine an advertised authority boundary. |
| Medium | Creates significant ambiguity, weakens traceability or repeatability, or leaves an important operational path insufficiently defined. |

## Findings Index

| ID | Priority | Finding |
| --- | --- | --- |
| F01 | High | Cluster gates conflict with intermediate approvals and final readiness review. |
| F02 | High | Readiness is confused with a lifecycle status and incompletely checked. |
| F03 | High | Transition rules are not sufficiently typed by artifact and packet. |
| F04 | High | Revision and downstream approval invalidation are undefined. |
| F05 | High | Supersession can retire the predecessor before successor approval. |
| F06 | High | Archiving a dependency makes it fail readiness. |
| F07 | High | Implementer instructions conflict with permissions and verifier ownership. |
| F08 | High | Shell access defeats file-edit boundaries and exposes policy files. |
| F09 | High | Promotion and stage-output ownership are largely advisory. |
| F10 | High | Release relocation lacks safe ordering, preflight, and recovery. |
| F11 | High | Observation routing bypasses PRD and epic workflow stages. |
| F12 | High | Verification lacks a complete behavior-to-source evidence chain. |
| F13 | Medium | Concrete verification and release gates drift from the constitutions. |
| F14 | Medium | Artifact relationships lack a complete typed reference model. |
| F15 | Medium | Identifier rules conflict and cannot guarantee allocation invariants. |
| F16 | Medium | Isolated contexts lack durable decision and interview handoffs. |
| F17 | Medium | Recovery routes ambiguity to the wrong stage and conflates failure types. |
| F18 | Medium | Universal draft-status validation rejects valid later-stage outputs. |
| F19 | Medium | Topology adoption, startup, and runtime assumptions are unverified. |
| F20 | Medium | Workflow invariants have no executable regression protection. |
| F21 | Medium | Skill prerequisites and template consumption are inconsistent. |
| F22 | Medium | Observation intake and closure do not support reliable release accounting. |
| F23 | Medium | Mandatory frontend styling inputs lack ownership and readiness rules. |
| F24 | Medium | Template placeholder policies conflict with approval rules. |
| F25 | Medium | PRD hierarchy and supersession semantics live outside the authority. |
| F26 | Medium | Behavior-level identities and test traceability are underspecified. |

## High-Priority Findings

### F01. Cluster Gates Conflict With Required Intermediate Approvals

**Evidence:** [orchestrator, lines 39-58](.opencode/agents/sdd-orchestrator.md#L39-L58); [ADR-005, lines 60-71](specs/adr/ADR-005.md#L60-L71); [epic prerequisites, lines 30-38](.agents/skills/refine-epic/SKILL.md#L30-L38); [Gherkin prerequisites, lines 25-28](.agents/skills/user-story-to-gherkin/SKILL.md#L25-L28); [task prerequisites, lines 27-38](.agents/skills/create-tasks/SKILL.md#L27-L38).

The topology says subagents run freely inside clusters and the orchestrator obtains confirmation at cluster boundaries. However, epic refinement needs an approved PRD, Gherkin formulation needs an approved story, design needs approved requirements, and task planning needs approved design/ADRs. Authoring produces drafts, and subagents cannot promote them.

**Impact:** starting cluster 1 with no PRD produces a draft PRD, then immediately blocks epic refinement. The same pattern recurs in clusters 2 and 3. In addition, cluster 3 ends with a spec-ready review before promotion, while the reviewer correctly requires all packet specifications to be approved. A conforming agent must stop or introduce undocumented intermediate gates.

**Recommendations:** retain clusters as organizational groupings, not approval batches. Specify authoring, return, human confirmation, promotion, and resumption at each dependent artifact boundary. Distinguish draft-completeness review from final spec-ready validation; run the latter after all required approvals.

**Acceptance check:** walk a new product from PRD through tasks without consuming an unapproved prerequisite or asking a reviewer to declare an unapproved packet spec-ready.

### F02. Readiness Is Confused With a Status and Incompletely Checked

**Evidence:** [promotion invocation and transitions, lines 20-34](.agents/skills/promote-artifact/SKILL.md#L20-L34); [readiness and write procedure, lines 54-81](.agents/skills/promote-artifact/SKILL.md#L54-L81); [normative predicate, lines 176-190](specs/SDD_WORKFLOW.md#L176-L190); [README sequence, lines 19-32](README.md#L19-L32).

The skill offers promotion "to spec-ready," labels readiness as advancing a packet, and generically writes `status: <target_state>`. The lifecycle does not contain `spec-ready`; it is a predicate. The documented readiness checklist also omits several final checks required by the authority: scenario path coverage, normative TBDs, explicit design contracts, ordered RED/GREEN tasks, empty blockers, and complete cross-reference resolution.

**Impact:** a literal execution could write an unsupported status or declare readiness from approval labels alone. The more complete reviewer checklist mitigates this in some multi-agent runs, but the supported single-agent route still uses the deficient skill instructions.

**Recommendations:** make readiness a validation-only operation with per-check results and no lifecycle edits. Reject unknown state targets. Reconcile the skill checklist item-for-item with the authority, separating mechanical checks from semantic review. Do not infer semantic completeness solely from `approved`.

**Acceptance check:** readiness leaves file contents unchanged; an approved packet with a blocker, missing boundary coverage, or unresolved reference fails with the specific reason.

### F03. Transition Rules Are Not Sufficiently Typed by Artifact and Packet

**Evidence:** [shared lifecycle, lines 91-103](specs/SDD_WORKFLOW.md#L91-L103); [promotion rules, lines 26-52](.agents/skills/promote-artifact/SKILL.md#L26-L52); [promotion write procedure, lines 76-81](.agents/skills/promote-artifact/SKILL.md#L76-L81).

The workflow defines states, but not a complete artifact-type transition matrix. Generic approval requires an approved "parent PRD/story," which does not fit a root PRD or every ADR, schema, chart, and release record. Packet-directory promotion does not specify which constituent files change, how aggregate status is determined, or how partial failure is handled. The metadata called `approval` has no defined schema.

**Impact:** approval of the first root PRD either blocks or depends on an invented exemption. Separate runs can apply different approval prerequisites or different packet status semantics. The reviewer also asks for promotion records without a corresponding durable record format.

**Recommendations:** define allowed transitions and prerequisites by artifact type, including parentless cases, mandatory review stages, terminal states, and packet membership. Specify the exact files and metadata changed by a packet transition. Define an approval record identifying reviewer, artifact baseline, source/target states, and authorization provenance.

**Acceptance check:** root PRD, ADR, schema, scenario, feature packet, and release transitions each have an unambiguous successful path and reject unsupported transitions without partial edits.

### F04. Revision and Downstream Approval Invalidation Are Undefined

**Evidence:** [ADR-002, lines 28-35](specs/adr/ADR-002.md#L28-L35); [PRD revision guide, lines 33-44](specs/prd_lifecycle_and_evolution_plan.md#L33-L44); [allowed promotions, lines 26-34](.agents/skills/promote-artifact/SKILL.md#L26-L34); [artifact boundaries and readiness](specs/SDD_WORKFLOW.md#L105-L118).

Authoring skills create or revise artifacts as draft, while every transition belongs to promotion. No approved-to-draft reopening procedure is defined. Approval is also not bound to a revision, digest, or commit, and upstream changes have no mandatory downstream impact/invalidation procedure.

**Impact:** revising an approved PRD as draft conflicts with promote-only control. Reapproving changed story behavior under the same ID can leave requirements, design, tasks, and old verification evidence marked approved or complete even though they describe the former behavior. Shared schema changes create the same issue across packets.

**Recommendations:** first resolve the lifecycle ambiguity in the authority. Define authorized reopening, retention of approval history, approved baselines, and editorial versus behavioral change handling. Identify affected dependents, invalidate stale approvals/evidence, and require revalidation before implementation resumes. Treat append-only execution evidence separately from changes to approved task scope.

**Acceptance check:** a business-rule revision cannot leave affected downstream artifacts automatically ready; a demonstrably editorial correction follows a documented lighter path.

### F05. Supersession Can Retire a Predecessor Before Successor Approval

**Evidence:** [PRD authoring, lines 31-35](.agents/skills/create-prd/SKILL.md#L31-L35); [design/ADR handling, lines 100-105](.agents/skills/create-design/SKILL.md#L100-L105); [supersession transition, line 34](.agents/skills/promote-artifact/SKILL.md#L34); [normative successor approval, line 103](specs/SDD_WORKFLOW.md#L103).

PRD authoring directly instructs marking the predecessor superseded despite prohibiting status advancement elsewhere. Design authoring pairs a draft successor ADR with predecessor supersession without explicitly waiting for successor approval. The promotion checklist requires a successor reference but does not repeat the authority's approved-successor condition.

**Impact:** an unapproved replacement proposal can appear to retire the currently approved source of truth. Missing reciprocal links or self-references can make replacement history ambiguous.

**Recommendations:** author and approve the successor first, then validate and promote the predecessor. Require reciprocal `supersedes` and `superseded_by` references, a valid distinct successor, and appropriate artifact identity/type checks. Remove authoring-time status-edit instructions. Define historical consumers separately from consumers required to migrate to the successor.

**Acceptance check:** supersession by a draft, nonexistent, or self-referencing successor fails without changing the predecessor.

### F06. Archiving a Dependency Makes It Fail Readiness

**Evidence:** [implemented/archive states, lines 100-101](specs/SDD_WORKFLOW.md#L100-L101); [dependency predicate, line 188](specs/SDD_WORKFLOW.md#L188); [promotion dependency check, line 63](.agents/skills/promote-artifact/SKILL.md#L63).

Dependencies must be exactly `implemented`, while successfully delivered features become `archived`.

**Impact:** feature B can satisfy readiness while dependency A is implemented, then fail the same gate after A is successfully released and archived. Normal delivery reduces apparent dependency satisfaction.

**Recommendations:** define dependency satisfaction independently of active-document location. At minimum, decide whether verified implemented and archived features both qualify, what evidence must remain accessible, and how withdrawn or superseded capabilities differ. This is an authority issue, not something an individual skill should silently reinterpret.

**Acceptance check:** releasing and archiving a still-available dependency does not invalidate a consumer's readiness; a withdrawn capability does not pass merely because it was once delivered.

### F07. Implementer Instructions Conflict With Permissions and Verifier Ownership

**Evidence:** [implementer permissions, lines 5-17](.opencode/agents/sdd-implementer.md#L5-L17); [evidence obligation and prohibition, lines 32-46](.opencode/agents/sdd-implementer.md#L32-L46); [implementation prerequisites and handoff](.agents/skills/implement-feature/SKILL.md#L24-L57); [orchestrator ownership, lines 47-58](.opencode/agents/sdd-orchestrator.md#L47-L58).

The implementer must record evidence in `tasks.md` but must not edit anything under `specs/**`. Its skill also validates through `promote-artifact` and invokes `verify-feature` itself, although promotion is orchestrator-owned and verification belongs to a separate agent. Loading another skill neither launches that agent nor changes the caller's permissions.

**Impact:** a correct implementation cannot complete its required evidence-writing step using permitted native edits. Literal skill execution overlaps or bypasses the designated verifier stage, or ends blocked on the same edit restriction.

**Recommendations:** make the orchestrator establish readiness before delegation. Have the implementer return task-linked RED/GREEN evidence and code changes; have the verifier execute independent gates and persist evidence; reserve status transitions for the orchestrator. Preserve a documented single-agent handoff without making multi-agent wrappers execute incompatible instructions.

**Acceptance check:** implementation can finish and verification can consume its RED evidence without the implementer editing specifications or loading a promotion operation.

### F08. Shell Access Defeats File-Edit Boundaries and Exposes Policy Files

**Evidence:** [implementer permissions, lines 5-17](.opencode/agents/sdd-implementer.md#L5-L17); [verifier permissions, lines 5-18](.opencode/agents/sdd-verifier.md#L5-L18); [releaser shell permissions, lines 5-19](.opencode/agents/sdd-releaser.md#L5-L19).

Implementer and verifier shell access is broadly allowed, except for `git push*` and `git commit*`. Native edit-path restrictions do not establish filesystem isolation for arbitrary shell commands. The implementer's ordinary edit allowlist also includes `AGENTS.md`, `.agents/skills/`, and `.opencode/agents/`. The releaser's `mv *` rule is not restricted to specification source/destination paths.

**Impact:** permitted shell capabilities can change specifications, source, tests, or gate scripts despite native edit restrictions. The implementer can alter the policy/configuration governing future runs. Prefix-based Git denials do not express a semantic prohibition on committing through every valid command form. Denying web tools does not eliminate shell-based network access.

**Recommendations:** protect policy/configuration files explicitly. Treat build/test shell access as high-trust, separate from native edits. Use controlled gate commands and, where strong isolation is required, a sandbox that protects source/tests/policy while allowing necessary build outputs. Constrain publication, credentials, dependency changes, and network access separately. Scope archive moves through validated operations. Do not claim arbitrary project scripts are read-only merely because their command names are allowlisted.

**Acceptance check:** negative permission tests show attempted policy edits, specification writes through shell, verifier test modifications, unauthorized publication, and out-of-scope archive moves are blocked under the supported runtime. No bypass was attempted for this report.

### F09. Promotion and Stage-Output Ownership Are Largely Advisory

**Evidence:** [orchestrator capabilities and claimed boundary](.opencode/agents/sdd-orchestrator.md#L5-L17); [orchestrator promotion ownership, lines 55-65](.opencode/agents/sdd-orchestrator.md#L55-L65); [product permissions, lines 5-14](.opencode/agents/sdd-product.md#L5-L14); [contract permissions, lines 5-14](.opencode/agents/sdd-contract.md#L5-L14); [verifier evidence-only rule, lines 39-42](.opencode/agents/sdd-verifier.md#L39-L42).

Most authoring agents can edit all of `specs/**` and load any skill. The orchestrator can edit all specifications although its prose permits only promotion metadata. The verifier can edit whole `tasks.md` files, not only evidence sections. An explicit human answer is not mechanically bound to a particular subsequent promotion diff.

**Impact:** the capability configuration permits changes outside the agent's declared stage, including lifecycle fields or normative content. Correct behavior depends on prompt compliance. Unlisted custom/MCP tools and delegated helpers also require effective-runtime review before read-only or least-privilege claims can be made.

**Recommendations:** allowlist mapped skills, narrow writable artifact paths, and deny unknown tools where supported. Validate stage-specific diffs: authoring cannot advance status, evidence updates cannot change task semantics, and promotion cannot change artifact meaning. Bind approval to a specific artifact baseline and transition. Use a project-owned explicitly read-only research agent if that guarantee is required rather than relying only on the name `explore`.

**Acceptance check:** an authoring agent cannot promote an artifact, an orchestrator promotion cannot rewrite behavior, and a verifier cannot change task scope through its permitted evidence operation. Skill allowlisting alone is not a sufficient control.

### F10. Release Relocation Lacks Safe Ordering, Preflight, and Recovery

**Evidence:** [release record and archive procedure, lines 24-41](.agents/skills/record-release/SKILL.md#L24-L41); [archive prerequisite, line 32](.agents/skills/promote-artifact/SKILL.md#L32); [release validation, lines 65-74](.agents/skills/promote-artifact/SKILL.md#L65-L74); [releaser evidence checks, lines 33-45](.opencode/agents/sdd-releaser.md#L33-L45).

The skill moves packets while compiling a draft release, then hands off archive and release transitions. Archive promotion requires a closed release milestone, but the handoff lists packet archival before release completion. Although the releaser wrapper checks evidence, the skill does not define complete preflight, collision handling, move authorization, reference repair, or resumable partial-failure behavior.

**Impact:** evidence rejection, a denied promotion, or an existing destination can leave packets moved or inconsistently located while the release is still draft. Moving a packet one level deeper also changes the meaning of relative links, and inbound links to its old location can break.

**Recommendations:** define a single authoritative sequence for release drafting, membership/evidence validation, human authorization, release finalization, relocation, archive promotion, and final consistency checks. Specify source/destination validation, no-overwrite behavior, link repair, an operation manifest, and safe retries. Choose the exact ordering through the workflow authority; do not let individual agents infer it.

**Acceptance check:** failed preflight leaves packet locations unchanged; interruption after one move is recoverable without overwriting, duplicating, or losing a packet; all incoming and outgoing references resolve afterward.

### F11. Observation Routing Bypasses PRD and Epic Workflow Stages

**Evidence:** [triage routing, lines 32-39](.agents/skills/triage-observation/SKILL.md#L32-L39); [story prerequisites, lines 32-43](.agents/skills/refine-user-stories/SKILL.md#L32-L43); [mapped skill ownership, lines 198-213](specs/SDD_WORKFLOW.md#L198-L213); [releaser role, lines 27-32](.opencode/agents/sdd-releaser.md#L27-L32).

Triage instructs the caller to update a PRD with a new epic and decision, mark the observation promoted, then invoke story refinement. It neither routes the PRD change through `create-prd` nor creates and approves the epic brief required by story refinement. The releaser owns triage but cannot delegate or interview the user.

**Impact:** literal execution either bypasses mapped ownership or stops at a missing approved epic brief. Even a regression fix within existing approved scope is steered toward an unnecessary new PRD epic.

**Recommendations:** separate classification from scope decisions and authoring. Return a routing proposal to the orchestrator. Reuse existing approved scope where appropriate; use PRD revision only for product-intent changes; refine and approve a new epic before its story. Populate authoritative `promoted_to` only when a real destination exists. Observation intake states are explicitly separate from specification statuses; using an operational `promoted` state is not itself a violation.

**Acceptance check:** an existing-scope bug routes without inventing an epic, while a genuine new capability follows the complete PRD/epic/story approval sequence.

### F12. Verification Lacks a Complete Behavior-to-Source Evidence Chain

**Evidence:** [implementation loop, lines 34-43](.agents/skills/implement-feature/SKILL.md#L34-L43); [verification evidence, lines 54-60](.agents/skills/verify-feature/SKILL.md#L54-L60); [task template, lines 28-65](specs/templates/feature/tasks.md#L28-L65); [release evidence template, lines 31-39](specs/templates/supporting/release.md#L31-L39).

Observed RED/GREEN and gate output are mandatory, but the process does not fully define how historical RED output survives the implementer/verifier handoff, how each specified behavior maps to an executed test, or how evidence identifies a dirty working tree. A date and commit hash do not necessarily identify the source actually tested. The templates lack a structured execution ledger and include affirmative example release-evidence statements.

**Impact:** a passing suite can omit a specified failure path. A recorded `HEAD` can differ from the tested working tree. Later source/specification changes can leave old evidence apparently usable. The verifier cannot reconstruct the original failing test state merely from final green code.

**Recommendations:** record the approved specification baseline, tested source identity including dirty-tree handling, behavior-to-test mapping, commands and working directories, exit results, metrics, retained output references, and RED/GREEN pairing. Distinguish not-run, failed, skipped, and passed checks. Invalidate evidence after relevant changes. Default template evidence to not run, not observed passing. Promotion should check applicability and completeness, not just the presence of an evidence paragraph.

**Acceptance check:** missing behavior coverage, absent RED evidence, dirty-source ambiguity, and changes after verification each prevent an unsupported implemented/released claim.

## Medium-Priority Findings

### F13. Concrete Gate Instructions Drift From the Constitutions

**Evidence:** [verification gates, lines 25-52](.agents/skills/verify-feature/SKILL.md#L25-L52); [.NET tooling and CI, lines 446-481](specs/CONSTITUTION.md#L446-L481); [frontend security and CI, lines 269-295](specs/CONSTITUTION_FRONTEND.md#L269-L295); [release template, lines 33-39](specs/templates/supporting/release.md#L33-L39).

The skill says to run every constitutional gate, but its concrete suite underrepresents required CI/prerequisite checks and treats canonical wrappers as optional if present. Locked restore is conditional on lockfiles existing rather than reporting missing required lockfiles. The release template repeats a fixed .NET/frontend list, changes vulnerability wording, and narrows domain evidence to ADR-defined criteria although PRD constraints can also define it.

**Impact:** following the printed commands can omit required evidence, fail on an unprepared checkout, or ask a frontend-only release to report nonexistent .NET execution. Missing required tooling may be mistaken for non-applicability.

**Recommendations:** derive a scoped gate manifest from the applicable constitution and approved PRD/ADR constraints. Record execution venue, prerequisites, applicability, thresholds, and evidence. Distinguish local, CI, and scheduled checks rather than forcing every check into one local invocation. Treat missing mandatory wrappers, lockfiles, or CI evidence as blockers. Avoid a second hand-maintained gate list in the release template.

**Acceptance check:** .NET-only, frontend-only, mixed, and missing-tooling cases each produce the correct gate obligations without silent omissions or irrelevant passing claims.

### F14. Artifact Relationships Lack a Complete Typed Reference Model

**Evidence:** [story metadata, lines 9-15](specs/templates/feature/user-story.md#L9-L15); [readiness references, lines 180-190](specs/SDD_WORKFLOW.md#L180-L190); [artifact responsibility table, lines 124-136](specs/SDD_WORKFLOW.md#L124-L136); [selected promotion checks, lines 45-63](.agents/skills/promote-artifact/SKILL.md#L45-L63).

Fields such as `parent`, `epic`, `feature`, `depends_on`, `requires`, and `related` are widespread but lack a complete normative definition of target types, qualification, cardinality, and required target states. CHART has a declared lifecycle identity but no responsibility-table completion gate; promotion groups it with schemas. Prose questions and `blockers: []` can also disagree.

**Impact:** authors can encode the same feature dependency as a story ID or packet slug. Different agents can validate different subsets of the graph. Cycles, ambiguous references, or open prose blockers can survive superficial checks.

**Recommendations:** define typed fields, canonical reference syntax, namespace context, cycle policy, archive/supersession resolution, and the distinction between gating and informational relationships. Specify whether checks aggregate across the entire packet and its upstream graph. Reconcile question tables with blockers. Add a CHART completion rule or explicitly define charts as non-gating references.

**Acceptance check:** the same resolver handles active, archived, and historical artifacts and rejects ambiguous, dangling, mistyped, or cyclic gating relationships according to documented policy.

### F15. Identifier Rules Conflict and Cannot Guarantee Allocation Invariants

**Evidence:** [identifier rules, lines 140-165](specs/SDD_WORKFLOW.md#L140-L165); [epic namespace decision, lines 36-45](specs/adr/ADR-003.md#L36-L45); [story allocation, lines 59-64](.agents/skills/refine-user-stories/SKILL.md#L59-L64); [bare epic destination example, lines 25-29](specs/templates/supporting/release.md#L25-L29).

Epic IDs are described as per-PRD scoped but also as never reused across PRDs, contrary to ADR-003's rejection of a global epic sequence. Filesystem-only allocation cannot guarantee non-reuse after deletion. Exactly three digits provides no successor after 999. The story skill's directory-first allocation example can obscure the explicit independence of packet and story numbering.

**Impact:** the first epic in a second PRD has contradictory numbering guidance; bare epic references outside parent context are ambiguous. Deleted IDs can be reallocated, independent branches can collide, and packet numbering can incorrectly drive story IDs.

**Recommendations:** make canonical epic identity `(PRD-ID, EPIC-ID)` and require qualification outside unambiguous parent context. Allocate packet and artifact sequences independently. Define durable reservation through a registry, tombstone, or explicit history procedure, plus merge-collision and exhaustion policies. Keep the solution proportional to expected scale, but state its limits.

**Acceptance check:** two PRDs can have unambiguous local epic identities, deleted IDs remain reserved, imported high-numbered stories do not corrupt allocation, and exhaustion fails explicitly.

### F16. Isolated Contexts Lack Durable Decision and Interview Handoffs

**Evidence:** [disk handoff decision, lines 80-82](specs/adr/ADR-005.md#L80-L82); [product question fallback and report](.opencode/agents/sdd-product.md#L26-L47); [contract decision report, lines 49-53](.opencode/agents/sdd-contract.md#L49-L53); [current-session engineering approvals, lines 43-44](.opencode/agents/sdd-contract.md#L43-L44).

The packet is the handoff, but some necessary state exists before any file can be written: synthesized content awaiting confirmation, an interrupted interview, or a human decision from the current session. Report formats do not define durable provenance or when to resume the same child context rather than start fresh.

**Impact:** the next agent may lose the content the user approved, repeat interviews, or confuse historical ADR approval with current-session authorization to add a dependency. RED evidence has the same cross-context persistence problem.

**Recommendations:** define a small structured handoff containing the exact operation, artifact paths/baselines, outcome, pending content, exact blocking question, scoped human decisions, evidence locations, and resume identifier when appropriate. Resume a blocked interview deliberately; retain fresh contexts between disciplines. Do not forward entire conversation history when a bounded handoff suffices.

**Acceptance check:** an interrupted interview resumes without lost decisions, and an implementer can distinguish present authorization from historical design rationale.

### F17. Recovery Routes Ambiguity to the Wrong Stage and Conflates Failures

**Evidence:** [orchestrator recovery, lines 66-72](.opencode/agents/sdd-orchestrator.md#L66-L72); [contract-owned outputs, lines 22-27](.opencode/agents/sdd-contract.md#L22-L27); [workflow mismatch branch, lines 73-84](specs/SDD_WORKFLOW.md#L73-L84); [implementation discrepancy rule, lines 48-49](.agents/skills/implement-feature/SKILL.md#L48-L49).

Implementation ambiguity is always routed to cluster 3, although the owning artifact may be a PRD, epic, story, or scenario. The workflow diagram sends every behavior/tests/evidence mismatch toward specification revision, even when the specification is correct and the code or evidence is wrong. Restart, rejected-gate, denied-permission, and infrastructure-failure paths are not fully defined.

**Impact:** an ambiguous story may be silently reinterpreted in requirements, or an implementation bug may trigger unnecessary specification changes. Partial runs can be repeated without reconciling existing files and user changes.

**Recommendations:** classify the failure first. Fix code defects against unchanged approved behavior; repair missing tests/evidence through verification; route genuine specification changes to the earliest owning artifact and invalidate affected descendants. Define blocked/failed/partial/cancelled outcomes, restart reconciliation, retry limits, and escalation. Permission denial must not trigger a workaround that bypasses the boundary.

**Acceptance check:** code bugs, story ambiguity, missing tools, rejected approval, and interrupted execution each follow distinct correct recovery paths while preserving unrelated worktree changes.

### F18. Universal Draft-Status Validation Rejects Valid Later-Stage Outputs

**Evidence:** [orchestrator report checks, lines 74-79](.opencode/agents/sdd-orchestrator.md#L74-L79); [reviewer approved inputs, lines 23-40](.opencode/agents/sdd-reviewer.md#L23-L40); [verifier status prohibition, lines 39-42](.opencode/agents/sdd-verifier.md#L39-L42).

The orchestrator requires statuses to be draft whenever a subagent reports. Correct reviewers inspect approved packets; correct verifiers preserve approved statuses; correct releasers relocate implemented packets unchanged.

**Impact:** a valid later-stage result fails the generic report check, or the instruction tempts an unnecessary and forbidden reset to draft. The universal observed-evidence wording also needs stage-specific interpretation: a product interview does not produce build output.

**Recommendations:** validate expected before/after changes by stage: draft authoring, no-change review, code-only implementation, evidence-only verification, draft release plus preserved moved statuses, and explicitly authorized promotion. Define appropriate evidence for each stage rather than treating all outputs as authoring or engineering execution.

**Acceptance check:** valid reviewer, verifier, and releaser outputs pass without resetting lifecycle fields.

### F19. Topology Adoption, Startup, and Runtime Assumptions Are Unverified

**Evidence:** [ADR-005 draft status, lines 1-11](specs/adr/ADR-005.md#L1-L11); [topology adoption summary, lines 44-59](AGENTS.md#L44-L59); [startup instructions, lines 9-15](README.md#L9-L15); [orchestrator mode and tools, lines 1-17](.opencode/agents/sdd-orchestrator.md#L1-L17); [ADR follow-up verification, lines 113-119](specs/adr/ADR-005.md#L113-L119).

ADR-005 remains draft while operational guidance presents the topology as adopted and authoritative. `mode: primary` defines an available primary agent, not by itself the selected runtime default. The tracked repository does not package a default-agent configuration, supported runtime/client matrix, or topology acceptance results. Question-tool availability and the permissions of delegated built-in research are not proven by their names or local allow declarations.

**Impact:** users may start the valid single-agent route without activating the advertised topology, or encounter missing interaction tools without a parent fallback. Claims such as "provably read-only" exceed the available effective-runtime evidence.

**Recommendations:** label the topology proposed/experimental until its conflicts and tests are resolved, then approve it through the mapped lifecycle. Document explicit orchestrator selection or intentionally configure a default. State supported runtime/client assumptions, skill discovery, question fallback, and extension policy. Fail closed when explicit approval cannot be obtained. Do not equate an ignored local plugin-package dependency version with a pinned OpenCode executable or a policy-enforcement plugin.

**Acceptance check:** a clean supported checkout launches the intended agent, discovers the skills, relays questions, and demonstrates expected effective permissions. Single-agent use remains explicitly supported.

### F20. Workflow Invariants Have No Executable Regression Protection

**Evidence:** the tracked repository inventory contains agent/skill Markdown and templates but no first-party workflow validator, lifecycle test suite, or CI conformance configuration; [workflow documentation checks, lines 240-249](specs/SDD_WORKFLOW.md#L240-L249); [ADR topology verification follow-up, lines 118-119](specs/adr/ADR-005.md#L118-L119).

No local executable implementation protects transitions, ID allocation, reference resolution, evidence freshness, or release preflight. This is not a missing application-test complaint: the repository intentionally contains no application. It is a gap in tests for the workflow product itself.

**Impact:** duplicated checklists and permission promises can drift without detection, as the current contradictions demonstrate. A future edit can silently reopen a resolved failure path.

**Recommendations:** first make the authority internally consistent, then add a small deterministic validation/test layer for machine-checkable rules. Use fixtures for accepted and rejected transitions, references, archival, and metadata. Add runtime smoke/negative tests for orchestration separately. Keep semantic product approval human-owned; do not build a large orchestration framework merely to replace a few reliable checks.

**Acceptance check:** CI or an equivalent repeatable command detects invalid states, missing references, approval-order regressions, and unsafe archive plans. Test outputs are recorded, not just test plans.

### F21. Skill Prerequisites and Template Consumption Are Inconsistent

**Evidence:** [requirements prerequisites, lines 30-38](.agents/skills/create-requirements/SKILL.md#L30-L38); [implementation reading list, lines 26-30](.agents/skills/implement-feature/SKILL.md#L26-L30); [complete packet requirement, lines 217-228](specs/SDD_WORKFLOW.md#L217-L228); [mandatory template rules, lines 168-174](specs/SDD_WORKFLOW.md#L168-L174); [Gherkin output procedure, lines 40-59](.agents/skills/user-story-to-gherkin/SKILL.md#L40-L59).

Requirements authoring requires an approved story but only scenario presence, not scenario approval. The implementation skill explicitly reads design/tasks/ADRs rather than the complete packet, although the implementer wrapper corrects that omission. Gherkin, release, and triage do not explicitly consume their canonical templates; some other skills make template use conditional on availability.

**Impact:** single-agent and multi-agent runs can have different safeguards. A contract can be based on draft behavior, and authoring can diverge from template metadata despite the mandatory-template rule.

**Recommendations:** make each mapped skill independently conformant: require approved scenarios before requirements, explicitly read the complete packet before implementation, and name the exact template for every new artifact. Treat missing mandatory templates as a reported prerequisite failure. Keep wrappers thin rather than relying on them to repair skill omissions.

**Acceptance check:** running each mapped skill directly enforces the same prerequisites and artifact format as running it through its stage agent.

### F22. Observation Intake and Closure Do Not Support Reliable Release Accounting

**Evidence:** [triage actions, lines 23-39](.agents/skills/triage-observation/SKILL.md#L23-L39); [release observation lists, lines 26-29](.agents/skills/record-release/SKILL.md#L26-L29); [observation identity and lifecycle exception](specs/SDD_WORKFLOW.md#L153-L162); [observation operational-state rule, lines 192-194](specs/SDD_WORKFLOW.md#L192-L194).

Triage covers classification, code-line confirmation, impact, and promotion, but not a complete create/update/disposition/closure process. It does not operationalize ID preservation when moving from the catalogue to a dedicated record, duplicate handling, rejected or deferred reports, closure evidence, or regression-test linkage. Not every initial operational report can be confirmed against exact source lines.

**Impact:** a promoted observation has no defined path to proven resolution, yet release recording expects a reliable list of closed observations. Duplicate intake and premature closure can corrupt delivery accounting.

**Recommendations:** define intake and disposition procedures using the operational states in the observation template. Permit unconfirmed reports with reproduction/operational evidence, maintain stable identity, and tie closure to a verified fix and release where applicable. Distinguish selected, promoted, implemented, and actually resolved outcomes.

**Acceptance check:** an unconfirmed report can be recorded safely; duplicate, deferred, rejected, promoted, and verified-closed cases have explicit evidence and identity rules.

### F23. Mandatory Frontend Styling Inputs Lack Ownership and Readiness Rules

**Evidence:** [visual-design guidance, lines 3-20](design/README.md#L3-L20); [frontend token authority, lines 208-210](specs/CONSTITUTION_FRONTEND.md#L208-L210); [frontend completion requirement, line 309](specs/CONSTITUTION_FRONTEND.md#L309); [readiness predicate, lines 180-190](specs/SDD_WORKFLOW.md#L180-L190).

Visual materials are described as optional, while the frontend constitution makes `design/epic-*/design-tokens.md` the styling source of truth. Readiness does not require this input or define who creates/approves it. Per-PRD epic IDs can also collide in the suggested unqualified `design/epic-NNN/` layout.

**Impact:** a frontend packet can pass written readiness and then be blocked by a mandatory but absent design input. Separate PRDs can ambiguously share an epic-token path.

**Recommendations:** distinguish optional mockups from mandatory styling inputs. Define token ownership, project-wide versus epic-specific scope, PRD-qualified paths where needed, and the stage that establishes tokens. Include applicable token availability in frontend readiness and define how token changes affect approved design/evidence.

**Acceptance check:** a frontend slice cannot be declared ready without its required styling source, and two PRDs cannot accidentally resolve to the same epic-specific token artifact.

### F24. Template Placeholder Policies Conflict With Approval Rules

**Evidence:** [placeholder prohibition, line 171](specs/SDD_WORKFLOW.md#L171); [PRD checklist, lines 84-92](specs/templates/project/prd.md#L84-L92); [epic checklist, lines 61-67](specs/templates/supporting/epic.md#L61-L67); [promotion completeness rules, lines 38-49](.agents/skills/promote-artifact/SKILL.md#L38-L49).

PRD and epic checklists allow success measures to remain explicitly TBD, while workflow/promotion rules prohibit unresolved normative placeholders before approval.

**Impact:** every template checkbox can be satisfied while the higher-priority approval gate fails. If an agent silently accepts the TBD, downstream delivery may lack a measurable success criterion.

**Recommendations:** remove approval-time TBD permission or define a precise nonblocking-unknown policy with owner, rationale, deadline, and the gate at which resolution becomes mandatory. Align templates, skills, and the normative rule. Do not replace unknown product targets with invented values merely to pass a validator.

**Acceptance check:** an unresolved normative success measure produces the same decision in template review, promotion, and readiness validation.

### F25. PRD Hierarchy and Supersession Semantics Live Outside the Authority

**Evidence:** [PRD scope/parent metadata, lines 5-13](specs/templates/project/prd.md#L5-L13); [lifecycle guidance, lines 9-16](specs/prd_lifecycle_and_evolution_plan.md#L9-L16); [hierarchy and replacement guidance, lines 46-70](specs/prd_lifecycle_and_evolution_plan.md#L46-L70); [immediate-parent readiness, lines 180-181](specs/SDD_WORKFLOW.md#L180-L181).

Templates and subordinate guidance introduce project versus major-feature PRDs, child relationships, and replacement behavior without a complete normative hierarchy contract. Readiness checks the immediate PRD and epic, not the disposition of PRD ancestors. The consequences of superseding a project PRD for child PRDs, epics, and active packets are unspecified.

**Impact:** an approved child can remain apparently ready under a draft or superseded ancestor without an explicit policy deciding whether that is valid. Agents may infer different ancestry and replacement requirements from derivative documents.

**Recommendations:** move hierarchy invariants into the workflow authority: allowed parents, nesting, ancestor-state requirements, active-root expectations, and treatment of dependents on replacement. Keep the lifecycle guide as examples. Establish reciprocal supersession metadata before approval or through a defined metadata-finalization operation.

**Acceptance check:** child-PRD readiness and project-PRD supersession have deterministic effects on the relevant epic/feature graph.

### F26. Behavior-Level Identities and Test Traceability Are Underspecified

**Evidence:** [requirements traceability table, lines 41-45](specs/templates/feature/requirements.md#L41-L45); [scenario template, lines 1-21](specs/templates/feature/scenarios.feature#L1-L21); [task test planning, lines 28-46](specs/templates/feature/tasks.md#L28-L46); [generic implementation trace comment, line 43](.agents/skills/implement-feature/SKILL.md#L43); [qualified frontend example, lines 53-57](specs/CONSTITUTION_FRONTEND.md#L53-L57).

Artifact IDs are specified, but local requirement/example/question identities and scenario identities are not standardized. Requirements link to scenario titles, while implementation suggests a document-level `REQ-NNN` trace comment. Neither alone proves which behavior within a document is covered.

**Impact:** two `FR-001` references can be ambiguous, scenario renaming can break traceability, and a single generic requirements reference can conceal missing alternate or boundary tests.

**Recommendations:** define qualified local references, such as `REQ-012/FR-003`, and stable scenario IDs independent of display titles. Add a compact behavior-to-test/evidence matrix. Specify scenario-outline and renamed/deleted-behavior handling. Avoid duplicating the same matrix in every artifact; choose an authoritative location and link to it.

**Acceptance check:** a scenario rename preserves identity, duplicate local IDs in different packets are unambiguous, and every specified behavior resolves to an executed test or an explicit unresolved coverage gap.

## Agent-by-Agent Assessment

These summaries evaluate implementation fitness, not the competence of a model assigned to the role. Finding IDs identify the detailed evidence and recommendations above.

| Agent | What Works | Main Gaps | Recommended Direction |
| --- | --- | --- | --- |
| `sdd-orchestrator` | Clear delegation ownership, human authority, and no direct shell access. | Cluster ordering, broad specification edits, universal draft check, wrong-stage recovery, incomplete handoffs (F01, F09, F16-F19). | Make it a precise coordinator of individual operations and explicit transitions, with stage-specific diff checks and resumable handoffs. |
| `sdd-product` | Focused product/epic interviews, draft-only intent, no shell or ordinary application edits. | PRD approval is needed inside its cluster; broad specification and skill access; interrupted interview context is underspecified (F01, F09, F16). | Return after each prerequisite boundary; narrow outputs and preserve pending interview state. |
| `sdd-story` | Valuable-slice focus and separation of story from executable behavior. | Gherkin depends on intermediate story approval; inherited broad permissions and generic reporting (F01, F09, F18, F21). | Split story and scenario operations around the promotion checkpoint while keeping one discipline-level agent. |
| `sdd-contract` | Clear contract/design/task responsibilities, constitution awareness, and current-session approval rule. | Internal approval dependencies, broad specification permissions, and decision provenance gaps (F01, F09, F16). | Delegate one approved-input operation at a time; retain explicit scoped human technical decisions. |
| `sdd-implementer` | Complete-packet reading in the wrapper, explicit RED/GREEN, stops on ambiguity and unapproved dependencies. | Evidence edit contradiction, cross-role skill handoff, broad shell/policy access (F07-F08, F12). | Return preserved implementation evidence; leave specification evidence persistence and verification to the verifier. |
| `sdd-verifier` | Independent gate role, concrete output requirement, no native source/test edits. | Shell defeats the intended boundary; whole-task-file edits; incomplete evidence freshness and gate model (F08-F09, F12-F13). | Use controlled execution and evidence-only changes; verify behavior coverage and tested source identity. |
| `sdd-releaser` | Requires observed evidence and distinguishes relocation from status advancement. | Broad move capability, incomplete release transaction, triage crossing into other stages (F08, F10-F11, F22). | Use a preflighted release/move plan and return observation-routing proposals to the orchestrator. |
| `sdd-reviewer` | Most restrictive native posture and most complete explicit readiness checklist. | Scheduled before prerequisites can pass; promotion-record format absent; effective extension permissions unverified (F01, F03, F09, F19). | Preserve independent read-only review, run it after approvals, and consume defined provenance records. |

The topology does not need more agents to resolve these findings. Most improvements concern operation boundaries, explicit state semantics, and handoff contracts within the existing roles.

## Skill-by-Skill Assessment

| Skill | Assessment | Recommendations |
| --- | --- | --- |
| `create-prd` | Strong interview/content-confirmation discipline; revision and supersession instructions conflict with sole promotion ownership. | Apply typed reopening and approved-successor rules; define root/child PRD prerequisites (F03-F05, F25). |
| `refine-epic` | Useful just-in-time outcome refinement, candidate slices, and parent linkage. | Keep the approved-PRD gate; fix cluster scheduling, revision handling, namespaces, and TBD policy (F01, F04, F15, F24). |
| `refine-user-stories` | Explicit approved-epic prerequisite and independently valuable slice discipline. | Keep that prerequisite; separate story-ID allocation from directory allocation and repair triage entry (F11, F15). |
| `user-story-to-gherkin` | Good separation of behavior formulation from implementation. | Require its template explicitly, preserve the approved-story checkpoint, and add stable scenario identities (F01, F21, F26). |
| `create-requirements` | Strong observable-contract focus and bidirectional scenario coverage intent. | Require approved scenarios, consistent template use, and revision invalidation (F04, F21). |
| `create-design` | Substantial treatment of interfaces, alternatives, failures, ADRs, and schema links. | Make successor approval precede supersession and establish required frontend design inputs (F05, F23). |
| `create-tasks` | Good dependency ordering, test-first intent, and prohibition on changing behavior. | Preserve approved-design prerequisites; add precise behavior coverage and separate plan/evidence sections (F01, F12, F26). |
| `promote-artifact` | Correct conceptual owner for lifecycle control, but highest concentration of correctness gaps. | Repair predicate/state separation, typed transitions, evidence validation, and supersession before adding automation (F02-F06, F12, F14). |
| `implement-feature` | Explicit expected RED failure before minimal GREEN implementation. | Read the complete packet directly and make verification/evidence handoff topology-compatible (F07, F12, F21). |
| `verify-feature` | Runs observed gates, recognizes both stacks, and does not itself advance lifecycle state. | Complete the scoped gate manifest, source identity, RED history, and behavior reconciliation (F12-F13, F26). |
| `record-release` | Captures membership, evidence, observations, migration, and rollback intent. | Consume its template and define preflight, sequencing, collision/link handling, and recovery (F10, F13, F21). |
| `triage-observation` | Useful impact-oriented categories and routing intent; procedure is much thinner than authoring skills. | Complete intake/closure, reuse existing scope, and route PRD/epic work through their owners (F11, F21-F22). |
| `qmd` | Small optional bootstrap avoids embedding stale CLI instructions and is not an application dependency. | Keep it optional, document CLI availability/version assumptions, and treat retrieved content as research rather than workflow authority. Full external CLI behavior remains outside this assessment. |

## Strengths Worth Preserving

1. **Explicit normative precedence.** Lifecycle and engineering authorities are separated, and derivative guidance is told to defer rather than silently override them.
2. **Clear artifact responsibilities.** Product intent, business value, behavior, observable contract, technical approach, and ordered work are deliberately distinct.
3. **Independently valuable vertical slices.** Just-in-time epic refinement and flat feature packets avoid specifying an entire product before obtaining implementation feedback.
4. **Human semantic authority.** Content confirmation, lifecycle approval, and passing tests are correctly recognized as different concepts, even though the implementation needs stronger binding between them.
5. **Real test-first intent.** The implementation skill explicitly requires expected RED output before production implementation and observed GREEN afterward.
6. **Separate implementation and verification roles.** This is a useful review boundary once evidence ownership and execution capabilities are made consistent.
7. **Meaningful native restrictions on authoring/review.** Authoring agents deny shell and application edits; the reviewer denies native mutation/delegation tools. These controls should be strengthened, not dismissed because they are incomplete.
8. **Supporting schema and decision modeling.** DB/TABLE backlinks and ADR alternatives/consequences provide useful technical traceability.
9. **Template/application separation.** The repository accurately states that it is a kit, not a scaffolded application or an approved sample product.
10. **Optional research remains optional.** QMD is not made a runtime requirement of generated applications or normal SDD execution.

## Remediation Roadmap

The order below minimizes rework. It is a recommendation, not authorization to change the workflow or introduce tooling.

### Phase 1: Make the Lifecycle Internally Consistent

Resolve F01-F06, F18, and the authority portions of F19/F25 first.

**Deliverables:** an artifact-type transition matrix; explicit readiness-only semantics; intra-cluster approval checkpoints; reopening and supersession rules; archive-aware dependency satisfaction; defined packet status and approval provenance.

**Exit condition:** documented walkthroughs of first-time authoring, rejected review, approved-spec revision, shared-decision replacement, and archived dependency use contain no contradictory instructions or invented exceptions.

### Phase 2: Align Agent and Skill Contracts

Resolve F07, F11, F16-F17, and F21.

**Deliverables:** single-operation delegation contracts; stage-specific expected diffs; resumable interview/decision handoffs; implementer-to-verifier evidence ownership; earliest-owner ambiguity routing; triage-to-orchestrator routing.

**Exit condition:** each skill works conformantly both directly and through its mapped agent, without permission workarounds or reliance on undocumented context.

### Phase 3: Make Verification and Release Auditable

Resolve F10, F12-F15, and F22-F26.

**Deliverables:** typed references and stable identities; behavior coverage mapping; evidence records tied to approved/tested baselines; scoped constitutional gates; frontend input prerequisites; preflighted, resumable release operations.

**Exit condition:** a reviewer can establish what was approved, what was tested, what was delivered, and why each gate passed, including after archival or an interrupted release.

### Phase 4: Harden and Test Enforcement

Resolve F08-F09, F20, and runtime packaging portions of F19.

**Deliverables:** least-privilege configuration, policy-file protection, appropriately constrained gate execution, deterministic metadata/reference checks, documented runtime selection, and positive/negative topology tests.

**Exit condition:** observed tests substantiate the claimed boundaries under a supported runtime configuration. Document any remaining prompt-only controls and trust assumptions explicitly.

## Recommended Acceptance Suite

Tests should assert rejected paths as well as happy paths. Fixture-driven mechanical validation and live agent/runtime tests serve different purposes and should remain distinguishable.

| Case | Expected Result | Findings |
| --- | --- | --- |
| First root PRD through first feature packet | Every dependent skill consumes approved inputs; all required approvals are obtainable. | F01, F03 |
| Readiness validation on an approved packet | Returns a predicate result and changes no statuses or approval metadata. | F02 |
| Approved packet with dangling reference, blocker, or missing boundary case | Fails the relevant mechanical or semantic check. | F02, F14 |
| Rejected review and later revision | Reopens through the authorized path and preserves approval history. | F03-F04 |
| Upstream business-rule or shared-schema change | Identifies affected descendants and invalidates/reviews stale approvals and evidence. | F04 |
| Supersession by draft or invalid successor | Leaves predecessor unchanged and reports the failed prerequisite. | F05 |
| Consumer of an archived delivered capability | Satisfies the defined dependency predicate without pretending archived means unimplemented. | F06 |
| Implementer evidence handoff | Verifier receives durable RED/GREEN evidence without implementer specification edits. | F07, F12, F16 |
| Forbidden native/shell/policy/publication operations | Effective runtime blocks the actions, not merely warns the agent in prose. | F08-F09 |
| Missing or changed source/evidence baseline | No implemented/released claim is made from stale or ambiguous evidence. | F12 |
| .NET-only, frontend-only, and mixed verification | Correct gate obligations and applicability; missing mandatory setup is a blocker. | F13, F23 |
| Two PRDs, imported IDs, deletion, allocation collision, and exhaustion | Identities remain unambiguous and reserved; unsupported allocation fails explicitly. | F15 |
| Interrupted interview or human gate | Resumes the correct state without inferring or losing authorization. | F16, F19 |
| Code defect versus story ambiguity versus infrastructure failure | Routes to the correct owner and preserves unrelated changes. | F17 |
| Reviewer/verifier/releaser reports | Valid later-stage statuses are preserved and accepted. | F18 |
| Archive destination collision or partial relocation | Preflight/recovery prevents overwrite and leaves a reconciliable state. | F10 |
| Incoming and outgoing references after archival | All supported references resolve to the intended artifacts. | F10, F14 |
| Existing-scope regression versus new product capability | Triage reuses scope or invokes the complete required authoring path. | F11, F22 |
| TBD metrics and missing required frontend tokens | Approval/readiness decisions are consistent and do not invent inputs. | F23-F24 |
| Scenario rename and duplicate local requirement IDs | Traceability remains stable and behavior coverage remains explicit. | F26 |

## Decisions Requiring Human Resolution

These are genuine policy/design decisions, not details an implementing agent should silently choose:

1. Whether intermediate approvals remain mandatory inside organizational clusters or provisional downstream drafting is deliberately allowed.
2. The legal reopening transitions and which changes invalidate downstream approvals or verification evidence.
3. What represents a packet's lifecycle state and the approved/tested baseline.
4. Whether archived dependencies satisfy readiness and how withdrawn/superseded capabilities are represented.
5. The exact release-finalization, relocation, and archive-promotion ordering and recovery policy.
6. Whether agent boundaries are intended as cooperative prompt guidance or stronger security/isolation guarantees.
7. The canonical identity/reference scheme, including PRD-scoped epic references and deleted-ID reservation.
8. Which tools/runtime configurations are supported, and which added validators or sandbox mechanisms are worth maintaining.

## Final Judgment

The kit has a useful foundation and does not need a wholesale redesign. Its disciplined authoring model and existing agent roles are worth retaining. The immediate work is to make lifecycle semantics, approval ordering, stage handoffs, evidence, and release operations agree with one another.

Until the high-priority conflicts are resolved and exercised, the strongest defensible claim is **a human-supervised SDD workflow with partially constrained agent roles**, not an automatically enforced, end-to-end delivery process. Confidence should increase through observed lifecycle and permission tests rather than additional assurances in agent prompts.
