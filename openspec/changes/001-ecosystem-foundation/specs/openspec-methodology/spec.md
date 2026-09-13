# openspec-methodology

## ADDED Requirements

### Requirement: The issue is the unit of work, the proposal is the specification

Work SHALL enter through an issue and SHALL be specified by an OpenSpec change. The
issue says *what is wanted and why now*; the change says *what the system will be
required to do*. Neither replaces the other.

An issue SHALL NOT reach `ready-for-agent` unless the behaviour it changes is
covered by an accepted change, or it is a `chore` or `docs` item that alters no
requirement.

#### Scenario: a behavioural issue without a proposal is held
- **GIVEN** an issue labelled `feat` with no linked OpenSpec change
- **WHEN** `ready-for-agent` is applied
- **THEN** the triage workflow removes the label and asks for the proposal

### Requirement: Every repository owns its OpenSpec tree

Each repository SHALL carry `openspec/specs/` for its living capability
specifications and `openspec/changes/` for proposals in flight. `Meta` SHALL hold
proposals whose scope is **ecosystem-wide only**; a change touching one repository
belongs to that repository.

#### Scenario: a single-repository change is not raised in Meta
- **GIVEN** a proposal affecting only `Here.Sdk.Rest`
- **WHEN** it is opened
- **THEN** it lives in that repository's `openspec/changes/`, not in `Meta`

### Requirement: A change is a directory with four parts

A change SHALL be a directory `openspec/changes/<nnn>-<slug>/` containing:

- `proposal.md` — why, what changes, and the impact, including whether it is destructive.
- `design.md` — the reasoning that settled each decision, and the alternatives
  rejected with their motive. Recording only the conclusion loses what a later
  reader needs most.
- `tasks.md` — checkboxes in execution order.
- `specs/<capability>/spec.md` — the deltas, using `## ADDED`, `## MODIFIED` and
  `## REMOVED Requirements`.

Every requirement SHALL use SHALL or MUST and SHALL carry at least one scenario in
`GIVEN / WHEN / THEN` form. A requirement without a scenario is an intention, not a
specification.

#### Scenario: an unscenarioed requirement is refused
- **GIVEN** a change containing a requirement with no scenario
- **WHEN** `openspec validate --changes --strict` runs in CI
- **THEN** it fails and the pull request is blocked

### Requirement: Task boxes reflect reality

The checkboxes of `tasks.md` SHALL be ticked as the work completes, in the same
pull request that performs it. A proposal whose boxes contradict the state of the
repository SHALL be reported by the **ecosystem audit**.

This requirement exists because the previous attempt carried a proposal that was
three-fifths done with none of its five boxes ticked, which made every status
report worthless.

#### Scenario: an unticked completed task is caught
- **GIVEN** a change whose tasks are implemented and merged
- **WHEN** the ecosystem audit compares the boxes against the merged pull requests
- **THEN** it reports the divergence as an issue in the owning repository

### Requirement: Accepted deltas are promoted and archived

When a change is fully implemented, its deltas SHALL be promoted into
`openspec/specs/<capability>/spec.md` and the change directory SHALL be moved to
`openspec/changes/archive/`. A change SHALL NOT remain open with all its tasks done.

#### Scenario: a finished change does not linger
- **GIVEN** a change whose tasks are all ticked and merged
- **WHEN** the ecosystem audit runs
- **THEN** it reports the change as awaiting promotion and archival

### Requirement: `to-spec` writes into the OpenSpec tree

The `to-spec` skill SHALL be configured, in every repository, to write into
`openspec/changes/<slug>/` in the format above. It SHALL NOT create a parallel
specification directory.

#### Scenario: a generated specification lands in the right place
- **GIVEN** an agent invoking `to-spec`
- **WHEN** the skill writes its output
- **THEN** the files appear under `openspec/changes/`, and `openspec validate` accepts them

### Requirement: Architecture decisions are recorded uniformly

An ADR SHALL be recorded whenever a decision constrains future work and its reason
would otherwise be lost — a presentation pattern, a trimming contract value, a
coverage contract value, a deviation from a platform's official guidance, a frozen
toolchain, a rejected alternative.

Conventions, identical in every repository:

- Path `docs/architecture/decision-records/ADR-NNNN-<kebab-slug>.md`, numbered from
  `0001` within the repository. Ecosystem-wide ADRs, held by `Meta`, use the `ADR-M`
  prefix.
- Sections: **Status** (`proposed`, `accepted`, `superseded by ADR-NNNN`), **Date**,
  **Deciders**, **Context**, **Decision**, **Consequences**.
- An ADR is **never edited to reverse it**. It is superseded by a new one, and both
  remain readable. The previous ecosystem contradicted an accepted ADR twelve days
  later with no superseding record; that must not recur.
- A decision that contradicts an accepted ADR without superseding it SHALL fail the
  ecosystem audit.

#### Scenario: a reversal creates a record rather than erasing one
- **GIVEN** `ADR-0004` accepted, and a later decision that contradicts it
- **WHEN** the new decision is recorded
- **THEN** `ADR-0009` is created stating the reversal and its reason
- **AND** `ADR-0004` is marked `superseded by ADR-0009` and its body is left intact

#### Scenario: an orphaned contradiction is detected
- **GIVEN** tooling committed to a repository that contradicts an accepted ADR
- **WHEN** the ecosystem audit runs
- **THEN** it opens an issue naming both the ADR and the contradicting artefact
