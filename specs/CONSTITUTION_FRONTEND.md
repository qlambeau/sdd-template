# Frontend Engineering Constitution

**Status:** Normative. **Audience:** human and agentic contributors. **Applies to:** all React / TypeScript / Node code in this repository — the `Web.Spa` build and its tests, tooling, and configuration.

This document is the normative engineering authority for the frontend. Specifications govern *what* is built; this constitution governs *how* the `Web.Spa` is written. For C#/.NET code, the separate `.NET Engineering Constitution` (`specs/CONSTITUTION.md`) governs. When this document conflicts with the workflow (`specs/SDD_WORKFLOW.md`) on workflow matters, the workflow wins; on frontend engineering form, this document wins and the conflict MUST be raised, not silently resolved.

This constitution applies to `Web.Spa` — React/TypeScript, Vite, Tailwind,
npm, and their tests. It does not apply to backend projects under `src/`.

---

## 0. How to use this document

### 0.1 Rule grammar

| Keyword | Meaning |
| --- | --- |
| **MUST** / **MUST NOT** | Non-negotiable. A violation blocks merge. |
| **SHOULD** / **SHOULD NOT** | Strong default. Deviation requires a code comment naming the rule ID and the reason. |
| **MAY** | Permitted, no justification needed. |

Every rule has a stable ID (`R-FE-<AREA>-<NN>`). Cite rule IDs in commit messages, review comments, and deviation comments. IDs are append-only: a retired rule is marked `RETIRED`, never renumbered or reused.

### 0.2 Recording a deviation

A `SHOULD` may be broken inline. A `MUST` may not — it requires an ADR.

```ts
// DEVIATION(R-FE-EFF-03): this effect intentionally runs once per mount to
// bootstrap a third-party widget that has no cleanup contract. Contained here.
```

### 0.3 Rules for the agent

- **R-FE-AGT-01** — An agent MUST read this document before its first `Web.Spa` edit in a session, and MUST NOT rely on a summary of it.
- **R-FE-AGT-02** — An agent MUST NOT introduce a new npm dependency, new build step, or new architectural layer (state store, routing, codegen) without explicit human approval in the current session.
- **R-FE-AGT-03** — An agent MUST NOT weaken a test, delete an assertion, add `.skip`/`.only`/`it.todo`, or loosen an ESLint/TS rule to make a build pass. If the code cannot satisfy the test, stop and report.
- **R-FE-AGT-04** — An agent MUST NOT use `any`, `@ts-ignore`, or `@ts-expect-error` in production code. TypeScript strictness is a hard contract.
- **R-FE-AGT-05** — An agent MUST leave the working tree passing §13 gates at the end of every unit of work. "I'll fix the tests next" is not a valid end state.
- **R-FE-AGT-06** — When a specification is ambiguous, an agent MUST ask rather than choose. Guessing at behaviour is the single most expensive failure mode in spec-driven development.
- **R-FE-AGT-07** — An agent MUST NOT mark work complete based on intent. Completion requires the §14 checklist to have actually been executed, with command output observed.

---

## 1. Spec-driven development loop

The SPA exists to satisfy a specification. The loop is fixed:

```
SPEC  →  CONTRACT  →  RED TEST  →  IMPLEMENTATION  →  GREEN  →  REFACTOR  →  TRACE
```

- **R-FE-SDD-01** — No production code is written before a specification exists for the behaviour it implements. The specification MAY be a single paragraph, but it MUST be written down and addressable by an ID.
- **R-FE-SDD-02** — Every specified behaviour MUST be traceable to at least one test. Encode the link in the test name or a doc comment:
  ```ts
  // Covers: REQ-001 FR-010 — after a successful create the list shows the new PRD.
  it("returns to the list containing the newly created PRD", async () => { /* ... */ });
  ```
- **R-FE-SDD-03** — Every public component, hook, and module MUST be attributable to a specification. Speculative abstraction ("we'll need this later") is forbidden; it is the primary source of dead weight in agent-written codebases.
- **R-FE-SDD-04** — Types are the first line of the specification. Before writing a validation function, ask whether the type system can make the invalid state unrepresentable (see §7).
- **R-FE-SDD-05** — A behavioural change MUST update the specification in the same commit as the code. Documentation drift is a defect of equal severity to a logic defect.
- **R-FE-SDD-06** — When implementation reveals the specification is wrong, stop. Fix the specification first, then resume. Never let code silently become the spec of record.

---

## 2. Naming conventions

### 2.1 Casing and file naming (mechanical, enforced by ESLint/Prettier)

| Item | Convention | Example |
| --- | --- | --- |
| Component, type, interface, enum, generic `T...` | `PascalCase` | `PrdList`, `PrdDto`, `TProps` |
| Hook | `use` + `PascalCase` | `usePrds`, `useDebouncedValue` |
| Function, local, parameter, property | `camelCase` | `fetchPrds`, `scopeFilter` |
| Constant (module scope) | `UPPER_SNAKE_CASE` | `MAX_LIST_SIZE` |
| CSS custom property | `kebab-case` | `--surface-container` |
| File for a component/hook | `kebab-case` matching export | `prd-list.tsx`, `use-prds.ts` |
| Test file | `*.test.ts` / `*.test.tsx` | `prd-list.test.tsx` |

- **R-FE-NAM-01** — Acronyms are treated as words: `HttpClient`, `parseUrl`, `OauthToken`. Never `HTTPClient` or `parseURL`.
- **R-FE-NAM-02** — Do not stutter. Inside a `prds` module, export `usePrds`, not `usePrdsHook`. The module path carries context.
- **R-FE-NAM-03** — Never abbreviate a domain term. `subscription`, not `sub`. Abbreviate only universally-understood mechanics: `id`, `ref`, `props`, `fn`.
- **R-FE-NAM-04** — Components are named for what they render, not their container: `PrdList`, not `PrdListContainer`. Avoid `Page`/`View`/`Widget` suffixes unless genuinely distinct.

### 2.2 Type and interface naming

- **R-FE-NAM-05** — Types describing API payloads use a `Dto` suffix (`PrdDto`); domain-facing types use the domain term (`Prd`). Wire types live at the API boundary and are never the domain model (see R-FE-SEP-05).
- **R-FE-NAM-06** — Discriminated unions are named for the state they model (`PrdScope = 'project' | 'major-feature'`); boolean-flag substitutions are forbidden where an enum reads better (R-FE-TYP-05).
- **R-FE-NAM-07** — Event handler props are named `on*` (`onCreate`); callbacks passed to hooks are named for their role (`onSuccess`). Never `handle*` for props.

### 2.3 Component naming (React semantics carry meaning)

- **R-FE-NAM-08** — "Pure" presentational components have no side effects and no data fetching; "connected" components use hooks. Name by behaviour, not by layer.
- **R-FE-NAM-09** — Custom hooks return a plain object or tuple with stable, documented keys (`{ prds, status, refetch }`), never a class instance.

---

## 3. Repository and directory organization

### 3.1 `Web.Spa` layout

```
Web.Spa/
├── package.json              # dependencies & scripts (single source)
├── package-lock.json         # committed (R-FE-DEP-02)
├── tsconfig.json             # strict mode, noUncheckedIndexedAccess
├── vite.config.ts            # build & dev server
├── tailwind.config.ts        # token-derived theme (R-FE-STY-02)
├── index.html
├── public/
├── src/
│   ├── main.tsx              # entry point — thin (R-FE-DIR-04)
│   ├── app/                  # app shell: providers, routing, layout
│   ├── components/           # reusable presentational components
│   ├── features/             # feature folders, each owning its UI+hooks
│   │   └── feature-name/     # feature-owned UI, hooks, and tests
│   ├── hooks/                # cross-feature custom hooks
│   ├── api/                  # API client + DTO types + endpoint functions
│   ├── styles/               # tokens.css, global styles (no component CSS)
│   └── test/                 # test utilities, mocks, fixtures
└── e2e/                      # Playwright end-to-end specs
```

- **R-FE-DIR-01** — One primary concept per file. A component/hook file over **400 lines** is a review trigger; over **700**, it MUST be split.
- **R-FE-DIR-02** — Folder structure follows features, never technical kind. `features/prds/`, not `components/`, `hooks/`, `api/` as the top-level dump. Reusable cross-feature pieces live under the shared folders above.
- **R-FE-DIR-03** — Barrel files (`index.ts`) contain re-exports only. No logic in a barrel.
- **R-FE-DIR-04** — `main.tsx` MUST be thin: it mounts the root, wraps providers, and sets up routing. Wiring is testable via integration tests; the entry point itself is not.
- **R-FE-DIR-05** — A component's co-located test, story, or fixture lives beside it (`prd-list.test.tsx` next to `prd-list.tsx`).

### 3.2 Inside a feature

- **R-FE-DIR-06** — A feature owns: its components, its data hooks (calling `api/`), and its local state. Nothing outside the feature reaches into another feature's internals.
- **R-FE-DIR-07** — Cross-feature data access goes through `api/` or shared hooks, never by importing another feature's module internals.

---

## 4. Separation of concerns

### 4.1 The three responsibilities

| Concern | Owns | MUST NOT contain |
| --- | --- | --- |
| **Component** | Rendering props/state to DOM; user events; accessibility | Data fetching logic, business rules, raw fetch, localStorage |
| **Hook** | Encapsulated behaviour: data loading, derived state, effects, form state | DOM-specific rendering, business rules that belong to a domain module |
| **API client** | HTTP calls, DTO types, error translation, auth headers | UI state, rendering, business rules |

- **R-FE-SEP-01** — Components render; they do not orchestrate data flows inline. A component that mixes `useEffect`+`fetch`+derived state for the same concern is a missing hook.
- **R-FE-SEP-02** — Business rules live in typed pure modules (see §7), not in JSX. If a rule can be stated without mentioning React or the DOM, it is a rule and MUST NOT live in a component.
- **R-FE-SEP-03** — The API client is the only place that touches `fetch`/axios and network errors. Components and hooks receive translated domain results, not `Response` objects.
- **R-FE-SEP-04** — Server state and client state are distinct (R-FE-STT-01). A hook loads server state; derived UI state is computed, not duplicated.
- **R-FE-SEP-05** — Wire types and domain types are distinct. `PrdDto` lives in `api/` and is converted at the boundary into the feature's domain shape. A domain type MUST NOT be coupled to the wire contract it doesn't control.
- **R-FE-SEP-06** — One function does one of: **transform**, **fetch**, or **render**. Never two. This is what makes the logic unit-testable without a DOM.

### 4.2 Component discipline

- **R-FE-SEP-07** — Components accept explicit props; no ambient singletons, no reading global stores inside presentational components.
- **R-FE-SEP-08** — A component SHOULD fit on one screen (~120 lines of JSX). Length is a proxy for the real problem: it is doing more than one thing.
- **R-FE-SEP-09** — Prefer composition over prop-drilling. Props beyond ~6 for a component signal a missing composition boundary.
- **R-FE-SEP-10** — Keys in lists are stable and unique (`prd.id`), never array index where order can change.

---

## 5. Data access and the API boundary

- **R-FE-DAT-01** — All HTTP goes through the `api/` client. No bare `fetch` in components, hooks, or tests' production code.
- **R-FE-DAT-02** — Endpoint functions return typed results and translate HTTP/network errors into typed outcomes (a discriminated union or a small error model), never unhandled `Response` inspection.
- **R-FE-DAT-03** — The API client declares request and response types explicitly; the response type is validated against the wire contract (runtime parse for untrusted payloads — see R-FE-TYP-06).
- **R-FE-DAT-04** — Server state caching/refetch belongs in one layer (e.g. a data-fetching library or a single shared hook pattern), chosen deliberately per ADR. Ad-hoc per-component fetching with no shared strategy is forbidden.

---

## 6. State management

- **R-FE-STT-01** — Server state (what the API returns) and client state (UI/transient) are managed separately. Do not mirror server data into global client stores unless there is a measured reason.
- **R-FE-STT-02** — Prefer the smallest state solution that works: local component state → co-located hook state → context for shared UI state → a dedicated store only for genuinely cross-cutting client state (auth, theme, routing). A global store for one feature's data is a defect.
- **R-FE-STT-03** — No global mutable singletons. State lives in React state/context/stores with explicit ownership; hidden mutable modules make tests order-dependent and flaky.
- **R-FE-STT-04** — Derived state is computed, not stored. Use selectors/memoization rather than syncing derived values into state.
- **R-FE-STT-05** — Form state is explicit (controlled inputs or a deliberate form library); uncontrolled inputs with no validation path are a defect where the specification defines validation.

---

## 7. Type design — make invalid states unrepresentable

- **R-FE-TYP-01** — TypeScript runs in **strict mode** with `noUncheckedIndexedAccess` and `exactOptionalPropertyTypes` enabled. Project-wide `any` is forbidden (R-FE-AGT-04); project-wide `@ts-ignore` is forbidden.
- **R-FE-TYP-02** — Model alternatives with discriminated unions, not with optional fields or boolean flags. Three optional fields where exactly one is set is a union wearing a disguise.
- **R-FE-TYP-03** — Booleans in function signatures are forbidden where they select behaviour. `send(email, true)` is unreadable; use a union or enum.
- **R-FE-TYP-04** — Prefer `undefined`/nullability that models reality; avoid sentinel values (`-1`, `""`, `"unknown"`). Represent "not loaded", "loading", "error", and "ready" as a discriminated status union, not four booleans.
- **R-FE-TYP-05** — Enum-like string unions are preferred over numeric enums for wire-adjacent values (`PrdScope = 'project' | 'major-feature'`) so values are inspectable and match the API.
- **R-FE-TYP-06** — Untrusted payloads (API responses, user input) are parsed and validated at the boundary (e.g. a small runtime validator or zod at the API edge only), then flow as typed values. Do not cast `unknown`/`any` to the expected type.
- **R-FE-TYP-07** — `props` types are `Readonly<Props>`; parameters that must not be mutated are `readonly`/`ReadonlyArray`. Prefer immutable updates (spread) over mutation.

---

## 8. Effects and async

- **R-FE-EFF-01** — `useEffect` MUST NOT contain data fetching for server state. Data loading belongs in a data hook/library (R-FE-DAT-04); `useEffect` is for side effects tied to rendering (subscriptions, external widgets, imperative DOM).
- **R-FE-EFF-02** — Every `useEffect` with a subscription, timer, or async controller MUST clean up (unsubscribe/clear/abort) in its return; an effect that cannot clean up is a defect.
- **R-FE-EFF-03** — Effects declare all dependencies; the linter's exhaustive-deps rule is enabled and MUST NOT be silenced project-wide. If an effect intentionally runs once, justify it inline with the rule ID.
- **R-FE-EFF-04** — Asynchronous work in hooks tracks cancellation: on unmount or input change, stale results MUST NOT update state (abort controller or a mounted guard). A hook that can set state after unmount is a defect.
- **R-FE-EFF-05** — Derived values are computed with `useMemo` only when the computation is measurable; premature memoization is a defect. Referential stability for props is the exception where it matters (memoized components).
- **R-FE-EFF-06** — Long-running CPU work blocks the main thread; delegate to web workers only with a measured need and an ADR.
- **R-FE-EFF-07** — No floating promises in event handlers or effects without error handling. Every async call that can reject has a `.catch`/try-catch and a user-visible or logged outcome.

---

## 9. Styling and design tokens

- **R-FE-STY-01** — The SPA is styled with Tailwind CSS according to the target project's approved design decisions; component files use utility classes, not bespoke CSS files, except where a utility cannot express the rule.
- **R-FE-STY-02** — `design/epic-*/design-tokens.md` is the **single source** of styling truth. The Tailwind theme (colors, fonts, spacing, radii) is derived from the tokens; token changes require the theme to be updated in the same change (R-FE-SDD-05).
- **R-FE-STY-03** — The design language is defined by the target project's design reference and tokens. Deviations require explicit design intent, not default styling.
- **R-FE-STY-04** — Accessibility is not optional: semantic HTML, visible focus states, color contrast per the token palette, `aria` labels where the design is icon-only, and keyboard-operable interactions.
- **R-FE-STY-05** — No global CSS blobs for component styling; global styles are limited to tokens/reset/fonts. Component layout lives with the component.

---

## 10. Testing

### 10.1 Non-negotiables

- **R-FE-TST-01** — Tests are written **before** the implementation. Red first — a test that has never failed has never been shown to test anything.
- **R-FE-TST-02** — Every public component, hook, and error path MUST have at least one test. Happy path alone is not coverage.
- **R-FE-TST-03** — Every specified behaviour has a test citing its spec ID (R-FE-SDD-02). Every fixed bug gains a regression test that fails before the fix.
- **R-FE-TST-04** — A test MUST be able to fail for exactly one reason. Multiple unrelated assertion groups in one test means multiple tests.
- **R-FE-TST-05** — Tests MUST be deterministic. No wall-clock dependence, no reliance on execution order, no un-mocked network. A flaky test is deleted or fixed within one working day — never skipped (R-FE-AGT-03).
- **R-FE-TST-06** — Assert on user-visible behaviour (Testing Library queries by role/text), never on implementation details (class names, internal state, snapshot of internal structure). A refactor that changes no behaviour MUST NOT change a test.
- **R-FE-TST-07** — No conditional logic in tests. An `if` in a test means it is two tests.

### 10.2 The pyramid

| Tier | Location | Scope | Speed |
| --- | --- | --- | --- |
| Unit | `*.test.ts` beside pure modules | transforms, rules, reducers — no DOM | µs–ms |
| Component | `*.test.tsx` beside components (Vitest + Testing Library) | render, events, interactions with mocked API | ms |
| Integration | feature-level tests | feature flow through hooks + components, mocked API client | ms |
| End-to-end | `e2e/` (Playwright) | real browser against the built app + API | s |

- **R-FE-TST-08** — Pure logic (R-FE-SEP-06) is unit-tested without a DOM; components are tested for behaviour with Testing Library.
- **R-FE-TST-09** — The API client is tested with a mocked/fixture transport; components/hooks depend on a mocked `api/` boundary, never on a live server in unit/component tests.
- **R-FE-TST-10** — E2E tests cover the critical user journeys only (create → see in list), against the running app with a controlled back end.

### 10.3 Coverage and rigour

- **R-FE-TST-11** — Line coverage ≥ **80%** for `Web.Spa/src` production code, measured by Vitest/V8. CI fails below threshold. Coverage is a floor, not a goal.
- **R-FE-TST-12** — Error/edge paths are explicit: every typed error state (loading, error, empty, ready) has a test that produces it.
- **R-FE-TST-13** — Pure functions with algebraic properties (parsers, validators, formatters) MUST have property/table-driven tests covering boundaries.
- **R-FE-TST-14** — Snapshot tests are for large stable outputs only and MUST be reviewed; accepting snapshots blindly converts tests into a changelog. Prefer behavioural assertions.

### 10.4 Structure and readability

- **R-FE-TST-15** — Test names describe behaviour and outcome: `returns to the list containing the newly created PRD`. Never `test 1`.
- **R-FE-TST-16** — Tests follow visible Arrange / Act / Assert structure, separated by blank lines.
- **R-FE-TST-17** — Test utilities (render helpers, mocks, fixtures) live in `src/test/` and are never `public` in production modules.
- **R-FE-TST-18** — Test code is production code: reviewed, formatted, linted, and held to §2 naming rules.

---

## 11. Documentation

- **R-FE-DOC-01** — Every exported hook and non-trivial pure module has a doc comment: purpose, parameters, and return shape. Public components carry a one-line purpose when non-obvious.
- **R-FE-DOC-02** — Comments explain *why*, never *what*. If code needs a comment to explain what it does, rename things until it doesn't.
- **R-FE-DOC-03** — Architectural decisions for the SPA are recorded as ADRs in `specs/adr/`, immutable once accepted, superseded rather than edited.
- **R-FE-DOC-04** — `TODO`/`FIXME` MUST reference a tracked issue: `// TODO(#412): ...`. Untracked TODOs are forbidden.
- **R-FE-DOC-05** — The design reference (`design/epic-*/`) is not code; implement from tokens and specifications, and flag discrepancies to the design instead of silently reinterpreting.

---

## 12. Dependencies

- **R-FE-DEP-01** — Every new dependency requires justification: maintenance status, licence, bundle weight, and alternatives. The target project's approved React/Vite/Tailwind base stack is the default; anything beyond it needs human approval (R-FE-AGT-02).
- **R-FE-DEP-02** — `package-lock.json` is committed; installs use `npm ci` in CI (reproducible builds).
- **R-FE-DEP-03** — `npm audit` / `npm outdated` run in CI; known-vulnerable direct dependencies block merge.
- **R-FE-DEP-04** — Feature switches are additive and non-breaking; no mutually exclusive build conditions that break the bundle.
- **R-FE-DEP-05** — Bundle weight is a review consideration: a large dependency for a small behaviour is a defect unless justified.

---

## 13. Tooling gates

- **R-FE-TOOL-01** — Node/npm versions are pinned (`.nvmrc` / `engines`); the Vite version is pinned in `package.json`.
- **R-FE-TOOL-02** — Lint/format configuration lives in `eslint.config.*` and Prettier config; analyzers are inherited by every source file. Baseline:
  ```jsonc
  // tsconfig.json — strict type contract
  { "compilerOptions": { "strict": true, "noUncheckedIndexedAccess": true, "exactOptionalPropertyTypes": true, "noFallthroughCasesInSwitch": true } }
  ```
- **R-FE-TOOL-03** — Suppressions are narrow and justified. Project-wide rule disabling is forbidden; per-line suppressions require a reason naming the rule and an ADR/issue where the rule is `MUST`.
- **R-FE-TOOL-04** — The following MUST pass before any commit is considered complete (provide as a single `npm run ci` so there is exactly one command and no divergence between local and CI):
  ```
  npm run typecheck      # tsc --noEmit
  npm run lint           # eslint . (errors as errors, no --fix in CI)
  npm run format:check   # prettier --check
  npm test -- --coverage # vitest run + coverage >= 80%
  npm run build          # vite build (type-safe production bundle)
  ```
- **R-FE-TOOL-05** — CI additionally runs `npm audit`, and E2E (`npm run e2e`) against the built app on a scheduled/merge basis.
- **R-FE-TOOL-06** — CI is not advisory. A red gate blocks merge. Never merge with a disabled or skipped gate.

---

## 14. Definition of Done

A unit of work is complete only when **every** box is checked, verified by observed command output rather than assumption (R-FE-AGT-07).

- [ ] A specification exists for the behaviour, and is current (§1)
- [ ] Tests were written first and observed to fail (R-FE-TST-01)
- [ ] Every specified behaviour is covered and traceable to its spec ID (R-FE-SDD-02)
- [ ] Error paths, edge cases, and boundary values are tested (R-FE-TST-12)
- [ ] Strict TypeScript passes with no `any` / `@ts-ignore` (R-FE-AGT-04)
- [ ] Effects clean up; no state-after-unmount; no floating promises (§8)
- [ ] Styling derives from design tokens; no global CSS blobs (§9)
- [ ] No new dependency without recorded approval (R-FE-AGT-02)
- [ ] `npm run typecheck` / `lint` / `format:check` / `test --coverage` / `build` pass clean (R-FE-TOOL-04)
- [ ] Coverage thresholds met (R-FE-TST-11)
- [ ] Deviations recorded with rule IDs; `MUST` deviations have an ADR (§0.2)
- [ ] Spec, ADRs, and design reference updated in the same commit (R-FE-SDD-05)

---

## 15. Amendment

This constitution is version-controlled and amended by pull request with an accompanying ADR stating the problem, the proposed rule change, and the migration path for existing code. Rules are added with new IDs; retired rules are marked `RETIRED` in place. Agents MUST NOT amend this document without explicit human authorization and an accompanying ADR.
