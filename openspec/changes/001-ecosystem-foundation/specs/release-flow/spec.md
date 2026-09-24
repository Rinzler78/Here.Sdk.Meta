# release-flow

## ADDED Requirements

### Requirement: Gate cost is proportional to irreversibility

| Event | What runs |
|---|---|
| Push on `feature/*` | Debug and Release build without costly AOT, unit tests, lint, coverage, trim analysis, `openspec validate --strict` |
| Pull request to `develop` | the above, plus integration tests, domain reviewers, `pack` without publication |
| Pull request to `master` | the above, plus full AOT, UI and end-to-end tests, measured sizes posted on the pull request |
| Release published | packages and applications published, all optimisations enabled |
| Nightly, from `Meta` | the integration job — building the graph from sources with full AOT — and the ecosystem audit |

Nothing costly SHALL be discovered at release time: the pull request to `master`
has already exercised it.

#### Scenario: a broken AOT path is caught before the release
- **GIVEN** a reflection pattern that survives a normal build but breaks WASM AOT
- **WHEN** the pull request to `master` runs
- **THEN** it fails there, not on the release

### Requirement: One branch, one worktree, never reused

The flow is Git Flow with two long-lived branches, `master` and `develop`. Work
SHALL happen on a `feature/<kebab-slug>` branch cut from `develop`, in a dedicated
worktree at `<repo>/.worktrees/<kebab-slug>/`.

A branch SHALL NOT be reused once its pull request is merged, and a worktree SHALL
be removed when its branch is. Reuse is how an agent inherits stale state and
reports success against code that is no longer there.

One agent SHALL own one pull request, with file scopes disjoint from any concurrent
agent — `src/` **and** `tests/` — so that two agents never contend for the same file.

#### Scenario: a merged branch cannot be picked up again
- **GIVEN** a `feature/*` branch whose pull request has merged
- **WHEN** an agent attempts to commit to it again
- **THEN** the hook refuses and directs it to cut a fresh branch from `develop`

#### Scenario: concurrent agents do not collide
- **GIVEN** two agents working on the same repository
- **WHEN** each is briefed
- **THEN** their file scopes are disjoint, and each has its own worktree

### Requirement: Blocking checks are automated; human approvals are zero

Both long-lived branches SHALL be protected by GitHub Rulesets with linear history,
signed commits, blocked force-push and deletion, administrators included.

Required status checks SHALL be the build, static quality, and three automated
domain reviewers, each with a defined assertion:

| Reviewer | Asserts |
|---|---|
| `spec-reviewer` | The change implements the OpenSpec delta it claims, and every requirement touched has at least one scenario exercised by a test. |
| `package-api-reviewer` | The `PublicAPI.Unshipped.txt` diff matches the version intent — no removed or changed public member without `breaking-change` and a major bump. |
| `manual-tag-guard` | No tag exists that was not created by merging a release pull request. |

Required human approvals SHALL be **zero**, the author being the sole maintainer;
the gate is the machine, not a signature.

#### Scenario: an unreviewed pull request still cannot merge broken code
- **GIVEN** a pull request with no human reviewer
- **WHEN** a domain reviewer check fails
- **THEN** merging is blocked by the ruleset

### Requirement: Versioning is SemVer 2.0.0, computed from commits

Every package SHALL be versioned according to Semantic Versioning 2.0.0, and the
version SHALL be **computed** from Conventional Commits by Release Please — never
edited by hand. Release Please SHALL be the single source of version truth; no
second versioning mechanism SHALL be added.

A removed or changed public member SHALL require the `!` commit prefix, the
`breaking-change` label and a major bump. The authority is the surface diff defined
in `sdk-updates`, not the commit prefix: a mislabelled commit fails its release
check rather than publishing a breaking change as a minor. Prerelease builds from `develop` SHALL be
versioned `X.Y.Z-develop.<run>` from the last tag.

#### Scenario: a breaking change cannot ship as a patch
- **GIVEN** a pull request removing a public member
- **WHEN** `package-api-reviewer` compares `PublicAPI.Unshipped.txt`
- **THEN** it fails unless the commit carries `!` and the issue carries
  `breaking-change`

### Requirement: Issue templates make triage possible

Each repository SHALL carry `.github/ISSUE_TEMPLATE/` with forms for `bug`,
`feature` and `chore`, delivered by the template. Each form SHALL require the
information triage needs — expected versus observed behaviour, affected target
frameworks and platforms, and for a behavioural change the linked OpenSpec proposal.

Each form SHALL apply `needs-triage` and its type label automatically, so that an
issue never enters the queue unlabelled.

#### Scenario: an issue arrives already typed
- **GIVEN** a bug reported through the form
- **WHEN** it is created
- **THEN** it carries `needs-triage` and `fix`, and names the affected target framework

### Requirement: No CODEOWNERS file

Required human approvals are zero and the author is the sole maintainer, so a
`CODEOWNERS` file would request review from the person opening the pull request and
add a permanently unsatisfiable entry to the ruleset. It SHALL NOT be created.

Should the ecosystem gain a second maintainer, that decision SHALL be revisited by
an ADR rather than by adding the file silently.

#### Scenario: the absence is deliberate, not forgotten
- **GIVEN** the ecosystem audit checking repository conventions
- **WHEN** it finds no `CODEOWNERS`
- **THEN** it passes, because the ADR records the absence as a decision

### Requirement: A release is a reviewed pull request, never a gesture

Release Please SHALL maintain a release pull request per repository. Merging it
creates the tag and the GitHub Release. Publication SHALL trigger on
`release.published` — deliberate and revocable — never on a tag push, which is
irreversible.

`manual-tag-guard` SHALL fail CI on any tag not created that way.

The release pull request SHALL carry the generated changelog, the computed version,
the public API diff, and the sizes measured after AOT.

#### Scenario: a hand-made tag is rejected
- **GIVEN** a tag pushed directly to the repository
- **WHEN** CI runs
- **THEN** `manual-tag-guard` fails and no package is published

### Requirement: Publication cascades in topological waves

nuget.org offers no webhook: its catalogue is an append-only, read-only feed
consumed with a cursor. Between repositories, `repository_dispatch` SHALL be used.

On `release.published`, the publishing repository SHALL notify `Meta`. `Meta` SHALL
poll the flat-container index until the version is restorable, then open bump pull
requests in the next topological wave — editing `Directory.Packages.props`, running
`dotnet restore --force-evaluate`, committing the regenerated lock file — and wait
for green and for publication before starting the next wave.

Fan-out notification SHALL NOT be used: a package two levels down would compile
against a dependency that still references the previous version.

#### Scenario: a bump wave waits for indexing
- **GIVEN** `Common` has just published
- **WHEN** `Meta` receives the dispatch
- **THEN** it polls `https://api.nuget.org/v3-flatcontainer/<id>/index.json` until
  the version appears, and only then opens the wave

### Requirement: Lock files are regenerated by the repository, not by the bot

Renovate does not update `packages.lock.json` when a version is bumped in
`Directory.Packages.props`; under `RestoreLockedMode` the pull request fails.

Each repository SHALL carry a `lockfile-sync` workflow, delivered by the template,
which on any bot-authored pull request runs `dotnet restore --force-evaluate` and
pushes the regenerated lock file. It SHALL serve both Renovate and the cascade.

#### Scenario: a Renovate pull request converges on its own
- **GIVEN** Renovate bumps a third-party package under central package management
- **WHEN** `lockfile-sync` runs on its branch
- **THEN** the lock file is regenerated and committed, and the build turns green

### Requirement: The triage state machine is enforceable

Thirteen labels SHALL exist identically in every repository, created at bootstrap:
five triage states (`needs-triage`, `needs-info`, `ready-for-agent`,
`ready-for-human`, `wontfix`), seven types aligned with Conventional Commits, and
`breaking-change`, plus `blocked`.

A workflow SHALL refuse `ready-for-agent` on an issue that does not carry exactly
one type label. `breaking-change` SHALL require the `!` commit prefix and a
migration section in the pull request, verified by a check.

No priority labels: priority is expressed by order in the aggregated board in `Meta`.

#### Scenario: an untyped issue cannot enter the agent queue
- **GIVEN** an issue with no type label
- **WHEN** `ready-for-agent` is applied
- **THEN** the workflow removes it and comments why

### Requirement: Applications publish through every channel open to them

On release, every demonstration application SHALL be published through **every
channel open to it**. Three exist, and not all three are open to every head:

| Channel | Applies to |
|---|---|
| Public stores | .NET Android and .NET iOS heads, MAUI heads |
| GitHub Release artifacts | every application, without exception |
| Test channels — TestFlight, Play internal testing | every application a store accepts |

**Neither Xamarin head can reach a public store, and both are published through
release artifacts only.**

- **Xamarin.iOS** — Apple has required Xcode 16 and the iOS 18 SDK for submission
  since 24 April 2025, and the frozen chain builds with Xcode 15.4.
- **Xamarin.Android** — Google Play has required target API 36 for new apps and
  updates since 31 August 2026, and apps below target API 35 are already hidden from
  new users on recent devices. Xamarin.Android's last supported target is API 34.

Both artefacts SHALL be signed for ad-hoc distribution, and each `README.md` SHALL
state the store limitation with its date and its cause. The limitation is part of
what the frozen chain demonstrates: it is named, dated and sourced, not hidden.

Web demonstrations SHALL additionally be published to GitHub Pages, and `Meta`
SHALL link them from the gallery.

#### Scenario: neither Xamarin head is offered to a store
- **GIVEN** the release workflows of the `iOS` and `Android` repositories
- **WHEN** they process their Xamarin heads
- **THEN** each produces a signed artifact attached to the GitHub Release
- **AND** neither attempts a store submission
- **AND** a check fails the workflow if a store upload step is ever added to a
  Xamarin head

### Requirement: Publication carries no long-lived credential

Packages SHALL be published through nuget.org **trusted publishing**: the release job
requests a short-lived OIDC token from the forge, nuget.org validates it against a
policy naming the repository and the workflow file, and returns an API key valid for one
hour.

No **long-lived** credential SHALL exist: no registry key in a repository secret, a
configuration file, a developer machine or a runner image. The short-lived key the
exchange returns lives in the publishing job's environment for the duration of that job,
is never written to a file, and is never passed to a step that does not push.

A policy SHALL be registered per repository, permitted to push **new packages as well as
new versions** — a scope limited to selected existing packages cannot push a new
identifier, and every identifier in this ecosystem is new.

A policy's scope SHALL name only the identifiers the repository it names publishes. A
namespace-wide scope such as `Rinzler78.*` SHALL NOT be used: a policy is a grant to
whatever workflow matches it, so a namespace-wide scope would let a compromise of any one
repository publish or replace any package of the ecosystem. Where a single glob cannot
express a repository's identifiers, that SHALL be one policy per identifier rather than
one wider glob.

The token SHALL be exchanged immediately before the push. It is valid for one hour, and a
job that builds first and requests the key at its start would lose it between the pack
and the push.

The alternative — one account key copied into nineteen repositories — SHALL NOT be used.
A personal account has no forge-level secret for workflows, so the per-repository work is
identical either way, and nineteen copies of one credential expiring within a year is
nineteen places to rotate and one to forget.

#### Scenario: no repository holds a publishing credential
- **GIVEN** any repository of the ecosystem
- **WHEN** its secrets are listed
- **THEN** none of them is a package-registry key

#### Scenario: a compromised repository cannot reach another's packages
- **GIVEN** a release workflow whose repository has been compromised
- **WHEN** it requests a key and attempts to push a package another repository owns
- **THEN** the exchange grants nothing for that identifier, because no policy scopes it
  to this repository

#### Scenario: a release published from an unregistered repository fails
- **GIVEN** a repository with no trusted publishing policy
- **WHEN** its release workflow requests a key
- **THEN** the exchange is refused and nothing is published

### Requirement: The version is written once, read everywhere

Release Please SHALL write the computed version to a single file in the repository, and
the packaging verb SHALL read it from there. No project file, command line or workflow
SHALL carry a version of its own: a second source of version truth is the one that goes
stale, and it goes stale silently.

A package built outside a release SHALL carry a prerelease suffix, so that a continuous
integration artefact cannot be mistaken for a released version.

#### Scenario: a continuous integration build is not a release
- **GIVEN** a build of `develop`
- **WHEN** it packs
- **THEN** the version carries a `develop.<run>` prerelease suffix, and nothing is pushed

### Requirement: A repository's forge settings are part of its bootstrap

A repository is not provisioned when its files are committed: settings that live only in
the forge decide whether its own workflows can run. They SHALL be applied when the
repository is created, by the same automated step that creates it, and SHALL be verified
by the ecosystem audit — a setting applied by hand once is a setting that differs on the
nineteenth repository.

At minimum:

- **Default workflow permissions SHALL remain read-only.** A job that needs more declares
  it, per job, in the workflow.
- **Workflows SHALL be permitted to create pull requests.** Release Please prepares a
  release as a pull request; without this the release run fails with *GitHub Actions is
  not permitted to create or approve pull requests*, which no workflow can grant itself.
- **Both long-lived branches SHALL carry their ruleset** — linear history, signed commits,
  no force-push, no deletion, administrators included.

The audit SHALL report a repository whose forge settings diverge from these, in the same
way it reports harness drift: the settings are part of the harness, they simply do not
live in the tree.

#### Scenario: a freshly created repository can run its own release flow
- **GIVEN** a repository created by the provisioning step
- **WHEN** a push to `master` runs the release workflow
- **THEN** Release Please opens its pull request, and the default token stays read-only

#### Scenario: a hand-edited setting is reported
- **GIVEN** a repository whose workflow permissions were widened by hand
- **WHEN** the ecosystem audit runs
- **THEN** it reports that repository and the setting that diverges
