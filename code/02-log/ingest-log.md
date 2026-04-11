# Ingest Log

## 2026-04-10
- 来源：知识库结构改造
- 输入文件：无
- 更新页面：
  - `00-schema/AGENTS.md`
  - `00-schema/工作流提示词.md`
  - `01-inbox/README.md`
- 新建页面：
  - `02-log/ingest-log.md`
  - `02-log/weekly-lint.md`
- 新增结论：
  - 知识库新增 `00-schema/`、`01-inbox/`、`02-log/` 三个工作流目录
  - 后续新增资料统一先进入 `01-inbox/`
  - 主题知识继续沉淀到 `工具/`、`跨平台开发/`、`Agent/`、`DevOps/`

## 2026-04-10 - 演示 ingest
- 来源：`01-inbox/2026-04-10 Docker 容器排障速记.md`
- 输入文件：
  - `01-inbox/2026-04-10 Docker 容器排障速记.md`
- 更新页面：
  - `DevOps/Docker.md`
- 新建页面：无
- 新增结论：
  - Docker 排障时可以按“日志 -> 状态 -> 端口映射 -> 进入容器”顺序检查
  - 进入精简镜像时，`/bin/sh` 往往比 `/bin/bash` 更通用
  - `docker compose` 项目应优先使用 compose 维度的状态和日志命令排障
