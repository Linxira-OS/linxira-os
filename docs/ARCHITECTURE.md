# Linxira OS Product Architecture v2.0

## 1. Product Definition

Linxira OS is a **scientific and AI workstation Linux distribution** based on
CachyOS (Arch ecosystem). It provides a modern, performant platform for
bioinformatics, AI/ML, mathematics, physics, chemistry, and engineering workflows.

The primary goals are:

1. **Performance-first** — leveraging CachyOS optimizations for maximum throughput.
2. **Reproducibility** — research and development environments must be portable,
   lockable, and restorable.
3. **Modern desktop** — KDE Plasma with NIRI compositor support.
4. **Developer-friendly** — pre-configured toolchains for scientific computing.

## 2. Technical Foundation

### 2.1 Base Distribution

- **CachyOS** (Arch Linux ecosystem)
- Rolling release with semi-annual stable snapshots
- Optimized packages for x86-64-v3/v4 architectures

### 2.2 Kernel Strategy

**Default dual-kernel configuration:**

| Kernel | Version | Purpose |
|--------|---------|---------|
| `linux-cachyos` | 6.18.x (mainline) | Daily use, latest features |
| `linux-cachyos-lts` | 6.18.x (LTS) | Backup, stability |

GRUB boot menu provides 3 options:
1. CachyOS (mainline kernel)
2. CachyOS LTS Kernel
3. CachyOS Legacy Hardware (nomodeset)

### 2.3 Desktop Environment

- **KDE Plasma** (primary)
- **NIRI** (scrolling-tiling Wayland compositor, optional)
- SDDM display manager

### 2.4 Package Management

| Manager | Purpose |
|---------|---------|
| pacman | System packages |
| yay/paru | AUR helper |
| mise | Multi-language version manager |
| Miniforge3 | Scientific computing (bioconda + conda-forge) |
| Distrobox | Containerized environments |
| uv | Fast Python packages |

## 3. Layered Architecture

### 3.1 Base System Layer

- CachyOS base with optimized packages
- Kernel and driver management
- System initialization and services

### 3.2 Linxira Platform Layer

- Linxira Config Hub (source management, workflow templates)
- Linxira Welcome (onboarding)
- Branding and desktop defaults

### 3.3 Reproducibility Layer

- `mise` for multi-language version management
- `Miniforge3` for scientific computing
- `Distrobox` for containerized environments
- `uv` for fast Python package management

### 3.4 Domain Environment Layer

- Bioinformatics (via BioArchLinux repository)
- AI/ML (PyTorch, TensorFlow)
- Scientific computing (NumPy, SciPy, etc.)

## 4. ISO Variants

### 4.1 Desktop ISO

- User selects desktop environment during install
- Options: KDE Plasma / NIRI
- Pre-installed development and scientific toolchain
- For: general users, researchers

### 4.2 Base ISO

- No desktop environment, CLI only
- User configures desktop and tools manually
- Pre-installed base system tools and package managers
- For: servers, developers, advanced users

### 4.3 WSL ISO (future)

- Based on Base version
- Optimized for WSL
- For: Windows users

## 5. Repository Structure

### 5.1 Current Repositories

| Repository | Role |
|------------|------|
| `linxira-os` | Meta-repository, architecture docs |
| `linxira-artwork` | Brand assets (logos, wallpapers) |
| `linxira-wiki` | Documentation (Linxira-specific only) |
| `Linxira-OS.github.io` | Official website |
| `linxira-iso` | ISO build system (forked from CachyOS-Live-ISO) |
| `linxira-config-hub` | Configuration center (GUI + CLI) |

### 5.2 AI Repositories (independent)

| Repository | Role |
|------------|------|
| `extendai-lab-Studio` | AI research orchestration |
| `extendai-lab-cli` | AI coding agent |
| `linxira-pulse` | System-level AI assistant |

## 6. Implementation Waves

### Wave 1 (Current)

- Establish product architecture
- Fork CachyOS-Live-ISO for build system
- Create artwork, website, wiki

### Wave 2

- Configure ISO build system
- Implement Linxira Config Hub
- Build and test first ISO

### Wave 3

- NIRI compositor integration
- WSL tarball
- Workflow templates

### Wave 4 (Future)

- Domestic GPU support (when hardware available)
- Additional desktop environments
- Server line (if needed)

## 7. Non-Goals

- No attempt to embed every scientific tool in ISO
- No attempt to support domestic GPU in v1.0
- No server or WSL line in v1.0
- No NIRI integration in v1.0
