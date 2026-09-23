# Tasks — 001 Ecosystem foundation

Two orders govern this change. **Construction** builds the natives first, because an
abstraction is extracted from its implementations, not designed against
documentation. **Publication** is strictly topological, and nothing goes stable or
public before extraction is complete.

## Phase 0 — Archive and reset

- [x] 0.1 `git bundle --all` for each of the nine existing GitHub repositories.
      The local `Here.Sdk.Blazor` and `Here.Sdk.Premium.Meta` directories are not
      repositories and have no remote; the first holds an OpenSpec tree and a
      pre-commit configuration, the second is empty
- [x] 0.2 Export issues, pull requests and releases via `gh` for each repository
- [x] 0.3 Present the destruction inventory for explicit authorisation
- [x] 0.4 Delete the GitHub repositories — **irreversible**
- [x] 0.5 Empty the local `Here.Sdk.Meta` directory of everything except
      `openspec/changes/001-ecosystem-foundation/`, the seed of the new cockpit;
      remove `.git` and re-initialise. The four condemned proposals go with the rest.
- [x] 0.6 Verify no `Rinzler78.*` or `Here.Sdk.*` identifier is taken on nuget.org

## Phase 1 — Toolkit and cockpit

- [x] 1.1 `Rinzler78.Toolkit` — harness written by hand, domain-agnostic
      (skills pending, issue #1)
- [x] 1.2 `Rinzler78.Build` — props/targets, domain-agnostic: named target
      framework sets, analysers, `TreatWarningsAsErrors`, deterministic build,
      `ContinuousIntegrationBuild` following the environment, SourceLink, `.snupkg`
      symbols, XML documentation with `CS1591` as error, `<RinzlerTrimContract>`,
      `<RinzlerCoverageContract>`, and the mechanisms the domain declares against —
      layer vocabulary and validation, forbidden dependencies, required analysers,
      required description fragment. A project declaring no policy builds
- [x] 1.3 `Rinzler78.Templates` — `rinzler-lib`, `rinzler-binding`, `rinzler-app`,
      each expanding: the seventeen skills in `.agents/skills/` and
      `.claude/skills/`, `CLAUDE.md` and `AGENTS.md` with the agreement check,
      `.claude/settings.json` with the `PreToolUse` / `PostToolUse` / `Stop` hooks,
      `.pre-commit-config.yaml` with the four blocking checks per language —
      vulnerability scan, dependency freshness, linter and formatter, and `cspell`
      with `language: en` over the whole tree — `mise.toml` and the `setup-env`
      script with its non-mutating `--check` mode,
      `Directory.Packages.props` and lock files with `--locked-mode` in CI,
      a `README.md` with the required sections and the check that keeps it true,
      `.github/ISSUE_TEMPLATE/`, the thirteen labels, Release Please, the
      `lockfile-sync` workflow, the ruleset definition, DocFX, the script facade,
      and a `net10.0` xUnit v3 test project at `tests/<PackageId>.Tests` — the
      single test framework of the ecosystem, no project targeting a platform
      moniker — wired to `AwesomeAssertions`, `NSubstitute` and
      `NSubstitute.Analyzers.CSharp`. The three templates SHALL share one physical
      copy of the harness, each `.template.config` sourcing the shared directory,
      and SHALL declare their seed files in the template itself. Three gates —
      `pre-commit`, `pre-push`, `pre-merge-commit` — each running what it can afford,
      with change-scoped test selection on the commit gate. `Rinzler78.Build` is
      consumed as an **MSBuild project SDK**, pinned in `global.json`: a package
      reference cannot deliver a named target-framework set, because restore needs
      TargetFrameworks before the package's props exist
- [ ] 1.4 `Here.Sdk.Build` — the domain specialisation: the layer vocabulary of the
      eight-layer map, the forbidden dependencies of the inward layers, the required
      analysers, and the HERE non-affiliation fragment, all declared through the
      generic mechanism of `Rinzler78.Build`. Verified by a scratch consumer, since
      a props/targets package never imports itself
- [x] 1.5 Regenerate the Toolkit harness from its own template; assert byte identity
      for every harness file, and presence only for the files the template declares
      as seeds — the README, the entry points, the solution, the central versions,
      the dictionary and the scaffold, whose content is repository-specific by
      construction
- [ ] 1.6 Publish the three packages
- [ ] 1.7 Request the `Rinzler78.` prefix reservation. This does not wait on
      phase 1: `Rinzler78.CometBFT.Client` is already published under the same
      owner, so the reservation prerequisite is met today. The search index
      reports the prefix as unverified, meaning no reservation exists yet
- [ ] 1.8 `Meta` — generated from the template, made public
- [ ] 1.9 `Meta` — founding ADRs: poly-repository rationale, public visibility,
      templates generated not synchronised, frozen Xamarin toolchain, deferred Linux
      desktop, self-hosted runner isolation, deliberate absence of `CODEOWNERS`
- [ ] 1.10 `Meta` — every repository added as a submodule under `repos/<name>`
      tracking `develop`, and the nightly refreshing them with
      `git submodule update --remote` before building, so it builds the tips
- [ ] 1.11 `Meta` — NUKE orchestrator skeleton, aggregated board, gallery page,
      nightly integration job, ninety-day secret rotation reminder, and the HERE
      SDK release detection job reading `heremaps/here-sdk-examples` releases through
      the GitHub API against a committed cursor, opening a `needs-triage`/`chore`
      issue in the affected binding repository and linking — never parsing — the
      documentation release notes
- [ ] 1.12 `Meta` — separate detection for the JavaScript column, which versions
      independently on `js.api.here.com`, is absent from the examples repository and
      returns 404 on the npm registry
- [ ] 1.13 `Meta` — the **ecosystem audit**: catalogue containment, demonstration
      coverage, task-box truth, ADR integrity and dead deny rules, harness drift,
      and version literals in specifications — a requirement naming a concrete
      version that is not one of the two declared freezes is reported

## Phase 2 — Native columns, standing alone

- [ ] 2.1 Self-hosted macOS runner: Xcode 15.4 pinned, ephemeral, dedicated account
      with no keychain access, version verification job, fork-exclusion condition,
      `pull_request_target` ban check — **before any Xamarin build**
- [ ] 2.2 Self-hosted Windows runner, or `windows-2022` while it lives, for the
      Xamarin.Android chain; same verification and isolation rules
- [ ] 2.3 `Bindings.iOS` — pinned `.xcframework` version, C# projection, and the
      API-coverage test reflecting over the bound assembly with its justified
      exclusion list
- [ ] 2.4 `Bindings.iOS` — commit the Objective Sharpie output, `ApiDefinition.cs`
      and `StructsAndEnums.cs`, as the upstream baseline, so a version raise is a
      plain `git diff` before any hand binding
- [ ] 2.5 `Bindings.iOS` — sample: .NET 10 iOS, raw surface, no pattern
- [ ] 2.6 `iOS` — idiomatic wrapper, referencing its binding only
- [ ] 2.7 `iOS` — samples: Xamarin.iOS in MVC Cocoa, .NET 10 iOS in MVVM-C
- [ ] 2.8 `Bindings.Android` — pinned AAR version, C# projection, and its
      API-coverage test
- [ ] 2.9 `Bindings.Android` — commit the generator's `api.xml` as the upstream
      baseline; it is an intermediate artefact and is discarded unless committed
      deliberately
- [ ] 2.10 `Bindings.Android` — sample: .NET 10 Android, raw surface, no pattern
- [ ] 2.11 `Android` — idiomatic wrapper, referencing its binding only
- [ ] 2.12 `Android` — samples: Xamarin.Android in MVP, .NET 10 Android in MVVM/UDF

## Phase 3 — JavaScript spike, then extraction

- [ ] 3.1 Throwaway spike on the HERE Maps API for JavaScript — capability matrix
      only, never merged, no commit in any repository
- [ ] 3.2 `Common` — extracted: value objects validating at construction
- [ ] 3.3 `Common` — sample: Blazor WASM page, no HERE key required
- [ ] 3.4 `Abstractions` — extracted from all three capability sets:
      `Abstractions`, `.Navigation`, `.Offline`, `.Testing` — prerelease
- [ ] 3.5 `Abstractions` — sample: Blazor WASM driven by the in-memory provider,
      demonstrably running with no HERE credential
- [ ] 3.6 `Core` — extracted from both natives: credentials, cache, logging, resilience
- [ ] 3.7 `Core` — sample: Blazor Server, secrets held server-side, cache visible
- [ ] 3.8 Reconcile the native wrappers against the extracted contracts
- [ ] 3.9 Trimming test applications for every `Committed` library

## Phase 4 — Standardisation

- [ ] 4.1 `Standard.Android` and `Standard.iOS` — adapters
- [ ] 4.2 `Standard` — multi-targeted façade, conditional dependencies, `AddHereSdk()`,
      `README.md` stating the two consequences: `Standard.Js` outside the façade, and
      no map renderer in the portable fallback
- [ ] 4.3 Samples: `Standard.iOS` on .NET 10 iOS and `Standard.Android` on
      **Xamarin.Android**, sharing one **linked** business-logic file; CI asserts both
      projects resolve the identical path and that no copy exists under `samples/`

## Phase 5 — Portable implementations

- [ ] 5.1 `Rest` — Geocoding & Search v7, Routing v8, Traffic v7, Raster Tile v3
- [ ] 5.2 `Rest` — sample: Blazor Server
- [ ] 5.3 `Navigation` — REST guidance engine: progress along the polyline, manoeuvre
      advance, deviation with hysteresis, rerouting, voice
- [ ] 5.4 `Navigation` — recorded GPS traces replayed as pure unit tests
- [ ] 5.5 `Navigation` — sample: Blazor Server, guidance replayed on a trace
- [ ] 5.6 `Presentation` — neutral engine, immutable serialisable state, cancellable
      intents
- [ ] 5.7 `Presentation` — sample: Blazor WASM showing the engine state bare
- [ ] 5.8 `Presentation.Mvp`, `.Mvvm` — the two pattern adapters, the MVVM one
      carrying the diff and the key-stable collections
- [ ] 5.9 `Presentation.Android`, `.iOS` — UI-thread marshalling, lifecycle,
      rotation survival

## Phase 6 — JavaScript column

- [ ] 6.1 `Bindings.Js` — pinned Maps API for JavaScript version, C# interop, and
      its API-coverage test
- [ ] 6.2 `Bindings.Js` — sample: Blazor WASM, raw `H.Map`, no pattern
- [ ] 6.3 `Js` — idiomatic C# wrapper, referencing its binding only
- [ ] 6.4 `Js` — sample: Blazor WASM
- [ ] 6.5 `Standard.Js` — adapter to the contract
- [ ] 6.6 `Standard.Js` — sample: Blazor WASM driven by the contract
- [ ] 6.7 Provision `HERE_JS_DEMO_APIKEY`, whitelisted to the Pages domain, and the
      check that the server-side key never reaches a Pages artefact
- [ ] 6.8 `Blazor` — Razor components, Blazor adapter of the presentation engine
- [ ] 6.9 `Blazor` — samples: WASM, Server, WPF host, WinForms host, with FlaUI
      launch smoke tests on the two desktop hosts

## Phase 7 — Cross-platform

- [ ] 7.1 `Forms` — Xamarin.Forms renderers, Android and iOS heads, MVVM
- [ ] 7.2 `Maui` — MAUI handlers, four heads, `BlazorWebView` handler on Windows and
      Mac Catalyst
- [ ] 7.3 Façade sample — MAUI four heads, `.csproj` with no platform condition, plus
      the native ↔ JavaScript provider selector and its honesty notice

## Phase 8 — Verification harness

- [ ] 8.1 Compatibility projects per intermediate target framework (`net8.0`,
      `net9.0`) referencing each published package, blocking on regression
- [ ] 8.2 One parameterised user interface suite — a single `net10.0` xUnit v3
      project holding the canonical scenario once behind a driver abstraction, with
      an Appium adapter for the sixteen Apple and Android heads and a FlaUI adapter
      for the two Windows heads — executed once per head by an eighteen-entry
      matrix, Xamarin included
- [ ] 8.3 Emulator and simulator substrate for the fourteen matrix entries that
      need one — seven iOS, seven Android; the two Mac Catalyst and two Windows
      heads run natively on their runner. The matrix routes by generation: the
      Xamarin heads to the self-hosted runner pinned at Xcode 15.4 on macOS 14, the
      modern iOS heads to a GitHub-hosted macOS image carrying the Xcode the current
      `dotnet/macios` release requires
- [ ] 8.4 `bUnit` on the Razor components, Playwright on the browser journeys
- [ ] 8.5 Stryker wired to the diff on every pull request and full nightly, break
      threshold 70 %, target 85 %
- [ ] 8.6 Integration test projects — `tests/<PackageId>.IntegrationTests` — in
      `Rest`, the three binding repositories and `Navigation`, and nowhere else,
      never executed in the unit pass
- [ ] 8.7 Release check asserting the published version matches the
      `PublicAPI.Unshipped.txt` diff — a removed or reshaped member forces a major,
      whatever the commit prefix says
- [ ] 8.8 Verify integration tests skip cleanly with no credentials configured, exit
      code zero

- [ ] 8.9 Cascade diagnostic: the bump pull request opened in the next topological
      wave carries the publishing package's surface diff as an attachment, so a
      downstream repository receives what changed rather than a red build to
      decompose
- [ ] 8.10 Upgrade merge check: a proposal raising a pinned HERE artefact does not
      merge on an empty surface diff alone — the integration suite and the canonical
      scenario have both run and are green

## Phase 9 — Credentials and signing

- [ ] 9.1 Request one HERE credential set **per demonstration bundle identifier**;
      never share one across two applications
- [ ] 9.2 Wire programmatic credential injection at start-up in the mobile
      demonstrations, with `SecureStorage` caching; assert no literal remains in
      `AndroidManifest.xml` or `Info.plist`
- [ ] 9.3 Android keystore and Apple certificate, private key and provisioning
      profiles into GitHub Actions secrets; temporary keychain created and destroyed
      per job, nothing written to the workspace
- [ ] 9.4 Scheduled expiry check in `Meta`, opening an issue sixty days ahead
- [ ] 9.5 Verify the dedicated runner account holds no signing material in its login
      keychain

## Phase 10 — Publication

- [ ] 10.1 Publish stable in topological order, one repository at a time
- [ ] 10.2 Make each repository public only once conforming
- [ ] 10.3 Store submissions for the .NET and MAUI heads; **release artifacts only
      for both Xamarin heads**, each `README.md` naming its dated cause
- [ ] 10.4 Enable the release cascade in `Meta` and verify a full propagation
- [ ] 10.5 Publish the gallery of all thirty-one demonstrations and the sixteen DocFX
      sites
- [ ] 10.6 Promote the deltas of this change into `openspec/specs/` and archive it
