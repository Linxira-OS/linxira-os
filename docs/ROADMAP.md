# Linxira OS Roadmap

## v1.0 — Deb Desktop (Current Focus)

### Phase 1: Foundation
- [x] Create repository structure
- [x] Define product architecture
- [x] Create artwork repository
- [x] Create website
- [x] Create wiki

### Phase 2: Build System
- [ ] Create linxira-live repository
- [ ] Set up live-build configuration
- [ ] Configure package lists (based on Linux Mint 22.x)

### Phase 3: Configuration
- [ ] Create linxira-config-hub (GUI + CLI)
- [ ] Create linxira-default-settings (KDE Plasma)
- [ ] Create linxira-calamares-settings (installer)
- [ ] Create linxira-meta-packages

### Phase 4: Testing
- [ ] Build first ISO
- [ ] Verify live environment (auto-login, wallpaper, installer)
- [ ] Verify installation flow (partitioning, user creation, bootloader)
- [ ] Verify btrfs + Timeshift snapshots
- [ ] Test on various hardware (USB boot, Ventoy, Hyper-V)

### Phase 5: Release
- [ ] Optimize package list
- [ ] Add pre-installed tools (mise, Miniforge, Distrobox)
- [ ] Add development toolchain (C/Rust/Go)
- [ ] Prepare WSL tarball
- [ ] Release v1.0

## v1.1 — NIRI Integration

- [ ] Verify NIRI on Linux Mint/Ubuntu
- [ ] Add NIRI session to SDDM
- [ ] Configure waybar, app launcher, notifications
- [ ] Test KDE app compatibility under NIRI
- [ ] Document NIRI-specific features

## v1.2 — Server & WSL

- [ ] Create server variant (desktop - desktop environment)
- [ ] Create WSL variant
- [ ] Test on various cloud platforms
- [ ] Document server-specific features

## Future

- [ ] Arch-based variant (pac series)
- [ ] More workflow templates
- [ ] Hardware-specific optimizations
- [ ] Community contributions
