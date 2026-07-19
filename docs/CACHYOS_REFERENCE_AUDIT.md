# CachyOS Reference Audit

Status: design reference for Linxira Phase 1. This document records the
behavior audited from the supplied `cachyos-desktop-linux-260628.iso` and the
local CachyOS Welcome/Calamares sources. It is not a claim that Linxira uses
CachyOS packages or repositories.

## Product Layers

Cachy separates four related layers:

1. Live Welcome exposes the installer.
2. Calamares `netinstall.yaml` presents category and subgroup checkboxes for
   selected packages and desktop environments.
3. The installed system receives `cachyos-packageinstaller`, whose executable
   is `cachyos-pi`.
4. Installed Welcome opens PackageInstaller, Kernel Manager, Tweaks,
   maintenance, DNS, gaming, VRAM and Winboat actions.

The installer is not a single gaming or application meta-package. Its default
selection contains required Cachy packages, Cachy tools, shell configuration,
base/common groups, network, firewall, Bluetooth, package management, desktop
integration, filesystem, fonts, audio, hardware, power, general applications
and Firefox. Desktop environments, printing, HP support and accessibility are
separate choices.

## Default Applications

The supplied KDE selection includes Ark, Dolphin, Gwenview, Kate, KCalc,
Konsole, Partition Manager, Spectacle, KDE Connect, Filelight and the normal
KDE integration stack. Haruna is the default KDE video player. VLC support is a
separate playback option and backend, not the entire media application policy.

Linxira therefore models Haruna as a reviewed KDE default and VLC as a
separately selectable application. The catalog must expose individual
applications by category; profiles remain optional presets and cannot be the
only install unit.

## Installed Welcome Functions

The audited Rust Welcome implementation provides these installed-system
capabilities:

- `cachyos-pi` PackageInstaller and `cachyos-kernel-manager` launchers;
- service/tweak controls for profile-sync-daemon, systemd-oomd, BPFtune,
  Bluetooth, Ananicy Cpp and Cachy Update;
- update, orphan removal, cache cleanup, keyring reset, package reinstall,
  mirror ranking and KWin diagnostics;
- NetworkManager DNS, IPv4/IPv6, DoT, DoH/DoQ through Blocky, latency tests and
  DHCP restoration;
- gaming packages, VRAM management, Winboat/Docker and hardware profile
  detection through `chwd`.

Linxira's current Python Welcome is narrower. Its fixed launchers cover the
installer, Shelly, System Settings, KInfoCenter, Timeshift and Konsole. The
Package Center and later control pages are being implemented as separate
catalog-backed capabilities rather than silently copying Cachy behavior.

## Linxira Catalog Contract

`catalog/catalog-v2.json` now contains `applications[]`. Each application has:

- stable ID and localized name/description;
- categories and a source ID;
- package identifiers without shell commands;
- architecture/network availability;
- review status and presentation order/default state.

Supported source kinds are represented in the schema for pacman, AUR, Flatpak,
PyPI, npm, Conda, Cargo, Go, OCI and containers. Adding a source kind does not
enable it. AUR and third-party sources remain explicit opt-in choices.

The installed software center and Calamares transaction code use application
IDs and catalog allowlists. Receipts record selected application IDs and
expanded package IDs. The remaining migration work is to replace the static
Calamares profile chooser with a catalog-generated category/application chooser
and to bind profile presets to application IDs.

## Package And Runtime Scope

The catalog and Config Hub currently reserve opt-in entries for Nix, Python,
uv, Node.js/npm, Rust/Cargo, Go, Podman and Apptainer. `linxira-config runtime
status` reports their installed availability.

Nix is marked `source-review` and is not default-selected. The workspace has no
verified WISE package, source, implementation or license metadata. WISE must
not be installed under a guessed package name; its exact product identity and
distribution source must be established before catalog inclusion.

The intended package/runtime CLI must eventually cover Arch/Linxira packages,
regional Arch mirrors, npm, PyPI, AUR, Flatpak, Cargo, Go, Conda/Bioconda,
OCI/container sources and scientific environments. This is separate from the
AI-oriented `extendai-lab-Studio` CLI.

## ISO Size Decision

The supplied Cachy ISO is `3,152,297,984` bytes. RC13 is `4,585,148,416`
bytes. Their package counts are nearly identical, but RC13 embeds 776 package
files under `/opt/linxira/offline-repo/x86_64`, totaling about
`2,103,327,253` bytes. This is the dominant size difference.

Before the next ISO build, Linxira must choose between an online-first desktop
ISO, a separate offline repository medium, or separate desktop and rescue
artifacts. Package deletion alone is not the correct first response.

## Build Gate

No new ISO is a release candidate until:

- the shared application selector is complete;
- Package Center and Calamares consume the same catalog and receipt model;
- the real transaction backend is implemented and audited;
- Nix/runtime/source policies are explicit;
- WISE is either identified and reviewed or excluded;
- desktop, first boot, installation target and recovery tests pass.
