# Linxira OS

> **Name notice:** Linxira OS is unrelated to LinxISA, linx-isa, or any custom
> instruction-set architecture project. It is an independent Linux distribution.

**Linxira OS is a Linux workstation distribution for scientific and AI
workflows.**

The baseline uses packages and tooling from the official Arch Linux repositories.
Live media, the installed target, and first-online completion are separate
delivery scopes. Phase 1 supports Plasma and COSMIC, while offline installation
guarantees the complete Plasma and COSMIC closures. GNOME remains hidden until
its online installation and independent acceptance pass.

The target uses Btrfs, Timeshift, `linux`, and `linux-lts`. Desktops, system
capabilities, scientific capabilities, and applications have separate data
semantics. Category and preset nodes never become indivisible installation
units. Welcome provides status and routing; Package Center, Config CLI, driver,
kernel, update, and recovery products keep independent ownership boundaries.

RC17 was rejected after a real disk installation exposed an initramfs failure,
and remains frozen as a rejected diagnostic. The diagnosed fix passed clean
Calamares packaging and disposable Btrfs dual-kernel target tests; the r-series
rebuild that followed is the current downloadable test image (r17, 2026-09-30);
no official release has been announced yet.

## Product Architecture

- [Phase 1 Architecture](docs/PHASE1_ARCHITECTURE.md) - current authority
- [Architecture](docs/ARCHITECTURE.md)
- [Milestone 1](docs/MILESTONE_1.md)
- [Installer UX](docs/INSTALLER_UX.md)
- [Linxira Welcome](docs/LINXIRA_WELCOME.md)
- [Software Catalog](docs/SOFTWARE_CATALOG.md)
- [Repository Strategy](docs/REPOSITORY_STRATEGY.md)
- [Roadmap](docs/ROADMAP.md)

Documents that describe RC7-RC13 are retained as historical evidence. When they
conflict with the Phase 1 architecture, the Phase 1 document is authoritative.

## Governance

- [Repository ownership](governance/repositories.yaml)
- [Repository policy](governance/REPOSITORY_POLICY.md)
- [Release manifests](releases/README.md)

## Related Repositories

### Distribution

- [linxira-artwork](https://github.com/Linxira-OS/linxira-artwork)
- [linxira-wiki](https://github.com/Linxira-OS/linxira-wiki)
- [Linxira-OS.github.io](https://github.com/Linxira-OS/Linxira-OS.github.io)
- [linxira-iso-direct](https://github.com/Linxira-OS/linxira-iso-direct)
- [packages](https://github.com/Linxira-OS/packages)

### System Software

- [linxira-welcome](https://github.com/Linxira-OS/linxira-welcome)
- [linxira-package-center](https://github.com/Linxira-OS/linxira-package-center)
- [linxira-config-hub](https://github.com/Linxira-OS/linxira-config-hub)
- [linxira-components](https://github.com/Linxira-OS/linxira-components)
- [linxira-catalog](https://github.com/Linxira-OS/linxira-catalog)
- [linxira-hooks](https://github.com/Linxira-OS/linxira-hooks)

### Bioinformatics

- [linxira-bio-sdk](https://github.com/Linxira-OS/linxira-bio-sdk) — Local-first bioinformatics analysis platform with 50+ analysis capabilities
  - [Website](https://linxira-os.github.io/bio-sdk/) · [Wiki](https://github.com/Linxira-OS/linxira-bio-sdk/wiki)

### Independent AI Projects

- [extendai-lab-Studio](https://github.com/Linxira-OS/extendai-lab-Studio)
- [extendai-lab-cli](https://github.com/Linxira-OS/extendai-lab-cli)
- [linxira-pulse](https://github.com/Linxira-OS/linxira-pulse)
- [linxira-skills](https://github.com/Linxira-OS/linxira-skills)

## Relationship Notice

Linxira OS is built directly on Arch Linux and is an independent project not
affiliated with or endorsed by Arch Linux. Linxira OS is not affiliated with or
endorsed by CachyOS.

---

## 简体中文

**Linxira OS 是一个面向科研与 AI 工作流的 Linux 工作站发行版。**

当前基线直接使用 Arch Linux 官方仓库的软件包与工具链。Live ISO、正式目标系统和
首次联网完成是三个独立交付范围。Phase 1 正式支持 Plasma 与 COSMIC，离线 ISO
保证 Plasma 与 COSMIC 的完整安装闭包；GNOME 必须完成联网安装和独立验收后才开放。

目标系统使用 Btrfs、Timeshift、`linux` 与 `linux-lts`。桌面、系统能力、科学能力和
应用拥有不同数据语义；应用与能力按树形叶子独立选择，分类和 preset 不是不可拆分
安装单位。Welcome 只负责状态和路由，Package Center、Config CLI、驱动、内核、
更新和恢复工具分别维护自己的产品边界。

RC17 因真实写盘安装中的 `crc32c-intel` initramfs 故障被拒绝，并冻结为 rejected
diagnostic。故障修复已通过 clean-built Calamares 包和 disposable Btrfs 双内核 target
验证；其后重建的 r 系列即为当前对外提供的测试镜像（r17，2026-09-30），尚未对外宣称正式版。
