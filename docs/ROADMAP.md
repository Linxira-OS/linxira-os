# Linxira OS Roadmap

## v1.0 — CachyOS Desktop (Current Focus)

### Phase 1: Foundation
- [x] Create repository structure
- [x] Define product architecture
- [x] Create artwork repository
- [x] Create website
- [x] Create wiki

### Phase 2: Build System
- [ ] Fork CachyOS-Live-ISO as linxira-iso
- [ ] Configure ISO branding (name, label, GRUB)
- [ ] Add Linxira tools to package list
- [ ] Test ISO build

### Phase 3: Configuration
- [ ] Create linxira-config-hub (GUI + CLI)
- [ ] Implement source management
- [ ] Implement workflow templates
- [ ] Configure KDE Plasma defaults

### Phase 4: Testing
- [ ] Build Desktop ISO and Base ISO
- [ ] Verify live environment (desktop selection, installer)
- [ ] Verify installation flow (partitioning, user creation, bootloader)
- [ ] Verify btrfs snapshots
- [ ] Verify dual kernel (mainline + LTS)
- [ ] Verify NIRI desktop
- [ ] Test on various hardware

### Phase 5: Release
- [ ] Optimize package list
- [ ] Add pre-installed tools (mise, Miniforge, Distrobox)
- [ ] Add development toolchain (C/Rust/Go)
- [ ] Prepare WSL tarball (future)
- [ ] Release v1.0

## v1.1 — NIRI Integration

- [ ] Verify NIRI on CachyOS
- [ ] Add NIRI session to SDDM
- [ ] Configure waybar, app launcher, notifications
- [ ] Test KDE app compatibility under NIRI
- [ ] Document NIRI-specific features

## v1.2 — WSL & Server

- [ ] Create WSL variant
- [ ] Create server variant (CLI only)
- [ ] Test on various platforms
- [ ] Document server-specific features

## Future

- [ ] Domestic GPU support (when hardware available)
- [ ] More workflow templates
- [ ] Hardware-specific optimizations
- [ ] Community contributions
