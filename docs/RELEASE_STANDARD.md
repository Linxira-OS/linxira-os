# Linxira OS · 发布与测试规范 (Release Standard)

> 本文件是**发布/测试的唯一权威规范**。工作区总纲见工作区根 `AGENTS.md`。
> release manifest 的字段规格见 [`releases/README.md`](../releases/README.md) 与
> [`releases/manifest.schema.json`](../releases/manifest.schema.json);
> 验收项清单见 [`docs/ROADMAP.md`](ROADMAP.md)。

## 1. 两种产物口径(勿混)

- **测试版 (test build)**:迄今所有构建的镜像(含 r17 / 2026-09-30)都属此类。
  可对外下载,但**不称正式版**、不宣称"已发布 (released)"。
- **正式版 (release)**:须通过第 3 节全部验收门,并有 release manifest 记录在案。
  **目前尚无正式版。**

文案与 README 必须用 "test build / 测试镜像 / 可下载测试镜像" 的措辞,
不得用 "正式版 / 官方发布 / released"。

## 2. 全自动发布链

```
源仓改 VERSION → release.yml 建 v-tag Release → packages/auto-bump.yml 扫描建 PR
→ 自动 squash 合并 → packages.yml 构建 + 签名 → 同步到 Linxira-OS/linxira-packages
```

- **排除**:calamares / shelly(外部上游)、`linxira-zeta`(独立,稳定前进链)。
- `packages.yml` 的 `pull_request` 事件**跳过 publish 是安全设计**(PR 代码不自动发布);
  真正发布靠 squash 合并后的 **push 事件**。日志里 "publish skipping" 指 PR run,不是故障。
- 全链**零人工**;不要手工改 `linxira-packages` 或手工 bump PKGBUILD 绕开链。

## 3. 正式版验收门(全部通过方可称 release)

1. 固件引导:BIOS + UEFI
2. 写盘安装:BIOS + UEFI
3. 双内核(`linux` / `linux-lts`)引导 + initramfs
4. 首次启动
5. Package Center 事务
6. 恢复路径(含 Timeshift 快照创建/还原、live-ISO 恢复)
7. 包签名与已安装文件完整性校验

对应 `docs/ROADMAP.md` 的 `Verification` 与 `Release` 段;未全部勾选前,不得称 release。

## 4. Release manifest(发布档案)

- 每张**正式发布**镜像配一份不可变 JSON,按 `releases/manifest.schema.json` 校验。
- 必须绑定的证据:源码提交、Arch/Linxira 包清单、`[linxira]` repo 数据库哈希、
  ISO SHA-256、SBOM、构建者身份、各验收门结果。
- `acceptance: rejected-diagnostic` 的记录可保留,但不作为 release candidate。
- **从下一次构建起**,构建过程随构建一并产出这些证据;已被拒绝或已丢失的历史构建
  **不回溯补建**。

## 5. 签名与密钥

- 签名密钥金库在工作区根 `.keys/` —— **禁止提交、禁止外传**。
- 包文件与仓库库文件(`.db` / `.files`)必须签名;keyring 随 `[linxira]` 仓库分发。

## 6. 回滚

- 目标系统为 Btrfs + Timeshift 快照;正式发布须同时公布**已知限制与回滚程序**。

## 7. 发布禁令

- 未过验收门不得称 release,不得把 README / 官网改成"正式版"。
- 不得在无 manifest 证据的情况下勾选 ROADMAP 发布项。
- 不得把测试镜像的构建期产物(包清单、db、SBOM)当作正式发布证据使用。