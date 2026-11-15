# Tasks — 001 Ecosystem foundation

Two orders govern this change. **Construction** builds the natives first, because an
abstraction is extracted from its implementations, not designed against
documentation. **Publication** is strictly topological, and nothing goes stable or
public before extraction is complete.

Three rules govern the boxes. A task is one deliverable, and its **Done when** line
says how to see it done; a task without one is done when its sentence is true of the
repository. A task never depends on a later one: the order below is the execution
order, and a dependency on something outside the repository's control — hardware, a
portal, a store — is named in the task and never blocks the tasks that do not need it.
The task that completes a repository's packages also requires what the repository
always requires: provisioning by 1.12, a declared coverage and trimming contract, a
trimming test application for a `Committed` library, an ADR for each architectural
choice, `README.md`, its DocFX site, and a green CI. A demonstration that needs a HERE
credential set requests it when its first sample starts; 2.4 does so for phase 2.

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
- [x] 1.2 `Rinzler78.Build` — props/targets, domain-agnostic: named target
      framework sets, analysers, `TreatWarningsAsErrors`, deterministic build,
      `ContinuousIntegrationBuild` following the environment, SourceLink, `.snupkg`
      symbols, XML documentation with `CS1591` as error, `<RinzlerTrimContract>`,
      `<RinzlerCoverageContract>`, and the mechanisms the domain declares against —
      layer vocabulary and validation, forbidden dependencies, required analysers,
      required description fragment. A project declaring no policy builds
- [x] 1.3 `Rinzler78.Templates` — `rinzler-lib`, `rinzler-binding`, `rinzler-app`,
      each expanding: `CLAUDE.md` and `AGENTS.md` with the agreement check,
      `.claude/settings.json` with the `PreToolUse` / `PostToolUse` / `Stop` hooks,
      `.pre-commit-config.yaml` with the four blocking checks per language —
      vulnerability scan, dependency freshness, linter and formatter, and `cspell`
      with `language: en` over the whole tree — `mise.toml` and the `setup-env`
      script with its non-mutating `--check` mode,
      `Directory.Packages.props` and lock files with `--locked-mode` in CI,
      a `README.md` with the required sections and the check that keeps it true,
      `.github/ISSUE_TEMPLATE/`, the thirteen labels, the tag-driven release workflow
      and its tag check, MinVer, the `lockfile-sync` workflow, the forge provisioning
      script, `WORKFLOW.md`, DocFX, the script facade,
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
- [x] 1.4 Regenerate the Toolkit harness from its own template; assert byte identity
      for every harness file, and presence only for the files the template declares
      as seeds — the README, the entry points, the solution, the central versions,
      the dictionary and the scaffold, whose content is repository-specific by
      construction
- [x] 1.5 Forge settings of `Rinzler78.Toolkit`, applied by `scripts/_provision-forge.sh`:
      a read-only workflow token that cannot approve pull requests, the `release`
      environment admitting tags `v*` only, `develop` and `master` rulesets, immutable
      tags `v*` created by administrators only. Applied on 2026-09-26, the hand-made
      ruleset and the branch the previous, bot-driven release flow left behind removed
- [x] 1.6 Publish the toolchain packages keylessly. `Rinzler78.Build` and
      `Rinzler78.Templates` 0.1.0 were published on 2026-09-24 through the previous,
      bot-driven release flow, since replaced; 0.1.1 was published on 2026-09-27 from
      the signed tag `v0.1.1`. The procedure and its traps are in
      `docs/runbooks/nuget-publication.md`
- [ ] 1.6a Vendor the seventeen skills of `agent-harness` from their public upstream
      into `.agents/skills/` and `.claude/skills/` of `Rinzler78.Toolkit`, and into the
      three templates through the shared harness directory (Toolkit issue #1). The
      boxes of 1.1 and 1.3 do not claim them.
      Done when: both skill directories hold the seventeen, byte-identical, and a
      regenerated repository carries them
- [ ] 1.7 Release check for packages with no public API surface — the toolchain's
      props/targets packages and template pack: refuse a tag whose version disagrees
      with the Conventional Commits since the previous tag — `!` or a `BREAKING
      CHANGE` footer breaking (minor before 1.0.0), `feat` minor, `fix` and `perf`
      patch, other types no effect, the strongest effect deciding, no release for a
      range with none — and require the annotated tag's message to carry the evidence
      the proposal cited; 1.0.0 after a 0.x version when, and only when, the ADR
      declaring the surface stable exists, whatever the commits.
      Done when: a scratch repository refuses a tag of each wrong effect and accepts
      the right one, and the check runs in the release workflow before the credential
      is requested
- [ ] 1.8 `Rinzler78.Templates` — Stryker wired to the diff on every pull request and
      in full nightly, break threshold 70 %, target 85 %, absent from a repository
      declaring no assembly. Mutation is a per-repository gate: wiring it after the
      repositories exist would reopen every one of them.
      Done when: a regenerated repository runs both modes and a surviving mutant under
      the threshold fails its pull request
- [ ] 1.9 `Rinzler78.Templates` — release check asserting the published version matches
      the `PublicAPI.Unshipped.txt` diff — an empty diff a patch, an additive diff a
      minor, a removed or reshaped member a major, or a minor before 1.0.0, whatever
      the commit prefix says — with one exception: 1.0.0 after a 0.x version is
      accepted when, and only when, the ADR declaring the public surface stable
      exists, whatever the diff; and the annotated tag's message carrying the evidence
      the proposal cited.
      Done when: a scratch library with a public API refuses the three wrong
      outcomes, and the check precedes the first prerelease of any package
- [ ] 1.10 `Rinzler78.Templates` — an opt-in integration-test project,
      `tests/<PackageId>.IntegrationTests`, never executed in the unit pass and
      selected only by `Rest`, the three binding repositories and `Navigation`. It
      skips cleanly, with exit code zero, when no HERE credential is configured.
      Done when: a regenerated repository with the option runs its unit pass without
      it, and its integration pass reports skipped, not failed, with no credentials
- [ ] 1.11 The domain reviewers `release-flow` requires, `spec-reviewer` and
      `package-api-reviewer`: written, joined to the required checks in
      `scripts/_provision-forge.sh`, and carried by the template.
      Done when: each fails a seeded violation in a scratch repository and passes the
      conforming one, and the script requires them alongside `verify` and `lint`
- [ ] 1.11a `scripts/_provision-forge.sh` — the `copilot_code_review` rule, and an
      idempotent verification mode that reads every setting back and reports drift
      without changing anything. The script carries neither today.
      Done when: the verification mode reports `Rinzler78.Toolkit` conforming after an
      apply, and reports a seeded drift
- [ ] 1.12 The repository provisioning procedure, one script or one runbook section:
      generate the repository from the right template, apply `_provision-forge.sh`
      (Copilot review included), register the nuget.org trusted-publishing policy
      when it publishes — a portal step only you can take — and add it to `Meta` as a submodule under `repos/<name>`
      tracking `develop`. Every repository task below starts from it, so no repository
      is provisioned by hand and no task waits for the other seventeen.
      Done when: it has provisioned `Here.Sdk.Build` end to end, and a second run
      changes nothing
- [ ] 1.13 `Here.Sdk.Build` — the domain specialisation: the layer vocabulary of the
      eight-layer map, the forbidden dependencies of the inward layers, the required
      analysers, and the HERE non-affiliation fragment, all declared through the
      generic mechanism of `Rinzler78.Build`.
      Done when: a scratch consumer violating the vocabulary, a forbidden dependency
      or the fragment cannot build, and a conforming one does; a props/targets package
      never imports itself, hence the scratch consumer
- [ ] 1.14 Publish `Here.Sdk.Build` by signed tag, following
      `docs/runbooks/nuget-publication.md`.
      Done when: the package is restorable from nuget.org at the tagged version
- [ ] 1.15 Request the `Rinzler78.` prefix reservation. This does not wait on anything
      else: `Rinzler78.CometBFT.Client` is already published under the same owner, so
      the prerequisite is met. The outcome is decided by nuget.org and never blocks a
      task; the search index reports the prefix as unverified while no reservation
      exists.
      Done when: the request is filed and its date recorded in the runbook
- [ ] 1.16 `Meta` — brought to the template's conformance. This repository exists
      already, with its history; it is not generated again. Add the harness files it
      lacks, the CI whose `verify` and `lint` checks the rulesets require, and
      `promotion.yml` on `master` before `promotion-source` is required: without it no
      pull request into `master` can merge.
      Done when: a pull request into `develop` runs `verify` and `lint` green, and a
      promotion pull request into `master` is judged by `promotion-source`
- [ ] 1.17 `Meta` — forge settings applied by `_provision-forge.sh`, once 1.16 gives
      the rulesets the checks they require.
      Done when: the script's verification mode reports every setting of 1.5 plus the
      Copilot review rule and the domain reviewers, with nothing to change
- [ ] 1.18 `Meta` — founding ADRs: poly-repository rationale, public visibility,
      templates generated not synchronised, frozen Xamarin toolchain, deferred Linux
      desktop, self-hosted runner isolation, deliberate absence of `CODEOWNERS`.
      Done when: each exists under `openspec/`, with context, decision and the
      alternatives rejected
- [ ] 1.19 `Meta` — made public. It carries the specification and the plan, so it
      opens before the other repositories, which wait for 10.2. The public-visibility
      ADR of 1.18 says why.
      Done when: a secret scan of the whole history is clean, then visibility is public
- [ ] 1.20 Decision: the identity the cascade writes with across repositories. No
      repository's `GITHUB_TOKEN` can reach another, and `release-flow` records that
      the identity "is not yet specified". Candidates: a GitHub App installation
      token, a fine-grained token, workflow dispatch from each repository.
      Done when: an ADR in `Meta` chooses one, names the permissions it holds and how
      long it lives, and `release-flow` states it
- [ ] 1.20a `Rinzler78.Templates` — the release workflow notifies `Meta` by
      `repository_dispatch` once it has published, with the identity chosen in 1.20.
      The Toolkit and `Here.Sdk.Build` receive it through the template's regeneration.
      Done when: a publication from a scratch repository raises the dispatch in a scratch
      `Meta`, and the Toolkit's regenerated workflow carries the step
- [ ] 1.21 `Meta` — NUKE orchestrator skeleton, with its unit tests where it holds
      logic.
      Done when: its target list runs locally and in CI and its unit tests pass
- [ ] 1.22 `Meta` — the release cascade, receiving side: accept the publishing
      repository's dispatch of 1.20a and poll the flat-container index until the
      version is restorable.
      Done when: a dispatch for a published `Rinzler78.Build` version is held until
      nuget.org serves it, then released
- [ ] 1.22a `Meta` — the release cascade, writing side: open the next topological
      wave's bump pull requests (`Directory.Packages.props`, then
      `dotnet restore --force-evaluate`, then the regenerated lock file), and wait for
      green and for publication before the next wave.
      Done when: publishing a `Rinzler78.Build` patch from `Rinzler78.Toolkit` opens a
      green bump pull request in `Here.Sdk.Build` — the first real edge of the graph
- [ ] 1.23 `Meta` — cascade diagnostic: the bump pull request carries the publishing
      package's surface diff as an attachment, so a downstream repository receives
      what changed rather than a red build to decompose.
      Done when: the bump pull request of 1.22 shows the diff of a package with a
      public API, from a scratch publisher
- [ ] 1.24 `Meta` — nightly integration job: refresh every submodule with
      `git submodule update --remote`, then build the graph from sources with each
      repository against the others' `develop`. A repository not yet created is
      reported as absent, never as a failure.
      Done when: the job runs green on the repositories that exist and its report
      lists the absent ones
- [ ] 1.25 `Meta` — aggregated board across repositories.
      Done when: it renders the task boxes and open issues of `Rinzler78.Toolkit` and
      `Here.Sdk.Build`
- [ ] 1.26 `Meta` — gallery page, ordered by layer, each application with its
      one-sentence delta.
      Done when: a fixture catalogue renders in layer order, and an empty one renders
      without failure
- [ ] 1.27 `Meta` — scheduled-issue reminders: a ninety-day rotation reminder for each
      HERE secret, and an expiry reminder sixty days ahead for each signing
      certificate. The jobs read metadata, never a secret value.
      Done when: a fixture of two secrets and one certificate opens the expected
      issues in `Meta`
- [ ] 1.28 `Meta` — HERE SDK release detection: read `heremaps/here-sdk-examples`
      releases through the GitHub API against a committed cursor, open a
      `needs-triage` / `chore` issue in the affected binding repository, and link —
      never parse — the documentation release notes.
      Done when: a recorded release fixture produces the expected issue in a dry run;
      real issues start with the binding repositories of phase 2
- [ ] 1.29 `Meta` — separate detection for the JavaScript column, which versions
      independently on `js.api.here.com`, is absent from the examples repository and
      returns 404 on the npm registry.
      Done when: a recorded page fixture produces the expected issue in a dry run
- [ ] 1.30 `Meta` — the **ecosystem audit**, first half: catalogue containment,
      task-box truth, and forge settings read through the verification mode of 1.11a.
      A check whose subject does not exist yet reports not applicable, never a
      failure; equality with the catalogue is asserted only once the realisation order
      is complete.
      Done when: each check has a failing and a passing fixture, and the audit runs
      green on the repositories that exist
- [ ] 1.30a `Meta` — the **ecosystem audit**, second half: demonstration coverage, ADR
      integrity and dead deny rules, harness drift, and version literals in
      specifications — a requirement naming a concrete version that is not one of the
      two declared freezes is reported.
      Done when: each check has a failing and a passing fixture

## Phase 2 — Native columns, standing alone

- [ ] 2.1 Self-hosted macOS runner: Xcode 15.4 pinned, ephemeral, dedicated account
      with no keychain access, version verification job, fork-exclusion condition,
      `pull_request_target` ban check — **before any Xamarin build**. It needs a Mac
      able to run the pinned Xcode, a prerequisite of hardware only you can supply.
      Done when: the verification job reports the pinned version, fails when it moves,
      and a canary job runs on the runner
- [ ] 2.2 Decision: the runner of the Xamarin.Android chain. The earlier wording,
      "`windows-2022` while it lives", names no date.
      Done when: an ADR in `Meta` chooses a self-hosted Windows runner or the hosted
      image, and dates the last day the hosted image can serve
- [ ] 2.3 The Windows runner of the Xamarin.Android chain, per 2.2, with the same
      verification and isolation rules as 2.1; a no-op when 2.2 keeps the hosted image.
      Done when: a canary job builds on it, or 2.2 records that none is needed
- [ ] 2.4 Request one HERE credential set **per demonstration bundle identifier**, for
      the mobile applications of this phase — `Bindings.iOS`, `Bindings.Android`,
      `iOS` and `Android` — and store each in the owning repository's GitHub Actions
      secrets. Never share one across two
      applications. A step only you can take, on the HERE portal.
      Done when: every identifier of this phase has a set, and none sits in a file
- [ ] 2.5 `Rinzler78.Templates` — credential injection check of `rinzler-app`, for the mobile demonstrations: the application
      binds its credential from configuration at start-up, caches it in
      `SecureStorage`, and CI fails when a literal remains in `AndroidManifest.xml` or
      `Info.plist`.
      Done when: the check fails a seeded literal in each of the two files
- [ ] 2.6 `Bindings.iOS` — pinned `.xcframework` version, and the Objective Sharpie
      output, `ApiDefinition.cs` and `StructsAndEnums.cs`, committed as the upstream
      baseline, so a version raise is a plain `git diff` before any hand binding
- [ ] 2.7 `Bindings.iOS` — C# projection, and the API-coverage test reflecting over the
      bound assembly with its justified exclusion list; the integration-test project
      of 1.10
- [ ] 2.8 `Bindings.iOS` — sample: .NET 10 iOS, raw surface, no pattern
- [ ] 2.9 `Bindings.Android` — pinned AAR version, and the generator's `api.xml`
      committed as the upstream baseline; it is an intermediate artefact and is
      discarded unless committed deliberately
- [ ] 2.10 `Bindings.Android` — C# projection, and its API-coverage test; the
      integration-test project of 1.10
- [ ] 2.11 `Bindings.Android` — sample: .NET 10 Android, raw surface, no pattern
- [ ] 2.12 `iOS` — idiomatic wrapper, referencing its binding only
- [ ] 2.13 `iOS` — samples: Xamarin.iOS in MVC Cocoa, .NET 10 iOS in MVVM-C, each
      written by hand in its pattern, with the ADR of the owning repository that
      justifies it
- [ ] 2.14 `Android` — idiomatic wrapper, referencing its binding only
- [ ] 2.15 `Android` — samples: Xamarin.Android in MVP, .NET 10 Android in MVVM/UDF,
      each written by hand in its pattern, with `ADR-0001` explaining why Google's
      MVVM guidance does not apply to .NET Android

## Phase 3 — JavaScript spike, then extraction

- [ ] 3.1 Throwaway spike on the HERE Maps API for JavaScript. The code is discarded and
      committed nowhere; the capability matrix is the deliverable, and extraction
      cannot start without it.
      Done when: the matrix is recorded as an ADR in `Meta`
- [ ] 3.2 `Common` — extracted: value objects validating at construction
- [ ] 3.3 `Common` — sample: Blazor WASM page, no HERE key required
- [ ] 3.4 `Abstractions` — extracted from the two native wrappers and the capability
      matrix of 3.1: `Abstractions`, `.Navigation`, `.Offline`, `.Testing` — prerelease,
      after the checks of 1.7 and 1.9
- [ ] 3.5 `Abstractions` — sample: Blazor WASM driven by the in-memory provider,
      demonstrably running with no HERE credential
- [ ] 3.6 `Core` — extracted from both natives: credentials, cache, logging, resilience
- [ ] 3.7 `Core` — sample: Blazor Server, secrets held server-side, cache visible
- [ ] 3.8 Reconcile the native wrappers against the extracted contracts

## Phase 4 — Standardisation

- [ ] 4.1 `Standard.Android` and `Standard.iOS` — adapters to the contract
- [ ] 4.2 Samples: `Standard.iOS` on .NET 10 iOS and `Standard.Android` on
      **Xamarin.Android**, sharing one **linked** business-logic file; CI asserts both
      projects resolve the identical path and that no copy exists under `samples/`

## Phase 5 — Portable implementations

- [ ] 5.1 `Rest` — Geocoding & Search v7, Routing v8, Traffic v7, Raster Tile v3; the
      integration-test project of 1.10
- [ ] 5.2 `Rest` — sample: Blazor Server
- [ ] 5.3 `Navigation` — REST guidance engine: progress along the polyline, manoeuvre
      advance, deviation with hysteresis, rerouting, voice
- [ ] 5.4 `Navigation` — recorded GPS traces replayed as pure unit tests; the
      integration-test project of 1.10
- [ ] 5.5 `Navigation` — sample: Blazor Server, guidance replayed on a trace
- [ ] 5.6 `Standard` — multi-targeted façade, conditional dependencies, `AddHereSdk()`.
      It needs the portable fallback of 5.1 and 5.3, hence its place here and not in
      phase 4. Its `README.md` states the two consequences: `Standard.Js` outside the
      façade, and no map renderer in the portable fallback
- [ ] 5.7 `Presentation` — neutral engine, immutable serialisable state, cancellable
      intents
- [ ] 5.8 `Presentation` — sample: Blazor WASM showing the engine state bare
- [ ] 5.9 `Presentation.Mvp`, `.Mvvm` — the two pattern adapters, the MVVM one
      carrying the diff and the key-stable collections
- [ ] 5.10 `Presentation.Android`, `.iOS` — UI-thread marshalling, lifecycle,
      rotation survival

## Phase 6 — JavaScript column

- [ ] 6.1 `Bindings.Js` — pinned Maps API for JavaScript version, C# interop, and its
      API-coverage test; the integration-test project of 1.10
- [ ] 6.2 Provision `HERE_JS_DEMO_APIKEY`, whitelisted to the Pages domain, and the
      check that the server-side key never reaches a Pages artefact. It precedes the
      first sample, which needs the key, and the key itself comes from the HERE portal,
      a step only you can take.
      Done when: the check fails a seeded server-side key in a Pages artefact
- [ ] 6.3 `Bindings.Js` — sample: Blazor WASM, raw `H.Map`, no pattern
- [ ] 6.4 `Js` — idiomatic C# wrapper, referencing its binding only
- [ ] 6.5 `Js` — sample: Blazor WASM
- [ ] 6.6 `Standard.Js` — adapter to the contract
- [ ] 6.7 `Standard.Js` — sample: Blazor WASM driven by the contract
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
- [ ] 8.2 Emulator and simulator substrate for the matrix entries that need one; the
      Mac Catalyst and Windows heads run natively on their runner. The matrix routes
      by generation: the Xamarin heads to the self-hosted runner pinned at Xcode 15.4
      on macOS 14, the modern iOS heads to a GitHub-hosted macOS image carrying the
      Xcode the current `dotnet/macios` release requires. It precedes 8.3, which
      cannot run without it
- [ ] 8.3 One parameterised user interface suite — a single `net10.0` xUnit v3
      project holding the canonical scenario once behind a driver abstraction, with
      an Appium adapter for the Apple and Android heads and a FlaUI adapter for the
      Windows heads — executed once per head by a matrix, Xamarin included
- [ ] 8.4 `bUnit` on the Razor components, Playwright on the browser journeys
- [ ] 8.5 Upgrade merge check: a proposal raising a pinned HERE artefact does not
      merge on an empty surface diff alone — the integration suite and the canonical
      scenario have both run and are green

## Phase 9 — Signing

- [ ] 9.1 Android keystore and Apple certificate, private key and provisioning
      profiles into GitHub Actions secrets; temporary keychain created and destroyed
      per job, nothing written to the workspace.
      Done when: a canary job signs a dummy artefact and leaves no file in the workspace
- [ ] 9.2 Verify the dedicated runner account of 2.1 holds no signing material in its
      login keychain

## Phase 10 — Publication

- [ ] 10.1 Publish stable in topological order, one repository at a time, driven by
      the release cascade of 1.22 and 1.22a, and verify a full propagation.
      Done when: every package of `package-topology` is restorable at its stable
      version, and the audit reports the ecosystem complete
- [ ] 10.2 Make each repository public once the audit reports it conforming; `Meta`
      opened earlier, at 1.19.
      Done when: no repository remains private
- [ ] 10.3 Store submissions for the .NET and MAUI heads; **release artifacts only
      for both Xamarin heads**, each `README.md` naming its dated cause.
      Done when: each submission is filed and each artefact is attached to a release;
      a store's verdict is recorded when it arrives and never blocks this task
- [ ] 10.4 Publish the gallery of every demonstration and the DocFX site of every
      repository that carries code
- [ ] 10.5 Promote the deltas of this change into `openspec/specs/` and archive it
