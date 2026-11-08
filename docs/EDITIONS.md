# Editions and build strategy

Status: working document (2026-11-08). User-facing announcements:
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
| Ubuntu x86_64 | Ubuntu 26.04 LTS | Inference, scientific computing, Chinese GPUs | LTS point releases + upstream security; HWE at point releases | Removable derivative layer, not rolling | Research |
| Ubuntu ARM64 | Ubuntu 26.04 LTS | Inference on a hardware allowlist (DGX Spark, N1X, …) | Same as Ubuntu x86_64 | Same as Ubuntu x86_64 | Research |
| NixOS | NixOS (VM / container) | Reproducible reference environment | Pinned flake, on-demand update | Configuration/flake only | Experimental, no SLA |

Non-goals:

- no Mint/Cinnamon base;
- no third desktop beyond KDE (primary) and an optional COSMIC spin;
- no home-grown kernel (v3 / CachyOS-derived) until a complete automation
  pipeline exists;
- no sponsor-bundled software or “agent count” marketing.

## 3. Release cadence

- **Arch**: monthly ISO snapshot of the rolling repository. The snapshot is a
  build artifact, not a new distribution release. Users keep rolling after
  install. Versioning: `YYYY.MM`.
- **Ubuntu**: LTS point releases + upstream security updates. ISO refresh at
  point releases; no monthly ISO requirement. Versioning:
  `26.04.x-linxira.N`.
- **NixOS**: flake pin; `flake update` on demand.

### Ubuntu timeline (authoritative external facts)

- 26.04 LTS: released 2026-04-23; standard security maintenance to 2031-05;
  ESM to 2036.
- 26.10: released 2026-10; 9-month support ends 2027-07.
- 27.04 “Tenacious T-Rex”: 2027-04-22; 9-month interim.
- 27.10: October 2027; interim.
- Next LTS: 28.04 (April 2028).

There is no 2027 LTS. For LTS + kernel 7.3, target the 26.04.2 point release
(2027-02-18), which should carry the 26.10 HWE kernel. Verify against the Ubuntu
release notes before committing.

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
- `linxira-ubuntu-iso` — Ubuntu Base + live/autoinstall profile;
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
  mirrors; NixOS mirrors exist but are not guaranteed, so NixOS stays
  experimental. Our ISOs are hosted on GitCode / domestic object storage.
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
