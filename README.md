# Here.Sdk.Meta

The orchestration cockpit of the `Here.Sdk.*` ecosystem — a .NET wrapper family
around the HERE SDK 4.x, built as a demonstration of .NET craft across four
generations of the platform, from Xamarin to .NET 10.

This repository publishes no package. It holds the ecosystem-level specification,
the realisation order, and the five obligations that make a cockpit something other
than documentation.

## What the ecosystem is

Nineteen repositories publishing twenty-nine packages, demonstrated by thirty-one
applications. One canonical journey — search an address, compute a route, follow it
with guided navigation — replayed at thirty-one altitudes, from a raw binding
projection to a MAUI application with a provider selector.

Three native columns stand alone and reference nothing but their own binding:
Android, iOS, and the HERE Maps API for JavaScript. Standardisation is an adaptation
layer placed **above** them, exposing a single multi-targeted façade. Guided
navigation reaches all five targets: the native SDK where it exists, a REST guidance
engine everywhere else.

Two generations coexist deliberately. The Xamarin chain is frozen, operated on a
self-hosted runner with a pinned Xcode, and treated as a subject of demonstration
rather than a liability — neither Xamarin head can reach a public store, and the
specification says so and explains why.

## The five obligations of the cockpit

1. **The release cascade** — topological, wave by wave, waiting for each package to
   become restorable before opening the next wave's bump pull requests.
2. **Nightly integration from sources** — the whole graph built with each repository
   against the others' `develop`, which is the only mechanism that catches an API
   break before it reaches nuget.org. No repository's own pipeline can see it.
3. **The aggregated board and the demonstration gallery.**
4. **The ecosystem audit** — one nightly job checking catalogue containment,
   demonstration coverage, task-box truth, ADR integrity, harness drift, and version
   literals that escaped into specifications.
5. **Ecosystem-level decision records**, including the one that argues the
   poly-repository choice. Without it, a cockpit reads as a cure for a self-inflicted
   disease.

## Layout

    openspec/changes/   the specification, as OpenSpec change proposals
    docs/runbooks/      procedures carried out outside any tree — forge settings,
                        registry policies — written down the first time they are done
    repos/              every repository of the ecosystem, as a git submodule

`docs/runbooks/nuget-publication.md` gives a repository the right to publish: its forge
settings, its `release` environment, and its keyless nuget.org policy, with the traps
the first repository met.

Submodules track `develop`. The commit a submodule pins is a bootstrap default, not
a dependency: the nightly refreshes them to the tips before building, because a pin
that is never refreshed turns the job designed to catch drift into a job that
reproduces it.

    git clone --recurse-submodules https://github.com/Rinzler78/Here.Sdk.Meta.git

## Status

Phase 0 of `001-ecosystem-foundation` is complete: the previous attempt was
archived and deleted, and this repository was re-founded on the change that
justifies it. Phase 1 is in progress — the toolkit exists, the templates do not.

Read `openspec/changes/001-ecosystem-foundation/proposal.md` for what is being built
and why, and `design.md` for the reasoning behind the decisions, including the
alternatives that were rejected and what each of them would have cost.

## Licence

MIT.

Not affiliated with, endorsed by, or supported by HERE Technologies. Consumers must
obtain their own HERE credentials and comply with HERE's Developer Agreement
independently.
