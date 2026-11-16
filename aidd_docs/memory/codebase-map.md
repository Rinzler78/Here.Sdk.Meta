# Codebase Map

## Areas

- `openspec/changes/`: the specification, one change per proposal, with `proposal.md`, `design.md`, `tasks.md` and per-capability `specs/`.
- `docs/runbooks/`: procedures carried out outside any tree, such as forge settings and registry policies.
- `repos/`: every ecosystem repository as a git submodule, tracking `develop`.
- `.worktrees/`: one worktree per feature branch, ignored by git.

## Entry points

- Start reading at `openspec/changes/001-ecosystem-foundation/proposal.md`, then `design.md` for the reasoning.
- Clone with `git clone --recurse-submodules`.

## Packages

- None. This repository publishes nothing.
