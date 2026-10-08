# Linxira OS Roadmap

## What we are building, and why

Linxira OS is a Linux workstation distribution for scientific and AI workflows,
built directly on the official Arch Linux base with no downstream binary
repository. The target is a workstation a researcher can install offline, audit
after the fact, and keep under their own control: Btrfs + Timeshift snapshots,
two official kernels (`linux`, `linux-lts`), Plasma and COSMIC as supported
desktops, and first-party products (Welcome, Package Center, Config CLI,
driver/kernel managers, update, recovery) that each own a single capability.

It is built this way because scientific and AI work depends on a trustworthy,
reproducible supply chain. A workstation assembled from an opaque mix of
repositories — or one that cannot state exactly which packages it shipped — is
not something a lab can rely on. That is why the design insists on an explicit
pinned package cohort, an offline installation closure, and separated
privilege / policy / UI ownership.

## Why the release standard exists

Every image built so far (including r17 / 2026-09-30) is a **test build**; no
official release has been announced. Before any image may be called a release it
must be able to prove what it is: its source commits, package cohort, build
environment, SBOM, and acceptance evidence (firmware boot, disk install, both
kernels, first boot, Package Center, recovery). `releases/` and
`manifest.schema.json` define that proof. It is not a build gate but an
after-the-fact audit and reproducibility record, enforced from the next build
onward; r17's build-time evidence is gone and will not be reconstructed.

## Milestone 1: Direct-Arch Installer Baseline

### Architecture

- [x] Select direct Arch Linux package and tooling base
- [x] Remove CachyOS binary repository dependency from the target design
- [x] Select Timeshift as the first-release snapshot frontend
- [x] Define two-kernel profiles using official Arch kernels
- [x] Replace the rejected restricted session with a complete Plasma Live session

### Package Pipeline

- [x] Package pinned upstream Calamares in a clean Arch build root
- [ ] Sign Calamares and Linxira integration packages
- [ ] Build a signed offline installation repository
- [ ] Record exact Arch Linux Archive cohort date and package manifest
- [ ] Verify clean-room package rebuilds

### Archiso And Installer

- [x] Create a clean direct-Arch Archiso profile
- [x] Implement the complete Plasma Live account and session
- [x] Autostart Welcome and provide manual Calamares launchers
- [x] Implement package-based offline target installation
- [x] Implement the documented Timeshift-compatible Btrfs subvolumes
- [ ] Configure GRUB, Timeshift, grub-btrfsd, and initial snapshot
- [ ] Add Standard and Responsive desktop kernel profiles
- [x] Remove every CachyOS repository, package, service, and branding dependency

### Verification

- [x] QEMU BIOS menu boot for RC7
- [x] QEMU UEFI menu boot for RC7
- [ ] Complete BIOS write-to-disk installation and first boot
- [ ] Complete UEFI write-to-disk installation and first boot
- [ ] Encrypted and unencrypted Btrfs installation
- [ ] Boot both selected kernels
- [ ] Timeshift GUI create and restore
- [ ] Live-ISO recovery restore
- [ ] grub-btrfs snapshot discovery and documented recovery behavior
- [ ] NVIDIA DKMS validation on both kernels
- [ ] Package signature and installed-file integrity checks

### Release

> See "Why the release standard exists" above. Items stay unchecked until an
> image passes the acceptance gates and its manifest is recorded.

- [ ] Publish direct-Arch architecture and installation documentation
- [ ] Publish ISO, package manifest, checksums, and signatures
- [ ] Publish known limitations and rollback procedure
- [ ] Promote the first validated Arch package cohort
- [ ] Emit release-manifest evidence as part of the build (Arch/Linxira package-cohort manifests, `[linxira]` repository-database hashes, SBOM, and firmware-boot / disk-install / dual-kernel / first-boot / Package Center / recovery acceptance records), starting with the next build

## Milestone 2: Scientific Workstation Profiles

- [ ] Miniforge and bioconda profile (channel configuration is available in
      Config Hub; bootstrap and environment manifest remain pending)
- [ ] Apptainer and BioContainers profile
- [ ] Development toolchain profile
- [ ] Versioned `linxira env` manifests for rootless Podman/Distrobox environments
- [ ] Signed OCI digest and Apptainer SIF verification
- [ ] Optional offline environment packs separate from the base ISO
- [ ] CUDA-enabled container validation matrix
- [ ] AI-operation pre-change snapshot interface
- [x] Config Hub source/channel integration baseline

## Milestone 3: Measured Performance Options

- [ ] Establish representative bioinformatics and numerical benchmarks
- [ ] Compare `linux` and `linux-zen` on workstation workloads
- [ ] Pilot ALHP `x86-64-v3` on internal canary hardware
- [ ] Evaluate locally built kernel and package candidates
- [ ] Promote only optimizations with reproducible material gains

## Deferred

- Secure Boot
- Custom Linxira kernel
- Complete self-hosted Arch binary mirror and archive
- Snapper coexistence
- Niri edition
- WSL and server editions
- Additional desktop environments
- ARM support
- Distribution-wide `x86-64-v4` packages
