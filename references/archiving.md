# 归档策略

## 触发

- `tasks/todos.json` 超过 **200 行** 或 **20 条任务** 时触发归档评估。
- `tasks/logs/tasks/` 中单个日志超过 **400 行** 时触发归档评估。
- 用户明确提出 `archive`/`归档` 时触发归档流程。

## 归档规则

- 只归档 `status=done` 的任务。
- `tasks/todos.json` 保留全部非 done 任务 + 最近 3 条 done 任务。
- 当前任务日志默认保留最新 200 行。
- 归档前必须征询用户确认。
- 归档后更新 `tasks/todos.json` 的 `updated_at`。

## 归档对象与命名

- `tasks/archive/todos/archive-{task_name}-todos-YYYYMM.json`
- `tasks/archive/logs/archive-{task_name}-{id}-YYYYMMDD.log`
- 命名必须包含任务名、任务 id 与日期，避免覆盖并便于追溯。

## 覆盖约定

- 项目可按需覆盖阈值与保留数量。
- 覆盖项应记录在项目文档或任务说明中，并注明原因与有效期。
