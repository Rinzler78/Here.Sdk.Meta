# ecosystem-architecture

## ADDED Requirements

### Requirement: A native wrapper depends only on its own binding

A package in the wrapper layer (`Here.Sdk.Android`, `Here.Sdk.iOS`, `Here.Sdk.Js`)
SHALL declare exactly one `PackageReference` inside the ecosystem: its own binding
package. It SHALL NOT reference `Common`, `Abstractions`, `Core`, or any other
ecosystem package.

Its public surface SHALL speak the projected HERE types, in the idiom of its
platform, so that a consumer targeting a single platform obtains the full power of
the native SDK with no abstraction cost.

#### Scenario: Android wrapper stays standalone
- **GIVEN** `Here.Sdk.Android` is built
- **WHEN** its resolved dependency graph is inspected
- **THEN** the only ecosystem package present is `Here.Sdk.Bindings.Android`
- **AND** a CI check fails the build if any other ecosystem package appears

#### Scenario: a single-platform consumer pays no abstraction cost
- **GIVEN** an application that targets Android only
- **WHEN** it references `Here.Sdk.Android` alone
- **THEN** it compiles and runs without `Here.Sdk.Abstractions` being restored

### Requirement: Standardisation is a layer above, with one entry point

Standardisation SHALL be implemented as adapter packages placed above the wrappers
— `Standard.Android`, `Standard.iOS`, `Standard.Js` — each translating one wrapper
to the common contract.

A single multi-targeted façade, `Here.Sdk.Standard`, SHALL be the only package an
integrator references to obtain platform abstraction **on the native targets**. Its
dependencies SHALL be conditioned by target framework so that no condition is
required in the consuming project:

| Consumer target framework | Resolved |
|---|---|
| `net8/9/10.0-android`, `monoandroid12.0` | `Standard.Android` — full capability, offline included |
| `net8/9/10.0-ios`, `xamarinios10` | `Standard.iOS` — full capability, offline included |
| `netstandard2.0`, `net8/9/10.0` | `Rest` and `Navigation` — services and guidance |

Two consequences SHALL be documented in the façade's `README.md`, because an
integrator discovers them otherwise at run time:

- **`Standard.Js` is deliberately outside the façade.** Blazor WebAssembly targets
  `net10.0` exactly like a console, a WPF host or a server application; the target
  framework cannot distinguish them, so no conditional dependency can select it.
  `Here.Sdk.Blazor` references `Standard.Js` explicitly, and an application wanting
  JavaScript rendering references `Here.Sdk.Blazor`.
- **The portable fallback carries no map renderer.** It provides services and
  guidance only. A Windows or server application that needs a map SHALL additionally
  reference `Here.Sdk.Blazor` and host it in a `BlazorWebView`.

#### Scenario: the portable fallback is honest about the map
- **GIVEN** a WPF application referencing only `Rinzler78.Here.Sdk.Standard`
- **WHEN** it resolves the service collection
- **THEN** search, routing, traffic and guidance resolve
- **AND** no map view type is registered, the README naming `Here.Sdk.Blazor` as the
  renderer to add

#### Scenario: one reference, no condition
- **GIVEN** a MAUI application targeting Android, iOS, Windows and Mac Catalyst
- **WHEN** it declares `<PackageReference Include="Rinzler78.Here.Sdk.Standard" />`
  with no `Condition` attribute
- **THEN** restore resolves `Standard.Android` on the Android head, `Standard.iOS`
  on the iOS head, and the portable REST implementation on the others
- **AND** a single `AddHereSdk()` registers the correct provider on each head

### Requirement: Capability asymmetry is carried by the type system

Capabilities that a target cannot honour SHALL be absent from the contracts that
target implements, never present and failing at runtime.

| Contract | Content | Implemented by |
|---|---|---|
| `Abstractions` | map, search, geocoding, routing, traffic | `Standard.Android`, `Standard.iOS`, `Standard.Js`, and `Rest` for the portable fallback |
| `Abstractions.Navigation` | guidance, manoeuvres, deviation, rerouting, voice | `Standard.Android` and `Standard.iOS` through the native SDK; `Here.Sdk.Navigation` — the REST guidance engine — everywhere else |
| `Abstractions.Offline` | offline maps, HERE positioning, lane guidance | `Standard.Android` and `Standard.iOS` only |

Two implementations of `Abstractions.Navigation` therefore coexist, and that is the
point: one interface, a native engine on one side and a REST engine on the other.

#### Scenario: the web cannot call what it cannot do
- **GIVEN** a Blazor application referencing the ecosystem
- **WHEN** the developer looks for offline map APIs in completion
- **THEN** no such type is visible, because `Abstractions.Offline` is not in its
  dependency graph

### Requirement: Repositories reference each other only through published packages

An ecosystem repository SHALL NOT contain a `ProjectReference` to a project owned
by another repository. Cross-repository dependencies SHALL be expressed as
`PackageReference` to nuget.org.

During extraction of a contract from existing implementations, prerelease versions
(`1.0.0-alpha.N`) SHALL be used to close the loop. Nothing SHALL be published in
stable form, nor any repository made public, before extraction is complete.

#### Scenario: no project reference crosses a repository boundary
- **GIVEN** any repository in the ecosystem
- **WHEN** its project files are scanned for `ProjectReference`
- **THEN** every target resolves inside the same repository

### Requirement: The eight layers are enumerated, and every package belongs to one

The architecture SHALL be described as eight layers, and every published package
SHALL map to exactly one of them. The map governs what an application and a package
may reference: nothing may depend on a layer above its own. It does not group
demonstrations, which are counted per package in `demonstration`.

| # | Layer | Packages |
|---|---|---|
| 1 | Binding | `Bindings.Android`, `Bindings.iOS`, `Bindings.Js` |
| 2 | Wrapper | `Android`, `iOS`, `Js` |
| 3 | Contracts | `Abstractions`, `.Navigation`, `.Offline`, `.Testing` |
| 4 | Portable services | `Common`, `Core`, `Rest`, `Navigation` |
| 5 | Presentation | `Presentation`, `.Mvp`, `.Mvvm`, `.Android`, `.iOS` |
| 6 | Standardisation adapters | `Standard.Android`, `Standard.iOS`, `Standard.Js` |
| 7 | Façade | `Standard` |
| 8 | Cross-platform heads | `Blazor`, `Forms`, `Maui` |

`Rinzler78.Build` and `Rinzler78.Templates` belong to no layer: they are toolchain
artefacts rather than ecosystem packages. Their demonstration exemption is declared
in `demonstration`, which remains the single authority on what needs a sample.

#### Scenario: a new package cannot exist outside the map
- **GIVEN** a proposal adding a package
- **WHEN** the ecosystem audit runs
- **THEN** it fails unless the package is assigned to one of the eight layers

### Requirement: The cockpit obtains sources through submodules, and follows the tips

Every repository of the ecosystem SHALL appear in `Meta` as a git submodule under
`repos/<repository name>`, tracking `develop`.

The second cockpit obligation — building the whole graph from sources, each
repository against the others' `develop` — presupposes that `Meta` has those
sources. Nothing said how it obtained them, and a nightly job that cannot check out
the graph cannot build it.

The commit a submodule pins SHALL be treated as a bootstrap default, not as a
dependency. The nightly integration build SHALL run `git submodule update --remote`
first, so that it builds the **tips** of `develop` and not the commits `Meta`
happens to have recorded. A pinned commit that is never refreshed would turn the
one job designed to catch drift into a job that reproduces it.

This SHALL NOT weaken the rule that repositories reference each other only through
published packages. A submodule gives `Meta` a working copy; it creates no
`ProjectReference`, and no repository gains a source dependency on another. The
from-sources nightly is the deliberate exception, and it exists precisely to see
what the package boundary hides — an API break that no repository's own continuous
integration can observe.

#### Scenario: the nightly builds the tips, not the pins
- **GIVEN** a submodule pinned to a commit three days old
- **WHEN** the nightly integration job runs
- **THEN** it refreshes every submodule to the tip of `develop` before building
- **AND** the recorded pins are irrelevant to the result

#### Scenario: a submodule does not become a build dependency
- **GIVEN** a repository whose sources are present under `repos/`
- **WHEN** any project file in the ecosystem is inspected
- **THEN** it references its dependencies as `PackageReference`, never as a
  `ProjectReference` reaching into a sibling submodule

### Requirement: Meta is a cockpit with five obligations, and it is public

`Meta` SHALL be public and SHALL publish no package. Beyond the ecosystem-level
OpenSpec tree and the realisation order it documents, it SHALL carry five
obligations, and a cockpit missing any of them is documentation rather than
orchestration:

1. **The release cascade** — topological, wave by wave, as specified in `release-flow`.
2. **Nightly integration CI** — building the whole graph **from sources**, each
   repository against the others' `develop`, which is the only mechanism that
   catches an API break before it reaches nuget.org. No repository's own CI can see it.
3. **The aggregated board and the demonstration gallery**, published to GitHub Pages.
4. **The ecosystem audit** — one nightly job, non-blocking, reporting into issues.
   Four names circulated for it during design; there is exactly one, and it checks:
   - **catalogue containment** — every published package appears in the topology
     table, and no package exists outside it;
   - **demonstration coverage** — every package has its sample, exemptions aside;
   - **task-box truth** — no change whose boxes contradict its merged pull requests,
     and no change fully done yet neither promoted nor archived;
   - **ADR integrity** — no committed artefact contradicting an accepted ADR without
     a superseding one, and no deny rule naming a branch the flow never creates;
   - **harness drift** — which repositories lag the published template version.
5. **Ecosystem-level ADRs**, including one that **argues the poly-repository choice**
   — disjoint lifecycles, incompatible runner matrices, independent per-package
   publication. Without that ADR the cockpit reads as a cure for a self-inflicted
   disease.

`Meta` SHALL NOT hold templates, and SHALL NOT push commits into sibling
repositories other than the bump pull requests of the cascade.

#### Scenario: an API break is caught before publication
- **GIVEN** a breaking change merged to `develop` in `Common`
- **WHEN** the nightly integration job builds `Abstractions` against that `develop`
- **THEN** the job fails and opens an issue in `Meta`, before any release exists

#### Scenario: the cockpit is readable by a prospect
- **GIVEN** the public `Meta` repository
- **WHEN** a visitor opens it
- **THEN** the dependency graph, the realisation order, the gallery and the ADRs
  are readable without any account
