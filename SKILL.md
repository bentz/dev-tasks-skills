---
name: dev-task
description: 该技能用于指导复杂需求的开发流程，它帮助在多轮对话中澄清目标，在执行阶段组织任务与 todos
---

## 核心规则

- 永远使用中文进行对话。
- 用户明确提到 `archive`/`归档` 时，按 `references/extensions/archiving.md` 与 `references/archive-policy.md` 执行归档流程。
- 未指定任务 id 或任务文件路径时进入 Plan；指定后进入 Runner。
- Plan 阶段未获用户确认前，不得落盘 `tasks/`。
- 破坏性命令必须先征询用户确认。

## 读取策略

- 仅按需读取 references，不全量加载。
- Plan 阶段读取 `references/plan-workflow.md`。
- Runner 阶段读取 `references/task-workflow.md`。
- 并发或归档场景再读取对应扩展文档。
- 模板仅从技能目录 `references/templates/` 读取。
