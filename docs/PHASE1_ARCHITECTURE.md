# Linxira OS Phase 1 产品与软件工程架构方案

状态：产品架构已于 2026-07-20 确认，尚未全部实现。

更新时间：2026-07-20

## 1. 这次重新理解后的产品目标

Linxira Phase 1 不是“把用户提到的功能全部放进 Welcome”，也不是“做几个软件合集”。
目标是形成一套与 CachyOS 类似、但由 Linxira 自己拥有和维护的系统产品：

1. Live ISO 是体验、安装和恢复环境。
2. 正式安装的目标系统与 Live ISO 不是同一份软件清单。
3. 安装器负责桌面、系统能力、科学能力和应用软件的初始选择。
4. 安装后的 Package Center 继续管理同一套应用和能力状态。
5. Welcome 只负责状态、引导和跳转，不实现包管理、驱动、运行时和系统配置。
6. CLI 负责快速检查和受控系统配置，不负责应用安装。
7. 驱动、内核、更新、游戏环境、恢复分别由边界清楚的系统工具负责。
8. 所有安装和配置变化都要有计划、确认、执行、验证和收据。

“成熟”应当来自模块边界、统一数据模型、事务安全、离线/在线策略和完整测试，
而不是页面数量或按钮数量。

## 2. 从 CachyOS 实际证据中学到的内容

对 `cachyos-desktop-linux-260628.iso` 的直接拆解结果：

- Live ISO 有 987 个已安装包，约 2.77 GiB SquashFS。
- Live 环境包含 Plasma、Firefox、Calamares、CHWD、恢复工具、文件系统工具、
  硬件诊断和多种虚拟机 guest 工具。
- Live 环境不包含 Steam、Wine、Lutris、Heroic、Gamescope、GameMode、Node/npm、
  Flatpak、Docker 或 Podman。
- 正常安装不是简单复制 Live 系统，而是联网执行 pacstrap，重新构建目标系统。
- 安装时会下载目标桌面、系统设置、内核、驱动配置、网络、字体、音频、开发工具等。
- `cachyos-packageinstaller` 和 `cachyos-kernel-manager` 是正式系统工具，目标安装时获取，
  但不在 Live ISO 中。
- CHWD 在目标系统中根据硬件配置驱动，Calamares 只调用 CHWD，不复制其逻辑。
- 游戏环境是安装后由 Hello 触发的独立工作流，通过两个 metapackage 获取 Steam、
  Wine、Proton、Lutris、Heroic、MangoHud 等内容。
- CachyOS Hello 是编排入口；包浏览器、内核管理器、CHWD、更新工具都是独立产品。

因此，Linxira 不应把 Live ISO、目标系统、第三方软件和安装后工作流混成一个清单。

## 3. 统一术语

后续文档和代码必须停止混用 `profile`、`component`、`application` 和 `bundle`。

### 3.1 Base System / 基础系统

能够离线启动、安装、联网、登录、更新和恢复的最小 Linxira 系统。
它不是用户选择项。

### 3.2 Linxira System Tool / Linxira 系统工具

Linxira 自己拥有的系统产品，例如：

- Welcome
- Package Center
- Config CLI / Config Agent
- Hardware and Driver Manager
- Kernel Manager
- Update Agent
- Mirror/Source Manager
- Recovery/Diagnostic Tool

这些不是“应用分类里的普通软件”，也不是可以随意取消的安装合集。

### 3.3 Desktop Bundle / 桌面组件

一个完整、原子、可测试的桌面产品定义，包括：

- session 和显示协议
- display manager
- file manager
- terminal
- system settings
- network/audio/Bluetooth/power integration
- Polkit agent
- XDG portal
- lock screen
- Linxira settings 和主题

桌面安装页必须且只能选择一个。

### 3.4 Optional Capability / 可选能力组件

它表示系统能力，而不是软件分类或不可拆分的软件合集，例如：

- 打印与扫描
- 无障碍支持
- 虚拟化 guest/host 能力
- 容器运行能力
- 科学 Python 基础
- R 数据分析
- GIS
- LaTeX/科研写作
- 生物信息工具基础
- 游戏兼容基础

每个能力必须显示其必需项和可取消的推荐项。用户可以展开并细选；不能再用一个
不透明的 package array 表示整个“科学套装”。

### 3.5 Application / 应用或系统软件

用户可以单独选择和管理的软件，例如 Firefox、Thunderbird、Krita、GIMP、Steam、
Lutris、JupyterLab。每个应用是持久选择的最小单位。

### 3.6 Category / 分类

分类只是树形组织节点，例如网络、办公、开发、图形、媒体、系统工具。
分类不是安装对象。父复选框只是“选择/取消该分类下全部可用子项”的快捷操作。

### 3.7 Preset / 预设

预设是选择快捷方式，例如“推荐游戏环境”或“数据科学工作站”。应用预设后，用户
仍然可以取消其中任意非必需子项。预设绝不能成为不可拆分的安装单位。

### 3.8 Hardware Profile / 硬件配置方案

基于硬件 ID、内核和系统状态生成的驱动/固件计划。它由硬件工具管理，不属于应用树。

## 4. 软件交付分层

### 4.1 Class A：Live ISO 和离线基础

只放：

- 启动、安装、完整 Plasma Live 会话
- 网络和存储基础
- Calamares
- 恢复和诊断工具
- 广泛必要的开放驱动和固件
- 本地 catalog、图标、桌面截图和许可证
- 一个可离线安装的基础目标系统和默认 Plasma 桌面闭包

不因为“常用”就把开发运行时、游戏、第二包管理器和大型应用全部塞进 ISO。

### 4.2 Class B：安装期间从正式仓库获取

用于用户明确选择的桌面、能力和应用。必须使用一致的仓库状态，不能把旧离线基础与
新仓库做不受控的局部升级。

### 4.3 Class C：正式安装后从 Arch/Linxira 仓库获取

用于 Package Center、Kernel Manager、可选运行时、Wine、开发工具和大型应用。

### 4.4 Class D：外部生态 opt-in

包括 Flatpak remote、npm registry 包、PyPI、Conda、Nix cache、AUR、OCI、Proton
下载。必须显示来源、信任、许可证、更新责任和不可复现风险。

### 4.5 Class E：硬件和专有例外

例如 NVIDIA userspace、特定 Wi-Fi 驱动、CUDA。必须经过硬件匹配、许可证确认和
双内核兼容验证。

### 4.6 Class F：不分发

许可证/EULA/专利/出口/来源/更新链不明确的软件不进入 ISO 或 Linxira 仓库。

## 5. Live ISO、正式安装和首次联网完成

### 5.1 Live 模式

用户可以体验：

- Linxira 默认桌面和主题
- Welcome 的 Live 首页
- 安装器
- 只读系统状态、硬件和运行时检查
- 恢复工具
- 系统工具的功能说明和入口状态

Live 不需要包含全部正式系统工具。不存在的工具显示“安装后可用”，而不是假按钮。

### 5.2 在线安装

Calamares 根据固定 catalog 和当前安装计划获取：

- 基础系统
- 一个完整桌面
- Linxira 必需系统工具
- 硬件匹配的驱动/固件
- 用户选择的可选能力
- 用户选择的应用

所有内容在分区写入前完成依赖解析和可用性检查。

### 5.3 离线安装

必须能够安装基础系统、Plasma 和最低限度 Linxira 系统工具。在线专属项目禁用并说明。
不能接受选择后静默跳过。

### 5.4 首次联网完成

离线安装留下的、已经在安装器中明确同意的项目进入 `pending completion` 计划。
首次登录时由专门的 Completion Agent 显示：

- 需要下载的项目
- 来源和大小
- 许可证/EULA
- 仓库或 multilib 变化
- 是否可推迟

只有 Linxira 自有且被声明为关键的系统工具可以默认继续完成。Steam、专有驱动、
外部 registry、Conda、Nix 等必须再次显式确认，不能静默下载。

## 6. Calamares 安装器结构

### 6.1 Desktop / 桌面环境

- `required` 单选，默认 Plasma。
- 左侧桌面列表或紧凑卡片；右侧显示真实 Linxira 截图和说明。
- 说明包括交互模型、Wayland/X11、资源等级、适用用户、成熟度和已知限制。
- 截图是 Linxira 实际配置的本地资产，固定比例和版本，不从网络下载。
- 未通过安装、登录、网络、音频、portal、锁屏、更新和卸载测试的桌面不得出现。
- Phase 1 不应一次暴露八个未完成桌面。先接受 Plasma，再逐个接受 GNOME/Xfce 等。

### 6.2 Optional Capabilities / 可选组件

使用可展开树，不再显示七个不透明 profile：

```text
科学与专业计算
  [ ] Python 科学基础
      [ ] JupyterLab
      [ ] NumPy / SciPy
      [ ] Matplotlib
  [ ] R 数据分析
  [ ] GIS
  [ ] LaTeX 科研写作
系统能力
  [ ] 打印与扫描
  [ ] 无障碍扩展
  [ ] 容器工作站
```

- 每个用户可理解的能力或工具是叶子。
- 分类父节点支持 unchecked/partial/checked 三态。
- 必需依赖由 pacman/libalpm 解析，不冒充用户选择项。
- 能力的 required child 不可取消；recommend child 可以取消。
- 默认全部关闭，除非产品规范明确某项为基础功能。

### 6.3 Applications / 应用与系统软件

严格学习 CachyOS netinstall 的树形交互，但不照搬其混合 catalog：

```text
网络
  [ ] Firefox
  [ ] Chromium
  [ ] Thunderbird
办公
  [ ] LibreOffice
图形与创作
  [ ] Krita
  [ ] GIMP
开发
  [ ] VS Code
  [ ] Node.js LTS + npm
系统工具
  [ ] GParted
  [ ] Timeshift
```

- 箭头只负责展开；复选框负责选择；行或详情按钮显示说明。
- 父复选框只提供“全选该分类”，不会持久保存“安装了整个分类”。
- 子项变化实时更新父节点三态。
- 搜索不能改变父节点实际选择范围。
- 不可用子项显示原因，不参与父节点全选计算。
- 默认只由叶子定义。预设只是批量修改叶子后让用户继续调整。

### 6.4 Summary / 安装摘要

分栏显示：

- 桌面
- Linxira 必需系统工具
- 硬件配置方案
- 可选能力及其具体子项
- 应用列表
- 离线安装项
- 在线下载项和大小
- 首次登录待完成项
- 仓库、multilib、服务和许可证变化

### 6.5 执行与收据

安装器不再自己展开 package arrays。它调用统一 planning backend，保存：

- catalog digest
- 选择 ID
- 默认/用户/依赖来源
- 精确包和版本
- requestedBy provenance
- 安装模式和仓库状态
- 成功、失败、推迟项目
- initramfs、桌面登录和目标验证结果

## 7. Catalog v3

Catalog v3 至少包含：

```text
categories
applications
capabilities
presets
desktopBundles
systemTools
hardwarePolicies
sources
```

核心约束：

- stable ID 是跨安装器、Package Center、收据和 CLI handoff 的唯一身份。
- category 只引用 children，不包含 package array。
- preset 只引用 child IDs，不成为持久安装对象。
- profile 被废弃或迁移为 preset/capability。
- 每个叶子声明 provider、artifact、scope、availability、license、source、size、review、
  conflicts、requires、recommends 和 offline policy。
- 一个项目只有一个 primary category，其他关系作为 tags，避免重复复选框所有权。
- 任何 GUI/Calamares 配置都从 catalog 生成，不手写复制 ID、名称和说明。

## 8. Package Center 和安装后管理

Package Center 使用与安装器相同的树和稳定 ID，但展示实际状态：

- available
- installed and managed
- installed externally
- partially installed
- pending
- unavailable
- drifted
- reboot required

能力和预设的父状态从子项计算，不单独保存。

Package Center Phase 1 必须支持：

- 单个应用安装
- 多个应用安装
- 分类全选后继续细调
- 能力展开和部分安装
- 计划详情、依赖、冲突、大小、来源和许可证
- 安装进度、失败和重试
- 收据查看
- 外部 pacman/Flatpak 变化检测

移除必须晚于 ownership ledger：预先存在或被其他能力共享的包不能被误删。

## 9. 统一事务 backend

`linxira-components` 改为唯一的 Linxira 管理事务服务，但不直接接受任意命令、包名、
URL 或路径。

目标状态：

- system-activated D-Bus service
- caller 只提交 catalog ID 和动作
- backend 创建 root-owned plan
- Polkit 按风险分 action
- apply 前重新验证 catalog、pacman DB、仓库和机器状态
- pacman、Flatpak、用户运行时分别执行，承认它们不是一个原子事务
- durable immutable receipt 保存选择 ID、精确 artifact 和验证结果

当前 application-only plan 在 apply 时被错误要求必须有 profile，这是明确缺陷，必须先修。

## 10. Linxira 系统工具边界

### 10.1 Welcome

只做状态和路由。建议最终只有：

- Home
- Status
- Help

Live Home 主操作是安装。Installed Home 显示健康状态、首次完成状态和最多四个稳定入口。
移除当前重复的 Applications、Sources & Runtimes、System 页面和应用卡片。

### 10.2 Package Center

负责 curated applications、可选能力、游戏软件选择和 transaction history。

### 10.3 Config CLI / Config Hub

负责快速状态、doctor、运行时、源、SSH、网络、firewall 和受控服务配置。
不负责应用、桌面、内核和驱动包事务。

### 10.4 Hardware and Driver Manager

学习 CHWD 的数据驱动硬件 profile：

- PCI/DMI/CPU/VM 检测
- 开放/专有驱动建议
- 双内核 DKMS 验证
- initramfs 和 reboot 状态
- install/remove hooks 受 schema 和固定 operation ID 约束
- 计划、确认、回滚和收据

### 10.5 Kernel Manager

独立工具管理官方内核、fallback、默认启动项和模块兼容性。Phase 1 先限制为 Linxira
接受的 `linux` 和 `linux-lts`，不复制 CachyOS 的大量实验内核矩阵。

### 10.6 Update Agent

负责完整 Arch 更新、Arch News、pacnew、orphan/cache、服务重启和 reboot 状态。
不能让 Package Center 或 CLI 通过 `pacman -Sy` 形成 partial upgrade。

### 10.7 Recovery and Diagnostics

包括 Live chroot、keyring repair、pacman lock 诊断、initramfs 修复、Btrfs/Timeshift、
双内核检查和隐私清理后的支持包。

## 11. CLI 设计

公共命令统一为 `linxira`：

```text
linxira status
linxira doctor [AREA]
linxira runtime status|doctor
linxira source status|list|test|set|reset
linxira ssh status|key|server
linxira network status|doctor
linxira firewall status|configure
linxira service status|configure
linxira gaming doctor
linxira hardware report
linxira driver report
linxira plan show|verify|apply
linxira receipt list|show
```

规则：

- `status` 是快速只读摘要，目标 500 ms 内，不联网。
- `doctor` 才执行慢检查和网络探测。
- 所有只读命令支持 human/json/jsonl。
- mutation 默认只生成 dry-run plan，必须 `--apply` 才修改。
- CLI 不内部调用 sudo，不接受任意 shell、unit、sysctl、路径或包名。
- root helper 只接受固定 operation ID 和已签名/已绑定的 plan。
- 每个 adapter 实现 observe/plan/validate/apply/verify/rollback。

### 11.1 Runtime

运行时状态不是简单的“命令存在”：

- Python：版本、来源、系统包所有权、venv/pip 绑定、externally-managed 状态
- Conda：识别 Miniforge/Anaconda、base、channels 和 priority
- Node/npm：Node LTS/Current、npm 版本、registry、用户/系统来源
- Rust：system rustc 与 rustup、active toolchain、Cargo home
- Go：GOROOT/GOPATH/GOMODCACHE/GOPROXY
- Containers：rootless Podman、Docker socket/context、Apptainer
- Nix：安装模式、daemon、substituters、trusted keys，只读检查优先

运行时检查不激活环境、不修改 shell rc、不执行项目代码。

### 11.2 Source

source adapters：Arch、Linxira、AUR Git/RPC、Flatpak、npm、PyPI、Go、Conda、Cargo、
Nix。每个变更计划显示旧值、新值、信任、TLS、签名影响、scope、备份和回滚。

### 11.3 SSH 快速设置

- key generate 是用户操作，无 root。
- server configure 使用 Linxira-owned `sshd_config.d` drop-in。
- 默认 keys-only、禁止 root login。
- `sshd -t` 验证。
- 服务和 firewall 变化在同一计划中明确显示。
- 远程连接时不得先关闭旧端口造成锁死。

## 12. Node/npm 和第三方运行时政策

- Node.js/npm 本身有可接受的开源许可，可由 Arch 官方包提供。
- 它们不属于 Live ISO 和离线基础的必要内容。
- 可在安装器开发分类或安装后 Package Center 选择 Node LTS + npm。
- npm registry 配置属于 CLI source adapter。
- npm registry 中每个包有独立许可证、生命周期脚本和可变版本，属于外部生态 opt-in。
- 禁止 `sudo npm -g` 作为 Linxira 工作流；推荐用户 scope、lockfile 和 integrity。
- Miniforge、Nix、Flatpak remote、AUR 和 Proton 同样按外部信任根处理，不静默 bootstrap。

## 13. 游戏、Wine、Proton 和驱动

### 13.1 Gaming Setup

它是一个可展开、可调整的快速工作流，不是不可拆分合集：

```text
游戏平台
  [ ] Steam
  [ ] Heroic
  [ ] Lutris
兼容层
  [ ] Wine
  [ ] Winetricks
  [ ] Proton tools
性能与诊断
  [ ] MangoHud
  [ ] Gamescope
  [ ] GameMode
```

“推荐游戏环境”只是 preset。应用后用户可以取消任何非必需项目。

计划必须显示：

- 是否启用 multilib
- 32-bit Vulkan/驱动依赖
- native/Flatpak provider
- Steam EULA 或第三方许可
- 下载大小
- 服务、group、sysctl 变化
- Wine prefix 和游戏数据不会在卸载时删除

### 13.2 Wine/Proton

系统包由 Package Center 管理；Wine prefix、runner、DXVK、游戏数据属于用户，不由 root
backend 修改。Proton 优先由 Steam 管理。

### 13.3 Drivers

驱动不是应用树或游戏 preset 的隐藏副作用。Gaming doctor 可以指出缺失驱动并跳转到
Driver Manager，但驱动事务由 Driver Manager 独立执行。

## 14. 当前实现的主要问题

### Release blocker

1. RC17 写盘安装失败：Calamares 3.3.14 生成 `crc32c-intel`，清理代码只删除
   `crc32c_intel`，两个内核的 mkinitcpio 都返回 2。
2. `consolefont` hook 没有 FONT，虽然非致命，但产生错误噪音。
3. RC17 不能再称为可接受候选，只是 Live 诊断件。

### Installer

4. Desktop、Components、Applications 都是平铺列表，不是要求的树形模型。
5. Desktop chooser 允许多选，但 backend 又要求恰好一个。
6. 所有桌面使用同一错误/缺失截图，说明和真实界面没有建立。
7. 未接受的候选桌面被暴露；基础 target 固定带 Plasma，造成跨桌面污染。
8. 当前 profile 是 opaque package arrays，不是可调整的科学能力。
9. 分类只被拼进应用名称，不存在父子三态。
10. 手写三个 chooser 重复 catalog，必然漂移。
11. optional job 使用刷新仓库后局部安装，违反 Arch partial-upgrade 约束。
12. 桌面失败也可能被记录为 deferred 后继续成功，严重错误。

### Package Center/backend

13. Package Center 只是 kdialog install checklist，没有 installed/remove/progress/history 状态。
14. application-only apply 会因 backend 强制要求 profileIds 而失败。
15. durable receipt 不保存最终选择 ID 和精确 artifact，无法追踪安装内容。
16. installer receipt 与 components receipt 是两套不兼容模型。

### CLI/runtime/source

17. 当前 1353 行 Bash CLI 混合 doctor、源、SSH、网络、firewall、service、group、包安装。
18. 存在直接写 resolv.conf、mask service、打开 firewall、加入 docker group 等危险路径。
19. runtime 只是 presence check，没有版本、来源、健康和信任状态。
20. source 输出包含静态声明，Cargo/OCI 等没有完整 adapter。
21. Help 仍展示已拒绝的安装命令，内部遗留 direct pacman 代码。

### Welcome

22. Welcome 复制 Package Center 应用列表和 Config Hub 子命令，职责重复。
23. 当前六页是功能堆砌，不是状态与路由产品。
24. Live/installed/first-run 生命周期不完整。

### Artwork

25. 已确认的字符画是 `Block + Geometric` Chafa 版本，但它未被 Git 跟踪或打包。
26. RC17 使用手写 `LL|--` 文本，因此终端显示与已确认 HTML 不一致。

### Repository engineering

27. workspace root 不是 Git，活动仓库、参考、构建、release、私密材料混在一起。
28. ISO 从 sibling working tree 直接复制，没有记录 commit 和输入 hash。
29. catalog/schema/chooser/docs 有多份可编辑副本。
30. root workplan 是 append-only 叙事，缺少机器可验证的 current state 和 supersededBy。

## 15. mkinitcpio 安装失败修复要求

必须同时做：

1. 修 Calamares package 中的 producer：不再加入 `crc32c-intel`。
2. branding 防御性删除 `crc32c-intel` 和 `crc32c_intel`。
3. 测试覆盖两种拼写、Intel+Btrfs、两个内核和真实 packaged Calamares。
4. target validator 要求两个 initramfs 文件存在并重新运行 `mkinitcpio -P`。
5. 扫描所有 mkinitcpio 配置，拒绝 obsolete module。
6. 没有选定 FONT 时移除 consolefont hook。
7. 保留 mkinitcpio 官方 ALPM hook 创建 preset 和初始镜像；最终配置写入后再执行一次受控
   `mkinitcpio -P`。不得屏蔽整个 hook，因为 mkinitcpio 41 的 preset 生命周期也由它管理。

## 16. Phase 1 实施顺序

### P0：停止错误扩展

- 冻结 RC17 为 rejected diagnostic。
- 不再增加 Welcome 页面或 profile。
- 建立产品词汇、workspace manifest、contract matrix 和 current task 记录。

### P1：修复可安装基础

- 修复 mkinitcpio blocker。
- 验证 Plasma-only 离线安装、双内核、Btrfs、首次启动和恢复。
- 建立真实 release manifest 和包所有权。

### P2：Catalog v3 和选择模型

- categories/applications/capabilities/presets/desktops/systemTools/hardwarePolicies。
- 生成 Calamares 和 Package Center 数据。
- 树形三态、搜索、availability 和 summary。

### P3：统一事务与收据

- D-Bus/Polkit backend。
- application-only、capability partial selection、plan/apply/verify/receipt。
- 安装器和安装后共用模型。

### P4：正式系统工具

- Welcome 精简。
- Package Center 正式 UI。
- `linxira` status/doctor/runtime/source/ssh。
- Update Agent、Hardware/Driver Manager、Kernel Manager、Recovery。

### P5：桌面矩阵

- Plasma 完整接受。
- 每个额外桌面单独完成 bundle、截图、VM 和 first-login acceptance 后再开放。

### P6：游戏和第三方生态

- Gaming doctor、Gaming Setup、multilib、Wine/Proton、驱动 handoff。
- 每个外部来源建立许可、信任和收据策略。

## 17. Phase 1 成熟度验收

至少要求：

- Live/online/offline/first-boot 四条路径有明确测试。
- 安装器三类选择边界正确，分类树支持 partial state。
- 单个应用和部分科学能力可以安装，不依赖 preset/profile。
- 桌面严格单选并有真实截图和说明。
- 任何关键 package、desktop、initramfs 失败都不能显示安装成功。
- Package Center application-only plan/apply/receipt 通过真实测试。
- `linxira status` 快速只读；所有 mutation 默认 dry-run。
- SSH/source/runtime 有 JSON contract 和幂等测试。
- 驱动覆盖 Intel/AMD/NVIDIA/hybrid、双内核、DKMS failure 和 reboot。
- 游戏工作流不隐藏 multilib、专有协议和第三方下载。
- Welcome 不执行包事务、sudo、pkexec 或 shell string。
- release artifact 有 source commits、package cohort、hash、SBOM 和 test evidence。

## 18. 已确认的产品决策

以下决策由用户于 2026-07-20 确认：

1. Phase 1 正式支持 Plasma 和 GNOME；GNOME 必须完成独立 bundle、真实截图、安装和
   首次登录验收后才可在发布版本中开放。
2. 系统能力与科学能力拆成两个安装页；两页都使用可展开树、叶子独立选择和父节点
   三态选择。
3. Firefox 默认预选，其他普通应用默认不选。
4. 离线 ISO 只保证 Plasma 的完整安装闭包；GNOME 安装需要联网。
5. 离线安装后，首次联网必须先显示计划并获得一次确认，再补齐 Linxira 核心系统工具。
6. Package Center Phase 1 暂不开放 remove；完成 ownership ledger、依赖归属和漂移检测
   后再开放。
7. Gaming Setup 是 Package Center 中的专用 workflow，不开发独立 GUI。
8. Hardware/Driver Manager 先审计并封装 CHWD，由 Linxira 提供 UI、策略、计划、确认、
   验证和收据边界，不从头重写硬件 profile 引擎。

这些决策解除 P1 实现冻结。P2 及以后仍必须按第 16 节顺序推进，不能跳过可安装基础。
