# 归档与容量策略（tasks/）

本文件是归档实现细节的唯一来源。

## 1) 触发阈值

满足任一条件即可进入归档流程：
- `tasks/todos.json` 超过 **200 行** 或 **20 条任务**
- `tasks/logs/tasks/` 中单个日志超过 **400 行**

## 2) 归档对象与命名

- `tasks/archive/todos/archive-{task_name}-todos-YYYYMM.json`
- `tasks/archive/logs/archive-{task_name}-{id}-YYYYMMDD.log`
- 命名必须包含任务名、任务 id 与日期，避免覆盖并便于追溯。

## 3) 执行规则

- 仅允许归档 `status=done` 的任务。
- `tasks/todos.json` 保留全部非 done 任务 + 最近 3 条 done 任务。
- 其余 done 任务移动到 `tasks/archive/todos/archive-{task_name}-todos-YYYYMM.json`。
- 单个任务日志超过阈值时，将早期内容迁移到 `tasks/archive/logs/archive-{task_name}-{id}-YYYYMMDD.log`。
- 当前任务日志默认保留最新 200 行。
- 归档前必须征询用户确认；归档后更新 `tasks/todos.json` 的 `updated_at`。

## 4) 覆盖约定

- 项目可按需覆盖阈值与保留数量。
- 覆盖项应记录在项目文档或任务说明中，并注明原因与有效期。
