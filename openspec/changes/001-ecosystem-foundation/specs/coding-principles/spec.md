# coding-principles

## ADDED Requirements

### Requirement: These principles apply where the surface is ours to design

`ecosystem-architecture` requires the binding and wrapper layers to speak the
projected HERE types, in the idiom of their platform. Those types are dictated
upstream: HERE's native APIs take bare `double` coordinates and deliver results
through listeners and callbacks rather than `Task`.

Every principle below that constrains **the shape of a public surface** —
value objects over primitives, complete cancellable asynchrony, segregated
interfaces — SHALL therefore apply to the layers that own their surface:
`Common`, `Abstractions` and above, including `Standard`, `Rest`, `Navigation`,
`Presentation`, `Blazor`, `Forms` and `Maui`.

`Bindings.*` and the wrappers `Android`, `iOS` and `Js` SHALL be exempt from those
three, and from nothing else. Faithfulness to the projected API is their purpose;
imposing our shape on them would make them a second abstraction layer and destroy
the reason they are standalone products.

Every other principle — the dependency rule, constructor injection, analysers,
structured logging, English, test-first development, no lazy mechanism — SHALL apply
to **every** package without exception.

#### Scenario: a wrapper may expose a projected primitive
- **GIVEN** the HERE Android SDK exposing `GeoCoordinates(double, double)`
- **WHEN** `Here.Sdk.Android` projects it
- **THEN** the bare doubles are accepted, and the value-object rule does not fail the build

#### Scenario: an adapter may not
- **GIVEN** `Standard.Android` translating that call to the contract
- **WHEN** it exposes a public method taking a bare `double` latitude
- **THEN** the analyser fails the build, because this layer owns its surface

### Requirement: Clean Architecture, with the dependency rule enforced executably

Source dependencies SHALL point inward only. Contracts and domain types SHALL NOT
reference infrastructure, platform, or user-interface assemblies.

The rule SHALL be enforced by a check that inspects the compiled assemblies and
fails the build on any outward reference — not by review, and not by convention.
A rule that is only written is not a rule.

#### Scenario: an inward layer cannot reach outward
- **GIVEN** a contract assembly that gains a reference to an HTTP client
- **WHEN** the architecture check runs
- **THEN** the build fails and names the offending reference

#### Scenario: the check is part of every build
- **GIVEN** any repository in the ecosystem
- **WHEN** `./scripts/build.sh` runs
- **THEN** the architecture check runs as part of it, not as a separate optional step

### Requirement: Dependencies are injected through constructors only

Every collaborator SHALL be received through the constructor and held behind an
interface. Singletons, static mutable state and the service locator pattern SHALL
NOT be used.

Each package SHALL expose exactly one registration extension —
`AddHereSdk…(this IServiceCollection, …)` — as its composition entry point.

#### Scenario: no static mutable state
- **GIVEN** the analyser set applied by `Rinzler78.Build`
- **WHEN** a static mutable field is introduced in a library
- **THEN** the build fails

#### Scenario: a consumer can substitute any collaborator
- **GIVEN** a consumer that registers `Abstractions.Testing` instead of a provider
- **WHEN** the application runs
- **THEN** every dependency resolves and no HERE credential is required

### Requirement: SOLID is enforced by analysers, not by intention

`Rinzler78.Build` SHALL enable the .NET analysers, the code-style analysers and
`TreatWarningsAsErrors` on every project — **libraries and demonstration
applications alike**. A sample shipped with warnings undermines exactly what the
sample exists to demonstrate. Interfaces SHALL be segregated by
capability rather than aggregated per platform — the split between `Abstractions`,
`.Navigation` and `.Offline` being the ecosystem-level application of that rule.

#### Scenario: a warning cannot be ignored
- **GIVEN** a library project
- **WHEN** any analyser raises a warning
- **THEN** the build fails

### Requirement: Value objects carry their invariants

Domain values — coordinates, distances, durations, bearings, identifiers — SHALL be
represented by types that validate at construction and are immutable thereafter.
Primitive obsession SHALL be avoided: no public API SHALL accept a bare `double`
where a coordinate or a distance is meant.

#### Scenario: an invalid coordinate cannot be constructed
- **GIVEN** a latitude of 91 degrees
- **WHEN** the coordinate type is constructed
- **THEN** it throws, and no invalid instance can exist

### Requirement: Asynchrony is complete and cancellable

Every operation performing input or output SHALL be asynchronous, SHALL accept a
`CancellationToken`, and SHALL honour it. `async void` SHALL NOT be used outside
event handlers. Blocking on asynchronous work — `.Result`, `.Wait()`,
`GetAwaiter().GetResult()` — SHALL NOT appear in library code.

#### Scenario: a long search is cancelled
- **GIVEN** a search in flight
- **WHEN** the caller cancels its token
- **THEN** the operation stops and throws `OperationCanceledException`

### Requirement: Logging is structured and free of secrets

Libraries SHALL log through `Microsoft.Extensions.Logging` with structured message
templates and named placeholders. String interpolation into log messages SHALL NOT
be used. Credentials, tokens and full request URLs containing keys SHALL NOT be
logged at any level.

#### Scenario: a key never reaches the log
- **GIVEN** a request carrying an API key
- **WHEN** the request is logged at debug level
- **THEN** the key is redacted in the emitted message

### Requirement: The whole ecosystem is written in English

Source code, identifiers, comments, XML documentation, commit messages, pull request
descriptions, issues, ADRs and specifications SHALL be written in English (en-US).
`cspell` with `language: en` SHALL run as a blocking pre-commit hook over the whole
tree — tests, scripts, configuration and documentation included.

#### Scenario: a French comment is refused
- **GIVEN** a comment written in French
- **WHEN** the developer commits
- **THEN** the spelling hook fails and the commit is refused

### Requirement: Test-driven development, with the order visible in history

Every feature and every bug fix SHALL be developed test-first. The failing test and
the implementation that makes it pass SHALL be distinguishable in the **pull
request's commit list**, which is where the order survives.

`release-flow` requires linear history on the long-lived branches, so merges are
squashed and intermediate commits do not reach `develop` or `master`. Verification
SHALL therefore run on the pull request's own commits, before the squash, and its
verdict SHALL be recorded as a status check — the check outliving the commits it
inspected.

A bug fix SHALL be accompanied by a regression test that fails without the fix.

#### Scenario: the test-first order is verified before the squash
- **GIVEN** a pull request labelled `feat` or `fix`
- **WHEN** the check walks its commit list
- **THEN** at least one commit adds a failing test before any commit that changes
  production source
- **AND** the verdict is posted as a status check that survives the squash merge

#### Scenario: a fix without a regression test is refused
- **GIVEN** a pull request labelled `fix`
- **WHEN** the check reverts only the non-test files of the diff and runs the tests
  added or modified by that pull request
- **THEN** at least one of them fails; otherwise the pull request is blocked

### Requirement: No lazy mechanism without a diagnosis and an explicit decision

Suppression comments, ignore lists, disabled hooks, commented-out tests and renames
that dodge a check SHALL NOT be introduced. A blocking check signals a root cause to
fix, never an obstacle to bypass.

Where a suppression is genuinely unavoidable, it SHALL carry a justification naming
the root cause and an ADR or issue tracking its removal. A suppression without a
justification SHALL fail the build.

#### Scenario: an unjustified suppression fails
- **GIVEN** a `#pragma warning disable` with no justification comment
- **WHEN** the build runs
- **THEN** it fails, naming the file and the suppressed rule
