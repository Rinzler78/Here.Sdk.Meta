# Project Brief

## Purpose

- Orchestration cockpit of the `Here.Sdk.*` ecosystem: a .NET wrapper family around the HERE SDK, spanning Xamarin to current .NET.
- It exists as a commercial showcase of .NET craft for a freelance activity. Without it, the ecosystem has no shared specification, release order or audit.
- It publishes no package. It holds the specification, the realisation order and the ecosystem-level obligations.

## Users

- Owner: the freelance developer who builds the ecosystem and uses it as proof of skill.
- Prospects and clients: readers who judge .NET craft from the published repositories and demonstrations.
- Agents: AI assistants working on one repository at a time.

## Main journey

1. Search an address.
2. Compute a route.
3. Follow it with guided navigation.

The same journey is replayed in every demonstration application, from a raw binding to a MAUI application.

## Domain language

| Term | Meaning |
| --- | --- |
| Cockpit | This repository: orchestration, specification and audit, never a package. |
| Native column | A platform stack (Android, iOS, JavaScript) that references only its own binding. |
| Façade | The single multi-targeted adaptation layer placed above the native columns. |
| Cascade | Topological, wave-by-wave release across repositories. |
| Demonstration | An application replaying the canonical journey at one layer. |

## Key features

| Feature | Entry |
| --- | --- |
| Ecosystem specification | `openspec/changes/001-ecosystem-foundation/proposal.md` |
| Realisation order and task state | `openspec/changes/001-ecosystem-foundation/tasks.md` |
| Publication procedure | `docs/runbooks/nuget-publication.md` |

## Business rules

- Every arbitration is settled by what the choice proves to a client, never by what is most modern.
- Nothing is published in stable form before the abstraction is extracted from the natives.
- A native wrapper references only its own binding.
- The cockpit never owns another repository's decision.

## Out of scope

- Publishing any package from this repository.
- Replacing the HERE SDK or solving a mapping need.
