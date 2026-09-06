# Fork conventions

This repository is a fork of
[akitaonrails/distrobox-gaming](https://github.com/akitaonrails/distrobox-gaming).
It exists to develop changes that are offered upstream as pull requests, and to
keep them alive here whether or not upstream takes them.

That dual purpose is the whole reason this file exists: **two commit conventions
apply, and which one you use depends on where the commit is going.**

## Which convention applies where

| Where | Convention | Example |
|---|---|---|
| A branch that will become a **PR upstream** | Upstream's, from its `AGENTS.md`: concise imperative subject, no version prefix | `Add Atari 2600/5200/7800/Lynx and Atari ST as ES-DE systems` |
| A commit that stays **only on this fork** | [repodocs](https://github.com/samirhvbr/repodocs): `X.Y.Z - description in English`, `version.md` bumped in the same commit, entry in `CHANGELOG.md` | `0.1.0 - adopt the repodocs standard for fork-only work` |

Upstream's `AGENTS.md` states its commit convention in writing. Sending it
commits in the fleet's format would push a foreign convention into someone
else's `git log`, and the point of a pull request is to be easy to accept.
repodocs itself only claims authority over fleet repositories; this is not one.

`tools/git-hooks/commit-msg` enforces the fleet format and would therefore
**reject a correctly-written PR commit**. It is shipped but not enabled. Enable
it only if you work exclusively on fork-only commits:

```sh
git config core.hooksPath tools/git-hooks
```

With it enabled, a PR commit needs the documented bypass:

```sh
REPODOCS_NO_HOOK=1 git commit
```

## The bump clause, overridden in writing

repodocs requires `version.md` to be bumped **in the same commit as the change
it describes**, and allows a repository to stamp the version some other way only
when its own documentation says so and says why. This is that statement.

Here, a delivery that ships as an upstream pull request is recorded by the
**fork-only commit that follows it**, not by the PR commit itself. The PR commit
carries an upstream imperative subject and no version, because a `X.Y.Z` subject
would push the fleet's format into someone else's `git log` — the exact thing
the two-convention split above exists to prevent. `version.md` therefore numbers
this fork's deliveries, and `CHANGELOG.md` names the PR each one went to.

## What is adopted from repodocs, and what is not

**Adopted:** `version.md` (first semver in the file is the authority),
`CHANGELOG.md` (each `##` heading is the fork-only commit subject), this file,
and the `commit-msg` hook.

**Not adopted:** `README.md`, `CLAUDE.md`, `AGENTS.md`, `LICENSE`, `NOTICE`,
`SECURITY.md` and everything under `docs/`. Those belong to the upstream
maintainer. Replacing them with the fleet skeleton would delete the content a
pull request has to respect and turn every future upstream sync into a
conflict — the opposite of what a fork is for.

New documentation written here follows **upstream's** `docs/` conventions, so
that a page can move into a PR unchanged.

## Staying current with upstream

```sh
git remote add upstream git@github.com:akitaonrails/distrobox-gaming.git
git fetch upstream
git rebase upstream/master        # or merge, if a PR branch is already open
```

Fork-only commits (`version.md`, `CHANGELOG.md`, this file, `tools/`) are the
only expected divergence on `master`. Keep it that way: anything else that lands
here and not upstream is either a PR waiting to be opened or drift.
