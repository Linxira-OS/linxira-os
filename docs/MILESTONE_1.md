# Milestone 1: Direct-Arch Graphical Installer

## Deliverable

One x86_64 ISO from the independent `linxira-iso-direct` repository that boots a
complete Plasma Live session and installs a complete base system without network
access.

## User Flow

1. Boot in BIOS or UEFI mode.
2. Enter the full Plasma Wayland Live session and review Linxira Welcome.
3. Configure network only when needed for time, diagnostics, or clearly marked
   optional downloads.
4. Manually start Calamares from Welcome or the application menu, then complete
   its language, keyboard, storage, kernel profile, user, and
   summary pages.
5. Install the base system from the repository embedded in the ISO. RC6 uses
   unsigned development metadata; release media must use signed metadata.
6. Create an initial Timeshift snapshot and GRUB configuration.
7. Reboot into the installed system.

## Kernel Choice

Calamares presents one segmented choice:

- **Standard:** `linux` + `linux-lts`
- **Responsive desktop:** `linux-zen` + `linux-lts`

The LTS kernel is always retained as the recovery path. The installer never
installs all three kernels.

## Storage Contract

Automatic Btrfs installation creates `@`, `@home`, `@log`, `@cache`, `@tmp`,
and `@swap`. The default subvolume remains ID 5. `/boot` is part of `@`; UEFI
mounts its FAT32 system partition at `/boot/efi`.

Timeshift and grub-btrfs are configured from official Arch packages. Research
data backup is outside this contract.

## Package Contract

- Arch official repositories supply the base packages.
- The RC6 development image uses locally built Linxira integration packages and
  an unsigned embedded repository for the exact installation set.
- Release images require signed Linxira packages and signed offline repository
  metadata.
- Calamares is built from a pinned upstream release in a clean Arch chroot.
- AUR helpers and third-party binary repositories are not used by the installer.

## Current RC6 Evidence

- Source and artifact checks pass.
- QEMU BIOS and UEFI menu boots pass.
- Complete Plasma, Welcome, write-to-disk installation, installed-system first
  boot, and recovery acceptance remain outstanding.

## Acceptance Gate

- BIOS and UEFI offline installation succeeds. Menu boot alone does not satisfy
  this gate.
- Encrypted and unencrypted Btrfs layouts match the storage contract.
- Both selected kernels boot.
- Timeshift create, restore, and live-media recovery succeed.
- Package signatures and package ownership validate.
- No CachyOS repository, binary package, service, or branding dependency remains.
- The installed system contains no live user, installer autostart, temporary
  network credentials, or offline repository configuration.
