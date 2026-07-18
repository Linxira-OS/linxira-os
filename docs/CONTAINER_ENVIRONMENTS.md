# Linxira 自动环境补全设计

## 结论

建议设计，但不要把大型环境直接塞进基础 ISO，也不要在首次启动时静默下载或自动执行。

Linxira 应增加独立的环境层，由 `linxira env` 管理。基础系统继续负责桌面、驱动、容器运行时和恢复能力；科学、AI、生信和固定开发环境由经过签名、固定摘要的容器镜像提供。

## 交付模式

### 在线环境

- Podman/Distrobox：交互式开发、IDE、语言工具链和普通科学计算。
- Apptainer：生物信息、HPC、批处理和需要单文件 SIF 的工作流。
- 镜像必须固定 OCI digest 或 SIF 哈希，不能只依赖可变标签。

### 离线环境包

大型环境使用独立伴随介质或下载包，不扩大基础安装镜像：

```text
linxira-env-ai-cpu-2026.07.oci.tar.zst
linxira-env-bio-core-2026.07.sif
linxira-env-dev-rust-2026.07.oci.tar.zst
```

离线包可以放在第二张环境 ISO、移动硬盘或局域网缓存中。基础 RC 镜像只携带环境目录和导入工具。

## 用户流程

```text
linxira env list
linxira env plan ai-cpu
linxira env fetch ai-cpu
linxira env create ai-cpu
linxira env verify ai-cpu
linxira env remove ai-cpu
```

`plan` 必须先显示：

- 下载和解压后的体积；
- CPU、GPU、架构和内存要求；
- 容器运行时；
- 网络访问；
- 主机目录挂载；
- 持久数据目录；
- 镜像来源、摘要、签名、许可证和 SBOM；
- 将创建的桌面入口、终端命令和 Jupyter 服务。

用户确认后才能执行 `fetch` 或 `create`。

## 目录模型

软件目录后续版本可以增加 `environments`，但不能包含任意 shell 命令：

```json
{
  "id": "ai-cpu",
  "runtime": "podman",
  "image": "registry.example/linxira/ai-cpu",
  "digest": "sha256:...",
  "architectures": ["x86_64"],
  "downloadSize": 4200000000,
  "gpu": "none",
  "network": "user",
  "mounts": ["projects", "datasets-readonly"],
  "signaturePolicy": "linxira-cosign-v1",
  "sbom": "spdx-url",
  "entrypoints": ["shell", "jupyter"]
}
```

`entrypoints`、`mounts` 和运行参数必须来自 Linxira 程序中的固定枚举，目录不能注入命令行。

## 安全边界

- 默认 rootless，不使用特权容器。
- 不挂载 Docker/Podman socket。
- 不默认挂载整个 Home，只允许项目目录和显式数据集目录。
- 数据集默认只读，输出写入独立工作目录。
- GPU 透传必须单独确认，并验证主机驱动和容器运行时兼容性。
- OCI 使用固定 digest 和签名验证；Apptainer 使用固定 SIF 哈希和签名。
- 不从 AUR 自动构建环境，不执行网络下载脚本，不允许目录携带任意仓库配置。
- 创建、更新和删除都写入 `/var/lib/linxira/environments/` 回执。

## 恢复和数据策略

容器层与系统快照分开：

- 容器镜像和缓存可删除后重新下载，不进入 Timeshift 保护范围。
- 项目代码、实验结果和数据集不依赖 Timeshift，必须单独备份。
- 删除环境只删除容器和可再生缓存，持久项目目录需要用户再次确认。

## 实施顺序

1. RC7 完成写盘安装、安装后首次启动和恢复验收。
2. 扩展 catalog v2 schema，加入只读 metadata/allowlist 和验证器；catalog 不执行事务。
3. 实现 `linxira env plan/fetch/create/verify/remove`，先支持 Podman。
4. 加入 Distrobox 桌面和终端集成。
5. 加入 Apptainer SIF、BioContainers 和批处理工作流。
6. 设计独立的离线环境包或伴随环境 ISO。
7. 完成 CPU、NVIDIA、AMD 和无 GPU 验证矩阵后再公开推荐。

这个功能属于 Milestone 2，不应阻塞基础系统安装和 RC7 验收。
