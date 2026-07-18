# Linxira OS Product Architecture v3.0

## 1. Product Definition

Linxira OS is an independent scientific and AI workstation distribution built
directly from Arch Linux packages and tooling. It combines a reproducible Arch
host, a graphical installer, system rollback, and optional scientific profiles.

The first release prioritizes installation and recovery reliability over
distribution-wide compiler optimization.

## 2. Trust And Repository Model

- Arch official repositories provide the base system, desktop, kernels,
  Timeshift, grub-btrfs, firmware, and normal updates.
- The planned signed `[linxira]` repository contains Linxira-owned packages and a
  small set of explicitly adopted, source-built integration packages that Arch
  does not provide.
- Release installation never contacts AUR helpers or CachyOS repositories.
- Exact installation packages are copied into an offline repository in the ISO.
  The RC6 development repository is unsigned; signed package and repository
  metadata remain a release gate.
- Arch Linux Archive dates may be used to reproduce and promote tested package
  cohorts until Linxira operates a complete package archive.

Initial Linxira components include artwork, hooks, repository configuration,
Welcome, Config Hub, installer configuration, and a pinned Calamares build.
Shelly is the graphical package manager in both the Live session and installed
system, but it does not own or participate in Calamares installation
transactions.

## 3. Installer Architecture

The independent `linxira-iso-direct` repository builds the current ISO. It boots
to a complete Plasma Live desktop, not a restricted installer-only session:

```text
Archiso
  -> unprivileged installer user
  -> full Plasma Wayland session
  -> Welcome autostart
  -> user manually launches Calamares
  -> package-based offline target installation
```

The Live session keeps the normal Plasma panel, application menu, terminal,
file manager, networking, Shelly, system tools, and recovery tools available
while Calamares runs. Calamares does not autostart. The mutable Live root is not
copied into the target.

Calamares is not an official Arch package. Linxira pins an upstream release and
builds it independently in a clean Arch environment. RC6 uses that locally built
package; signing and publication through the Linxira package pipeline remain
pending. Existing CachyOS Calamares code may be consulted as licensed historical
reference material, but it is not a current binary, repository, or build
dependency.

## 4. Welcome Boundary

Linxira Welcome is the current `org.linxira.Welcome` Python/PySide6 application.
It reads catalog v2 metadata and the installer receipt, opens project resources,
and launches only fixed allowlisted desktop executables. It does not execute
shell strings, `sudo`, `pkexec`, or package transactions. Calamares handles
installation; Shelly is the graphical package manager.

The XDG autostart entry invokes Welcome at Plasma login when the per-user
`Show Welcome at login` setting is enabled. The setting defaults to enabled and
persists for an installed user. A Live profile is ephemeral, so Welcome opens on
each fresh Live boot. This is a login preference, not a one-time completion or
first-boot migration flag.

## 5. Kernel Profiles

Every installation has one primary kernel and one official LTS recovery kernel.
The release design defines two mutually exclusive profiles:

| Profile | Primary | Recovery | Intended use |
|---------|---------|----------|--------------|
| Standard | `linux` | `linux-lts` | General and compute workloads |
| Responsive desktop | `linux-zen` | `linux-lts` | Interactive workstation latency |

Matching headers are installed for both selected kernels. GRUB explicitly uses
the primary kernel as its top-level default. Linxira does not install all three
kernels because that unnecessarily expands DKMS, initramfs, snapshot, and test
matrices.

RC6 implements the Standard profile. The Responsive desktop profile remains a
release task and must not be presented as accepted current behavior.

NVIDIA installation uses a validated DKMS package so modules build for both
kernels. An installation is not promoted until both kernels boot and the GPU
passes native and container tests.

## 6. Btrfs And Timeshift

Timeshift is the supported graphical system-rollback tool for the first
release. Snapper is deferred to avoid two competing snapshot policies.

The automatic Btrfs layout is:

```text
@       -> /
@home   -> /home
@log    -> /var/log
@cache  -> /var/cache
@tmp    -> /var/tmp
@swap   -> /.swap
```

The Btrfs default remains top-level subvolume ID 5. `/etc/fstab` mounts by
subvolume name, not subvolume ID. `/boot` remains inside `@`; the UEFI system
partition is mounted at `/boot/efi`. This keeps kernels and initramfs files with
the matching root snapshot. BIOS installations use the same root layout.

Official Arch `timeshift`, `grub-btrfs`, `btrfs-progs`, `inotify-tools`, and
`cronie` packages provide snapshot management. `grub-btrfsd` runs with
`--timeshift-auto`. Snapshot boot is an emergency recovery path; the supported
normal rollback is a Timeshift restore followed by reboot.

Snapshots protect system state, not research data. Scratch data, container
caches, and large pipeline intermediates require separate storage policy.

## 7. AI Operations Boundary

AI automation does not receive a general root shell. Privileged mutations are
exposed through fixed, root-owned operations. Before package, service, boot,
driver, or system-configuration changes, the operation controller must:

1. Check Btrfs free space and Timeshift health.
2. Create an on-demand pre-change snapshot.
3. Record the operation and snapshot identifiers outside the snapshot set.
4. Abort when the snapshot fails.
5. Require human approval for destructive or cluster-wide operations.

Snapshots remain removable only through a separately authorized maintenance
path. Off-host backups are required for irreplaceable data and configuration.

## 8. Performance Policy

The public ISO initially uses Arch official packages only. Performance work is
promoted by measured workload results, not by distribution-wide claims.

Priorities are application SIMD dispatch, BLAS/FFTW selection, CUDA and NCCL,
NUMA and thread affinity, local scratch I/O, and reproducible Apptainer images.
ALHP `x86-64-v3` or locally built kernels may be tested on internal canary nodes,
but are not public installation dependencies before Linxira controls building,
signing, archiving, rollback, and redistribution compliance.

## 9. Build And Test Environment

- WSL is used for source work, linting, package-list validation, and CI parity.
- Release packages and ISOs are built in an Arch VM on a native Linux
  filesystem.
- Hyper-V Generation 1 validates legacy BIOS.
- Hyper-V Generation 2 validates UEFI with Secure Boot disabled.
- Physical NVIDIA hardware validates DKMS and CUDA paths.

RC6 has passed source and artifact checks plus QEMU BIOS and UEFI menu boots.
Complete write-to-disk installation, installed-system first boot, and recovery
acceptance have not yet passed.

## 10. Independent Projects

ExtendAI Lab Studio, ExtendAI Lab CLI, Linxira Pulse, Linxira Skills, and the
upstream OpenCode clone are independent projects. They are not rewritten or
versioned as part of the operating-system base migration.

## 11. First-Release Non-Goals

- No CachyOS binary repositories or redistributed CachyOS packages
- No custom Linxira kernel
- No self-hosted replacement for the full Arch repositories
- No Secure Boot guarantee
- No Snapper policy
- No online-only or minimal installer
- No multiple desktop editions
- No AUR helper during installation
- No distribution-wide `x86-64-v3/v4` package replacement
