# 正式发布前检查 · 2026-10-24

> 依据 `docs/RELEASE_STANDARD.md` 与 `docs/ROADMAP.md`；结论以代码/CI/仓库现状为准。
> 本文件只记录事实与待办，不构成法律意见。

## 1. 结论：现在不能称正式版

| 门槛 | 标准要求 | 现状 |
|---|---|---|
| 验收门 | 7 项（13 子项）全过 | **2/13**（仅 QEMU BIOS/UEFI 菜单启动）；写盘安装、Btrfs 加密/非加密、双内核、首启、Package Center、Timeshift、live-ISO 恢复、grub-btrfs、NVIDIA DKMS、签名/完整性 **无记录** |
| Release 段 | 安装文档 / ISO+清单+校验和+签名 / 已知限制+回滚 / 首个 Arch cohort / manifest 证据 | **0/5**；`releases/` 只有 README+schema，**0 份 manifest** |
| 构建产出 | manifest + SBOM 随构建生成 | `build-direct-iso.sh` **无** manifest/SBOM 代码 |
| 文档 | 安装指南 / 已知限制 / 回滚程序 | `linxira-os` 顶层只有 README+AGENTS |
| WSL | 有 CI、产物可追溯 | **无任何 CI run**；线上产物为 2026-09-29 构建 |
| 许可证 | 分发 GPL/AGPL 二进制须提供源码与 notices | **无 SOURCE-OFFER、无聚合 NOTICES**，manifest schema 无 notices/sourceOffer 字段 |

代码面本身健康（13 仓 CI 全绿），卡点是**证据与发布工程**，不是功能。

## 2. 发布链审计（已修 3 处，补发 5 个 release）

五段链：源仓 `VERSION` → `release.yml` → `v<ver>` Release → `auto-bump`（`upstream-sync.toml` 白名单）→ `packages.yml` 构建+签名 → `linxira-packages` → Pages。

已确认并修复：

1. **`linxira-artwork` release.yml 监听 `main`，实际分支 `master`** ⇒ VERSION 变更永不发 release（8 月那次是手发测试 release）。修 `.github/workflows/release.yml` + 手动 dispatch → `v1.1.0` 已出，auto-bump 已把 PKGBUILD 推到 `pkgver=1.1.0`/`pkgrel=1`（`_commit=fa84ae6`，两个 source 的 sha256 均已更新）。
2. **`packages/scripts/rollout-release-workflows.py` 两个根因**：REPOS 表把 artwork 分支写成 `main`；VERSION 与 release.yml 分两次 PUT（两个 commit）⇒ 首次铺开不匹配 `paths:["VERSION"]`，自动触发次数为 0。已修：模板改 `branches: [main, master]`；写入后补一次 `workflow_dispatch`（对已存在 tag 幂等）。
3. **5 个仓有 VERSION 却从未有 Release**（completion-agent 0.1.1 / gaming-manager 0.3.0 / hardware-driver-manager 0.4.0 / hwd-detector 1.23.0 / kernel-manager 0.1.0）——其中 3 个已在 ISO 包清单里。已全部 dispatch，`v<ver>` 已生成，等每日 sync 消费。

遗留（不影响发布）：`linxira-calamares/.github/workflows/push.yml` 监听不存在的 `calamares` 分支（C 档上游仓）；`linxira-hooks` 与 `linxira-kernel-manager` 不在 `upstream-sync.toml`（前者未进镜像，后者未进任何包清单，均无实际影响）。

## 3. 许可证与上游血缘（P0/P1）

### 3.1 阻断项

| 编号 | 级别 | 缺口 | 证据 |
|---|---|---|---|
| G1 | **P0** | 分发 GPL/AGPL 二进制但无源码提供、无聚合 NOTICE；manifest 无对应字段 | `iso-release/README.md`（仅 ISO+SHA）、`linxira-packages/README.md`、`linxira-wsl/build-wsl-rootfs.sh`、`releases/manifest.schema.json`（required 无 notices/sourceOffer） |
| G2 | **P0** | Calamares 包 `license=` 错项/漏项，且只装 BSD-2-Clause 文本 | `packages/packages/calamares/PKGBUILD:9`（`CC-BY-SA-4.0` 实为 `CC-BY-4.0`；漏 MIT/CC0/LGPL-3.0；`LGPL-2.1-or-later` 实为 `LGPL-2.1-only`）；`:111` 仅装 BSD |

### 3.2 高优先（正式版前）

| 编号 | 缺口 | 证据 |
|---|---|---|
| G3 | Shelly 包无 GPL 文本、无“已被 Linxira 修改”说明 | `packages/packages/shelly/PKGBUILD:10,109-199` |
| G4 | Zeta 5 个 `-bin` 不装 `LICENSE`/`THIRD-PARTY-NOTICES.txt`（Release 附件里有，PKGBUILD 不取） | `linxira-zeta-*/PKGBUILD`；`linxira-zeta/THIRD-PARTY-NOTICES.txt` |
| G5 | Bio SDK 包不装 AGPL 文本与 `THIRD_PARTY*` | `packages/packages/linxira-bio-sdk/PKGBUILD:10,30-33` |
| G6 | **艺术仓三份许可冲突**：`LICENSE` = All rights reserved；墙纸 `metadata.desktop` = GPL-3.0；PKGBUILD = GPL-3.0-or-later | `linxira-artwork/LICENSE:1-2`、`usr/share/wallpapers/*/metadata.desktop`、`packages/packages/linxira-artwork/PKGBUILD:10` |
| G7 | 官网法律页称"自研代码 GPL-3.0"，实际绝大多数仓是 MIT；文档 CC BY-SA vs 博客 CC BY 4.0 互相矛盾；网站仓无 LICENSE | `Linxira-OS.github.io/src/pages/en/legal/license.astro:11-22`、`CONTRIBUTING.md:50` |
| G8 | CachyOS 派生配置仓大量无 LICENSE；`linxira-settings` README 写 GPL-1.0、`LICENSE.md` 是 GPL-3.0；上游 `cachyos-settings`/`cachyos-hypr-noctalia` PKGBUILD 写 **GPL-1.0-only**（与 GPL-3.0 不可混用） | `linxira-settings/README.md:4` vs `LICENSE.md:1-5`；`linxira-{kde,gnome,hypr}-*/README.md`（无 LICENSE） |

### 3.3 中/低

- G9 hooks/keyring 声明 GPL-3.0-or-later 但仓内/包内无 GPL 文本（keyring 本质是密钥数据，license 字段可疑）。
- G10 `linxira-iso-direct` 无 LICENSE，但 metainfo 声明 GPL-3.0-or-later；含 archiso 派生脚本（`airootfs/usr/local/bin/choose-mirror`、`livecd-sound`）。
- G11 `linxira-update` 改自 `Antiz96/arch-update`，源码头缺上游版权行与逐文件修改说明。
- G12 Calamares 4 个补丁无 GPLv3 §5(a) 式修改声明。
- G13 `linxira-skills/LICENSE` 版权人写成 `BOHUYESHAN-APB`，与 `CITATION.cff` 不一致。
- G14 **`linxira-rate-mirrors` 是 CC BY-NC-SA 3.0（非自由、禁商用）** ⇒ 不得进 ISO/[linxira]（目前也确实没进）。
- G15 wiki 仓 LICENSE=MIT，官网称文档 CC BY-SA 4.0。
- G16 ISO/WSL 未汇总 Arch 基础包与非免费组件（firmware、unrar、libdvdcss）的许可索引。

### 3.4 无需动作

- `openoffice-bin` 仅 catalog 条目（`provider=aur`），Linxira 未重打包 ⇒ 暂无 Apache NOTICE 义务。
- 未发现 GPL-2.0-only 与 Apache-2.0 直接互斥；`linxira-components` 依赖 `pyalpm`（GPL-2.0-or-later）属依赖披露问题。

### 3.5 需决策（法律口径，需版权人拍板）

1. **艺术仓许可**：保留品牌权利（PKGBUILD 改 `LicenseRef-Linxira-Brand`，墙纸 metadata 同步）还是整体 GPL-3.0-or-later（改 `LICENSE`）？现状三份声明互相矛盾，必须选一。
2. **CachyOS 派生仓**：按上游 GPL-1.0-only 补文本（不得擅自升级到 GPL-3.0），还是确认上游实际按 GPL-3.0 发布？`linxira-settings` 需先定。
3. **源码提供方式**：随 ISO 附源码 tarball（GPLv3 §6(a)）＋同服务器 URL（§6(d)）＋书面 offer 兜底，选哪种组合。

## 4. Shell 策略（事实 + 建议）

| 环境 | 实际安装 | 默认 shell |
|---|---|---|
| ISO live | bash、zsh + grml-zsh-config | root = `/usr/bin/zsh`（跑 `~/.zlogin` 自动安装），安装器用户 `installer` = bash |
| 装机后系统 | bash | Calamares 新用户 `/bin/bash`；root bash |
| WSL | bash | root bash |
| fish | **未安装**；`linxira-fish-config` 未打包、未登记、硬编码 CachyOS 路径 | — |
| nushell | 未安装；Arch extra `0.116.1-1`，42.9 MB 包 / **210.2 MB 安装** | — |
| 其他 | starship / fisher / zoxide / fzf / atuin 均未装；carapace、oh-my-zsh、powerlevel10k 不在官方仓 | — |

建议：**默认保持 bash**（POSIX、脚本兼容）；fish/zsh/nushell 作为"按需安装"由 welcome 的独立终端页提供，不预装（nushell 体积 210 MB，预装无收益）。安装路径复用 `linxira-components plan → confirm → pkexec apply`（唯一合规提权面），不改默认 shell 除非用户显式操作（`chsh` 需密码，界面只给可复制命令）。

## 5. 品牌资产（增量替换现状）

已推送：站点 SVG+PNG 全套；艺术仓 v1.1.0（矢量/字符画/文档/生成器，含 release.yml 修复）；打包侧 hicolor 图标改自包含图标（深色面板可见）；WSL 补 `linxira-artwork`+`fastfetch`+配置；ISO 树 Calamares logo 副本。

镜像生效条件：`linxira-artwork 1.1.0` 包构建完成 ⇒ 下次 ISO/WSL 构建。左侧菜单图标 `icon=linxira`（`airootfs/etc/skel/.config/plasma-org.kde.plasma.desktop-appletsrc:36`）随主题图标自动更新。

待办：crash-watch 图标缺失（desktop 声明 `org.linxira.CrashWatch` 但无 SVG）；update 9 个状态图标；recovery 图标；plymouth `watermark.png`；WSL/ISO 重建。

## 6. 整改顺序（发布冲刺）

1. **G1 + G2**：manifest schema 增 `notices`/`sourceOffer`/`licenseManifest`；构建脚本生成 `THIRD-PARTY-NOTICES.md`、`license-manifest.json`、`SOURCE-OFFER`；Calamares 包补齐 LICENSES 与 `license=`。
2. **G6 + G8**（决策后一次做完）：统一艺术仓三份声明；派生仓补 `UPSTREAM.md` + 上游许可文本。
3. **G3/G4/G5**：Shelly、Zeta 5 个 `-bin`、Bio SDK 在 `package()` 里装许可证与 notices。
4. **G7 + G15**：官网法律页与各仓 LICENSE 对齐；网站仓补 LICENSE。
5. 验收门：按 `ROADMAP.md` 逐项跑并写进 manifest（这是"正式版"的最后一块）。
6. 资产收尾：crash-watch/update/recovery 图标、plymouth 位图、镜像重建。
