# sdk-updates

## ADDED Requirements

### Requirement: Upstream releases are detected from a machine-readable feed

A scheduled job in `Meta` SHALL detect a new HERE SDK release by reading the
releases of the public `heremaps/here-sdk-examples` repository through the GitHub
API, against a committed cursor. That repository publishes one dated release per
SDK version — twelve over the last ten months, roughly one every three weeks, some
carrying an `_LTS` suffix — and requires no credential.

The job SHALL open an issue in the affected binding repository naming both versions,
labelled `needs-triage` and `chore`. It SHALL NOT open a branch, a pull request or a
version change: an SDK upgrade is not a dependency bump.

The HERE documentation release notes SHALL be linked from the issue, never parsed.
That site is a single-page application whose Android and iOS release-note URLs
return byte-identical documents; a scraper against it would break silently.

The JavaScript column SHALL be checked separately. It versions independently, is
served from `js.api.here.com`, and is absent both from that repository and from the
npm registry, where `@here/maps-api-for-javascript` returns 404.

#### Scenario: a new release is surfaced, not applied
- **GIVEN** HERE publishes a new SDK version and the examples repository tags it
- **WHEN** the scheduled job runs
- **THEN** it opens an issue in the affected binding repository naming both versions
- **AND** no branch, no pull request and no version change is produced automatically

#### Scenario: the release notes are not scraped
- **GIVEN** the HERE documentation site changes its markup
- **WHEN** the detection job runs
- **THEN** it is unaffected, because it reads the GitHub API and only links the notes

### Requirement: The three columns are pinned independently, by decoupling

Each binding repository SHALL pin the exact upstream artefact it projects — the
Android AAR version, the iOS `.xcframework` version, the HERE Maps API for
JavaScript version — in a single committed manifest, never resolved as a floating
range.

The three columns SHALL be pinned independently so that no column can block another.
This is a decoupling decision, not an observation about HERE's release cadence: the
examples repository tags a **single** version covering Android, iOS and Flutter
together, so the two native columns are in fact released in lockstep. Only the
JavaScript column versions on its own rhythm.

#### Scenario: one column upgrades while another does not
- **GIVEN** a proposal raising the Android AAR
- **WHEN** it merges
- **THEN** the iOS and JavaScript pins are untouched, and neither column rebuilds

### Requirement: The upstream surface has a committed baseline where one can exist

Constating what changed upstream SHALL NOT require redoing the binding first.

For iOS, the Objective Sharpie output — `ApiDefinition.cs` and `StructsAndEnums.cs`
— SHALL be committed. Regenerating against the new `.xcframework` makes the upstream
change a plain `git diff`.

For Android, the `api.xml` produced by the binding generator SHALL be committed as a
baseline. The generator's own output is intermediate and discarded, so without this
deliberate commit there is nothing to diff.

For JavaScript there is no generator, no npm package and no type definitions, so no
mechanical verdict exists. The proposal SHALL record that the verdict was a human
reading of the release notes, rather than implying a diff that was never produced.

#### Scenario: the iOS change is visible before the work starts
- **GIVEN** a new `.xcframework` version
- **WHEN** Objective Sharpie is re-run and the output committed on a branch
- **THEN** the diff lists what appeared, disappeared and changed shape
- **AND** the proposal can state the effect on the contracts before binding by hand

#### Scenario: the JavaScript verdict is declared manual
- **GIVEN** a proposal raising the HERE Maps API for JavaScript version
- **WHEN** it is reviewed
- **THEN** it states that no mechanical surface diff exists for this column
- **AND** names the release-note entries it relied on

### Requirement: The projected surface is enforced by an analyser

Every project shipping a public surface SHALL reference
`Microsoft.CodeAnalysis.PublicApiAnalyzers` and SHALL commit
`PublicAPI.Shipped.txt` and `PublicAPI.Unshipped.txt`.

Any change to a public surface then becomes a compilation error until the file is
updated. The verdict is produced by the compiler, committed, and reviewable in the
pull request; the diff of `PublicAPI.Unshipped.txt` **is** the list of what changed.
No bespoke comparison tool SHALL be written for this.

The two artefacts are not redundant. `api.xml` and the Sharpie output say what HERE
changed, before anything is decided. `PublicAPI.*.txt` says what we expose, after
transformation.

#### Scenario: an unannounced surface change cannot merge
- **GIVEN** a branch that removes a public member
- **WHEN** it is compiled
- **THEN** the build fails until `PublicAPI.Unshipped.txt` records the removal

### Requirement: The surface diff decides the version, not the commit label

The published version SHALL be derived from the `PublicAPI.Unshipped.txt` diff: a
removed or reshaped member forces a major, an added member a minor, an empty diff a
patch. A release whose version contradicts its diff SHALL fail its own release
check.

Conventional Commits remain the authoring convention, but they SHALL NOT be the
authority: a mislabelled commit would otherwise publish a breaking change as a minor
and defeat the only signal downstream repositories receive.

#### Scenario: a mislabelled breaking change is caught
- **GIVEN** a commit prefixed `fix:` whose diff removes a public member
- **WHEN** the release check runs
- **THEN** it fails, naming the removed member and the version the diff requires

### Requirement: Propagation travels with the package, never through the cockpit

Repositories are autonomous. `Meta` SHALL detect, inform and aggregate; it SHALL NOT
open a proposal, own a decision or choreograph work on another repository's behalf.
The protocol between repositories is the published package and its version.

Propagation therefore takes two shapes, selected by the verdict:

- **Empty diff** — the upgrade changed nothing observable. The binding republishes,
  the topological cascade opens the bump pull requests in the next wave, lock files
  are regenerated, and they merge on green. No repository above has any work.
- **Non-empty diff** — the projected surface moved. The published major is the
  signal. The bump pull request SHALL carry the generated surface diff as a
  diagnostic attachment, so the downstream repository receives what changed rather
  than a red build to decompose. The decision to accept, adapt or defer stays with
  that repository, which opens its own proposal in its own OpenSpec tree.

#### Scenario: a patch upgrade costs nothing above
- **GIVEN** an SDK upgrade whose `PublicAPI.Unshipped.txt` diff is empty
- **WHEN** the binding publishes
- **THEN** the cascade opens bump pull requests that merge on green
- **AND** no proposal is opened in any repository above

#### Scenario: a downstream repository decides for itself
- **GIVEN** a binding publishes a major after a surface change
- **WHEN** the bump pull request reaches `Here.Sdk.Standard`
- **THEN** it carries the surface diff as a diagnostic
- **AND** the proposal adapting the contracts is authored in `Here.Sdk.Standard`,
  not in `Meta`

### Requirement: Behaviour changes at constant surface are caught by tests

The analyser sees surface alone. An upgrade that changes a computed route, an
isoline radius or the meaning of an error code passes it untouched.

Such changes SHALL be caught by the integration tests running against real HERE
credentials and by the canonical scenario replayed across the eighteen heads. A
proposal raising a pinned version SHALL NOT merge on an empty surface diff alone;
both suites SHALL have run.

#### Scenario: a silent behaviour change is caught
- **GIVEN** an upgrade whose surface diff is empty but which reorders route legs
- **WHEN** the integration suite and the canonical scenario run
- **THEN** the assertion on the expected itinerary fails
- **AND** the proposal must state the behavioural change before it can merge
