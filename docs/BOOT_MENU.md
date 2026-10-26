# 引导界面与多系统：现状、方案、决策

> 目标：把“用哪个引导界面、能不能配置、装双 Linux 会怎样”讲清楚。
> 结论先行：**默认 GRUB（可换 rEFInd / systemd-boot），不主动改固件启动顺序**。

## 1. 现在一共有四个“引导/启动界面”

| # | 界面 | 位置 | 现状 | 可配置项 |
|---|---|---|---|---|
| 1 | ISO GRUB（UEFI） | 镜像 `boot/grub/grub.cfg` | 深色背景 + 品牌 splash，菜单文字平铺 | 背景图、配色、超时、默认项 |
| 2 | ISO syslinux（BIOS） | 镜像 `boot/syslinux/` | 640×480 splash（已换新标） | 背景图、菜单配色 |
| 3 | 安装后 GRUB | 目标 `/etc/default/grub` + `/boot/grub` | r6 起带 splash + gfxterm 配色；之前是纯黑白 | `GRUB_THEME`/`GRUB_BACKGROUND`、配色、超时、默认项 |
| 4 | 安装器可选引导器 | Calamares 引导器页 | GRUB（默认）/ systemd-boot / rEFInd | 用户安装时选择 |

另外 Plymouth 只负责内核启动画面，不参与多系统选择。

## 2. 为什么看起来“像千禧年”

- GRUB 默认主题就是黑底白字、无圆角、无图标；CachyOS 那种“毛玻璃”不是新版 GRUB 的新特性，
  而是**自带一套主题**（背景图 + 半透明选中框 pixmap + 图标 + 自定义字体）。
- 我们此前连背景都没设，所以安装后是纯文本菜单；r6 已补背景与配色，
  但要接近 CachyOS 还需要 `theme.txt` + 选中框 pixmap + 条目图标。

## 3. 建议的界面策略

1. **默认继续用 GRUB**：兼容 UEFI/BIOS、能 os-prober、能 chainload Windows、ESP 占用小。
2. 给 GRUB 做一套自有主题（可配置）：
   - `GRUB_THEME=/boot/grub/themes/linxira/theme.txt`
   - 背景：现有品牌 splash（可换成 1920×1080 版本）
   - 选中框：半透明青绿圆角 pixmap（“毛玻璃”观感来自背景 + 半透明块）
   - 条目图标：`gnu-linux`、`windows`、`efi`、`reboot`、`shutdown`
   - 字体：沿用 `unicode.pf2`，避免中文缺字
3. ISO 与安装后使用**同一套主题**，避免“镜像好看、装完变丑”。
4. systemd-boot / rEFInd 保持可选，不默认；它们的美观与多系统能力各自独立。

## 4. 多系统：把所有情况列全

### 4.1 Windows + Linxira（两块盘，各自 ESP）
- 盘 2 写自己的 GPT/ESP/NVRAM 条目，盘 1 不变。
- GRUB（盘 2）用 os-prober 找 Windows，生成 chainload 条目。
- 固件默认仍可能进 Windows：**按用户要求，我们不修改 BootOrder**，由用户在固件里设默认。
- 风险：Windows 大版本更新可能重置 BootOrder（回到 Windows），但不影响盘 2 的引导项。

### 4.2 Windows + Linxira（同一块盘，共用 ESP）
- 需要缩 NTFS；共用 Windows 的 ESP（常见 100 MiB）。
- 我们的 `partition.conf` 写的是 `minimumSize: 512MiB`，**共用小 ESP 时会冲突/告警**。
- 风险：Windows 更新可能覆盖 `\EFI\BOOT`；BitLocker 必须先暂停保护。

### 4.3 两块盘，各装一个 Linux（含两个 Linxira）
- 各自 GPT/ESP；NVRAM 里会有两条记录。
- **坑：`efiBootloaderId` 固定为 `linxira`**，第二次安装会覆盖同名 NVRAM 条目
  （盘 2 的引导文件还在，但固件菜单里只剩一条）。需要在安装器里区分 ID
  （例如带磁盘序号），否则“双 Linxira”只能靠改 ID 或固件手动加条目。

### 4.4 同一块盘，Windows + 已有 Linux + Linxira（三系统）
- 共用 ESP 时，三个引导器都会往同一个 ESP 写目录（`EFI/<distro>/`），条目靠 os-prober 互相发现。
- 谁最后运行 `grub-mkconfig`，谁的菜单就是“主菜单”；另外两家的更新可能反客为主。
- 约定：**后装者负责主菜单**，且必须能列出前两者；否则用户会“丢系统”。

### 4.5 先装 Linxira，后装 Windows
- Windows 安装器通常把自己的引导项排到第一，并把 `\EFI\BOOT` 写回自己。
- Linxira 的 ESP 目录与 NVRAM 条目仍在；用户可在固件里切回，或在 Windows 里用
  `bcdedit`/固件菜单调整。

### 4.6 移除其中一块盘
- 独立 ESP 场景下，拔掉盘 2 后盘 1 仍可独立启动（需固件里还有 Windows 条目）。
- 共用 ESP 场景下，删掉 Linux 分区后要清理 `\EFI\linxira` 与 NVRAM 条目，否则固件菜单有死项。

## 5. 功能缺口（按优先级）

| 优先级 | 缺口 | 处理 |
|---|---|---|
| P0 | 同一磁盘/多 Linux 时 NVRAM ID 冲突 | 安装器按磁盘生成唯一 `efiBootloaderId`（或检测冲突并提示） |
| P0 | 共用 100 MiB ESP 与 `minimumSize: 512MiB` 冲突 | 兼容小 ESP：仅要求 ≥100 MiB，GRUB 实际占用 <10 MiB |
| P1 | 不做 BootOrder 调整 | 按决策保持现状；文档说明如何用固件/F12 选默认 |
| P1 | 无多系统验收 | QEMU 双盘矩阵（假 Windows ESP + NTFS）逐项断言 |
| P2 | GRUB 主题缺失 | 自有 theme.txt + 选中框 + 图标，ISO 与安装后共用 |
| P2 | Secure Boot 无提示 | 安装前检测并提示关闭 |

## 6. 已决定

- 引导器默认 GRUB；不主动改固件 BootOrder，默认跟随用户固件设置（Windows 优先就 Windows）。
- 目标系统安装 `os-prober`（r6 起），GRUB 能列出 Windows/其他 Linux。
- 引导视觉：ISO 与安装后统一，用自有主题，不用第三方主题包。

## 7. 三种引导器对比（只看自家可选项）

| | GRUB（默认） | systemd-boot | rEFInd |
|---|---|---|---|
| Windows 共存 | os-prober 自动生成条目 | 新版有 `auto-windows`；内核须在 ESP | 自动扫描所有 OS |
| 美观 | 可主题化（已做自有主题） | 纯文本，不可主题化 | 图标化，最好看 |
| 功能 | 双内核、快照子菜单、救援、chainload | 极简、启动快 | 图形化、多系统友好 |
| ESP 需求 | 几 MB | ≥1 GiB | ≥1 GiB |
| 固件风险 | 一条 `linxira` + 可选 fallback | 同左 | 同左，且 Secure Boot 需签名 |

## 8. 与固件 / NVRAM 的关系（笔记本重点）

- UEFI 启动项存在 **NVRAM 变量**里，不在磁盘；固件不会“缓存”启动项，但常见坏行为是：
  新增变量要冷启动才生效、反复把 Windows 排回第一、重装后留下同名重复项。
- 我们的策略：
  1. **不修改 BootOrder / BootNext**，默认跟随用户固件设置；
  2. `efiBootloaderId: linxira` 固定，`grub-install` 替换同名条目，不主动堆叠；
  3. **`installEFIFallback: false`** —— 不写 `\EFI\BOOT\BOOTX64.EFI`。共用 Windows ESP 时
     该路径是固件的通用回退，覆盖它等于和固件抢方向盘；独立盘场景 NVRAM 条目已足够。
- 若固件丢失 NVRAM：进固件菜单手动选盘/选 `\EFI\linxira\grubx64.efi`，或重跑
  `grub-install`（会重建条目）。
