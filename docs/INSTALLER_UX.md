# Linxira Installer UX

## Purpose

The installation media provides a complete Plasma Live desktop with a graphical
installer. It supports normal Live use and users who cannot or should not
navigate an English-only terminal installer.

## Boot Menu

The default boot menu contains:

1. Start Linxira OS
2. Start Linxira OS (safe graphics)
3. Recovery tools
4. Advanced boot options

The normal and safe-graphics entries start the same full Plasma Live session.
Safe graphics adds only documented fallback kernel parameters. Kernel package
selection belongs in Calamares, not in the firmware boot menu.

## Installer Session

Archiso automatically logs an unprivileged Live account into a full Plasma
Wayland session. Linxira Welcome opens through XDG autostart. Calamares starts
only when the user selects the fixed installer entry in Welcome, the panel, or
the application menu.

The session exposes the normal Plasma desktop, including:

- the application launcher, panel, and desktop;
- Dolphin, Konsole, browser, and system settings;
- NetworkManager and accessibility controls;
- Shelly and recovery tools;
- installer logs, diagnostics, power, and restart.

The Live home is ephemeral. The full session remains usable while Calamares is
open; Calamares is never an automatic or restricted full-screen session.

## Calamares Flow

The first release uses this sequence:

```text
Welcome
Language and locale
Keyboard
Time zone
Storage and encryption
Kernel profile
User account
Summary
Installation
Finish
```

Application recommendations are not installer pages. They are reversible
post-install choices presented as catalog metadata in Linxira Welcome and
managed through Shelly or another audited transaction owner.

## Storage Choice

Automatic installation recommends Btrfs with Timeshift system rollback. The
summary explicitly states that snapshots protect system state and are not user
data backups.

The supported automatic layout is documented in `ARCHITECTURE.md`. Manual
partitioning is available, but Timeshift Btrfs support is shown as unavailable
unless the resulting root and home layout satisfies its requirements.

## Kernel Choice

The user selects one of two product profiles:

- **Standard:** Arch `linux` plus `linux-lts`
- **Responsive desktop:** Arch `linux-zen` plus `linux-lts`

The page explains that Zen targets interactive responsiveness and is not a
general compute-throughput upgrade. The LTS kernel is always retained as a
recovery path.

## Offline Contract

The complete base system installs without a network connection from the embedded
repository. RC6 uses unsigned development metadata; release media must use
signed package and repository metadata. Calamares does not copy the temporary
file-based repository configuration into the target.

Clearly marked optional components may require networking during installation or
may be completed later through their owning tool. An optional download failure
must not corrupt or invalidate the offline base installation. Flatpak remotes,
vendor software, and scientific environments require explicit user consent.

## Failure Behavior

- Installation never reports success after a failed package, signature, DKMS,
  initramfs, or bootloader transaction.
- Logs remain accessible from the installer session after failure.
- The target does not retain the installer user, autostart entry, temporary
  network credentials, offline repository, or live-session logs.
- Interrupted installation is treated as incomplete and is never marked
  bootable without target validation.
