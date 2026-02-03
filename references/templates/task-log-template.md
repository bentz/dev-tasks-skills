# `tasks/logs/tasks/{id}.log` 模板（任务执行日志）

任务执行日志用于记录“做了什么、跑了什么命令、结果如何、失败原因与下一步”。

建议结构（Markdown 即可）：

```md
# Task Log: 001-TASK

- 2026-02-01 10:05: Started
- 2026-02-01 10:12: Ran `pnpm --filter server test` (pass)
- 2026-02-01 10:18: Found regression in xxx; next: adjust yyy
```
