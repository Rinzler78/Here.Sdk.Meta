# toolchain

## ADDED Requirements

### Requirement: Four blocking checks per language, each at the gate it belongs to

`pre-commit` SHALL be installed natively in every repository, and SHALL run four
blocking checks for each language present, over the **whole tree** — tests, scripts,
configuration and documentation included. Each check SHALL run at the gate its cost
belongs to: a check that queries the network belongs to the push, because a commit
that waits on an advisory database is a commit a developer learns to avoid. Blocking
is about there being no way past the check, not about which gate holds it.

1. **Vulnerability scan** of the dependency graph, failing on any known advisory of
   moderate severity or above — at the **push** gate: it queries an advisory database
   over the network.
2. **Dependency freshness**, blocking, at the **push** gate for the same reason —
   measured as *unattended drift*, not as
   instantaneous lag. A dependency fails the commit when the bump pull request
   Renovate opened for it has been open for more than fourteen days. Failing on any
   lag at all would deadlock the repository: the very pull request that raises a
   version could not be committed while the version is still behind.
3. **Linter and formatter** for the language — at the **commit** gate: offline and
   fast.
4. **`cspell` with `language: en`** — at the **commit** gate, for the same reason.

There SHALL be no opt-out and no bypass instruction. A blocking hook signals a root
cause to fix; a systematic false positive SHALL be answered by refining the
detection, never by disabling it.

#### Scenario: a vulnerable transitive dependency stops the push
- **GIVEN** a transitive package with a published advisory
- **WHEN** the developer pushes
- **THEN** the scan fails, naming the package, the advisory and the path to it, and no
  instruction to bypass it is offered

#### Scenario: the same checks run in CI, at every gate
- **GIVEN** a commit pushed without hooks installed
- **WHEN** CI runs
- **THEN** it executes the identical set across **every** gate, and fails identically —
  a job that ran only the commit-stage hooks would let the network checks through

### Requirement: Uniform script facade, one verb per action

Each repository SHALL expose the same thin `bash` scripts, at the same location,
with exit code 0 or non-zero and no interactive prompt: `setup-env`, `restore`,
`build`, `test`, `coverage`, `watch`, `run`, `deploy`, `package`, `publish`, `docs`,
`lint`, `format`, `checks`, `e2e`, `bench`, `clean`.

Behind the facade, plain `dotnet` SHALL be used. Build semantics — target
frameworks, analyzers, deterministic build, SourceLink, thresholds, packaging
metadata — SHALL live in `Rinzler78.Build`, so that they apply identically under F5 in
an IDE and in CI.

`Rinzler78.Build` SHALL be consumed as an **MSBuild project SDK**, its version pinned
once in `global.json`. A package reference cannot carry semantics a project reads while
it is being evaluated: restore needs `TargetFrameworks` before the package's props
exist, so a named target-framework set delivered that way evaluates to empty and NuGet
reports an invalid framework identifier. The package SHALL also ship the props and
targets under `build/`, so that a project which only references it still receives every
semantic that is read after evaluation.

NUKE SHALL appear only in `Meta`, the sole place where orchestration is real.

#### Scenario: IDE and CI agree
- **GIVEN** a rule violated by the code
- **WHEN** the developer builds in the IDE
- **THEN** the same error appears as in CI, because both read the same targets

### Requirement: `setup-env` converges the machine to a declared toolchain

Each tool SHALL be declared in its native manifest — `global.json` for the SDK and
the workload set, `.config/dotnet-tools.json` for .NET tools, `mise.toml` for the
JDK, Node and the Android SDK — and `setup-env` SHALL orchestrate them.

Activation SHALL be **per directory** through `mise`: nothing global is overwritten,
and two repositories requiring two JDKs coexist. Xcode is the exception: it cannot
be installed by a script, and `setup-env` SHALL verify it and fail with instructions.

`setup-env --check` SHALL be non-mutating and SHALL be a CI gate.

#### Scenario: two repositories, two JDKs
- **GIVEN** two repositories declaring different JDK versions
- **WHEN** the developer moves between their directories
- **THEN** each shell resolves its own JDK, and neither machine-wide default changes

### Requirement: Release enables every optimisation the project supports

Debug SHALL serve diagnosability: no optimisation, full symbols,
`AndroidFastDeploymentType=AssembliesOnly`, Hot Reload.

Release SHALL enable everything the project can enable — full R8 and profiled AOT on
Android, AOT and LLVM on iOS, WASM AOT and Brotli on Blazor, ReadyToRun or Native AOT
on server demonstrations, deterministic build, SourceLink, symbol packages, and
warnings as errors on every project — samples included, per `coding-principles`.

Costly optimisations — WASM AOT, Android profiled AOT, iOS LLVM — SHALL NOT run on
`feature/*` pull requests; they run on the pull request to `master`, on the nightly
job, and on release.

#### Scenario: a fast pull request is still a faithful one
- **GIVEN** a pull request on a feature branch
- **WHEN** CI runs
- **THEN** Release is built without costly AOT, and the nightly job covers it

### Requirement: Package versions are centralised and restore is locked

Each repository SHALL use central package management through
`Directory.Packages.props`, and SHALL commit `packages.lock.json` for every project.
CI SHALL restore with `--locked-mode`, so an unintended version drift fails the
build rather than being silently absorbed.

#### Scenario: an unlocked drift fails
- **GIVEN** a pull request that changes a version in `Directory.Packages.props`
  without regenerating the lock files
- **WHEN** CI restores with `--locked-mode`
- **THEN** restore fails, and `lockfile-sync` regenerates the lock on bot branches

### Requirement: Specifications name libraries, versions live in the manifest

A requirement SHALL name the library it mandates and the property that motivated the
choice, never its current version. Concrete versions SHALL live in
`Directory.Packages.props` and the lock files, where Renovate maintains them.

Writing a current version into a requirement would make every routine dependency
bump an OpenSpec change, and would put the specification in direct conflict with the
dependency-freshness gate that requires those bumps to land unattended. A
specification that has to be edited to stay true is a specification that will stop
being true.

Two exceptions SHALL hold, and both are deliberate freezes rather than current
values: the HERE SDK artefacts pinned per column, whose raise is a proposal by
design, and the frozen Xamarin toolchain with its pinned Xcode. A version that is
itself the decision belongs in the specification; a version that merely happens to
be the latest does not.

#### Scenario: a routine bump does not touch the specification
- **GIVEN** Renovate raises the assertion library by one minor version
- **WHEN** the pull request is reviewed
- **THEN** it changes `Directory.Packages.props` and the lock file alone
- **AND** no requirement is edited

#### Scenario: a deliberate freeze stays written down
- **GIVEN** a proposal raising the pinned HERE Android artefact
- **WHEN** it is authored
- **THEN** the specification records the version, because the pin is the decision

### Requirement: The Xamarin toolchain is frozen and operated deliberately

No GitHub-hosted runner carries Xcode 15 after 2 November 2026, and
`Component.Xamarin` was removed from the Windows Server 2025 image. The Xamarin
target frameworks SHALL therefore build on a self-hosted runner with Xcode 15.4
pinned.

Four conditions SHALL hold, without which this reads as neglect rather than
stewardship:

1. A dated ADR recording the market survey — `macos-13` retired December 2025,
   `macos-14` unsupported 2 November 2026, Bitrise retiring Xcode 15.x on
   16 September 2026, Azure DevOps consuming the same images, and Apple's licence
   permitting two virtual machines per Apple host.
2. A job that **verifies** the runner reports Xcode 15.4 and fails if it moves.
3. A disjoint matrix, so that an unavailable self-hosted runner never blocks a pull
   request on a portable repository.
4. A written exit condition stating when Xamarin leaves the scope.

The self-hosted machine SHALL carry the Xamarin column and nothing else. The two
generations are separated by two major operating system versions, not by two Xcode
minors: the current .NET 10 iOS servicing release requires Xcode 26.6, which
requires macOS 26.2, while Xcode 15.4 dates from May 2024. The modern iOS heads
SHALL therefore build on GitHub-hosted macOS images, which carry the required Xcode.

Neither generation SHALL be moved onto the other's machine. Installing both Xcode
versions side by side and selecting one per job would keep a May 2024 Xcode running
on a 2026 operating system, unsupported by Apple and with no recourse the day it
stops launching, since that version will no longer be signed. It would also require
keeping current a machine whose entire purpose is to stay frozen: a runner that is
updated is no longer a frozen runner. Virtualising a second macOS on the same Apple
host stays available as a fallback — Apple's licence permits two virtual machines
per host — but SHALL NOT be adopted while a hosted image does the work.

#### Scenario: the modern heads never touch the frozen machine
- **GIVEN** a pull request building a `net10.0-ios` head
- **WHEN** the workflow selects its runner
- **THEN** it runs on a GitHub-hosted macOS image
- **AND** the self-hosted runner is not solicited

#### Scenario: the freeze is tested, not believed
- **GIVEN** the self-hosted runner
- **WHEN** its Xcode version changes
- **THEN** the verification job fails before any build is attempted

### Requirement: Self-hosted runners are never reachable from a fork

GitHub advises against self-hosted runners on public repositories because a fork
can execute arbitrary code on the machine. The mitigation SHALL be structural, not
a setting.

Self-hosted jobs SHALL be conditioned on
`github.event.pull_request.head.repo.full_name == github.repository`, and SHALL run
on `push`, on schedule and on `workflow_dispatch`. The runner SHALL be ephemeral,
SHALL run under a dedicated macOS account with no access to personal keychains, and
`pull_request_target` SHALL be forbidden — verified by a check.

#### Scenario: a fork pull request never reaches the machine
- **GIVEN** a pull request opened from a fork
- **WHEN** its workflows are scheduled
- **THEN** only hosted jobs run, and every self-hosted job is skipped

#### Scenario: an unavailable runner blocks only what it owns
- **GIVEN** the self-hosted runner is offline
- **WHEN** a pull request touching a Xamarin target framework is opened
- **THEN** its Xamarin job queues and the pull request cannot merge — the frozen
  chain accepts blocking rather than being waved through
- **AND** a pull request on a portable repository merges normally, its matrix
  containing no self-hosted job
