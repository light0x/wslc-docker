---
name: wslc-docker
description: 在 Windows 或 WSL 上用 wslc 替换 Docker CLI 命令，适用于容器构建、运行、脚本迁移和 Compose 兼容性评估。
---

# 用 wslc 替换 Docker CLI

- 仅替换面向 Windows 或 WSL 的 Docker 命令。保留原生 Linux 和 macOS 分支、Shell 控制流及引号。
- Windows 使用 `wslc`，WSL 通过 Windows 互操作调用 `wslc.exe`。以目标版本的 `wslc.exe <子命令> --help` 核对参数。
- 常见映射为 `docker ps -a` → `wslc container list --all`、`docker images` → `wslc image list`。转换 `run`、`build` 和 `exec` 时保持原参数语义。构建时保留原构建文件，必要时用 `-f Dockerfile` 显式指定。
- 从 WSL 传递宿主路径时，核对 Windows 进程能否访问该路径，必要时用 `wslpath -w` 转换。容器内路径保持 Linux 格式。
- 核对 Compose、Buildx、重启策略和 Docker Engine API 依赖的支持情况，说明无法保留的行为。将 Compose 服务转换为单独的 `wslc run` 命令时，核对网络、数据卷、启动顺序和就绪条件。
- 如果 `wslc-mcp` 可用，可批量查询有疑问的独立 Docker 命令。按原脚本的分支回填，并检查转换工具报告的行为变化。
