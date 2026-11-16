# VCS

## Platform

- Platform: GitHub, repository `Rinzler78/Here.Sdk.Meta`. Agents use `gh`.

## Branches

- `master` receives promotions only and carries (pre-)releases. `develop` is where work lands.
- Work happens on `feature/<kebab-slug>` cut from `develop`, in `.worktrees/<kebab-slug>/`. A branch is never reused.
- Hotfixes branch from `master`, then merge back into `develop`.

## Commits

- Conventional Commits, as in the history: `feat`, `fix`, `docs`, `chore`, `spec`.
- Commits are signed with the SSH key registered as a signing key. GitHub blocks an unsigned pull request.
- Agent names never appear in commits or pull requests.

## Pull requests

- Every change to `develop` or `master` goes through a pull request, submodule bumps and promotions included.
- Merge only at zero review findings, local and GitHub. Post the review verdict as a comment, then merge, pull and clean up.
- A release is a signed tag on `master`, and the tag publishes.

## Agent may

- Commit after each finished task, signed.
- Push a feature branch and open a pull request. Never commit or push directly to `develop` or `master`.
