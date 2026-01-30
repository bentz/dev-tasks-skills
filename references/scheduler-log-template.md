# `tasks/logs/scheduler.md` 模板（调度器日志）

调度器日志用于重启恢复与反查“为什么这样分配/为什么 blocked/下一步是什么”。

建议结构（Markdown 即可）：

```md
# Scheduler Log

## Agent IDs
- 2026-01-30 10:00: agent-1 = "scheduler/main"
- 2026-01-30 10:00: agent-2 = "executor/backend"

## Events
- 2026-01-30 10:01: mode=long；Doctor=enabled
- 2026-01-30 10:02: claimed 001-TASK by agent-2; todos: todo → doing
- 2026-01-30 10:15: 001-TASK blocked (needs DB access); next: user to provide creds
- 2026-01-30 10:40: 001-TASK unblocked; resumed; todos: blocked → doing
- 2026-01-30 11:10: 001-TASK done; todos: doing → done
```
