# 双系统安装（UEFI）设计与缺口清单

> 本文回答“整块独立硬盘装第二个系统，分区表和引导项到底写在哪”，并列出当前安装器
> 已经做到什么、还缺什么。**不要据此宣称双系统已就绪**，完成第 5 节验收才算。

## 1. 两条不同的路径

| 场景 | 磁盘 | ESP | 风险 |
|---|---|---|---|
| A. 同盘共存 | 一块盘，缩 Windows NTFS | 共用 Windows 的 ESP（通常 100 MiB） | Windows 功能更新可能重写 ESP/引导顺序 |
| B. 独立硬盘 | 两块盘，各自的 GPT | **各自独立的 ESP** | 需在固件启动菜单里选盘，或由安装器调整 BootOrder |

用户当前描述的是 **场景 B**：整块物理盘交给 Linxira，Windows 在另一块盘上。

## 2. UEFI 的事实（决定“写在哪”）

- **GPT 是每块盘各自的**：主表在 LBA1，备份表在盘尾。在盘 2 建 GPT 不会改盘 1 的任何字节。
- **启动项存在主板 NVRAM**，不在分区表里。每条记录形如
  `Boot0003* linxira  HD(1,GPT,<ESP 分区 GUID>,0x800,0x100000)/\EFI\linxira\grubx64.efi`：
  指向“哪块盘、哪个 ESP 分区、哪个路径”。
- **ESP 是 FAT32 分区**。场景 B 下盘 2 会新建一个 1 GiB ESP，GRUB 及其配置写在里面；
  `installEFIFallback: true` 还会写 `\EFI\BOOT\BOOTX64.EFI`，作为 NVRAM 丢失时的兜底。
- Windows 的启动项 `Windows Boot Manager` 指向盘 1 的 ESP，**不受我们影响**。

## 3. 当前安装器的实际行为（代码依据）

- `airootfs/etc/calamares/modules/partition.conf`
  - `efi.mountPoint=/boot/efi`、`recommendedSize=1024MiB`、`minimumSize=512MiB`
  - `defaultPartitionTableType: gpt`、`requiredPartitionTableType: gpt`
  - `allowManualPartitioning: false`
- `airootfs/etc/calamares/modules/bootloader.conf`
  - `efiBootloaderId: linxira` → NVRAM 条目名为 `linxira`
  - `installEFIFallback: true` → 写 ESP 的 `\EFI\BOOT\BOOTX64.EFI`
  - 未调用 `efibootmgr -o`，**不调整 BootOrder**
- 目标系统已加 `os-prober`（构建期门禁强制），`GRUB_DISABLE_OS_PROBER=false`
  → `grub-mkconfig` 会生成 Windows 条目。

结论：场景 B 下“选盘 → 全盘安装 → 写盘 2 的 GPT/ESP → 注册 NVRAM 条目”这条链路
是通的；`os-prober` 负责让 GRUB 列出 Windows。

## 4. 已知缺口（必须处理，不能算开发完）

1. **启动顺序未设置**：装完默认仍可能进 Windows（固件把 Windows Boot Manager 排第一）。
   用户需要按 F12 选 `linxira`。要不要由安装器把 `linxira` 调到第一位，是产品决策。
2. **手动分区被禁用**：无法在盘 2 上保留数据分区；只能整盘擦除。场景 B 可接受，
   场景 A（同盘缩容）更依赖自动“共存”。
3. **无自动化验收**：目前没有任何测试证明“盘 1 未被改动”“NVRAM 两条记录并存”
   “GRUB 能 chainload Windows”。
4. **前提条件未校验**：Secure Boot、Windows 快速启动、BitLocker、Intel RST 都靠文档提醒；
   安装器不做检测（至少应检测 Secure Boot 并提示）。

## 5. 验收矩阵（QEMU 可完成）

两盘 UEFI 虚拟机：盘 1 = 假 Windows（ESP 内放 `EFI/Microsoft/Boot/bootmgfw.efi` 占位 +
NTFS 分区放 `Windows/System32` 目录），盘 2 = 目标盘。

| # | 断言 |
|---|---|
| 1 | 安装到盘 2 后，盘 1 的 GPT/ESP 内容哈希不变 |
| 2 | 盘 2 存在 GPT + 1 GiB ESP（FAT32）+ btrfs 根 |
| 3 | NVRAM 同时存在 `linxira` 与 `Windows Boot Manager` 两条记录 |
| 4 | `linxira` 的 `grub.cfg` 含 Windows 条目（os-prober 生效） |
| 5 | 首启默认进入 Linxira；从菜单可回到 Windows 占位 |
| 6 | 拔掉盘 2 后，盘 1 仍能独立启动 Windows 占位 |

## 6. 待决策

- 安装后是否把 `linxira` 设为 BootOrder 第一？（推荐：是，并在文档给出改回命令）
- 是否开放手动分区（给高级用户保留盘 2 的数据分区）？
- 是否在安装器里检测 Secure Boot 并给出明确阻断/提示？

## 7. “在 Windows 里直接装 Linux”的来龙去脉

用户记忆里的方案是 **Wubi（Windows-based Ubuntu Installer）**：

- 在 NTFS 分区里创建一个磁盘镜像文件（如 `root.disk`），把整个 Linux 装进这个文件；
- 在 Windows 引导器里加一条入口，重启后由 GRUB4DOS/GRUB2 的 **loopback** 挂载该镜像启动；
- 不是虚拟机，性能接近原生但受 NTFS 与镜像文件限制；
- **Ubuntu 13.04 起从下载页移除，14.10 后停止提供**。

放弃的原因（也是我们必须知道的坑）：

| 限制 | 说明 |
|---|---|
| 硬断电易损坏 | 镜像文件在 NTFS 上，异常关机/断电可能丢文件系统 |
| 不支持休眠 | 与 Windows 快速启动/休眠冲突 |
| 无 TRIM | SSD 上长期使用性能下降 |
| 依赖父文件系统 | Windows 崩溃或重装 → Linux 一起没了 |
| 引导耦合 Windows | Windows 大版本更新可能覆盖引导 |
| Secure Boot | 需要自定义签名的 loopback 引导链，主流发行版不提供 |

现状：社区分支 **WubiUEFI** 仍在维护（1165 stars / 233 issues），但要求发行版主动适配
（loop 文件启动、自动安装、preseed hook、签名引导器），没有任何主流发行版默认支持。
Ubuntu 现在的官方安装器**不再提供**从 Windows 内安装。

### 7.1 主流替代方案

| 方案 | 机制 | 适用 |
|---|---|---|
| Live USB + “与 Windows 共存” | 安装时用 `ntfsresize` 缩小 NTFS，新建分区 | 当前的推荐路径 |
| WSL2 | 不双系统，Windows 里跑 Linux 用户态 | 只想用工具链（我们已发镜像） |
| 虚拟机（Hyper-V/VirtualBox） | 完全隔离 | 试玩/隔离环境 |
| 装进 VHDX 再原生引导 | 与 Wubi 同类（loop 镜像） | 不建议作为产品路径 |

### 7.2 对我们的结论

- **不实现 Wubi 式“Windows 内安装”**：收益是省一次 U 盘，代价是一整套镜像文件
  引导链、签名与数据安全风险，且上游已证明不可持续。
- 保留两条路径：**Live USB 安装（单盘/双盘）** 与 **WSL 镜像**（已发布）。
- 若将来要做“Windows 内一键体验”，优先做 **WSL 发行版**，而不是 loop 双系统。

## 9. ESP 容量：必须算，不能赌

Windows 默认 ESP 只有 **100 MiB**（部分 OEM 128/260 MiB）。同一块 ESP 上还要放
Windows 自己的引导文件，剩余空间通常 50–60 MiB。

| 内容 | 占用 |
|---|---|
| Windows（bootmgfw.efi、字体、BCD 等） | 约 30–50 MiB |
| GRUB（引导器 + 配置 + 字体，内核在 /boot 不在 ESP） | 约 10–15 MiB |
| systemd-boot / rEFInd（内核 + initramfs 放进 ESP） | 每套约 120–180 MiB，双内核 250–350 MiB |

因此：

- **当前安装器只提供 GRUB**；systemd-boot / rEFInd 的选择已暂时关闭，下面的容量规则保留给未来启用时。
- **默认 GRUB** 可以装进 Windows 的 100 MiB ESP（前提是剩余 ≥ 32 MiB）。
- **systemd-boot / rEFInd** 在 100 MiB ESP 上必然塞不下，必须专用大 ESP（≥900 MiB）。

未来重新开放时，安装器仍有硬门禁（`linxirapacstrap._efi_capacity_error`）：

| 条件 | 结果 |
|---|---|
| ESP 总容量 < 64 MiB | 拒绝安装 |
| 选 GRUB 且剩余 < 32 MiB | 拒绝安装，提示清理或扩容 |
| 选 systemd-boot/rEFInd 且总容量 < 900 MiB 或剩余 < 512 MiB | 拒绝安装，建议改选 GRUB |
| BIOS/legacy（无 ESP） | 跳过检查 |

拒绝发生在写盘之前，不会出现“装到一半才失败”。我们**不缩放、不格式化**已存在的 ESP。

## 10. 选盘与分区

### 10.1 能选具体硬盘吗

可以。Calamares 的「擦除磁盘」页会列出所有块设备（如 `/dev/nvme0n1`、`/dev/sda`），
按容量与型号选择。默认预选改为「与 Windows 共存」，避免把第一项「擦除磁盘」当默认。

### 10.2 怎么区分 USB 和真实 SSD

Calamares 列表本身对 USB/内置盘区分不强，所以安装器在写盘前做硬校验：

| 检查 | 依据 | 结果 |
|---|---|---|
| 目标盘是否可移动 | `/sys/block/<disk>/removable == 1` | 拒绝，提示型号 |
| 目标盘是否就是 live 介质 | `findmnt /run/archiso/bootmnt` 的父盘与目标父盘相同 | 拒绝 |
| 无法判定 | `findmnt` 不可用 | 放行（避免误伤） |

错误信息会带上磁盘型号（`/sys/block/<disk>/device/model`），例如
“`/dev/sda2` 挂在 USB DISK 3.0 上，系统不能安装到 U 盘/移动硬盘”。

### 10.3 Linux 的分区策略

Linux 不需要像 Windows 那样为“系统盘/数据盘”反复分区：所有路径最终挂到同一棵树。
我们的默认就是**一块 btrfs 根分区 + 一个 ESP**，不单独分 `/home`，也不建 swap 分区
（`userSwapChoices: [ none ]`）。这样：

- 空间按需增长，不会出现“/home 满了但 / 空着”；
- 快照（Timeshift）对整棵树生效，不需要跨分区协调。

手动分区已开放（`allowManualPartitioning: true`）给高级用户，但不推荐拆得过细；
即使手动分区，ESP 容量与可移动盘校验仍然生效。
