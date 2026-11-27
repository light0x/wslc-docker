---
name: wslc-docker
description: 在 Windows 或 WSL 上将 Docker CLI 命令迁移到 wslc，将 Docker Compose 项目迁移到 wslc-compose，适用于容器构建、运行和脚本迁移。
---

# 用 wslc 替换 Docker CLI

- 仅替换面向 Windows 或 WSL 的 Docker 命令。保留原生 Linux 和 macOS 分支、Shell 控制流及引号。
- Windows 使用 `wslc`，WSL 通过 Windows 互操作调用 `wslc.exe`。以目标版本的 `wslc.exe <子命令> --help` 核对参数。
- 常见映射为 `docker ps -a` → `wslc container list --all`、`docker images` → `wslc image list`。转换 `run`、`build` 和 `exec` 时保持原参数语义。构建时保留原构建文件，必要时用 `-f Dockerfile` 显式指定。
- 从 WSL 传递宿主路径时，核对 Windows 进程能否访问该路径，必要时用 `wslpath -w` 转换。容器内路径保持 Linux 格式。
- 核对 Buildx、重启策略和 Docker Engine API 依赖的支持情况，说明无法保留的行为。
- 如果 `wslc-mcp` 可用，可批量查询有疑问的独立 Docker 命令。按原脚本的分支回填，并检查转换工具报告的行为变化。

## Docker Compose 迁移

- Windows 和 WSL 暂用 PyPI 上的 `wslc-compose==0.3.0`。需要 Python 3.9 或更高版本及可用的 WSL 容器 CLI。可用 `pipx install "wslc-compose==0.3.0"` 隔离安装，并用 `wslc-compose --version` 核对版本。若工具找不到原生 CLI，将 `WSLC_COMPOSE_BIN` 设为实际的 `wslc.exe` 路径。
- 优先调用独立的 `wslc-compose`，避免 `PATH` 顺序影响 `wslc compose` 调用的是随包安装的包装器还是原生 CLI。保留现有 Compose 文件，将 `docker compose up -d`、`docker compose ps`、`docker compose logs` 和 `docker compose down` 的命令前缀替换为 `wslc-compose`。将 `-f`、`-p`、`--env-file` 和 `--profile` 等全局选项放在子命令前，保留多个 `-f` 的顺序。
- 先运行 `wslc-compose -f compose.yaml config` 检查解析结果，再运行 `wslc-compose --dry-run -f compose.yaml up -d` 查看生成的命令。使用项目实际的文件和选项，核对端口、网络、数据卷及挂载路径。WSL 下工具会尝试将绑定挂载的源路径转换为 Windows 路径，需确认目标环境可以访问转换后的路径。
- 0.3.0 忽略 `restart` 策略和 `healthcheck`，`depends_on: condition: service_healthy` 降级为按依赖顺序启动。检查配置警告和各子命令的 `--help`，说明项目中受影响的行为。
- 需要改用单独的 `wslc run` 命令时，核对网络、数据卷、启动顺序和就绪条件。
