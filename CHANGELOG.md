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

## 0.3.0 - the Atari 5200 and 8-bit line boot with no BIOS file

Chasing a question about whether downloaded ROMs could go in this public fork
led back into the atari800 core's source, and it contradicted something the
Atari PR asserted twice: that `5200.ROM` was required and the core would not
boot without it.

libretro-atari800 **compiles in** Avery Lee's AltirraOS, a clean-room and
freely redistributable replacement for Atari's OS ROMs —
`atari800/src/roms/altirra_5200_os.c`, `altirraos_800.c`, `altirraos_xl.c`
and `altirra_basic.c` — selected through the `atari800_os_5200`,
`atari800_os_xl`, `atari800_os_800` and `atari800_basic_version` options. Of
everything `dg_retroarch_system_bios` links, only the Lynx's `lynxboot.img`
is genuinely required; the `handy` core bundles no equivalent.

`bin/retroarch-atari800` therefore stopped warning and started fixing: it
writes `atari800_os_5200 = "Original"` when the BIOS is present and
`"AltirraOS"` when it is not, turning a silently dead launch into a working
one. The 8-bit equivalents are documented rather than automated — someone
holding the real ROMs may prefer them, and AltirraOS compatibility is high
but not identical. The option write became a `set_opt` helper so a second key
did not duplicate the read-filter-replace logic.

Pushed to the open PR as `0d4eff6`. Y rather than Z: this removes a hard
prerequisite from two of the four systems, which is a capability change, not
a wording fix.

## 0.2.1 - replace the Atari800 caveat with the core's own source

`docs/atari.md` shipped a caveat asking whoever ran it first to confirm the
per-core config directory the wrapper writes to, because the machine switch
had been written from the libretro documentation. Reading
[libretro-atari800](https://github.com/libretro/libretro-atari800)'s
`libretro/libretro-core.c` settles it: `info->library_name = "Atari800"`, the
option key is `atari800_system`, and the accepted machine values are
`400/800 (OS B)`, `800XL (64K)`, `130XE (128K)`, `XEGS`, the three
`Modern XL/XE` sizes and `5200`. The caveat is gone.

It also corrected the reasoning, which had been weaker than the truth. The
core already forces 5200 mode for a cart it recognises — `.a52` by extension,
or `.bin`/`.rom` by CRC32 against a built-in database — but
`if (autorunCartridge == A5200_CART || strcmp(var.value, "5200") == 0)` only
turns 5200 mode **on**. Nothing turns it off, so a `.atr` launched after a
5200 session boots as a 5200 with 16 KB of RAM and a 5200 joystick layout.
That silent failure is the wrapper's actual purpose; the 5200 direction is a
fallback for carts the CRC database does not carry.

No behaviour change — the wrapper was already writing the correct value to
the correct path. Pushed to the open PR as `63c25c7`. Z rather than Y: a
documentation correction, not a new phase.

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
