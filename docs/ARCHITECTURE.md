# Linxira OS Product Architecture v1.0

## 1. Product Definition

Linxira OS is a **personal scientific workstation platform** oriented first toward
bioinformatics, then toward broader AI, mathematics, physics, chemistry, and
engineering workflows.

The four primary goals are:

1. **Bioinformatics-first** — the first-class user is a serious scientific user
   with an independent machine.
2. **Reproducibility** — research and development environments must be portable,
   lockable, and restorable.
3. **Hardware friendliness** — the platform must be usable on mainstream
   personal workstation hardware with explicit support priorities.
4. **Personal workstation UX** — the platform needs a real GUI, not only a
   shell-driven experience.

## 2. Supported Product Lines

### 2.1 Deb Desktop (v1.0)

- **Role**: stable desktop line for most users.
- **Base**: Linux Mint (Ubuntu LTS ecosystem).
- **Audience**: users who value predictable behavior, documentation, and broad
  compatibility.
- **Surface**: GUI-first, installer-equipped, workstation-focused.

### 2.2 Not in the First Delivery Wave

- Server line stays deferred until Desktop is stable.
- WSL line stays deferred until Desktop is stable.
- NIRI compositor integration stays deferred until v1.1+.

## 3. Layered Architecture

Linxira should combine the strengths of multiple ecosystems by separating
responsibilities into layers, not by forcing everything into one ISO.

### 3.1 Base Distribution Layer

Responsible for:

- boot and init
- desktop session
- installer
- system networking
- device drivers and kernel integration
- native package manager (apt)

This layer is distribution-specific (Linux Mint / Ubuntu LTS).

### 3.2 Linxira Platform Layer

Responsible for Linxira-specific product capabilities:

- GUI control center and system tools
- source / mirror management
- runtime management
- hardware detection and guidance
- onboarding and first steps
- branding and desktop defaults
- terminal experience and MOTD

This layer should be as shared as possible across future product lines.

### 3.3 Reproducibility Layer

Responsible for environment portability and locking:

- `mise` as a first-class multi-language version manager
- `Miniforge3` as a scientific computing package manager
- `Distrobox` for containerized environments
- `uv` for fast Python package management

This layer must not replace the system package manager. It complements it.

### 3.4 Domain Environment Layer

Responsible for domain-specific stacks:

- bioinformatics
- AI / ML
- mathematics
- chemistry
- physics
- engineering

These should be delivered primarily through profiles, templates, meta packages,
and reproducible environment definitions rather than by embedding everything in
the base ISO.

## 4. GUI Product Modules

### 4.1 Linxira Config Hub

Primary responsibilities:

- manage APT mirrors
- manage mise runtimes
- manage Miniforge channels
- benchmark mirrors and show region / provider metadata
- provide safe source switching and restore paths
- configure system services (SSH, remote desktop, firewall)
- offer preset workflow templates

### 4.2 Linxira Welcome

Primary responsibilities:

- onboarding
- initial system setup
- source/runtime/hardware guidance
- launch points into Config Hub

## 5. Package and Preinstall Strategy

### 5.1 Principles

- Do **not** embed every important scientific package into the ISO.
- Keep the base image focused on a stable workstation entry experience.
- Heavy or license-sensitive tools should usually be user-selected.

### 5.2 Deb Desktop Base

Deb Desktop should ship with:

- KDE Plasma desktop and installer
- browser, terminal, file manager, editor set
- `mise` (multi-language version manager)
- `Miniforge3` (with bioconda + conda-forge)
- `Distrobox` (containerized environments)
- Linxira Config Hub
- a small number of scientific entry tools

Large scientific application sets should move to profiles/templates rather than
the default ISO payload.

## 6. Asset and Branding Pipeline

Linxira branding assets must be treated as a system, not scattered files.

### 6.1 Asset Categories

- core branding: SVG logo, simplified logo, ASCII logo, slogans
- desktop branding: wallpapers, login/installer branding, icons
- terminal branding: MOTD, fastfetch/neofetch templates, shell banners
- docs/web branding: release pages, README assets, website headers

### 6.2 Required Outputs

- Chinese desktop wallpaper set
- English desktop wallpaper set
- shared low-language-dependence fallback wallpaper set
- terminal ASCII identity derived from the same brand source

## 7. Repository Boundaries

### 7.1 Current Workspace Roles

- `linxira-artwork` — shared visual assets (canonical source)
- `linxira-os` — meta-repository for product architecture and top-level docs
- `linxira-wiki` — documentation (only Linxira-specific features)
- `Linxira-OS.github.io` — official website

### 7.2 Future Repositories

- `linxira-live` — Deb Desktop live ISO build system
- `linxira-config-hub` — cross-ecosource configuration tool
- `linxira-default-settings` — KDE Plasma defaults
- `linxira-calamares-settings` — installer configuration
- `linxira-meta-packages` — meta package definitions

### 7.3 External Repositories

- `extendai-lab-Studio` — AI research orchestration layer
- `extendai-lab-cli` — AI coding agent
- `linxira-pulse` — system-level AI assistant

## 8. Implementation Waves

### Wave 1 (Current)

- establish product architecture and repository structure
- create artwork, website, and wiki
- stabilize Deb Desktop base image

### Wave 2

- create linxira-live build system
- create linxira-config-hub
- build and test first ISO

### Wave 3

- create linxira-default-settings
- create linxira-calamares-settings
- create linxira-meta-packages
- integration testing

### Wave 4

- NIRI compositor integration
- WSL tarball
- Server line (if needed)

## 9. Non-Goals for the Current Iteration

- no attempt to make one ISO contain every scientific tool
- no attempt to collapse all ecosystems into one package manager
- no server or WSL line in v1.0
- no NIRI integration in v1.0
