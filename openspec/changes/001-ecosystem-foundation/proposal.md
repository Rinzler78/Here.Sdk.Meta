# 001 — Ecosystem foundation

- **Status:** draft
- **Date:** 2026-09-12
- **Author:** @rinzler78
- **Scope:** ecosystem-wide (all repositories)

## Why

The previous attempt at the `Here.Sdk.*` ecosystem stalled after four months with
nothing published on nuget.org, seven of nine repositories private, a governance
ADR contradicted by its own tooling twelve days after acceptance, and a release
pipeline that failed silently on its only tag. The root cause was not any single
defect: decisions were taken in isolation, recorded inconsistently, and never
reconciled against each other.

This proposal replaces the whole ecosystem from zero. Nothing of the existing
content survives. No package was ever published, so the identifiers start clean.

## What the ecosystem is for

It is a **commercial showcase**. It exists to prove .NET craft to prospects and
clients — not to solve a mapping need. Every arbitration is settled by *what does
this prove to a client*, never by *what is most modern*. That rule is why Xamarin
stays in scope: clients run Xamarin applications in production, and being able to
serve them is market proof.

## What changes

- **18 repositories, 28 packages, 31 demonstration applications.** Applications
  outnumber packages because a package with several project systems — MAUI's four
  heads, Xamarin.Forms' two — is demonstrated once per head.
- A generic engineering toolkit — `Rinzler78.Templates` and `Rinzler78.Build` —
  extracted from the ecosystem and published on its own, with no mention of HERE.
- An eight-layer architecture in which a native wrapper references **only its own
  binding**, and standardisation is an adaptation layer placed above it, exposing a
  single multi-targeted façade.
- Guided navigation available on **all five targets**: the native SDK where it
  exists, a REST guidance engine everywhere else. Only offline maps, HERE
  positioning and lane guidance remain native-exclusive.
- Six presentation idioms driven by one neutral engine, each justified by an ADR in
  its own repository.
- A frozen Xamarin toolchain operated on a self-hosted runner, treated as a subject
  of demonstration rather than a liability.

## Impact

- **Destructive.** All existing repositories are archived (`git bundle --all`,
  plus issue and pull-request export) and then deleted. Archives are a safety net,
  not a source of reuse.
- **Two orders, not one.** Construction builds the native columns first and extracts
  the contracts from them; publication is strictly topological. Prerelease versions
  close the loop during extraction.
- **Sequential.** Repositories go public one at a time, in dependency order, only
  when conforming, and **nothing is published in stable form before the abstraction
  is extracted**.
- **Neither Xamarin head can reach a public store.** Apple has required Xcode 16
  since 24 April 2025 and Google Play target API 36 since 31 August 2026; both
  exceed what the frozen chain produces. Both ship as signed release artifacts, with
  the cause dated in their `README.md`.
- **Prerequisite, not an afterthought.** One HERE credential set per demonstration
  bundle identifier, a domain-restricted key for the public web pages, and the
  Android and Apple signing material, all provisioned before the publication phase.

## Affected capabilities

- `ecosystem-architecture` — ADDED
- `package-topology` — ADDED
- `coding-principles` — ADDED
- `demonstration` — ADDED
- `agent-harness` — ADDED
- `toolchain` — ADDED
- `secrets` — ADDED
- `release-flow` — ADDED
- `quality` — ADDED
- `testing` — ADDED
- `sdk-updates` — ADDED
- `openspec-methodology` — ADDED
