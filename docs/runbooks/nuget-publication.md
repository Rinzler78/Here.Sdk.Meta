# Runbook — publishing a repository to nuget.org

The procedure, and the reasons behind each step, for giving one repository of the
ecosystem the ability to publish its packages. It was first carried out on
`Rinzler78.Toolkit` on 2026-09-24, and every trap below was met there.

Publication uses nuget.org trusted publishing over GitHub OIDC: **no long-lived key
exists** — not in a repository secret, not on a machine. The only key is the one the
exchange returns, valid for one hour, held in the publishing job's environment for the
duration of that job and never written anywhere. Each step below closes a specific way in
which that could go wrong; none of them is optional.

## What the template already provides

A repository generated from `Rinzler78.Templates` arrives with:

- `.github/workflows/release.yml` — Release Please on `master`, then a `publish` job that
  runs in the `release` environment, asks GitHub for an OIDC token **immediately before
  the push**, and hands the one-hour key to `scripts/publish.sh`;
- `release-please-config.json` — `release-type: simple`, `initial-version: 0.1.0`;
- `version.txt`, `.release-please-manifest.json` — seeds, owned by the repository;
- `scripts/package.sh` — reads `version.txt`, and `VERSION_SUFFIX` for prereleases;
- `scripts/publish.sh` — refuses any package whose file name does not carry
  `EXPECTED_VERSION`, the version the release claims.

Nothing in the tree needs editing. Everything below lives outside it.

## Step 1 — forge settings, by API

Run once per repository, right after it is created. `REPO` is `Rinzler78/<name>`.

```bash
REPO=Rinzler78/<name>

# Workflows may open pull requests — Release Please cannot open its release pull request
# otherwise, and no workflow can grant itself that right. The default token stays
# read-only: a job that needs more declares it, per job.
gh api -X PUT "repos/$REPO/actions/permissions/workflow" \
  -f default_workflow_permissions=read -F can_approve_pull_request_reviews=true

# The `release` environment, admitting deployments from `master` only.
echo '{"deployment_branch_policy":{"protected_branches":false,"custom_branch_policies":true}}' |
  gh api -X PUT "repos/$REPO/environments/release" --input -
gh api -X POST "repos/$REPO/environments/release/deployment-branch-policies" \
  -f name=master -f type=branch
```

Verify:

```bash
gh api "repos/$REPO/actions/permissions/workflow"
# {"default_workflow_permissions":"read","can_approve_pull_request_reviews":true}
gh api "repos/$REPO/environments/release/deployment-branch-policies" --jq '.branch_policies[]|"\(.type): \(.name)"'
# branch: master
```

**Why the environment is not optional.** A nuget.org policy matches the repository owner,
the repository, the workflow's *file name* and the environment — **never the branch**.
Without an environment, any branch carrying a file named `release.yml` can mint a key: a
feature branch that rewrites that file to trigger on its own push would publish without
review. The `push: branches: [master]` trigger cannot prevent it, because it lives in the
very file the branch rewrites.

## Step 2 — the `master` branch

The release workflow triggers on `master`. A repository cut from the template has only
`develop`; create `master` from it once, **before** the rulesets are applied — once they
are, `master` only moves through the promotion the ruleset allows:

```bash
git push origin develop:master
```

## Step 3 — the nuget.org policy

nuget.org → profile menu → **Trusted Publishing** → *Create*. One policy per repository.

| Field | Value |
|---|---|
| Policy Name | `<repository> release` |
| Package Owner | `Rinzler78` |
| CI/CD Provider | GitHub Actions |
| Repository Owner | `Rinzler78` |
| Repository | the repository name |
| Workflow File | `release.yml` — the file name only, no path |
| Environment | `release` |
| Scopes | **Push** → *Push new packages and package versions* |
| Unlist or relist | **unchecked** |
| Glob Patterns and Packages | the repository's package identifiers, **exactly**, one per line |

**Exact identifiers, never a namespace glob.** A policy is a grant to whatever workflow
matches it. `Rinzler78.*` on nineteen repositories would let a compromise of any one of
them publish or replace every package of the ecosystem. The field accepts several
identifiers, one per line, so one policy per repository is enough.

**List only what the repository actually packs.** Run `./scripts/package.sh` and read the
file names: a repository's name is not a package identifier. `Rinzler78.Toolkit` packs
`Rinzler78.Build` and `Rinzler78.Templates`, and no `Rinzler78.Toolkit` package exists.

**Push new packages as well as new versions.** A scope limited to existing packages
cannot push a new identifier, and every identifier here is new at its first release.

**Leave unlist unchecked.** The release does not need it, and it is exactly the right a
compromised workflow would use to make a version disappear.

The saved policy is pinned to the **permanent GitHub IDs** of the owner and the
repository, not only to their names: deleting the repository and recreating one with the
same name does not make the policy usable again. A policy on a **private** repository
starts as pending for seven days and becomes inactive if nothing is published in that
window; a public repository's policy is active immediately.

## Step 4 — the first release

1. Push to `master`. The release workflow runs and Release Please opens
   `chore(master): release 0.1.0`, changing exactly `version.txt`,
   `.release-please-manifest.json` and `CHANGELOG.md`.
2. Merge that pull request. Release Please tags and creates the GitHub release, and the
   `publish` job runs in the `release` environment.
3. The job verifies the tree — locked restore, build, test, every repository check, pack
   — then exchanges its OIDC token and pushes. `publish.sh` refuses any artefact that
   does not carry the release's version: a version pushed to nuget.org can never be
   replaced.

## The ID prefix reservation

Done once for the whole ecosystem, not per repository. There is no form: the documented
procedure (nuget.org documentation, *ID Prefix Reservation*, "application process") is an
e-mail to NuGet's public account team, `account@nuget.org`, naming the owner display name (`Rinzler78`)
and the prefix (`Rinzler78.*`, private, no delegation), with evidence against the three
acceptance criteria. Send it **from the address registered on the nuget.org account**,
since the team may need to verify the requester's identity.

A reservation protects the prefix against third parties. It grants nothing to a workflow,
and it does not replace the per-repository policy.

## After the first release

`master` now carries the release commit that `develop` does not. Bring `develop` level by
fast-forward, so that its `version.txt` and `CHANGELOG.md` match and both branches point
at the same commit:

```bash
git push origin <release-commit>:refs/heads/develop
```

What the first publication looked like, for comparison:

```
==> 2 package(s), all at version 0.1.0
Pushing Rinzler78.Build.0.1.0.nupkg ...      Created   Your package was pushed.
Pushing Rinzler78.Templates.0.1.0.nupkg ...  Created   Your package was pushed.
```

`Created` means accepted, not available. nuget.org validates and indexes asynchronously:
the flat container served both packages about four minutes after the push, the search
index later still. Do not announce a package as available until
`https://api.nuget.org/v3-flatcontainer/<id-lowercase>/index.json` lists the version.

## Traps met on the first repository

| Symptom | Cause | Fix |
|---|---|---|
| `release-please--branches--develop` appears after a push to `master` | Release Please targets the repository's **default** branch, not the branch that triggered the run | `target-branch: master` in the action — already in the template |
| `GitHub Actions is not permitted to create or approve pull requests` | A forge setting, not code | Step 1 |
| First release proposed as `1.0.0` | Release Please's default for a repository with no tag | `initial-version: 0.1.0` — already in the template |
| A key that pushes new versions but not a new package | Scope limited to existing packages | *Push new packages and package versions* |
| A feature branch could publish | The policy matches the file name, never the branch | The `release` environment, restricted to `master` |
| The release pull request shows no checks | GitHub starts no workflow for events caused by `GITHUB_TOKEN`, which is what Release Please opens its pull request with | Nothing unverified is published — the `publish` job re-verifies the tree before pushing — but a ruleset that *requires* checks on `master` would make the release pull request unmergeable; see the open question below |

## Open question: promoting `develop` to `master`

GitHub has no fast-forward merge method for pull requests: *squash* and *rebase* rewrite
the commits, so a promotion pull request makes `master` diverge from `develop` at every
release, and the release commit then has to travel back the same way. The first release
was promoted and back-merged by fast-forward pushes instead, which keeps both branches on
identical commits but cannot coexist with a rule requiring a pull request on either
branch. Until that is decided, the ruleset on both branches carries only what every
model agrees on: no deletion, no force-push, linear history, signed commits.

## Record of repositories provisioned

| Repository | Date | Packages in the policy | Forge settings | Policy |
|---|---|---|---|---|
| `Rinzler78.Toolkit` | 2026-09-24 | `Rinzler78.Build`, `Rinzler78.Templates` | yes | active — `0.1.0` published 2026-09-24 |
