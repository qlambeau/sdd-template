# .NET Engineering Constitution

**Status:** Normative. **Audience:** human and agentic contributors. **Applies to:** all C# / .NET code in this repository.

This document is the highest-precedence engineering authority in the repository. It governs *how* C# / .NET code is written. Specifications govern *what* is built. When a specification and this constitution conflict, the constitution wins on form and the specification wins on behaviour — and the conflict MUST be raised, not silently resolved.

This constitution is the repository's C#/.NET engineering authority. If the
target project uses a different backend stack, replace this document with an
equivalent stack-specific constitution before implementation begins.

---

## 0. How to use this document

### 0.1 Rule grammar

| Keyword | Meaning |
| --- | --- |
| **MUST** / **MUST NOT** | Non-negotiable. A violation blocks merge. |
| **SHOULD** / **SHOULD NOT** | Strong default. Deviation requires a code comment naming the rule ID and the reason. |
| **MAY** | Permitted, no justification needed. |

Every rule has a stable ID (`R-<AREA>-<NN>`). Cite rule IDs in commit messages, review comments, and deviation comments. IDs are append-only: a retired rule is marked `RETIRED`, never renumbered or reused.

### 0.2 Recording a deviation

A `SHOULD` may be broken inline. A `MUST` may not — it requires an ADR.

```csharp
// DEVIATION(R-ASY-03): this lock is held across await because the
// upstream native handle is not thread-safe. Contained to this method.
```

### 0.3 Rules for the agent

- **R-AGT-01** — An agent MUST read this document before its first edit in a session, and MUST NOT rely on a summary of it.
- **R-AGT-02** — An agent MUST NOT introduce a new NuGet package, a new project/solution reference, or a new architectural layer without explicit human approval in the current session.
- **R-AGT-03** — An agent MUST NOT weaken a test, delete an assertion, add `[Fact(Skip = ...)]` / `[Ignore]`, or loosen an analyzer rule to make a build pass. If the code cannot satisfy the test, stop and report.
- **R-AGT-04** — An agent MUST NOT write `unsafe`. `unsafe` in C# is a human-authored, human-reviewed construct (see §8).
- **R-AGT-05** — An agent MUST leave the working tree passing §13 gates at the end of every unit of work. "I'll fix the tests next" is not a valid end state.
- **R-AGT-06** — When a specification is ambiguous, an agent MUST ask rather than choose. Guessing at behaviour is the single most expensive failure mode in spec-driven development.
- **R-AGT-07** — An agent MUST NOT mark work complete based on intent. Completion requires the §14 checklist to have actually been executed, with command output observed.

---

## 1. Spec-driven development loop

Code exists to satisfy a specification. The loop is fixed:

```
SPEC  →  CONTRACT  →  RED TEST  →  IMPLEMENTATION  →  GREEN  →  REFACTOR  →  TRACE
```

- **R-SDD-01** — No production code is written before a specification exists for the behaviour it implements. A specification MAY be a single paragraph, but it MUST be written down and MUST be addressable by an ID.
- **R-SDD-02** — Every specified behaviour MUST be traceable to at least one test. Encode the link in the test name or in a doc comment on the test:
  ```csharp
  /// <summary>Covers: SPEC-AUTH-014 — expired tokens are rejected before signature checks.</summary>
  [Fact]
  public void RejectsExpiredToken_WithoutVerifyingSignature() { /* ... */ }
  ```
- **R-SDD-03** — Every public type, interface, and function MUST be attributable to a specification. Speculative abstraction ("we'll need this later") is forbidden; it is the primary source of dead weight in agent-written codebases.
- **R-SDD-04** — Types are the first line of the specification. Before writing a validation function, ask whether the type system can make the invalid state unrepresentable (see §7).
- **R-SDD-05** — A behavioural change MUST update the specification in the same commit as the code. Documentation drift is a defect of equal severity to a logic defect.
- **R-SDD-06** — When implementation reveals the specification is wrong, stop. Fix the specification first, then resume. Never let code silently become the spec of record.

---

## 2. Naming conventions

### 2.1 Casing (mechanical, enforced by `.editorconfig` and analyzers)

| Item | Convention | Example |
| --- | --- | --- |
| Namespace, type, method, property, event | `PascalCase` | `OrderPricing`, `TokenStore`, `SettleInvoice` |
| Interface | `I` + `PascalCase` | `IOrderRepository`, `IClock` |
| Local, parameter | `camelCase` | `retryBudget`, `settledTotal` |
| Private field | `_camelCase` | `_tokenStore` |
| Constant | `PascalCase` | `MaxRetryCount` |
| Type parameter | `T` or descriptive `PascalCase` | `T`, `TEntity` |
| Async method | `PascalCase` + `Async` suffix | `SettleOrderAsync` |
| Feature flag | `kebab-case` | `postgres-store` |

- **R-NAM-01** — Acronyms of three or more letters are treated as words: `HttpClient`, `ParseUrl`, `OAuthToken`. Two-letter acronyms stay uppercase (`IOStream`). Never `HTTPClient` or `parseURL`.
- **R-NAM-02** — Do not stutter. Inside namespace `Token` the type is `Token.Store`, not `Token.TokenStore`. The namespace carries the context.
- **R-NAM-03** — Never abbreviate a domain term. `subscription`, not `sub`. Abbreviate only universally-understood mechanics: `ctx`, `cfg`, `id`, `db`, `tx`, `req`, `res`, `i`/`j` for indices.

### 2.2 Method and member naming (.NET semantics carry meaning)

- **R-NAM-04** — Conversion prefixes are load-bearing and MUST NOT be misused:
  - `As*` — cheap wrapper view over the same data, no allocation, infallible. `AsSpan`, `AsMemory`, `AsReadOnly`.
  - `To*` — allocating conversion, infallible. `ToString`, `ToArray`, `ToDictionary`.
  - `Parse*` — builds a value from text and throws on invalid input. `int.Parse`.
  - `Try*` — fallible attempt returning `bool` with an `out` value; never throws for the expected failure. `TryParse`, `TryDequeue`.
- **R-NAM-05** — Prefer properties for cheap accessors (`public string Name { get; }`). Reserve `Get*` methods for expensive computation, remote calls, or indexed lookup (`GetById`). Never pair a `Get*` method with an equivalent property.
- **R-NAM-06** — Predicates read as assertions and return `bool`: `IsExpired`, `HasCapacity`, `CanSettle`. Never `Check*` for something that returns `bool`.
- **R-NAM-07** — Constructors: overload the constructor for simple variants; use static factory methods for named construction (`Order.Create`, `OrderId.From`) and fallible construction (`Order.TryCreate`, returning `bool`/result). A constructor with more than four parameters SHOULD be replaced by a builder or options record.
- **R-NAM-08** — Fallible operations that mirror an infallible method take the `Try` prefix: `Reserve` / `TryReserve`.
- **R-NAM-09** — Iterator methods return `IEnumerable<T>` / `IAsyncEnumerable<T>` and are named as plurals (`Orders()`) or `Enumerate*`/`Iterate*`; never `GetAllList`.

### 2.3 Domain naming

- **R-NAM-10** — The domain layer uses the ubiquitous language of the specification verbatim. If the spec says "settlement", the code says `Settlement` — not `Payment`, not `Transaction`. A rename in the spec is a rename in the code, in the same commit.
- **R-NAM-11** — Names MUST NOT encode their technical layer inside the domain. `OrderService`, `OrderManager`, `OrderHelper`, `OrderUtil`, and `OrderData` are all banned in `Domain`. Name the behaviour: `OrderPricing`, `SettlementPolicy`, `RefundCalculator`.
- **R-NAM-12** — `Util`, `Common`, `Helpers`, `Misc`, `Shared`, and `Core` are forbidden as namespace, project, or folder names. They are landfills. Name the capability instead.
- **R-NAM-13** — Interfaces are named for the capability they confer, with the standard `I` prefix: `IOrderRepository`, `IClock`, `ISerializable`. Never suffix `Interface`, `Base`, or `Impl`.
- **R-NAM-14** — Concrete implementations of a port are named for their technology, not for the port: `PostgresOrderRepository`, `InMemoryOrderRepository`, `SystemClock` — implementing `IOrderRepository` and `IClock`. Never `OrderRepositoryImpl`.

---

## 3. Repository and directory organization

### 3.1 Repository and .NET solution layout

This repository currently contains the SDD Workflow Kit and uses the following
specification layout:

```
.
├── AGENTS.md
├── README.md
├── .agents/
│   └── skills/
└── specs/
    ├── SDD_WORKFLOW.md           # normative workflow — single authority for lifecycle/gates
    ├── CONSTITUTION.md
    ├── prds/
    ├── adr/
    ├── templates/
    ├── NNN-feature-slug/
    └── archive/
```

When the .NET solution is introduced, it follows the layout
below. Layering is enforced by project references and architectural tests, not by convention. A layer is a
project; an illegal dependency is a build error.

```
.
├── Project.sln                   # solution — projects, shared versions via Directory.Packages.props
├── global.json                   # pinned SDK version (R-TOOL-01)
├── Directory.Build.props         # shared MSBuild properties
├── Directory.Packages.props      # central package management (R-DIR-10)
├── .editorconfig                 # formatting & analyzer configuration
├── specs/
│   ├── CONSTITUTION.md           # this file
│   ├── prds/                     # PRD-NNN.md — product requirements
│   ├── adr/                      # ADR-NNN.md — architecture decisions
│   ├── templates/                # source templates, not active specifications
│   └── NNN-feature-slug/         # feature packet and executable scenarios
├── src/
│   ├── Domain/                   # layer 0 — pure. zero I/O, zero async, zero frameworks.
│   ├── Application/              # layer 1 — use cases + PORT INTERFACES. depends on Domain only.
│   ├── Infrastructure/           # layer 2a — cross-cutting: persistence, telemetry, config.
│   │   ├── Persistence/
│   │   └── Messaging/
│   ├── Adapters/                 # layer 2b — one project per technology, implements ports.
│   │   ├── Store.Postgres/
│   │   ├── Http.Client/
│   │   └── Queue.Kafka/
│   ├── Web.Api/                  # layer 3 — ASP.NET Core host, composition root wiring
│   └── Web.Spa/                  # React/TypeScript frontend (separate build, no Blazor)
├── tests/
│   ├── Domain.Tests/             # unit tests for Domain
│   ├── Application.Tests/        # handler tests with in-memory fakes
│   └── Api.IntegrationTests/     # integration / E2E tests
└── build/                        # build automation (MSBuild targets / scripts)
```

- **R-DIR-01** — The dependency graph is acyclic and points inward only:
  ```
  app  →  adapters  →  application  →  domain
   └──────────────────────┴───────────────┘
  ```
  `Domain` depends on nothing in the solution. `Application` depends only on `Domain`. Adapters depend on `Application` (for the port interfaces) and `Domain` (for the types). **Nothing depends on an adapter except `app`.**
- **R-DIR-02** — `Domain` MUST NOT depend on `Microsoft.AspNetCore.*`, `EntityFrameworkCore`, `StackExchange.Redis`, `System.Net.Http.Json` serialization on domain types, or any other I/O or framework package. Permitted: the Base Class Library, and pure computation packages (`NodaTime` without I/O features). Enforce with project-reference rules and architecture tests scoped to the project.
- **R-DIR-03** — Composition (constructing concrete adapters and injecting them) happens **only** in `Web.Api` (composition root). If any other project names a concrete adapter type, the layering is broken.
- **R-DIR-04** — `Program.cs` MUST be thin (~50 lines). It parses configuration, builds the host, and runs it. Host wiring is testable via integration tests; the entry point itself is not.
- **R-DIR-05** — File and namespace layout follows feature/domain structure (e.g. `Domain/Billing/`), never technical kind (`Models/`, `Interfaces/`, `Dtos/`).
- **R-DIR-06** — One primary concept per file. A file over **400 lines** of non-test code is a review trigger; over **700**, it MUST be split. Test files do not count toward the limit.
- **R-DIR-07** — Barrel-like files (`_Imports.cs`, `Usings.cs`, re-export hubs) contain declarations and usings only. No logic in a barrel.
- **R-DIR-08** — Folders inside a project follow the domain, never the technical kind. `Domain/Billing/`, not `Domain/Structs/` or `Domain/Interfaces/`.
- **R-DIR-09** — Shared code between two projects that has no home does not go in a `Common` project (R-NAM-12). Either it belongs to `Domain`, or it is a genuine capability and gets a named project.
- **R-DIR-10** — Package versions are declared once in `Directory.Packages.props` (central package management); projects use `PackageReference` without version. Version drift between projects is a defect.

### 3.2 Inside a project

```
src/Application/
├── Application.csproj
├── Errors/
│   └── ApplicationError.cs        # this project's error hierarchy (§6)
├── Ports/                         # PORT INTERFACES — the seams (§5)
│   ├── IOrderRepository.cs
│   └── IClock.cs
├── SettleOrder/
│   ├── SettleOrderCommand.cs      # input record
│   ├── SettleOrderHandler.cs      # orchestration
│   └── SettleOrderValidator.cs
└── tests -> ../tests/             # tests live under /tests/<Project>.Tests
```

- **R-DIR-11** — Project roots contain no logic. Externally consumed types live in curated namespaces; internal implementation stays `internal`.
- **R-DIR-12** — Design the public API deliberately. Callers consume `Application.SettleOrder.SettleOrderHandler` through documented entry points only; internal namespaces are not part of the contract. Use `InternalsVisibleTo` only for test assemblies.
- **R-DIR-13** — Default visibility is `private`/`internal`. Escalate one level at a time: `private` → `internal` → `public`. Every `public` on a non-API item is a bug.

---

## 4. Separation of concerns

### 4.1 The four responsibilities

Each layer has exactly one job. Mixing them is the defect this section exists to prevent.

| Layer | Owns | MUST NOT contain |
| --- | --- | --- |
| **domain** | Entities, value objects, invariants, pure business rules and calculations | I/O, `async`, SQL, HTTP, clocks, RNG, env vars, logging, JSON serialization attributes |
| **application** | Use-case orchestration, transaction boundaries, port interface definitions, authorization decisions | Business rules, SQL, HTTP types, concrete adapter types |
| **adapters** | Translation between the outside world and port interfaces; wire formats; retries | Business rules, decisions the domain should make |
| **app** | Configuration, wiring, process lifecycle, shutdown | Anything else |

- **R-SEP-01** — Business rules live in `Domain`. If a rule can be stated without mentioning a database, a queue, or a request, it is a domain rule and it MUST NOT live in an adapter or handler.
- **R-SEP-02** — The domain is deterministic and pure. **All** nondeterminism — time, randomness, GUID generation, environment, filesystem, network — enters through a port (§5). A domain function given the same inputs MUST return the same output, forever.
  ```csharp
  // WRONG — domain reaches for ambient state
  public class Subscription
  {
      public bool IsExpired() => _clock.UtcNow > ExpiresAt;
  }

  // RIGHT — time is an input
  public class Subscription
  {
      public bool IsExpired(Timestamp now) => now > ExpiresAt;
  }
  ```
- **R-SEP-03** — A use-case handler orchestrates; it does not compute. Its body reads as: load via port → call domain → persist via port → return. Non-trivial branching logic inside a handler is misplaced domain logic.
- **R-SEP-04** — Adapters are thin and dumb. An adapter maps types and performs I/O. An `if` in an adapter that expresses a business condition is a layering violation.
- **R-SEP-05** — Wire types and domain types are distinct. HTTP DTOs and database row models live in the adapter that owns them and are converted at the boundary via explicit mapper methods. A domain type MUST NOT carry serialization attributes (`JsonPropertyName`, EF mappings) for an external format — that couples your business model to a wire contract you don't control.
- **R-SEP-06** — Transaction boundaries are declared by the application layer and executed by the adapter. The domain never knows a transaction exists.
- **R-SEP-07** — A single function does one of: **decide**, **compute**, or **perform I/O**. Never two. This is what makes the decision logic unit-testable without a runtime.
- **R-SEP-08** — Push I/O to the edges; keep the middle pure ("functional core, imperative shell"). Read everything you need, compute the decision purely, then write. Do not interleave.
- **R-SEP-09** — No global mutable state. No mutable `static` fields, no mutable singletons, no ambient service locator outside composition. Dependencies are passed explicitly via constructors. This is not stylistic — hidden state makes parallel tests flaky.
- **R-SEP-10** — Configuration is read once, at startup, in `app`, bound to an options type, and passed down as typed values. No `Environment.GetEnvironmentVariable` outside `app`.

### 4.2 Function and type discipline

- **R-SEP-11** — A function SHOULD fit on one screen (~40 lines) and MUST NOT exceed 80 lines of body. Length is a proxy for the real problem: it is doing more than one thing.
- **R-SEP-12** — Cyclomatic complexity per function SHOULD stay ≤ 10. Enforce with the Roslyn maintainability analyzers (CA1502).
- **R-SEP-13** — More than 5 parameters means a missing parameter object. More than 3 with the same primitive type means a missing value object (R-TYP-01).
- **R-SEP-14** — Accept the most general type you can use: `string` over concrete builder types, `IReadOnlyList<T>` over `List<T>`, `IEnumerable<T>` when you only iterate.
- **R-SEP-15** — Return concrete types from public APIs. Prefer a named type over `object`/dynamic on the return path unless heterogeneity is genuinely required.

### 4.3 Database and persistence schemas

- **R-DB-01** — When creating a table schema for a database, regardless of the database technology, every table MUST have a domain-independent primary key named `id` with a unique identifier enforced by the database system, if supported.

---

## 5. Ports and interfaces — abstraction at the seams

**The stance:** interfaces are for *seams*, not for *decoration*. C#'s type system already gives you encapsulation and change-detection; an interface's job here is to invert a dependency so the pure core does not know about the impure world. An interface with exactly one implementation that will never grow one is negative value — it adds indirection, defeats devirtualization, and obscures the call graph. Interface-per-class is a Java reflex, not a .NET one.

### 5.1 When an interface is mandatory

- **R-TRT-01** — Every crossing of the process boundary MUST be behind a port interface defined in `Application.Ports`: databases, HTTP calls, message queues, filesystem, clock, randomness, ID generation, secrets, email/SMS, feature flags, payment providers.
- **R-TRT-02** — Every source of nondeterminism MUST be behind a port, however trivial. `IClock` and `IIdGenerator` are the two most valuable interfaces in most codebases; without them, half your tests need `Task.Delay` or accept unpredictable output.
- **R-TRT-03** — Any point where the specification anticipates more than one strategy MUST be an interface (e.g. `IPricingStrategy`, `INotificationChannel`).
- **R-TRT-04** — Ports are **defined by the consumer, in the inner layer** (dependency inversion). `Application` owns `IOrderRepository`; the Postgres adapter implements it. Never the reverse — that would invert the dependency arrow and break R-DIR-01.

### 5.2 When an interface is forbidden

- **R-TRT-05** — MUST NOT define an interface for a pure, deterministic, in-process type. Value objects, entities, and calculators are concrete. There is nothing to fake — call them directly.
- **R-TRT-06** — MUST NOT define an interface with one implementation and no second implementation in prospect, *unless* it exists to satisfy R-TRT-01/02 (in which case the test double is the second implementation, and that counts).
- **R-TRT-07** — MUST NOT create an interface purely to enable mocking of code you own and that is already pure. If a unit is hard to test without a mock, the fault is usually R-SEP-07, not a missing interface. Fix the seam, don't mock the mess.

### 5.3 Designing a port

- **R-TRT-08** — Ports are narrow and role-based. Prefer three focused interfaces over one twelve-method `IRepository`. A test double for a fat interface is mostly `throw new NotSupportedException()`, which is a design smell made visible.
- **R-TRT-09** — Port signatures speak the domain's language: domain types in, domain types out, domain errors on failure. No `SqlException`, no `HttpResponseMessage`, no `JsonElement` in a port signature — that leaks the technology the port exists to hide.
- **R-TRT-10** — Port methods signal failure with error types owned by the port's layer (§6). Adapters map their technology exceptions into them (R-ERR-08).
- **R-TRT-11** — Ports SHOULD remain free of static abstract members (`abstract` interface members, DIM) unless a measured reason exists, so classic fakes and dynamic proxies stay available.
- **R-TRT-12** — Async port methods return `Task`/`ValueTask` and accept a `CancellationToken` parameter. Choose one convention per workspace for cancellation defaults and record it in an ADR.
- **R-TRT-13** — Ports injected across threads or registered in DI as singletons MUST be thread-safe or explicitly documented as scoped/transient. Do not discover lifetime bugs at the call site.
- **R-TRT-14** — Interfaces carry documented contracts, not just signatures. Document invariants, error semantics, idempotency, and ordering guarantees on the **interface**; implementations inherit the obligation. An undocumented port contract will be implemented inconsistently.
- **R-TRT-15** — Default interface methods only for genuine convenience derived from required members. Never put behaviour an implementor is expected to override in a default method — make it required.

### 5.4 Injection

- **R-TRT-16** — Dependencies are injected via constructor through the host's DI container, wired only in the composition root. No service locator (`IServiceProvider.GetService` deep inside a component), no global registry, no singleton mutable holder.
- **R-TRT-17** — Default to direct constructor injection of the interface type. Generic constraints (`SettleOrderHandler<TRepo, TClock> where TRepo : IOrderRepository`) are reserved for measured hot paths where devirtualization matters; DI-managed instance registration is the norm.
- **R-TRT-18** — Shared ports across concurrent tasks must be safe to call concurrently. Never wrap a port in application-level locks to fake safety — an implementation is responsible for its own interior synchronization.
- **R-TRT-19** — Constructor parameter count per type SHOULD stay ≤ 5. Beyond that, bundle collaborators into a named context/parameters record.
- **R-TRT-20** — Every port interface MUST ship with at least one in-memory test double in the test projects, so handlers can be tested without infrastructure:
  ```csharp
  // tests/Application.Tests/Fakes/InMemoryOrderRepository.cs
  /// <summary>Deterministic in-memory IOrderRepository. Honours the same contract as production impls.</summary>
  public sealed class InMemoryOrderRepository : IOrderRepository { /* ... */ }
  ```
- **R-TRT-21** — Prefer hand-written fakes over generated mocks. A fake that actually behaves (an in-memory dictionary) tests behaviour; a mock that asserts call order tests your implementation's internals and calcifies refactoring. Use Moq/NSubstitute only for verifying that an interaction *happened* when that interaction is the specified behaviour.

---

## 6. Error handling

- **R-ERR-01** — Typed exceptions for expected failures; `Debug.Fail`/`Environment.FailFast`-class aborts for programmer error only. A malformed user input is a caught, mapped domain/application exception — never a crash.
- **R-ERR-02** — Swallowing exceptions (`catch { }`), catching `Exception` broadly without rethrow or mapping, `!` (null-forgiving) without proof, and `throw` for control flow are forbidden in production code paths. Permitted in tests, and where a comment proves impossibility:
  ```csharp
  // IMPOSSIBLE-NULL: regex is built from a compile-time literal validated by RegexValidationTests.
  var pattern = _compiledPattern!;
  ```
  Enforce with analyzers (`CA1062`, `CS8602`) in library projects.
- **R-ERR-03** — Each project defines its own exception hierarchy rooted in one base type (e.g. `ApplicationError : Exception`). Applications MAY surface a generic error page/handler at the top level only. Raw `Exception` MUST NOT cross a library's public boundary as a documented contract — callers must be able to catch specific failures.
- **R-ERR-04** — Exceptions are named for the *situation*, not the source: `OrderNotFoundException`, `InsufficientFundsException` — not `SqlExceptionWrapper`.
- **R-ERR-05** — Public exception types are sealed unless extension by consumers is a specified scenario.
- **R-ERR-06** — Preserve the cause chain with `InnerException` (or `ExceptionDispatchInfo.Capture(...).Throw()`). Never stringify an exception to pass it along.
- **R-ERR-07** — Exception messages describe the failure with context and match .NET style: `"Order {id} was not found."` Not `"Error!"`. Messages are for operators; do not encode control-flow information only readable by code.
- **R-ERR-08** — Adapters translate technology exceptions at the boundary. A `SqlException`/`HttpRequestException` MUST NOT escape the adapter's public API un-mapped.
- **R-ERR-09** — Let exceptions propagate naturally. Never `catch` a specific exception only to rethrow it unchanged (`catch (Exception e) { throw e; }` destroys stack traces).
- **R-ERR-10** — Never silently discard an exception. `catch (Exception) { /* ignore */ }` requires a comment explaining why the failure is genuinely irrelevant.
- **R-ERR-11** — Distinguish retryable from terminal failures in the error type when the specification cares (e.g. a transient marker on the application error). Retry policy lives in the adapter; the decision about whether retrying is meaningful lives in the type.
- **R-ERR-12** — Exceptions are logged **once**, at the boundary that handles them. Log-and-rethrow produces duplicate noise and destroys signal.

---

## 7. Type design — make invalid states unrepresentable

- **R-TYP-01** — Wrap every domain identifier and constrained primitive in a value object. `readonly record struct OrderId(Guid Value)`, `Email`, `Quantity`. Bare `string`/`Guid`/`int` as domain concepts are forbidden — they permit `Charge(customerId: orderId, orderId: customerId)` to compile with the arguments swapped.
- **R-TYP-02** — Validate once, at construction, in a factory (`Create`/`TryCreate`) or constructor guard clause. Downstream code receives the type and MUST NOT re-validate. *Parse, don't validate.*
- **R-TYP-03** — A type with an invariant keeps its state private and exposes read-only accessors (`init`-only or `get` only). A publicly settable property is a public promise that any value of that type is valid.
- **R-TYP-04** — Model alternatives with discriminated unions of records or enums plus exhaustive switch expressions, not with nullable fields or boolean flags. Three nullable properties where exactly one is set is a union wearing a disguise.
- **R-TYP-05** — Booleans in signatures are forbidden where they select behaviour. `Send(email, true)` is unreadable; use a two-variant enum or distinct methods.
- **R-TYP-06** — Prefer nullable reference types (`string?`) over sentinels (`-1`, `""`, `0`) and express non-empty/non-zero invariants in value objects where real.
- **R-TYP-07** — Model specified lifecycles with explicit states (sealed state classes/records or an enum + guarded transitions) so illegal transitions require deliberate circumvention. Do not apply speculative state machinery (R-SDD-03).
- **R-TYP-08** — Choose equality deliberately: `record`/`readonly record struct` give value semantics; classes default to reference equality. Implement `IEquatable<T>` when the type is compared or keyed. Never expose a parameterless constructor on a type with an invariant — it manufactures an invalid value.
- **R-TYP-09** — Prefer immutable inputs; share references instead of copying, but clone deliberately at trust boundaries where callers could mutate shared state. Defensive `.ToArray()`/`.ToList()` sprinkled to silence analyzers is a design defect, not a fix.
- **R-TYP-10** — Use collection interfaces (`IReadOnlyList<T>`, `IEnumerable<T>`) in signatures for flexibility; named domain types when the concept is referenced more than once.

---

## 8. Unsafe, panics, and correctness

- **R-UNS-01** — `<AllowUnsafeBlocks>false</AllowUnsafeBlocks>` in every `.csproj`. Remove it only with an ADR.
- **R-UNS-02** — Where `unsafe` is unavoidable (interop), it is confined to a single dedicated adapter project that exposes a fully safe managed API.
- **R-UNS-03** — Every `unsafe` block carries a comment stating the invariants relied upon and why they hold. Reviewers treat undocumented `unsafe` as a defect.
- **R-UNS-04** — Unsafe code MUST be exercised under additional verification (runtime CodeQL/security scanning) in CI where applicable.
- **R-UNS-05** — Integer arithmetic on domain quantities uses explicit checked arithmetic (`checked`, `Math.Clamp`) where overflow is possible. Silent release-mode wrapping on a monetary quantity is a defect waiting for production traffic.
- **R-UNS-06** — Money is never `float`/`double`. Use `decimal` or integer minor units.
- **R-UNS-07** — Indexing (`list[i]`) is forbidden outside tests where the index is not provably in range; validate or use safe access patterns. Null-forgiving `!` likewise requires proof.
- **R-UNS-08** — Exceptions MUST NOT escape a request handler unobserved. The top-level middleware catches, logs once, and maps to an error response.

---

## 9. Async and concurrency

- **R-ASY-01** — `async` only where there is real I/O concurrency. The `Domain` project is synchronous (R-DIR-02). Do not make a method async because its caller is; do not use `async void` anywhere except event handlers, and avoid those in production paths.
- **R-ASY-02** — One host, created once, in `app`. Library projects MUST NOT start hosts, spawn unmanaged threads, or block on async work (`.Result`, `.Wait()`, `GetAwaiter().GetResult()`) — see R-ERR-02.
- **R-ASY-03** — Never hold a `lock` (Monitor) across `await`. Use `SemaphoreSlim.WaitAsync` when the critical section must span an await, and prefer message passing/channels over shared locks.
- **R-ASY-04** — Never block a thread in an async context. CPU-bound work inside a request goes to a queued background worker (`IHostedService`/channel consumer), not to `Task.Run` sprinkles.
- **R-ASY-05** — Every spawned `Task`'s exceptions MUST be observed: awaited, or routed through logging in a supervised background loop. Silently dropped tasks are lost errors.
- **R-ASY-06** — Every external call accepts a `CancellationToken` and enforces a timeout. Every retry has a bounded budget and jittered backoff. Both are configuration, not literals.
- **R-ASY-07** — Cancellation safety is documented on any async operation that is not cancel-safe. Composing non-cancel-safe work with cancellation tokens is a data-loss bug.
- **R-ASY-08** — Shutdown is graceful and coordinated by `app` via `IHostApplicationLifetime` and hosted-service stopping hooks.

---

## 10. Testing

Testing is not a phase; it is the specification made executable. In an agentic workflow it is the *only* reliable signal that generated code does what was asked.

### 10.1 Non-negotiables

- **R-TST-01** — Tests are written **before** the implementation. Red first — a test that has never failed has never been shown to test anything.
- **R-TST-02** — Every public function, every enum variant with behaviour, and every error path MUST have at least one test. Happy path alone is not coverage.
- **R-TST-03** — Every specified behaviour has a test citing its spec ID (R-SDD-02). Every fixed bug gains a regression test that fails before the fix.
- **R-TST-04** — A test MUST be able to fail for exactly one reason. Multiple unrelated assertion groups in one test means multiple tests.
- **R-TST-05** — Tests MUST be deterministic. No wall-clock dependence, no network, no random seeds without pinning, no reliance on execution order, no `Thread.Sleep`/`Task.Delay` for synchronization. A flaky test is deleted or fixed within one working day — never skipped and forgotten (R-AGT-03).
- **R-TST-06** — Tests run in parallel by default. Disabling xUnit parallelism globally is a defect in the tests, not a configuration need.
- **R-TST-07** — Assert on values and behaviour, never on log output or on private internals.
- **R-TST-08** — No conditional logic in tests. An `if` in a test means it is two tests, and the branch that never runs is a lie.
- **R-TST-09** — Throwing helpers are fine in tests. Prefer `Assert.Throws<T>` with the exact expected exception type so a failure names itself.

### 10.2 The pyramid

| Tier | Location | Scope | Speed | Share |
| --- | --- | --- | --- | --- |
| Unit | `tests/Domain.Tests/`, `tests/Application.Tests/` | one function/type, no I/O, in-memory fakes at ports | µs–ms | ~70% |
| Contract | shared contract suite run against every port implementation | every impl of a port, same suite | ms | as needed |
| Integration | `tests/Api.IntegrationTests/` | public API via `WebApplicationFactory`, fakes at ports | ms–s | ~25% |
| End-to-end | `tests/Api.IntegrationTests/` (tagged) | real adapters via Testcontainers | s | ~5% |

- **R-TST-10** — Unit tests exercise one type/function through its public API. Integration tests go through the HTTP/public entry points and thereby prove the API is usable.
- **R-TST-11** — Unit tests MUST NOT touch the network, filesystem, database, clock, or RNG. If a unit test needs any of those, either the code violates R-SEP-02 or the test is misclassified.
- **R-TST-12** — The domain project is tested with **zero** doubles. It is pure; there is nothing to fake. If a domain test needs a mock, the domain is impure — fix the domain.
- **R-TST-13** — Application handlers are tested with in-memory fakes (R-TRT-20), not with a database.
- **R-TST-14** — **Contract tests are mandatory for every port with more than one implementation.** Write the suite once, generically over the interface, and run it against every implementation including the fake. Without this, your fake and your production adapter drift and your fast tests become fiction.
  ```csharp
  // tests/Application.Tests/Contracts/OrderRepositoryContract.cs
  public static async Task VerifyContract(IOrderRepository repo)
  {
      // insert → find → returns the same order
      // find on absent id → throws OrderNotFoundException, not null
      // insert duplicate id → conflict
  }
  ```
  Then in the Postgres adapter's tests: `await OrderRepositoryContract.VerifyContract(new PostgresOrderRepository(pool));`
- **R-TST-15** — End-to-end tests use real dependencies via Testcontainers, never a shared or developer-local database. They are few and cover only critical paths.

### 10.3 Coverage and rigour

- **R-TST-16** — Line coverage ≥ **85%** solution-wide and ≥ **95%** in `src/Domain`, measured by coverlet. CI fails below threshold. Coverage is a floor for spotting untested regions, not a goal — 100% coverage with weak assertions is worthless.
- **R-TST-17** — Branch/error-path coverage is explicit: every exception a function can throw for specified reasons has a test that produces it.
- **R-TST-18** — Pure functions with algebraic properties (round-trips, invariants, idempotency, ordering) MUST have property tests (e.g. CsCheck/FsCheck). Serialization round-trips MUST be property-tested. Failing seeds are committed as regression cases.
- **R-TST-19** — Every parser or any code handling untrusted input MUST have fuzz-style testing (e.g. SharpFuzz) or systematic adversarial property suites. The corpus/cases are committed.
- **R-TST-20** — Mutation testing (Stryker.NET) SHOULD run on `src/Domain` in CI on a schedule. Surviving mutants indicate assertions that assert nothing — this is the check that coverage cannot give you.
- **R-TST-21** — XML documentation on public API items is kept accurate; where behaviour is subtle, a unit test doubles as the executable example. Documentation that cannot be trusted is documentation that is already wrong.

### 10.4 Structure and readability

- **R-TST-22** — Test names describe behaviour and outcome, not the method under test: `ReturnsInsufficientFunds_WhenBalanceBelowAmount`. Never `Test1`.
- **R-TST-23** — Every test has visible Arrange / Act / Assert structure, separated by blank lines. Three sections, in that order.
- **R-TST-24** — Build test data with builders or fixture helpers (Object Mother) that default every irrelevant field, so each test names only the values it depends on. Twenty-line object initializers in tests hide the one field that matters.
- **R-TST-25** — Shared test helpers live in a dedicated `TestSupport` project/folder referenced only by test projects. Never `public` helpers in production assemblies.
- **R-TST-26** — Test the observable contract, not the implementation. A refactor that changes no behaviour MUST NOT change a single test. If it does, the tests were over-specified (usually via mocks — see R-TRT-21).
- **R-TST-27** — Table-driven tests via xUnit `[Theory]` with `[InlineData]`/`[MemberData]` for input/expectation matrices. Each case is independently reported.
- **R-TST-28** — Assertion messages carry context; use fluent assertions sparingly and prefer precise expected-vs-actual ordering so diffs read correctly.
- **R-TST-29** — Snapshot/approval tests (e.g. Verify) for large structured output only. Reviewing a snapshot diff is mandatory — accepting snapshots blindly converts tests into a changelog.
- **R-TST-30** — Test code is production code. It is reviewed, formatted, linted, refactored, and held to the naming rules of §2. Duplication in tests is tolerated only where it improves local clarity.

---

## 11. Documentation

- **R-DOC-01** — Every public item has XML documentation comments. `GenerateDocumentationFile=true` (with warnings-as-errors on missing docs) in every library project.
- **R-DOC-02** — Doc comments open with a `<summary>` in the third person: "Returns the settled total." Not "This method will return...".
- **R-DOC-03** — Document `<exception>` for every thrown exception type, `<param>`/`<returns>` where non-obvious, and `<remarks>` for invariants and threading guarantees.
- **R-DOC-04** — Public API docs include a short usage `<example>` where the shape is non-obvious; the example MUST stay compilable against the public API.
- **R-DOC-05** — Comments explain *why*, never *what*. If code needs a comment to explain what it does, rename things until it doesn't.
- **R-DOC-06** — Each assembly carries a namespace/assembly-level doc file stating its responsibility and its layer.
- **R-DOC-07** — Architectural decisions are recorded as ADRs in `specs/adr/`, numbered, immutable once accepted, superseded rather than edited.
- **R-DOC-08** — `TODO`/`FIXME` MUST reference a tracked issue: `// TODO(#412): ...`. Untracked TODOs are forbidden.

---

## 12. Dependencies

- **R-DEP-01** — Every new dependency requires justification: maintenance status, licence, transitive weight, and `unsafe` footprint. BCL first; a 30-line helper beats a micro-package.
- **R-DEP-02** — `packages.lock.json` is committed, for libraries as well as hosts.
- **R-DEP-03** — `dotnet list package --vulnerable` and locked-mode restore (`--locked-mode`) and licence allow-list checks run in CI.
- **R-DEP-04** — Feature switches are additive and non-breaking. No mutually exclusive MSBuild conditions that break restore.
- **R-DEP-05** — Domain-layer dependencies are held to a stricter bar (R-DIR-02): no ASP.NET, EF Core, or any I/O/framework package.

---

## 13. Tooling gates

- **R-TOOL-01** — The .NET SDK is pinned in `global.json` with the exact version and required workloads.
- **R-TOOL-02** — Workspace-wide analyzer and formatting configuration lives in `Directory.Build.props` and `.editorconfig` and is inherited by every project. Baseline:
  ```xml
  <!-- Directory.Build.props — shared analyzer configuration -->
  <PropertyGroup>
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
    <EnforceCodeStyleInBuild>true</EnforceCodeStyleInBuild>
    <AnalysisLevel>latest</AnalysisLevel>
    <EnableNETAnalyzers>true</EnableNETAnalyzers>
  </PropertyGroup>
  ```
  Additional analyzer packages (e.g. StyleCop.Analyzers, SonarAnalyzer.CSharp) are added only with human approval (R-AGT-02).
- **R-TOOL-03** — Analyzer suppressions are narrow and justified. `#pragma warning disable` at project scope is forbidden; at member scope it requires a reason:
  ```csharp
  #pragma warning disable CA1031 // generated interop shim, see ADR-NNNN
  ```
- **R-TOOL-04** — The following MUST pass before any commit is considered complete (provide as a single `build.ps1 ci` / MSBuild target so there is exactly one command and no divergence between local and CI):
  ```
  dotnet format --verify-no-changes
  dotnet build --no-restore -warnaserror
  dotnet test --no-build --collect:"XPlat Code Coverage" --settings coverlet.runsettings
  dotnet list package --vulnerable
  ```
  Coverage threshold enforcement (≥85% line, ≥95% Domain) is part of the same target (R-TST-16).
- **R-TOOL-05** — CI additionally runs `dotnet test -c Release`, security scanning (`dotnet list package --vulnerable`), and unused-package detection.
- **R-TOOL-06** — CI is not advisory. A red gate blocks merge. Never merge with a disabled or skipped gate.

---

## 14. Definition of Done

A unit of work is complete only when **every** box is checked, verified by observed command output rather than assumption (R-AGT-07).

- [ ] A specification exists for the behaviour, and is current (§1)
- [ ] Tests were written first and observed to fail (R-TST-01)
- [ ] Every specified behaviour is covered and traceable to its spec ID (R-SDD-02)
- [ ] Error paths, edge cases, and boundary values are tested (R-TST-17)
- [ ] Contract tests pass for every implementation of every touched port (R-TST-14)
- [ ] Layering holds: `Domain` is pure, composition only in `Web.Api` (R-DIR-01, R-SEP-02)
- [ ] Every new process-boundary dependency sits behind a port with a fake (R-TRT-01, R-TRT-20)
- [ ] No new interface with a single implementation and no prospect of a second (R-TRT-06)
- [ ] No swallowed exceptions, no throw-for-control-flow, no `Console.WriteLine`/`Debug.WriteLine` in production paths (§6)
- [ ] Domain identifiers are value objects; invalid states are unrepresentable (§7)
- [ ] Database table schemas have a domain-independent primary key named `id` (R-DB-01)
- [ ] Public items documented with XML docs, including `<exception>` tags (§11)
- [ ] `dotnet format` / `dotnet build` / `dotnet test` passes clean, no new warnings, no new suppressions (R-TOOL-04)
- [ ] Coverage thresholds met (R-TST-16)
- [ ] Deviations recorded with rule IDs; `MUST` deviations have an ADR (§0.2)
- [ ] Spec, ADRs, and glossary updated in the same commit (R-SDD-05)

---

## 15. Amendment

This constitution is version-controlled and amended by pull request with an accompanying ADR stating the problem, the proposed rule change, and the migration path for existing code. Rules are added with new IDs; retired rules are marked `RETIRED` in place. Agents MUST NOT amend this document without explicit human authorization and an accompanying ADR.
