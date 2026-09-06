# Changelog

Entries in the commit-message format (`X.Y.Z - description in English`, see
[FORK.md](FORK.md)), newest first. **Each `##` heading is literally the commit
subject.**

This file covers **fork-only commits**. Work destined for a pull request
upstream follows `akitaonrails/distrobox-gaming`'s own convention instead —
concise imperative subjects, no version prefix, no entry here. See
[FORK.md](FORK.md) for why the two coexist.

Bodies are narrative: what changed, why, and what was measured. This file is
never rewritten.

## 0.2.0 - record the macOS baseline and how the version tracks PR work

Second delivery on this fork: the native macOS baseline under `macos/`,
merged from `macos-baseline` and submitted as
[PR #2](https://github.com/akitaonrails/distrobox-gaming/pull/2). ES-DE plus
Dolphin, PCSX2 and PPSSPP over Homebrew, with its own Ansible playbook and no
code shared with the Linux tree.

Review of that work against the Ansible its own `Brewfile` installs turned up a
first-run blocker: the missing-library message read `item.invocation.module_args.path`,
and registered results stopped carrying `invocation` in ansible-core 2.19, so it
passed on 2.15 and failed on 2.21 with `object of type 'dict' has no attribute
'invocation'`. The task fires exactly when a ROM directory is absent — every new
user's first run — so `bootstrap.sh check` died before configuring anything.
Fixed by resolving the path from the loop item, and the integration test now has
a recorded pass on both 2.15.13 and 2.21.3. The Portuguese install guide was
translated to English as `macos/INSTALL.md`, since upstream's docs are English
throughout and the page has to be able to move into a PR unchanged.

`FORK.md` gains the clause repodocs requires to be written down: the version
here numbers fork deliveries, and a delivery that ships as an upstream PR is
recorded by the fork-only commit that follows it, not by the PR commit itself.
A PR commit carrying `X.Y.Z` would push the fleet's format into someone else's
`git log`, which is the thing the split convention exists to avoid.

Y rather than Z: a completed phase, per repodocs' bump criteria.

## 0.1.0 - adopt the repodocs standard for fork-only work

This is a fork of [akitaonrails/distrobox-gaming](https://github.com/akitaonrails/distrobox-gaming),
not a fleet repository, so the [samirhvbr/repodocs](https://github.com/samirhvbr/repodocs)
skeleton is adopted **in part**: `version.md`, this file, `FORK.md` and the
`commit-msg` hook. `README.md`, `CLAUDE.md`, `AGENTS.md`, `LICENSE` and `docs/`
are the upstream maintainer's and are deliberately left alone — overwriting them
with the fleet skeleton would destroy the very content a pull request has to
respect, and would make every future upstream sync a conflict.

The first delivery on this fork is the Atari work in `824a94f`, submitted as
[PR #1](https://github.com/akitaonrails/distrobox-gaming/pull/1): ES-DE systems
for the Atari 2600, 5200, 7800 and Lynx over cores `dg_retroarch_cores` already
carried, a `bin/retroarch-atari800` wrapper that resolves the 5200-vs-8-bit
machine type per launch, RetroArch system-BIOS symlinks in `link_storage`, and
an opt-in Atari ST install on the hatari core. That commit carries an upstream
imperative subject on purpose — it exists to be merged by someone else.

The version starts at `0.1.0` rather than tracking upstream: it numbers this
fork's own deliveries, and upstream publishes no version of its own to track.
