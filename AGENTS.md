# Linxira OS 工作区 · Agent 开发规范

> 本文件是工作区级 Agent 开发规范的**云端副本**,与工作区根目录
> `F:\Linxira-OS\AGENTS.md` 保持一致;工作区根目录那份是 agent 就地读取用的。
> `f:\Linxira-OS` 本身**不是 git 仓库**,是多仓库工作区根:下面并列着 40+ 个各自
> 独立的 git 子仓库。本文件约束 agent 在这片工作区里的通用行为。
> 每个子仓库若自带 `AGENTS.md`,其条款**优先于**本文件的通用条款;冲突时以子仓为准。

## 0. 先读什么(权威顺序)

1. 目标仓库自己的 `AGENTS.md`
2. 本文件
3. `HANDOVER.md` —— 工作交接与挂账清单(读「工作区拓扑」「挂账清单」)
4. `linxira-os/governance/repositories.yaml` —— 仓库归属与 lifecycle 的**唯一权威**
5. `linxira-os/README.md` + `linxira-os/docs/ROADMAP.md` —— 我们在开发什么、为什么、发布规范化
6. `linxira-os/docs/RELEASE_STANDARD.md` —— **发布/测试规范**(测试版 vs 正式版、验收门、发布禁令)

不要凭记忆回答"某功能归哪个仓库",一律查 `repositories.yaml`。
各子仓库的 `AGENTS.md` 按档位编写(S 治理发布核心 / A 系统源仓 / B 内容仓 / C 外部上游),
均引用本总纲与发布规范,不重复。

## 1. 工作区拓扑

| 位置 | 说明 |
|---|---|
| `F:\Linxira-OS\<repo>` | 子仓库本地镜像,**写入口**。F: 盘历史不稳定 —— 提交后尽快 `push`,勿留唯一副本 |
| `F:\Linxira-OS\iso-release\` | ISO 发布目录(GitCode LFS 仓),ISO 落地点 |
| `F:\Linxira-OS\.keys\` | 签名密钥金库,**禁止提交、禁止外传** |
| `F:\Linxira-OS\.opencode\ .codegraph\ .mimosa\ .workbuddy\ .tmp*\` | 工具/缓存态,**非源码,勿入仓** |
| `F:\Linxira-OS\_archive\ _backup\ _private\` | 归档/备份/私有区 |

**关于 `.agent` 路径**:工作区**没有** `.agent/` 这个路径。agent 配置的约定是
**各仓库根目录的 `AGENTS.md`**(大写)。另有 `linxira-skills` 生成的运行时态
`.agents/skills/` 与 `.linxira/`,属生成物、已被 git 忽略,不要手工填充。

## 2. 仓库分类(改动前先认清属于哪类)

- **治理仓**:`linxira-os`(架构/归属/发布档案)、`linxira-wiki`(用户文档)
- **系统源仓(有 `VERSION` + `release.yml`,进全自动发布链)**:catalog、components、
  component-manager、config-hub、welcome、update、package-center、recovery-diagnostics、
  hardware-driver-manager、kernel-manager、hwd-detector、gaming-manager、completion-agent、
  hooks、keyring 等
- **构建/发布仓**:`linxira-iso-direct`(ISO 构建)、`packages`(PKGBUILD + CI)、
  `linxira-packages`(签名仓托管)、`iso-release`(镜像)
- **配置/内容仓**:gnome-settings、kde-settings、fish-config、zsh-config、
  hypr/niri-noctalia、mangowc-dms、wallpapers、plymouth-theme、artwork、settings ——
  **多数未接入打包链**,不要默认它们会随 ISO 发布
- **外部上游**:`linxira-calamares`、shelly(不随链)
- **独立**:`linxira-zeta`(稳定前不进链)、`linxira-skills`、`linxira-bio-sdk` 等

## 3. 分支约定

改前先确认:`git rev-parse --abbrev-ref HEAD` 与 `git rev-parse --abbrev-ref '@{u}'`。

- 多数仓库默认 `main`
- `master`:`linxira-os`、`linxira-artwork`、`linxira-hooks`、`linxira-hypr-noctalia`、
  `linxira-niri-noctalia`、`linxira-mangowc-dms`、`linxira-plymouth-theme`、
  `linxira-rate-mirrors`、`linxira-settings`、`linxira-wiki`、`linxira-zsh-config`、
  `Linxira-OS.github.io`
- `develop`:`linxira-gnome-settings`、`linxira-kde-settings`、`linxira-wallpapers`

推错分支名会直接失败(`src refspec ... does not match any`),别盲目重试,先查分支。

## 4. 文档纪律

- **以代码为准**。发现文档与代码不一致:修文档(顺手),不要改代码去迎合文档。
- **不要手改生成物**:`linxira-wiki/ai/manifest.json` 由
  `linxira-wiki/scripts/generate-manifest.py` 生成(数据源 `repositories.yaml`,从
  GitHub raw 拉取)。正确流程:**改 yaml → push linxira-os → 重跑生成脚本**。
  直接手改 manifest 会挂 wiki CI 的 `test_generated_manifest_is_current`。
- **版本号唯一来源**是各仓根目录的 `VERSION` 文件,别在 README 里另写一份。
- **发布状态口径**:迄今所有镜像(含 r17 / 2026-09-30)均为**测试版**,未对外宣称
  正式版。正式发布需 release manifest 证据(见 `linxira-os/releases/README.md`);
  从下一次构建起随构建产出,不回溯补建。

## 5. 发布链(全自动,勿手工介入)

源仓改 `VERSION` → `release.yml` 建 v-tag Release → `packages/auto-bump.yml` 扫描建 PR
→ 自动 squash 合并 → `packages.yml` 构建+签名 → 同步到 `linxira-packages`。
排除:calamares / shelly(上游)、zeta(独立)。

`packages.yml` 的 `pull_request` 事件**跳过 publish 是安全设计**(PR 代码不自动发布),
不是故障;真正发布发生在 squash 合并后的 push 事件。

## 6. 硬性约束(违反即 CI 挂或出事故)

- **禁止**在 PKGBUILD 引入 `cachyos` / `aur` / `seafoam` 仓库引用(check-boundaries.sh 会拒)
- **LF 行尾**:`PKGBUILD` 与 `.sh` 必须 LF(makepkg 解析需要)
- 改 PKGBUILD 的 `_commit` 或 `sha256sums` 后,**必须同步** `scripts/check-boundaries.sh` 里的哈希
- `_commit` 与 `sha256` 必须**同时**更新(只改一个必挂)
- **不要 `git add -A`**:只加具体文件名,防止带入 `.pkg.tar.zst` / `src/` / `pkg/` 等产物
- **提权只走 `linxira-components` + polkit**:不直接 `sudo`/`su`/`doas`/`run0`;只读命令免提权
- **绝不提交** `.keys/`、密钥、凭据、`.env`
- 破坏性操作(`reset --hard`、`push --force`、批量删除)先确认,不擅自执行

## 7. 各仓库 `AGENTS.md` 现状

**已携带(已推送云端)**:

| 仓库 | 说明 |
|---|---|
| `linxira-bio-sdk` | 技能路由 + 仓库规则 + 执行安全 + 校验命令 |
| `linxira-skills` | 技能布局 + 阅读纪律 + 运行时边界 |
| `Linxira-OS.github.io` | 官网仓 agent 约定 |
| `linxira-zeta` | 根 + `editor/`、`python/robomp/`、`web-ui/` 子级 |

另有 `packages/docs/AGENT-GUIDELINES.md`(packages 仓的中文操作规范)。

**其余子仓库暂缺 `AGENTS.md`** —— 新建仓库或首次接管某仓时,应补一份根 `AGENTS.md`,
至少写清:仓库职责、目录布局、构建/校验命令、禁区与提权方式。

## 8. 新建子仓库检查清单

1. 在 `linxira-os/governance/repositories.yaml` 登记 `name` / `lifecycle` / `owns`
2. 补根 `AGENTS.md`
3. 系统源仓补 `VERSION` + `.github/workflows/release.yml`
4. 确认默认分支,并设好 `@{u}` 上游
5. 若改动归属数据,记得重跑 wiki manifest 生成脚本