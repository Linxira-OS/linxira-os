# Editions and build strategy

Status: working document (2026-11-10). User-facing announcements:
[site](https://linxira-os.github.io/zh/blog/editions-and-build-strategy/) and
[wiki](https://linxira-os.github.io/linxira-wiki/editions/).

This document defines the edition boundaries. It is the governance reference
for repository placement, transaction backends, release cadence and support
claims. Code changes that cross an edition boundary require an update here.

## 1. Positioning

Linxira is a **removable layer on top of an upstream distribution**, not a
distribution built from scratch. The layer owns:

- catalog data and selection;
- transaction planning, confirmation, privileged apply and receipts;
- signed package repository and keyring;
- configuration entry points, update status, crash watch and welcome.

The base owns the kernel, hardware enablement and base userland.

## 2. Editions

| Edition | Base | Role | Cadence | Exit semantics | Status |
|---|---|---|---|---|---|
| Arch x86_64 | Arch Linux | Primary development machine; also inference | Rolling; monthly ISO snapshot (planned) | Remove `linxira-*`, return to vanilla Arch | Current |
| Ubuntu interim validation | 26.10, then 27.04 / 27.10 if needed | Validate apt backend, desktop, drivers, ARM allowlist | Follows Ubuntu interims; 9 months | Reimage; layer removable | Validation |
| Ubuntu LTS (future) | 26.04 LTS or 28.04 LTS | Inference, scientific computing, Chinese GPUs | LTS point releases + upstream security | Removable derivative, not rolling | Pending |

NixOS is **not** a product line. Zeta already provides Nix builds; use upstream
Nix in a container or VM. Do not maintain a NixOS configuration, flake or
distribution promise.

Validation images are labelled “validation build, not long-term support, EOL
date”, kept out of the support matrix and given no long-term promise.

Non-goals:

- no Mint/Cinnamon base;
- no third desktop beyond KDE (primary) and an optional COSMIC spin;
- no home-grown kernel (v3 / CachyOS-derived) until a complete automation
  pipeline exists;
- no sponsor-bundled software or “agent count” marketing;
- no NixOS product line.

## 3. Release cadence

- **Arch**: monthly ISO snapshot of the rolling repository. The snapshot is a
  build artifact, not a new distribution release. Users keep rolling after
  install. Versioning: `YYYY.MM`.
- **Ubuntu interim**: validation images only, following 26.10 → 27.04 → 27.10.
  Each has a 9-month lifetime and must be migrated or retired before EOL.
- **Ubuntu LTS**: LTS point releases + upstream security updates. ISO refresh at
  point releases; no monthly ISO requirement. Versioning:
  `26.04.x-linxira.N`.
- **NixOS**: not applicable.

### Ubuntu timeline (authoritative external facts)

- 26.04 LTS: released 2026-04-23; standard security maintenance to 2031-05;
  ESM to 2036.
- 26.10: released 2026-10; 9-month support ends 2027-07.
- 27.04 “Tenacious T-Rex”: 2027-04-22; 9-month interim.
- 27.10: October 2027; interim.
- Next LTS: 28.04 (April 2028).

There is no 2027 LTS. The decision point for moving to LTS is the 26.04.2 point
release (2027-02-18): if HWE brings kernel 7.3 and hardware validation passes,
switch the long-term baseline to 26.04 LTS. Otherwise continue validation on
27.04 and retire 26.10 before 2027-07.

## 4. Repository and contract boundaries

Shared repositories (base-agnostic policy / UI / content):

`linxira-catalog`, `linxira-components` (policy layer), `linxira-package-center`,
`linxira-component-manager`, `linxira-gaming-manager`,
`linxira-completion-agent`, `linxira-crash-watch`, `linxira-welcome`,
`linxira-recovery-diagnostics`, `linxira-config-hub` (adapters per edition),
`linxira-artwork`, `linxira-wiki`, `Linxira-OS.github.io`, `linxira-skills`.

Arch edition repositories:

`packages`, `linxira-iso-direct`, `linxira-hooks`, `linxira-update`,
`linxira-keys`, `linxira-packages`, `linxira-wsl`.

Ubuntu edition repositories (planned):

- `linxira-ubuntu-packaging` — `debian/` packaging for all `linxira-*` sources;
- `linxira-ubuntu-iso` — Ubuntu Base + live/autoinstall profile (only if a full
  image is needed; the interim phase may validate via a setup tool instead);
- apt backend for `linxira-components` (package inside the components repo or a
  dedicated adapter repo);
- vendor driver integration (Chinese GPU, DGX/N1X) as reviewed packages.

Contracts:

1. `linxira-catalog`: every application has a logical ID and a per-edition
   target map (`arch`, `ubuntu`); no package names in UI or installer.
2. `linxira-components`: `plan → confirm → apply → receipt` is shared; the
   execution backend is selected by edition:
   - Arch: fixed-argv `pacman`;
   - Ubuntu: fixed-argv `apt-get` / `dpkg-query`.
3. `linxira-update` / `linxira-config-hub`: source and update adapters per
   edition; one status schema for the UI.
4. ISO: separate build scripts per edition; shared selection/confirmation UX.

Do not use Git branches to separate editions. Use edition directories,
adapters and build profiles.

## 5. Supply chain and network

- Arch: AUR off by default; official repositories are signed; our packages are
  built and signed in CI.
- Ubuntu: unreviewed PPAs off by default; vendor debs are reviewed and
  republished in our signed repository; `curl | sh` is not recommended.
- China: Arch mirrors are the strongest; Ubuntu has Chinese archive and ISO
  mirrors; NixOS mirrors are not guaranteed, which is one reason it is not a
  product line. Our ISOs are hosted on GitCode / domestic object storage.
- DGX OS / NVIDIA downloads may be account- or network-gated; ARM support is an
  allowlist, not generic ARM.

## 6. Acceptance gates

- **Exit test**: remove all `linxira-*` packages; the base still boots, updates
  and reaches the network.
- **No hidden agents**: no agent starts without explicit user action; crash
  watch does zero-token triage and never uploads.
- **Catalog completeness**: every preinstalled or recommended application has a
  catalog entry with source, license, trust level and per-edition target.
- **Kernel ownership**: Arch uses the official kernel; Ubuntu uses HWE/vendor
  kernels; no custom kernel without automation.
