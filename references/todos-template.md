# `tasks/todos.md` 模板（YAML in Markdown）

推荐将 `tasks/todos.md` 作为一个“可读的 Markdown 文件 + 一个稳定的 YAML 数据块”。

- 不要使用 Markdown table
- 字段名保持固定（便于多 agent 协作与 Doctor 检查）
- `status` 必须为：`todo | doing | blocked | done`

示例：

```yaml
updated_at: "2026-01-30 10:00"
mode: simple # simple | long

archives: [] # 可选：归档指针（见 references/memory-policy.md）

tasks:
  - id: 001-TASK
    title: "实现登录页 i18n"
    status: doing
    priority: high # high | medium | low（可选）
    claimed_by: agent-1 # agent id；未认领用空字符串
    task_file: tasks/items/001-TASK.md
    log_file: tasks/logs/tasks/001-TASK.md
    updated_at: "2026-01-30 10:00"

  - id: 002-TASK
    title: "补齐单测覆盖"
    status: todo
    priority: medium
    claimed_by: ""
    task_file: tasks/items/002-TASK.md
    log_file: tasks/logs/tasks/002-TASK.md
    updated_at: "2026-01-30 10:00"
```
