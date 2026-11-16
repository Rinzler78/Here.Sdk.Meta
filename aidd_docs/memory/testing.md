# Testing

## Layers

| Change | Test |
| --- | --- |
| Specification or runbook | Review against the spec and its scenarios. |
| Orchestrator logic, once it exists | Unit tests, as the specification requires. |

## Conventions

- Unit tests target a portable runtime and never need a device, an emulator or a simulator.
- One test framework across the ecosystem: xUnit v3.
- Code hard to test is code in the wrong place: move it, never add a harness to test it where it stands.
- Meta is exempt from coverage, mutation, trimming and API-documentation gates, because it has no public assembly.
- Narrowest run: only the tests a change can reach.
