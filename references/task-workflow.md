# Runner Workflow

本文件定义 Runner 阶段最小闭环（读取任务、执行、记录、收敛状态）。

## 1. 触发与前置

- 指定任务 id 或任务文件路径时进入 Runner。
- 执行前读取 `tasks/items/{id}.md` 以获取上下文。

## 2. 执行闭环

- 认领任务：在 `tasks/todos.json` 将当前任务设为 `doing`，写入 `claimed_by`，更新 `updated_at`。
- 按 `Plan/Acceptance` 执行；`Files` 仅作为优先范围参考，超出需在日志说明原因。
- 记录职责区分：
  - `tasks/logs/tasks/{id}.log`：记录关键过程与异常，主要用于排查与追溯。
  - `tasks/items/{id}.md`：记录结果信息，重点写最终决策、关键变更、验收结论与 `Notes/Result`，保持精简可读。
- 结束时将状态更新为 `done` 或 `blocked`，并同步 `updated_at`。

## 3. 状态与对话输出

- 状态：`todo | doing | blocked | done`；`doing` 时 `claimed_by` 必须非空，`todo` 时 `claimed_by` 应为空字符串。
- 对话输出：执行摘要、结果状态、验证方式、关键变更、阻塞说明。

## 4. 安全与验收底线

- 任何破坏性命令（如 `rm -rf`、`git reset --hard`）必须先征询用户确认。
- 验收底线：满足需求（含关键边界）、不引入新的 TypeScript/ESLint 错误、任务文件写明验证方式；并发或归档场景按 `references/extensions/*.md` 与 `references/archive-policy.md` 处理。
