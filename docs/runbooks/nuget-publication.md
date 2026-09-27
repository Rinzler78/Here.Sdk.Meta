# Runbook — publishing a repository to nuget.org

The procedure, and the reasons behind each step, for giving one repository of the
ecosystem the ability to publish its packages. It was first carried out on
`Rinzler78.Toolkit` on 2026-09-24 with Release Please, and moved to tag-driven releases
on 2026-09-26; every trap below was met there.

Publication uses nuget.org trusted publishing over GitHub OIDC: **no long-lived key
exists** — not in a repository secret, not on a machine. The only key is the one the
exchange returns, valid for one hour, held in the publishing job's environment for the
duration of that job and never written anywhere. Each step below closes a specific way in
which that could go wrong; none of them is optional.

## What the template already provides

A repository generated from `Rinzler78.Templates` arrives with:

- `.github/workflows/release.yml` — triggered by a tag `v*`; checks the tag, verifies the
  tree, asks GitHub for an OIDC token **immediately before the push**, publishes, and
  creates the GitHub release with generated notes;
- `scripts/_release-tag.sh` — refuses a tag outside `vMAJOR.MINOR.PATCH[-(alpha|beta|rc).N]`,
  a lightweight tag, a tag whose signature GitHub does not verify, and a tag on a commit
  `master` does not contain;
- MinVer, in `Directory.Packages.props` — the version comes from the tags alone;
- `scripts/publish.sh` — refuses any package whose manifest does not declare
  `EXPECTED_VERSION`, the tag's version; the release workflow runs that check on its
  own, before it requests the credential;
- `scripts/_provision-forge.sh` — every forge setting below, idempotent.

Nothing in the tree needs editing. Everything below lives outside it.

## Step 1 — sign commits in the clone

The rulesets require signed commits, and GitHub refuses to merge a pull request carrying
an unsigned one — `mergeStateStatus: BLOCKED`, with `--admin` as the only suggestion,
which the rulesets do not honour. The key is `~/.ssh/Rinzler78-GitHub.pub`, registered on
the account as a *signing* key. Per clone, before the first commit:

```bash
git config gpg.format ssh
git config user.signingkey ~/.ssh/Rinzler78-GitHub.pub
git config commit.gpgsign true
git config tag.gpgsign true
```

## Step 2 — the `master` branch

A repository cut from the template has only `develop`. Create `master` from it once:

```bash
git push origin develop:master
```

This is the one direct push the flow allows, and `release-flow` names it: it creates a
branch rather than changing one, before any promotion can exist. From then on `master`
moves only through promotion pull requests from `develop`, merged with a merge commit.

## Step 3 — forge settings

```bash
bash scripts/_provision-forge.sh Rinzler78/<name>
```

It applies, and re-applies without harm:

- a read-only default workflow token that cannot create or approve pull requests;
- the `release` environment, admitting tags `v*` only — any other deployment policy is
  deleted, not merely outnumbered;
- `develop`: pull request, squash only, linear history, signed, `verify` and `lint`
  required and bound to GitHub Actions;
- `master`: pull request, merge commits only, signed, the same checks, no linear
  history — a promotion keeps `develop`'s history;
- Copilot reviewing every push to a pull request into either branch;
- tags `v*`: neither updatable nor deletable by anyone, and created by administrators
  only — two rulesets, because a bypass applies to a whole ruleset.

The required checks are `verify` (build and tests) and `lint` (static quality), the jobs
of the harness's `ci.yml`. The domain reviewers `release-flow` also requires,
`spec-reviewer` and `package-api-reviewer`, join them in the script when they exist;
no task schedules them yet. A check the ruleset requires must exist: a repository without that
workflow — `Here.Sdk.Meta` before it is generated from the template — would have every
pull request blocked on checks that never run. Provision it once it carries the CI.

**Why the environment is not optional.** A nuget.org policy matches the repository owner,
the repository, the workflow's *file name* and the environment — **never the ref**.
Without an environment, any ref carrying a file named `release.yml` can mint a key, and
the `push: tags` trigger cannot prevent it, because it lives in the very file a branch
could rewrite. GitHub also silently *creates* an unprotected environment the first time a
job names one.

## Step 4 — the nuget.org policy

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

## Step 5 — a release

1. Promote: a pull request from `develop` to `master`, merged with a merge commit. Its
   checks are the costly ones; nothing expensive is discovered after this point.
2. When certain, tag the merge commit and push the tag:

   The tag's message is the version, then the evidence the proposal cited — the
   Conventional Commits since the previous tag, or the surface diff:

   ```bash
   git fetch origin --tags
   # Empty before a repository's first release: the log then covers all history.
   previous=$(git describe --tags --abbrev=0 origin/master 2>/dev/null || true)
   { echo v0.2.0; echo; git log --no-merges --format='- %s' "${previous:+$previous..}origin/master"; } |
     git tag --sign --annotate v0.2.0 --file - origin/master
   git push origin v0.2.0
   ```

   For a version derived from a public API, the evidence is the surface diff: replace
   the `git log` line with
   `git diff "${previous:-$(git hash-object -t tree /dev/null)}" origin/master -- '*PublicAPI.Unshipped.txt'`,
   against the empty tree before the first release. **1.0.0 after a 0.x version** is
   accepted only with an ADR of the repository declaring its public surface stable:
   add that ADR's path to the message; without it the release check refuses 1.0.0.

   The push publishes, directly. There is no draft and no second gesture: the tag check,
   the immutable tags and the promotion pull request are what bound an irreversible push.
3. The workflow checks the tag, verifies the tree — locked restore, build, test, every
   repository check, pack — then exchanges its OIDC token and pushes. `publish.sh`
   refuses any artefact that does not carry the tag's version: a version pushed to
   nuget.org can never be replaced. A mistaken tag is corrected with a new number.

## The ID prefix reservation

Done once for the whole ecosystem, not per repository. There is no form: the documented
procedure (nuget.org documentation, *ID Prefix Reservation*, "application process") is an
e-mail to NuGet's public account team, `account@nuget.org`, naming the owner display name (`Rinzler78`)
and the prefix (`Rinzler78.*`, private, no delegation), with evidence against the three
acceptance criteria. Send it **from the address registered on the nuget.org account**,
since the team may need to verify the requester's identity.

A reservation protects the prefix against third parties. It grants nothing to a workflow,
and it does not replace the per-repository policy.

## After a release

What a publication looks like, for comparison:

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
| A tag created by the workflow publishes nothing | GitHub starts no workflow for events raised with `GITHUB_TOKEN`, except `workflow_dispatch` and `repository_dispatch` | Tags are pushed by a repository administrator, the only role allowed to create them; this is why Release Please was removed |
| A pull request `BLOCKED` although every check is green | The ruleset requires signed commits and the branch carries unsigned ones | Step 1, then `git rebase --force-rebase --gpg-sign origin/develop` and `push --force-with-lease` on the feature branch |
| A rebase merge is refused on a branch requiring signatures | GitHub cannot sign the commits it rewrites | `develop` squashes, `master` merges; rebase is allowed on neither |
| A squashed promotion replays every earlier commit at the next one | Squash leaves the merge base where it was | Promotions are merge commits; `master` carries no linear-history rule |
| A key that pushes new versions but not a new package | Scope limited to existing packages | *Push new packages and package versions* |
| A feature branch could publish | The policy matches the file name, never the ref | The `release` environment, restricted to tags `v*` |
| The `release` environment still admits `master` after re-provisioning | Adding the tag policy left the old branch policy in place | `_provision-forge.sh` deletes every policy other than tags `v*` |
| Administrators could move a release tag | A bypass applies to every rule of its ruleset | Immutability and creation are two rulesets |
| Every release tag refused as lightweight | On a tag push, `actions/checkout` tests the tag by comparing the commit with `git rev-parse refs/tags/<tag>` — the tag object's SHA for an annotated tag — so the test always fails and a second fetch of `+<commit>:refs/tags/<tag>` rewrites it as lightweight (seen in the `v0.1.1` run's log) | `_release-tag.sh` fetches the tag from `origin` before reading it |
| A clone reports itself bare after a commit | A `git init` run from a hook inside a worktree inherits `GIT_DIR` and re-initialises the real repository with `core.bare=true` | Scripts and tests unset every inherited `GIT_*` variable; repair with `git config core.bare false` |

## Record of repositories provisioned

| Repository | Date | Packages in the policy | Forge settings | Policy |
|---|---|---|---|---|
| `Rinzler78.Toolkit` | 2026-09-24 | `Rinzler78.Build`, `Rinzler78.Templates` | `_provision-forge.sh`, 2026-09-26 | active — `0.1.0` published 2026-09-24; `0.1.1`, the first tag-driven release, 2026-09-27 |
