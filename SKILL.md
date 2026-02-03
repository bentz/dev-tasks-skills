---
name: dev-task
description: "该技能用于指导“复杂需求的开发流程”，强调抽象步骤与可复用的落地方式，而不是针对某个具体需求的复盘总结。它帮助在多轮对话中澄清目标、在执行阶段组织任务与 todos、在收尾阶段补齐测试与边界处理。"
---

## 重要
- 永远使用中文进行对话

## 核心规则（MUST/SHOULD/MAY）

MUST
- 未指定任务 id 或任务文件路径 → planner；指定了则进入 runner
- 用户未确认前不得落盘 `tasks/`
- 破坏性命令必须先征询用户确认

SHOULD
- 按需读取 references，默认不全量加载
- 执行时记录关键命令与结论到任务日志
- 变更范围优先遵循 `Files` 列表，超出需说明原因

MAY
- 触发并发或归档场景时加载扩展文档

## 读取策略（节省 token）

默认按“按需读取、逐步展开”执行，不要一次性加载所有 `references/*`：

- 必读：`SKILL.md`
- planner：读取 `references/plan-workflow.md`
- runner：读取 `references/task-workflow.md`
- 示例/模板/策略文件：仅在创建/校验对应文件时读取（不要默认全读）
- 扩展文档：仅在触发场景时读取 `references/extensions/*.md`

## 扩展索引（按需加载）

- 并发控制：`references/extensions/concurrency.md`
- 归档策略：`references/extensions/archiving.md`

触发建议：
- 多 agent 并行写入或认领 → 并发控制
- `tasks/` 体量增大或日志膨胀 → 归档策略

## 快速要点

- 信息不足时再问关键问题；否则直接给出可执行方案。
- 执行型需求：创建并维护项目内 `tasks/`（见下方），确保可审计、可恢复（除非用户明确只分析/建议）。
- 明显破坏性操作/命令（例如删除文件、重置历史）必须先征询用户确认。
- 涉及归档时：先说明将执行归档操作，征询用户确认后再落盘。

## 触发规则（planner / runner）

- **planner（默认）**：未指定任务 id 或任务文件路径。
- **runner**：指定了任务 id 或任务文件路径（进入 task-runner 流程）。
- **planner 落盘时机**：先输出方案摘要与可能涉及文件，待用户明确“确认创建/落盘/立项”后，才创建 `tasks/` 中的任务文件与 `todos.json` 记录；落盘后仍可补充/调整，需用户“最终确认/开始执行”才进入 runner。

### 方案与落盘流程（planner）

- **方案阶段（不落盘）**：给出方案摘要（目标/范围/非范围/关键风险/验收要点），列出可能涉及的文件，并询问是否落盘。
- **落盘阶段（初版文档）**：用户确认后，生成 `tasks/items/{id}.md` 草案（结构与 `references/templates/000-task-example.md` 一致），创建 `tasks/logs/tasks/{id}.log`，并在 `tasks/todos.json` 新增条目（`status: todo`）。
- **补充/调整**：用户提出“补充/调整/追加”仅更新当前任务文档，不新建任务；保持 `status: todo` 直至最终确认。

## 输出契约（planner / runner）

### planner（对话输出）
- **方案阶段**：输出方案摘要 + 可能涉及文件 + 落盘确认语
- **落盘阶段**：输出任务文档草案（模板一致），并提示可继续补充/调整

### runner（对话输出模板）
- 执行摘要：做了什么（1-3 条）
- 结果状态：`done` / `blocked`
- 验证方式：如何验证（或为何无法验证）
- 关键变更：涉及的关键文件
- 若阻塞：阻塞原因 + 需要用户提供的内容

### `tasks/` 最小结构（执行型需求默认启用）
- `tasks/todos.json`：任务清单（字段/状态机见 `references/task-workflow.md`）
- `tasks/items/{id}.md`：任务文件（规划/约束/验收/进展/结果；用于记录与反查）
- `tasks/logs/tasks/{id}.log`：任务执行日志（记录命令与关键摘要）
- 允许归档：`tasks/todos.archive-*.json`

### 触发词
- 用户输入 **“创建任务 / 新任务 / 立项 / 任务单”**：创建 `tasks/items/{id}.md` 并登记到 `tasks/todos.json`，然后请用户确认内容。
- 以下关键词视为“当前任务的增量需求”，不得新建任务：**补充 / 调整 / 追加**。
- 用户明确 **“最终确认 / 开始执行”** 时，允许进入 runner 执行当前任务。

### 短路模式（仅分析/建议）
当用户明确要求“只分析/建议、不执行”时：
- 不创建任务、不写 `tasks/`，仅给出建议与风险提示（这是“执行型需求都创建 tasks”约定的唯一例外）。

### 验收底线（简版）
- 行为符合需求（含关键边界）
- 不引入新的 TypeScript / ESLint error
- 任务文件补齐“如何验证”

### 错误处理（最小版）
- `error_type`: `TRANSIENT | PERMISSION | REQUIREMENT | ENVIRONMENT`
- `TRANSIENT` → `blocked`，提示可手动重试
- `PERMISSION` → `blocked`，说明需用户确认/授权
- `REQUIREMENT` → 返回 planner 澄清
- `ENVIRONMENT` → `blocked`，给出修复指引
- 失败时在对话与任务日志记录 `error_type`、摘要与建议动作

## 任务目录与引用指示

references 目录下包含 workflow 与模板/策略文件：仅在需要时读取；引用前确认存在。

- `references/plan-workflow.md`：planner 工作流
- `references/task-workflow.md`：runner 工作流与状态机规范
- `references/templates/000-task-example.md`：任务模板示例（按需参考）
- `references/templates/todos-template.md`：`tasks/todos.json` 模板
- `references/templates/task-log-template.md`：任务日志模板
- `references/archive-policy.md`：归档策略（详细）
- `references/extensions/concurrency.md`：并发控制（扩展）
- `references/extensions/archiving.md`：归档策略（扩展）
