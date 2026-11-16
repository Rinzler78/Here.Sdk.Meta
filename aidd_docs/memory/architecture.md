# Architecture

## Stack

- .NET with C#, targeting current .NET and the frozen Xamarin generations, because clients run both in production.
- Specification: OpenSpec change proposals, validated strictly.
- Generic engineering toolkit consumed as a submodule: `repos/Rinzler78.Toolkit`.

## Decisions

- Poly-repository: one repository per package family, argued by an ecosystem-level decision record so the cockpit is not a cure for a self-inflicted disease.
- Natives first: the abstraction is extracted from implementations, never designed against documentation.
- Guided navigation reaches every target: native SDK where it exists, a REST guidance engine elsewhere.
- Both Xamarin heads ship as signed release artifacts: neither can reach a public store.
- Toolkit stays domain-agnostic. Domain policy lives in a separate package.

## Constraints

- Build order and publication order differ: construction goes natives first, publication is topological.
- Unit tests never run on a device, so platform assemblies hold glue only.
- Meta publishes no package, so assembly-based gates do not apply to it.

## Placement

- Specification and decisions go under `openspec/changes/`.
- Procedures done outside any tree go under `docs/runbooks/`, written the first time they are done.
- Every other repository of the ecosystem is a submodule under `repos/`.

## Dependencies

- A native wrapper references only its own binding.
- The façade references the native columns, never the reverse.
- The cockpit references repositories only through submodules tracking `develop`.

## Gotchas

- A submodule pin is a bootstrap default, not a dependency. The nightly refreshes it to branch tips.
- After a merge in a child repository, advance its submodule here through a pull request.
- Specifications name a library, never its version: versions live in manifests.

## Flow

```mermaid
flowchart LR
  Natives --> Contracts --> Facade --> Presentation --> Demonstrations
```
