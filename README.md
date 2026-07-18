# Linxira OS

> **Name notice:** Linxira OS is unrelated to LinxISA, linx-isa, or any custom
> instruction-set architecture project. It is an independent Linux distribution.

## 中文

**Linxira OS 是一个面向科研与 AI 工作流的 Linux 工作站发行版。**

当前基线直接使用 Arch Linux 官方仓库的软件包与工具链。独立的
`linxira-iso-direct` 构建完整 KDE Plasma Live 会话；用户从 Linxira Welcome
或应用菜单手动启动 Calamares。已安装系统使用 Btrfs + Timeshift 系统回滚和
官方 Arch 双内核配置。

首发设计定义两种内核配置，每种都只安装两个内核；RC7 当前仅实现标准配置，
响应性桌面配置仍待完成：

- 标准：`linux` + `linux-lts`
- 响应性桌面：`linux-zen` + `linux-lts`

基础系统来自 Arch 官方仓库。Linxira 自有组件和必要集成包独立构建；发布仓库
签名仍待完成。CachyOS 资料仅可作为有许可证的历史实现参考，不是当前依赖。

RC7 已通过静态检查以及 QEMU BIOS/UEFI 菜单启动验证。完整写盘安装、安装后
首次启动和恢复流程仍待验收。

## English

**Linxira OS is a Linux workstation distribution for scientific and AI
workflows.**

The current baseline uses packages and tooling directly from the official Arch
Linux repositories. The independent `linxira-iso-direct` project builds a full
KDE Plasma Live session. Users start Calamares manually from Linxira Welcome or
the application menu. Installed systems use Btrfs + Timeshift rollback and an
official Arch dual-kernel configuration.

The release design defines two profiles, each with exactly two kernels. RC7
currently implements only Standard; Responsive desktop remains pending:

- Standard: `linux` + `linux-lts`
- Responsive desktop: `linux-zen` + `linux-lts`

Arch repositories provide the base system. Linxira-owned components and required
integration packages are built independently; release repository signing is
still pending. CachyOS material is historical licensed reference only, not a
current dependency.

RC7 has passed static checks and QEMU BIOS/UEFI menu boot tests. Full disk
installation, first boot of the installed system, and recovery acceptance are
still outstanding.

## Product Architecture

- [Architecture](docs/ARCHITECTURE.md)
- [Milestone 1](docs/MILESTONE_1.md)
- [Installer UX](docs/INSTALLER_UX.md)
- [Linxira Welcome](docs/LINXIRA_WELCOME.md)
- [Software Catalog](docs/SOFTWARE_CATALOG.md)
- [Repository Strategy](docs/REPOSITORY_STRATEGY.md)
- [Roadmap](docs/ROADMAP.md)

## Related Repositories

### Distribution

- [linxira-artwork](https://github.com/Linxira-OS/linxira-artwork)
- [linxira-wiki](https://github.com/Linxira-OS/linxira-wiki)
- [Linxira-OS.github.io](https://github.com/Linxira-OS/Linxira-OS.github.io)
- [linxira-iso-direct](https://github.com/Linxira-OS/linxira-iso-direct)
- [packages](https://github.com/Linxira-OS/packages)

### Independent AI Projects

- [extendai-lab-Studio](https://github.com/Linxira-OS/extendai-lab-Studio)
- [extendai-lab-cli](https://github.com/Linxira-OS/extendai-lab-cli)
- [linxira-pulse](https://github.com/Linxira-OS/linxira-pulse)
- [linxira-skills](https://github.com/Linxira-OS/linxira-skills)

## Relationship Notice

Linxira OS is built directly on Arch Linux and is an independent project not
affiliated with or endorsed by Arch Linux. Linxira OS is not affiliated with or
endorsed by CachyOS.
