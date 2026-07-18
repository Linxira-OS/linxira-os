# Linxira Welcome

## Product Boundary

Linxira Welcome is the current workstation control center. It is a Python 3 and
PySide6 application with desktop identity `org.linxira.Welcome`. It reads catalog
v2 metadata and the installer receipt, opens reviewed resources, and launches a
fixed allowlist of desktop tools. It is not a package manager.

The four user-facing responsibilities are separated:

| Surface | Responsibility |
|---------|----------------|
| Calamares | Install a bootable base system |
| Linxira Welcome | Live installer entry, status, metadata, and fixed launchers |
| Shelly | Live and installed-system package management |
| Config Hub | Catalog-backed configuration outside Welcome |

## Current Surfaces

The implemented pages provide:

- a Live-only manual Calamares entry;
- read-only workstation, release, kernel, and network status;
- catalog v2 profile metadata and the installer selection receipt;
- fixed launchers for Shelly, System Settings, Info Center, Timeshift, and
  Konsole when those executables exist;
- fixed links to Arch Linux News, ArchWiki, Linxira documentation, the project
  website, and GitHub;
- reviewed translations with English fallback.

## Login State

The ISO installs the `org.linxira.Welcome` application entry and one XDG
autostart entry. `Show Welcome at login` defaults to enabled. When disabled,
autostart invocations exit without opening the window; users can still open
Welcome from the application menu. The preference and selected language persist
per installed user through `QSettings`.

The Live profile is ephemeral, so Welcome opens on every fresh Live boot. This
switch controls login autostart; it is not a one-time first-run completion flag.

## Privilege Model

Welcome runs as the user and executes no shell strings, `sudo`, `pkexec`, or
package transactions. It launches only fixed executable paths compiled into the
application, and enables an action only when that executable exists. Catalog
strings are data and are never executable commands.

Calamares owns installation transactions. Shelly owns post-install graphical
package management. Any future privileged Config Hub operation remains outside
Welcome and requires its own audited interface.

## Implementation

The implementation lives in the independent `linxira-welcome` source tree. Its
runtime dependencies are Python 3, PySide6, and catalog v2 under
`/usr/share/linxira/catalog/`. The current implementation does not inherit the
superseded onboarding implementation or its Rust/GTK, CachyOS,
privileged-command, or package-manager assumptions.
