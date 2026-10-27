# Linxira OS Roadmap

只保留通往正式版的路线。与发布无关的事项放在最后，不占用发布排期。

## 目标

发布一个可安装的桌面 ISO：Arch 官方包为基线，加上我们自己的系统组件（含 Zeta 完整版），
双内核（`linux` / `linux-lts`），Btrfs + Timeshift 快照，首次启动即可用。

正式版的定义以 [`RELEASE_STANDARD.md`](RELEASE_STANDARD.md) 为准：7 个验收门全部通过，
并且每个发布的 ISO 有一份可校验的 release manifest。

## 阶段

### P1 — 构建可复现

完成标准：用当前包集从干净环境构建 ISO，产物同时包含许可三件套
（第三方声明、源码提供、许可清单）和 release manifest 草稿。

- [x] 自研包链路（CI 构建、签名、同步到 `[linxira]`）
- [x] Zeta 完整版进入独立仓（`[linxira-zeta]`，签名可校验）
- [x] 目标系统默认安装 Zeta，并登记附加仓以便随 `update` 升级
- [x] 构建期生成许可三件套（WSL 镜像已验证）
- [ ] ISO 用新包集实跑一遍（最后一次 ISO 是 2026-09-30 的 r17）
- [ ] 构建期生成 release manifest（字段见 `releases/manifest.schema.json`）

### P2 — 验收（QEMU，SeaBIOS + OVMF，虚拟盘）

完成标准：7 个门每项都有一条带时间、结果、日志位置的记录，写入 manifest。

### Verification

- [x] 菜单引导 BIOS / UEFI（RC7，需随新 ISO 重跑）
- [ ] 门 1 固件引导：新 ISO 在 BIOS 与 UEFI 下都能进入安装器
- [ ] 门 2 写盘安装：BIOS 与 UEFI 各装一次，普通 Btrfs 与加密 Btrfs 各一次
- [ ] 门 3 双内核：`linux` 与 `linux-lts` 都能引导，initramfs 正常
- [ ] 门 4 首次启动：装好的系统进入桌面，`update` 可执行
- [ ] 门 5 Package Center：装一个包、卸一个包，事务完整
- [ ] 门 6 恢复：Timeshift 建快照并还原；live 镜像能救援；GRUB 能列出快照
- [ ] 门 7 完整性：包签名校验通过，已安装文件与包清单一致

### P3 — 修复与复跑

完成标准：P2 中的缺陷全部修复，并对受影响的门重跑。不允许带着已知失败的门发布。

### P4 — 发布

完成标准：下列文件全部公开，且与 manifest 中记录的哈希一致。

### Release

- [ ] ISO、SHA-256 校验和、签名
- [ ] release manifest（源码提交、包清单、仓库索引哈希、ISO 哈希、许可清单、构建者、各门结果）
- [ ] 安装说明（简短，以安装器实际界面为准）
- [ ] 已知限制与回滚说明（回滚 = Timeshift 快照还原）

## 明确不做（发布前）

- 显卡硬件验证。驱动管理只负责生成可复制的安装/配置步骤，不做硬件兼容认证。
- Zeta 索引历史保留与版本回滚。升级失败时重装同一版本即可。
- 包中心里的"附加源"概念或目录条目。Zeta 是系统组件，由系统默认带上，不需要额外的 UI 层。
- WSL 作为正式发布物。WSL 镜像保留为开发环境，Zeta 在 WSL 中通过 npm 安装。
- AppImage 分发。系统内的 Zeta 只走包管理器，才能随 `update` 升级。
- 重型 SBOM 工具链。用构建期生成的许可清单（列出每个包的版本与许可）作为物料清单。

## 发布后再考虑

以下内容有价值，但与发布无关，不进入当前排期。

- 科研工作站档位：Miniforge / bioconda、Apptainer、开发工具链、环境清单、离线包、CUDA 验证矩阵。
- AI 操作前快照接口（与 Zeta 的集成，需先确定权限模型）。
- 性能：基准集、`linux` 与 `linux-zen` 对比、ALHP v3 试点。只有可复现且明显的收益才会晋级。
- 服务器 profile 包拆分：把 GUI、字体、音频、浏览器从无桌面基线里移出。至少稳定一年后再动，
  当前 `desktop-server` 保留完整基线；拆分还要同步改 `linxiravalidate` 的 profile 校验。
- 无头安装器 / 自动安装：`linxira-install --config`、ISO `linxira.autoinstall`、
  VM/cloud seed。当前只有 Calamares GUI；设计已调查，暂不排期。
- VM 镜像与 cloud 变体：qcow2 / OCI / cloud-init。只有出现明确的自动化开 VM 需求再做，
  且只做一个镜像，cloud-init 作为可选 datasource，不做 cloud/非 cloud 两份资产。
- Welcome 数据驱动 System 页：读取 `capabilities.json` 的 `welcome` 元数据渲染入口；
  当前 welcome 仍以应用目录为主，未接入。
