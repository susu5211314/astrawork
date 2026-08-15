# AstraWork

AstraWork 是一款面向软件开发全流程的 **Windows 本地优先、多 Agent 桌面工作台**。

项目目前处于产品设计与交互原型阶段，尚未创建正式应用工程。已确定的架构基线包括：

- Tauri Windows 桌面容器
- 应用内置 Windows 标题栏
- 左侧导航、中间对话、按需展开的右侧上下文面板
- SQLite WAL、Durable Journal 与 Transactional Outbox
- 多模型供应商适配层
- 可编辑 DAG Agent 编排
- Git worktree、独立分支和进程隔离
- Windows Credential Manager 凭据存储

## 当前内容

- [总控 Agent：任务入口与需求澄清规格](docs/superpowers/specs/2026-08-15-orchestrator-task-intake-design.md)
- [任务入口与需求澄清交互原型 V12](prototypes/task-intake-right-default-closed-v12.html)

## 项目状态

当前仓库包含正式设计规格和可交互 HTML 原型，不包含可发布的桌面应用。前端框架及正式工程结构将在后续实施规划中确定。

## 许可与使用限制

本项目采用 [PolyForm Noncommercial License 1.0.0](LICENSE)。

你可以在许可证允许的范围内：

- 查看和研究源码；
- 进行非商业使用；
- 修改源码并进行非商业二次开发；
- 分发符合许可证要求的非商业派生版本。

你不可以在未获得版权所有者另行书面授权的情况下，将本项目或其派生版本用于商业目的。

分发原项目或二次开发版本时，必须：

1. 保留完整的 [LICENSE](LICENSE)；
2. 保留 [NOTICE.md](NOTICE.md) 中所有以 `Required Notice:` 开头的声明；
3. 明确注明 AstraWork 是原始项目，并保留原项目地址；
4. 不得暗示你的派生版本获得原作者官方认可。

由于许可证限制商业使用，本项目属于 **source-available（源码可用）**，不是 OSI 定义的开源软件。

## 原始项目

- 项目名称：AstraWork
- 原始作者：susu5211314
- 原始仓库：https://github.com/susu5211314/AstraWork

## 免责声明

本项目按现状提供，不附带任何明示或暗示担保。完整法律条款以 [LICENSE](LICENSE) 为准。
