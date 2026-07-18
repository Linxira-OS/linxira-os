# Linxira OS Repository Strategy

## 1. Bootstrap Model

Linxira OS does not need a full self-hosted Arch mirror to publish its first
release. The bootstrap model separates ownership and update responsibility:

```text
Arch official repositories
  -> base system, desktop, kernels, drivers, recovery tools

Linxira GitHub Pages repository
  -> Linxira-owned packages and required integration packages

Linxira ISO GitHub Releases
  -> installation images, checksums, signatures, manifests
```

This model avoids redistributing CachyOS binaries and avoids taking ownership of
the complete Arch package lifecycle before Linxira has durable infrastructure.
RC7 still uses locally built packages and unsigned development repository
metadata; the signed public repository described below is a release requirement,
not a completed dependency.

## 2. Package Scope

The public `[linxira]` repository may initially publish:

- `linxira-keyring`
- `linxira-mirrorlist`
- `linxira-artwork`
- `linxira-hooks`
- `linxira-welcome`
- `linxira-config-hub`
- `linxira-profiles`
- `linxira-recovery-meta`
- `linxira-calamares`
- `linxira-calamares-config`
- `shelly`, as an explicitly adopted source-built post-install package

The repository must not initially replace Arch packages such as the kernel,
glibc, systemd, Mesa, pacman, firmware, or the desktop stack. Experimental
kernels and optimized packages belong in a separate non-default staging
repository.

## 3. Hosting

GitHub Actions builds Linxira packages in clean Arch environments. GitHub Pages
publishes only signed binary packages, signatures, repository databases, the
public signing key, and machine-readable manifests.

The client configuration is:

```ini
[linxira]
SigLevel = Required DatabaseOptional
Server = https://linxira-os.github.io/packages/$arch
```

The repository is enabled only after `linxira-keyring` installs the approved
public package-signing key.

GitHub Releases hosts early ISO files and their detached signatures. Release
assets are not used as mutable pacman repository endpoints.

## 4. Signing

- The offline primary key certifies a dedicated package-signing subkey.
- CI receives only the dedicated signing subkey.
- Every package and repository database is signed.
- The public key fingerprint is pinned in source and release documentation.
- Signing secrets never enter the ISO source tree or general build VM images.
- The package workflow must fail closed when signing material is unavailable.

The current encrypted recovery backup remains offline and is not uploaded to
GitHub Actions.

## 5. Update Channels

Linxira follows Arch's coherent full-system upgrade model. Normal users update
Arch and Linxira packages together with:

```bash
sudo pacman -Syu
```

Linxira packages use three promotion stages:

```text
pull request build
  -> staging artifact and tests
  -> signed production repository
```

System-critical integration packages are promoted with an ISO compatibility
test. User-facing applications such as Linxira Welcome and Config Hub may release
independently when their dependency and migration tests pass.

Repository publication must be atomic: upload packages and signatures first,
then publish the newly generated repository database last. Never expose a
database that references missing artifacts.

## 6. Offline Installer Repository

Each release ISO contains a separate signed repository with the exact package
closure used for installation. It is generated from a recorded Arch package
cohort and the promoted Linxira repository state. RC7 embeds the same structural
boundary with unsigned development metadata.

The installer uses the embedded repository only while installing the target.
Before first boot it removes the file-based repository configuration and enables
normal Arch mirrors plus `[linxira]`.

The release records:

- Arch package cohort date
- complete package manifest and versions
- Linxira repository commit and database checksum
- ISO checksum and signature
- Calamares package source revision

## 7. Rollback And Retention

GitHub Pages is a distribution endpoint, not a sufficient package archive.
Before publishing a new package database, preserve the previous database and all
referenced packages in offline storage.

Until an online archive exists:

- retain at least the current and previous promoted repository snapshots;
- retain package build logs, source revisions, signatures, and manifests;
- retain release artifacts on separate physical storage;
- use Arch Linux Archive dates for reproducible base-package recovery;
- never require a package version that can no longer be reconstructed.

## 8. Recommended Software

Linxira Welcome presents curated metadata and recommendation links, but it does
not redistribute third-party software merely because it is recommended.

Each catalog entry declares one supported source:

- Arch official repository
- Linxira repository
- Flatpak/Flathub
- vendor repository or vendor download
- source build, only when Linxira explicitly accepts maintenance ownership

Catalog v2 is metadata and an allowlist contract. It contains package identifiers
and presentation data, never executable commands, and it does not perform a
package transaction. Calamares owns installation transactions and Shelly owns
normal graphical package management.

Entries such as WPS Office require a separate license, source, architecture,
update, and regional-availability review. Proprietary software is optional and
must not block installation or system upgrades.

## 8.1 Shelly Software Manager

Shelly is the default graphical software manager in the Live and installed
systems. It may manage Arch repositories, AUR packages, Flatpaks, and AppImages,
but it is not used by Calamares or the offline installation repository.

- Linxira builds Shelly from a pinned upstream source archive. Release packages
  must be signed with the Linxira package key; RC7 uses a local development
  package.
- Linxira does not consume CachyOS, AUR, or Seafoam binary repositories to
  build or distribute Shelly.
- AUR remains disabled until the user explicitly accepts its risk notice.
- Flatpak and AppImage support may be enabled by the user; Linxira does not add
  Flathub or another remote without confirmation.
- Background update checks are disabled for a fresh profile and must honor the
  same AUR and Flatpak opt-in settings as the main interface.
- The Linxira patch removes Shelly's GUI action for deleting pacman's lock
  file. Pacman lock conflicts must resolve through a documented recovery flow.
- The packaged GUI does not download icon archives or release assets at startup.

## 9. Migration Beyond GitHub

Move package and ISO delivery to object storage or dedicated mirrors when one
or more of these conditions becomes true:

- release assets or repository traffic approach GitHub service limits;
- the repository needs a durable online historical archive;
- optimized kernels or a broad package set materially increase storage;
- regional download performance requires multiple mirrors;
- publication availability becomes a release-level service objective.

Suitable later targets include S3-compatible object storage, Cloudflare R2,
Backblaze B2, institutional mirrors, and community mirrors. The public URL
should be abstracted through `linxira-mirrorlist` so clients do not require a
manual pacman configuration migration.

## 10. Full Repository Readiness Gate

Linxira may consider self-hosted kernels or broad optimized repositories only
after it has:

- reproducible clean-chroot builders;
- isolated signing and key rotation;
- atomic publication;
- package and source retention;
- an online archive and tested rollback;
- monitoring and repository integrity checks;
- sufficient storage and bandwidth;
- license and source-offer compliance;
- a documented security-response process.

Until then, performance experiments remain internal and the public base remains
the official Arch package set.
