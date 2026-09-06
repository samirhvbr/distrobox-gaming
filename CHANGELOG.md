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

## 0.4.0 - the macOS stack plays the 8- and 16-bit libraries

The macOS baseline stopped at four console emulators, so the downloaded Atari,
Nintendo, Sega and Commodore collections had nothing to run them. RetroArch
Metal plus seven libretro cores closes that: Atari 2600 (two collections), NES,
SNES, Master System, Mega Drive, GBA, Game Boy/Color and C64.

Cores are not Homebrew packages, so `install-cores.py` fetches them from the
Libretro buildbot for the Mac's architecture, verifies each with `lipo
-verify_arch` and installs atomically. The buildbot URL is a rolling `latest`,
and the recorded SHA-256 is an integrity record of what was fetched, not a pin
like the Linux tree's tarballs. That divergence is stated in `INSTALL.md` and
in the PR rather than left for a reviewer to notice.

Two display findings came out of running it rather than reading about it.
Launching from ES-DE sat on a black screen until `pause_nonactive = "false"`,
because ES-DE keeps focus while RetroArch starts. Then the image advanced only
while the mouse moved — the experimental Metal driver — so the default became
`video_driver = "vulkan"`, which the same RetroArch Metal application provides
through MoltenVK. Both settings are applied with `--appendconfig`, so a user's
own RetroArch configuration is never touched.

`import-metadata.py` brings each collection's existing `gamelist.xml` into
ES-DE, rewriting paths to absolute form and symlinking cover art. Source ROMs
and metadata are never modified, and entries are confined to the library root.

Review of the work found the canonical `INSTALL.md` still describing a
four-application install, and the new retro documentation sitting in a separate
Portuguese file that the rest of the repository does not match. The retro
documentation is now merged into `INSTALL.md` in English, and `INSTALL.md`
lists RetroArch and the core step it had omitted. A `verify.py` assertion for
the four console systems' `(Standalone)` labels, dropped when the retro
assertions were added, is restored.

Gameplay is confirmed for the first time in this stack: a NES ROM under
Nestopia and an Atari 2600 ROM under Stella both loaded and rendered through
Vulkan/MoltenVK on Apple Silicon. Audio, controllers, saves and the remaining
systems are still unverified, and the four console emulators still need real
games and a PS2 BIOS.

## 0.3.1 - no Atari system needs a BIOS file, and ROMS/ stays out of git

The BIOS table still marked `lynxboot.img` as required. Same mistake as the
5200 one, made the same way — from documentation instead of the core.
libretro-handy passes `!bios_found` as `CSystem`'s `useEmu` argument, so with
no file it high-level-emulates the boot ROM (`CSystem::HLE_BIOS_FE00` in
`lynx/system.cpp`). That closes it: stella never needed one, prosystem's is
optional, atari800 compiles in AltirraOS, handy HLEs. `dg_retroarch_system_bios`
is not a prerequisite list, it is there so whoever owns the dumps runs the
originals — which is why every entry warns and skips instead of failing.

Three wrong BIOS claims in one PR, all from reading docs rather than source.
The pattern is worth naming: for an emulator core, the source is the only
authority on what it requires.

`.gitignore` gains `ROMS/` with the reason. Upstream's list is lowercase and
git patterns are case-sensitive, so a `ROMS/` directory would slip past
`roms/`. Downloaded game files are not going into a public fork: they are
copyrighted, it would contradict the upstream repo's own stated policy
(`docs/setup-assets.md`: "None of these are shipped with the repo — legal
distribution rules"), and git history makes a mistake permanent. This is a
fork-local pattern and stays out of the PR.

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
