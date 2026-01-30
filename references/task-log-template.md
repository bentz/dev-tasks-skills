# `tasks/logs/tasks/{id}.md` 模板（任务执行日志）

任务执行日志用于记录“做了什么、跑了什么命令、结果如何、失败原因与下一步”，便于并行协作与重启恢复。

建议结构（Markdown 即可）：

```md
# Task Log: 001-TASK

## Meta
- id: 001-TASK
- task_file: tasks/items/001-TASK.md
- claimed_by: agent-1

## Log
- 2026-01-30 10:05: Started
- 2026-01-30 10:12: Ran `pnpm --filter server test` (pass)
- 2026-01-30 10:18: Found regression in xxx; next: adjust yyy

## Blocked (if any)
- reason: "需要用户提供 xxx"
- next: "用户确认/提供后继续"
```
