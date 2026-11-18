# quality

## ADDED Requirements

### Requirement: These gates apply to repositories that carry code

Coverage floors, mutation scores, trimming contracts, API documentation,
API-coverage tests and the committed public API baseline of `sdk-updates` all
presuppose a compiled assembly with a public surface. Three repositories have none,
and SHALL be exempt from those six gates rather than made to fail requirements they
cannot satisfy:

| Repository | Why |
|---|---|
| `Here.Sdk.Meta` | Publishes no package. Its only code is the NUKE orchestrator, verified by the cascade actually propagating, and by its own unit tests where it has logic. |
| `Here.Sdk.Build` | Ships a props/targets package declaring the domain's layer vocabulary and non-affiliation fragment. Its verification is that a project violating either cannot build, exercised by a scratch consumer. |
| `Rinzler78.Toolkit` | Ships a props/targets package and a `dotnet new` template pack, neither of which produces an assembly with a public API. Its verification is that it regenerates its own harness byte-identically, and that the repositories import it and behave identically in IDE and CI. |

Every other gate — analysers, warnings as errors, English, pre-commit, linear
history, specification validation — SHALL apply to **all** repositories, these two
included. Exemption is from the gates that require an assembly, and from nothing
else.

#### Scenario: the cockpit is not failed for having no library
- **GIVEN** the `Meta` repository, which publishes no package
- **WHEN** CI runs
- **THEN** no coverage, mutation, trimming or DocFX gate is evaluated
- **AND** analysers, spelling, pre-commit and specification validation still are

#### Scenario: the toolkit is verified by regeneration
- **GIVEN** the `Rinzler78.Toolkit` repository
- **WHEN** CI runs
- **THEN** its harness is regenerated from its own template and compared
- **AND** a divergence fails the build

### Requirement: Each project declares its own trimming and AOT contract

Partial trimming is the default on Blazor WebAssembly and on iOS, and trims only
assemblies that opted in. `IsTrimmable` is therefore not a badge: it is the switch
that makes a library shrink in every integrator's Release build.

`Rinzler78.Build` SHALL provide the vocabulary as `<RinzlerTrimContract>` with values
`Committed`, `Analyzed` and `OutOfScope`. Each project SHALL choose its value,
justified by an ADR in its own repository.

- `Committed` — the project marked `IsTrimmable`, and `IsAotCompatible` on the target
  frameworks that support it (`net7.0` and later: on `net6.0` that marking switches the
  trim checks off), a dedicated trimming test application per library.
- `Analyzed` — analyser enabled, no `IsTrimmable` marking,
  `[RequiresUnreferencedCode]` propagated to the public API.
- `OutOfScope` — the project is not a trimming subject at all.

The value is declared **once per project** and applies only to the target frameworks
where trimming exists — `net6.0` and later. The `netstandard2.0` and Xamarin slices
of a multi-targeted project are out of scope by construction, whatever the declared
value; a project SHALL NOT be forced to declare `OutOfScope` because one of its
frameworks predates trimming. `Rinzler78.Build` SHALL apply the contract through
`$([MSBuild]::IsTargetFrameworkCompatible('$(TargetFramework)', 'net6.0'))`.

Native bindings SHALL NOT be marked trimmable: their JNI and Objective-C projections
are reflection-driven, and marking them would invite complacent suppressions.

#### Scenario: an integrator is warned rather than crashed
- **GIVEN** an application enabling `PublishTrimmed`
- **WHEN** it calls a binding API annotated `[RequiresUnreferencedCode]`
- **THEN** the build emits a warning naming the API, instead of failing at runtime

### Requirement: The trimming benefit is measured, not declared

Every `Committed` library SHALL record, in its `README.md`, the size in bytes of the
library and of the managed payload of its trimming test application, published trimmed
in `TrimMode` partial (the mode Blazor WebAssembly and iOS apply; the trimmer's own
default, `full`, trims every assembly and shows marking no benefit) with and without
the library marked. The sizes are recorded by a local `--write`: CI holds a token that
cannot commit. The promotion pull request into `master` SHALL re-measure, compile the
application ahead of time, post the sizes on the pull request, and fail when the marked
library exceeds the record by more than 5 %, when marking trims nothing, or when the
record is absent or unreadable. The payload is recorded as information and does not
gate: five percent of a framework is a hundred times a library. A WASM payload or an
IPA size belongs to the head that builds one.

#### Scenario: the README carries a number
- **GIVEN** a `Committed` library and its trimming test application
- **WHEN** `scripts/trimming.sh --write` runs
- **THEN** the measured sizes are written to the README block, and nothing around it changes

#### Scenario: a size regression fails the promotion
- **GIVEN** a promotion pull request into `master` whose marked library is more than 5 % over the README's record
- **WHEN** the trimming workflow runs
- **THEN** it fails, and the measured sizes are posted on the pull request beside the recorded ones


### Requirement: Coverage gates the diff, not the file

`Rinzler78.Build` SHALL provide `<RinzlerCoverageContract>` with three values, mirroring
the trimming contract, and each project SHALL declare one:

| Value | Floor | Applies to |
|---|---|---|
| `Enforced` | 90 % lines, 90 % branches | portable libraries, adapters, presentation |
| `Reduced` | a floor declared in the project, justified by an ADR | wrappers, whose surface is largely delegation |
| `Exempt` | none | binding projections, whose code is generated or reflective and cannot be meaningfully covered |

A `Reduced` floor SHALL be enforced exactly as an `Enforced` one, at the value the
project declares: the contract lowers the bar, it never removes the gate.

An `Exempt` project SHALL still be built, analysed, documented and reviewed;
exemption removes the coverage floor and the diff-coverage gate, and nothing else.
Mutation testing does not apply to it either — a projection produces surviving
mutants with no meaning — so its guarantee comes from the binding's API-coverage
test, which reflects over the bound assembly and fails when a public upstream type
is not projected. A project declaring
`Reduced` or `Exempt` without an ADR SHALL fail the build, a test project excepted since
the templates declare it `Exempt` by construction, and a `Reduced` floor
SHALL NOT be lowered without a new ADR superseding the previous one.

Without this mechanism a uniform 90 % floor is unreachable on the binding
projections, and an unreachable gate is a gate nobody enforces.

There SHALL be **no per-file threshold**: it forces tests on code with no behaviour — records,
data transfer objects, binding projections — which detect nothing and discredit the
rest of the suite.

Diff coverage SHALL be enforced at 95 % or above on `Enforced` and `Reduced`
projects: new untested code is refused without punishing trivial existing code. It
SHALL NOT apply to `Exempt` projects, where a large generated diff could never reach
it.

#### Scenario: a trivial record does not demand a test
- **GIVEN** a value object with only auto-properties
- **WHEN** coverage is computed
- **THEN** no gate requires a test that would assert nothing

### Requirement: Mutation score is the quality signal

Stryker SHALL run on the diff for every pull request and in full nightly on the
portable libraries, with the score published as a badge.

Thresholds SHALL be: **break below 70 %**, target 85 %. The diff score SHALL NOT
regress below the current full-run score of the project being modified.

It SHALL NOT run on the binding packages, whose projections would produce surviving
mutants with no meaning.

The nightly full run SHALL record each project's score and the badge on an orphan
`badges` branch of the repository, written without a pull request since no
integration branch ever merges it; the pull-request run SHALL read that branch as the
floor of the diff score. An origin that cannot be reached SHALL be an error, never a
first run, and a repository with no published score has no floor.

#### Scenario: a patch that lowers the score is refused
- **GIVEN** a project whose last full-run score is 90 %
- **WHEN** a pull request changes it and its diff scores 85 %
- **THEN** the pull request fails although 85 % is above the break threshold

#### Scenario: a test that traverses without verifying is caught
- **GIVEN** a test that calls a method and asserts nothing meaningful
- **WHEN** Stryker mutates that method
- **THEN** the mutant survives and the pull request fails the threshold

### Requirement: Bindings are guaranteed by an API-coverage test

Coverage and mutation do not apply to a projection, so a binding's guarantee SHALL
come from a different mechanism: a test that **reflects over the bound native
assembly** and fails when a public upstream type or member is not projected into C#.

The test SHALL carry an explicit, justified exclusion list — upstream members
deliberately not projected — and an unlisted omission SHALL fail the build. Adding
an entry SHALL require a justification naming why the member is out of scope.

This is what makes the pinned-version upgrade of `sdk-updates` safe: a new upstream
release that adds public surface fails the test until the projection catches up.

#### Scenario: an unprojected upstream type fails the build
- **GIVEN** the HERE Android AAR exposing a public class absent from the projection
- **WHEN** the API-coverage test runs
- **THEN** it fails, naming the type, unless the exclusion list justifies it

#### Scenario: an upstream upgrade surfaces its own gaps
- **GIVEN** a proposal raising the pinned AAR version
- **WHEN** CI runs the API-coverage test against the new artefact
- **THEN** every newly added public type is reported as unprojected

### Requirement: Every repository carries a README, and a check keeps it true

Every repository SHALL carry a `README.md`, written in the **initial commit** rather
than added later, and SHALL keep it true as the repository changes.

A README is the first thing a visitor sees and frequently the only thing. On an
ecosystem whose purpose is to demonstrate craft, an empty landing page cancels the
benefit of the code beneath it.

Existence SHALL NOT be the check, because a document that describes a command which
no longer exists is worse than an absent one: a reader acts on it. A blocking check
SHALL assert, at minimum:

- the required sections are present — what the repository is, how to run it, and
  under what licence;
- every script verb present in `scripts/` is named in the README;
- every script the README names exists.

The last two directions are both needed: they catch the two ways a README rots — a
capability added without being documented, and a capability removed while its
documentation survives.

For a repository that publishes a package, the `README.md` shipped inside the
`.nupkg` SHALL be the repository's own, so that the page on nuget.org cannot drift
from the page on GitHub.

#### Scenario: an undocumented verb fails the build
- **GIVEN** a new script added under `scripts/`
- **WHEN** the README check runs without the README mentioning it
- **THEN** the build fails, naming the verb

#### Scenario: a removed script leaves no lie behind
- **GIVEN** a README naming `scripts/deploy.sh`, which has been deleted
- **WHEN** the README check runs
- **THEN** the build fails, naming the script that does not exist

### Requirement: The public API is documented and the documentation is built

Every public type and member SHALL carry XML documentation. `CS1591` SHALL be an
error on all library projects, so an undocumented public member cannot merge.

Every repository publishing a package **with an API surface** SHALL build a DocFX
site from its XML documentation and publish it to GitHub Pages on release —
sixteen sites. `Meta` publishes no package, and `Rinzler78.Toolkit` ships a
props/targets package and a template pack, neither of which has an API surface to
document; both SHALL carry a written `README.md` instead. `Meta` SHALL link the
sixteen sites from its gallery.

#### Scenario: an undocumented public member blocks the merge
- **GIVEN** a new public method with no XML documentation comment
- **WHEN** the library project builds
- **THEN** `CS1591` is raised as an error and the pull request is blocked
