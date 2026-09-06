# Installing on macOS

This guide covers the first phase of macOS support. The commands are meant to
be run on the target Mac; reading this document installs nothing.

## What gets installed

| Application | Role | Homebrew package |
|---|---|---|
| ES-DE | Frontend for browsing the library and launching games | `es-de` |
| Dolphin | GameCube and Wii | `dolphin` |
| PCSX2 | PlayStation 2 | `pcsx2` |
| PPSSPP | PSP | `ppsspp-emulator` |

Plus the two tools that do the installing and configuring:

- **Homebrew** — prerequisite for the current installer. Install it from
  [brew.sh](https://brew.sh) if it is not already present.
- **Ansible** — applies the project's configuration; it is in the `Brewfile`.
  It is neither an emulator nor a frontend.

[Brewfile](Brewfile) is the list the installer actually uses. Applications are
expected in `/Applications`. Homebrew may pull additional dependencies, and
versions follow their packages — this is not a version-locked environment.

Windows games, Wine, Proton, CrossOver and Windows-only tooling are
deliberately out of scope. No ROMs, BIOS or firmware are supplied.

## 1. Check the environment

From the repository root:

```sh
command -v brew
brew bundle check --verbose --file=macos/Brewfile
```

Neither command installs anything. If a dependency is missing or outdated, the
check exits non-zero and names it.

## 2. Install the applications

The full installer covers Ansible plus the four applications:

```sh
./macos/install-apps.sh
```

The equivalent individual commands are:

```sh
brew install ansible
brew install --cask es-de
brew install --cask dolphin
brew install --cask pcsx2
brew install --cask ppsspp-emulator
```

Use whichever suits you — the full installer always considers every Brewfile
entry. Installing only some applications does not drop the other systems from
the generated configuration: an entry whose emulator is absent simply cannot
launch anything.

### Inspecting and extending the list

```sh
./macos/install-apps.sh --list
./macos/install-apps.sh --check
```

To add an application, append `cask "package-name"` to
[macos/Brewfile](Brewfile); for a CLI tool use `brew "package-name"`. Then run
the installer again — the script itself needs no edit. Note that adding an
emulator to the Brewfile installs the application only; wiring a new console
into ES-DE also means configuring its system in the Ansible playbook.

The script runs from any directory, installs or updates what is declared, and
verifies the result. If Homebrew fails it returns an error and can be rerun
once the cause is fixed. It never removes packages that are not on the list.
`./macos/bootstrap.sh install` remains available as a shortcut to the same
installer.

## 3. Configure your library paths

Create the local override only if it does not exist yet:

```sh
if [ ! -f macos/ansible/host_vars/localhost.yml ]; then
  cp macos/ansible/host_vars/localhost.yml.example macos/ansible/host_vars/localhost.yml
fi
```

Edit it with the absolute paths to your library:

```yaml
dg_macos_rom_root: /Volumes/Games/roms
```

For a split library use `dg_macos_rom_paths`, as shown in the example file. The
default is `~/Games/roms` with `gc`, `wii`, `ps2` and `psp` subdirectories.
External volumes must be mounted. Missing directories are reported, never
created — a disconnected `/Volumes/...` mount must not silently become a local
library.

## 4. Review and apply

Close ES-DE before applying:

```sh
./macos/bootstrap.sh check
./macos/bootstrap.sh configure
```

`check` previews without writing; `configure` writes. The generated file is
`~/ES-DE/custom_systems/es_systems.xml`, or elsewhere if you customised
`dg_macos_esde_home`. **The generated file owns the complete custom-system
list** and the previous version is backed up first, so review the diff if you
already had custom systems. ES-DE's built-in systems stay available.

## 5. First launch

1. Open Dolphin, PCSX2 and PPSSPP once and finish their first-run setup.
2. In PCSX2, select your own PS2 BIOS.
3. Configure your controller in each emulator. For Wii, pick the input profile
   that suits the game and the controller you are using.
4. Open ES-DE with `open -a ES-DE` and launch one game per configured system.
5. Check video, audio, controller input, clean exit, and saving/loading.

This phase automates installation and ES-DE integration. BIOS, controllers and
graphics settings are still configured inside each emulator, and the automated
tests are no substitute for testing with real games.

See [README.md](README.md) for architecture, scope boundaries and how the
automation is tested.
