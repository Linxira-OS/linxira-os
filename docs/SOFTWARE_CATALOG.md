# Linxira Software Catalog

Catalog v2 is canonical metadata and an allowlist contract shared by Calamares,
Linxira Welcome, `linxira-config`, Shelly recommendation links, and generated
documentation. It is not a package manager or transaction backend. Entries hold
package identifiers and presentation metadata, never shell commands.

## Catalog Rules

Linxira recommends software without automatically becoming the upstream
binary redistributor. Every entry records:

- stable Linxira catalog ID and AppStream ID where available;
- category and profile membership;
- package or Flatpak ID;
- source type and publisher;
- license class;
- supported architectures;
- conflicts and required repositories;
- last review date.

Source preference is Arch official, Linxira-owned or explicitly adopted
source-built package, verified Flatpak, then reviewed vendor delivery. AUR and
proprietary vendor packages require a specific maintenance and license decision.

## Preinstalled Desktop

- Plasma desktop and System Settings
- Dolphin, Konsole, Kate, Okular, Gwenview, Ark, and Spectacle
- Firefox
- Shelly
- NetworkManager Plasma integration
- `fwupd` where supported
- Timeshift and system recovery integration on Btrfs installations

## Software Management

Shelly is the default graphical software manager in both the Live session and
installed system. It is never part of a Calamares installation transaction. Its
source boundaries are visible in the interface and catalog:

| Source | Default state | Policy |
|--------|---------------|--------|
| Arch official and `[linxira]` | Available | Normal signed package transactions |
| AUR | Disabled | User opt-in, PKGBUILD review, normal-user build |
| Flatpak | Disabled | User opt-in; a remote such as Flathub requires confirmation |
| AppImage | Disabled | User opt-in; user chooses the download and update source |

The table describes release policy. RC7 uses locally built packages and unsigned
development metadata; production `[linxira]` transactions remain gated on the
signed repository.

KDE Discover is not part of the default Linxira desktop. Firmware updates stay
available through `fwupd` and Config Hub/System Settings integration.

## Initial Recommendations

| Category | Applications | Source | Status |
|----------|--------------|--------|--------|
| Office | LibreOffice Fresh | Arch | Recommended profile |
| Office alternative | ONLYOFFICE Desktop Editors | Verified Flatpak | Optional |
| Mail | Thunderbird | Arch | Optional |
| Passwords | KeePassXC | Arch | Recommended |
| Media | VLC | Arch | Recommended |
| Creation | Krita, GIMP, Inkscape, Kdenlive, OBS Studio | Arch | Creator profile |
| Development | Git, base-devel, CMake, Code OSS | Arch | Developer profile |
| Containers | Podman, Distrobox | Arch | Developer profile |
| Files | Nextcloud client, Syncthing, LocalSend | Arch/verified Flatpak | Optional |
| Flatpak control | Flatseal | Flatpak | Recommended with Flatpak |
| Scientific | Miniforge bootstrap and Jupyter profile | Vendor/Linxira integration | Scientific profile |
| Bioinformatics | Apptainer and workflow manifests | Arch/Linxira integration | Bioinformatics profile |

Communication, gaming, proprietary remote-access, and region-specific software
are not preinstalled. They may be presented after source and privacy review.

## Recovery Set

Installed Btrfs systems include:

- `timeshift`
- `grub-btrfs`
- `btrfs-progs`
- `inotify-tools`
- `cronie`
- Linxira Timeshift and grub-btrfs configuration

The installer medium additionally carries diagnostic and data-recovery tools
such as `smartmontools`, `nvme-cli`, `gparted`, `testdisk`, and GNU ddrescue as
space permits. Destructive tools are not promoted as one-click Welcome actions.

## WPS Office Decision

WPS Office is deferred from the initial catalog and must not be preinstalled,
mirrored, or included in a meta-package.

Before listing it, Linxira must verify the current vendor package, redistribution
terms, personal versus commercial-use license, architecture support, update
mechanism, locale availability, font dependencies, and whether installation can
be performed through an authorized vendor channel. An unofficial AUR recipe is
not sufficient evidence of redistribution permission.

If approved later, Welcome must label it as proprietary third-party software,
show the applicable license before download, and make clear that Linxira does
not provide the binary or vendor support.

## Profiles

Stable native system compositions may use Linxira meta-packages. Optional user
profiles use versioned declarative manifests and installation receipts because a
single profile may combine native packages, Flatpak applications, and user
configuration.

Profile installation is transparent and reversible. Package install scripts do
not invoke Flatpak or write user home configuration implicitly.
