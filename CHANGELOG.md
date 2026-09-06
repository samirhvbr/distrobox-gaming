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
