---
name: dev-task
description: "该技能用于指导“复杂需求的开发流程”，强调抽象步骤与可复用的落地方式，而不是针对某个具体需求的复盘总结。它帮助在多轮对话中澄清目标、在执行阶段组织任务与 todos、在收尾阶段补齐测试与边界处理。"
---

## 重要
- 永远使用中文进行对话

## 读取策略（节省 token）

默认按“按需读取、逐步展开”执行，不要一次性加载所有 `references/*`：

- 必读：`SKILL.md`
- `long` 模式：进入执行前必须读取 `references/task-workflow.md`
- `simple` 模式：通常不需要读取 `references/task-workflow.md`（除非遇到并发/漂移/需要对照规则）
- 模板与策略文件：仅在创建/校验对应文件时读取（不要默认全读）

## 模式选择（simple / long）

当用户未明确指定时，agent 需要根据需求复杂度选择模式，或询问用户确认：

- `simple`：也使用 `tasks/` 做可审计记录；默认单 agent 串行推进。
- `long`：引入 `scheduler`（调度器）负责拆分/并行/恢复；可选启用 Doctor（健康检查）。

> 说明：本 skill 约定 **无论 simple/long，执行型需求都应创建项目内 `tasks/` 结构**；`long` 只是增加“调度器”能力。

## 快速要点

- 信息不足时再问关键问题；否则直接给出可执行方案。
- 执行型需求：创建并维护项目内 `tasks/`（见下方），确保可审计、可恢复。
- 明显破坏性操作/命令（例如删除文件、重置历史）必须先征询用户确认。

### `tasks/` 最小结构（执行型需求默认启用）
- `tasks/todos.md`：任务列表（YAML code block；字段/状态机见 `references/task-workflow.md`）
- `tasks/items/{id}.md`：任务文件（规划/约束/验收/进展/结果）
- `tasks/info.md`：项目索引与信息（YAML code block；可覆盖 memory-policy）
- `tasks/logs/tasks/{id}.md`：任务执行日志（记录命令与关键摘要）
- `long` 额外：`tasks/logs/scheduler.md`（调度/恢复日志）
- 允许归档：`tasks/todos.archive-*.md`、`tasks/info.archive-*.md`

### 触发词
- 用户输入 **“创建”**：创建 `tasks/items/{id}.md` 并登记到 `tasks/todos.md`，然后请用户确认内容。
- 以下关键词视为“当前任务的增量需求”，不得新建任务：**补充 / 调整 / 追加**。

### 短路模式（仅分析/建议）
当用户明确要求“只分析/建议、不执行”时：
- 不创建任务、不写 `tasks/`，仅给出建议与风险提示（这是“执行型需求都创建 tasks”约定的唯一例外）。

### 验收底线（简版）
- 行为符合需求（含关键边界）
- 不引入新的 TypeScript / ESLint error
- 任务文件补齐“如何验证”

## 任务目录与引用指示

references 目录下包含 workflow 与模板/策略文件：不得声称其不存在；按需读取即可（优先依靠模型能力、仅在不确定时打开模板对照）。

- `references/task-workflow.md`：`long` 模式必读；定义状态机/并发/Doctor/格式
- `references/000-task-example.md`：创建/修改 `tasks/items/{id}.md` 时优先参考（结构不确定时再打开对照）
- 其它 templates / policy：仅在“格式不确定、Doctor 校验失败、触发归档策略”时读取
