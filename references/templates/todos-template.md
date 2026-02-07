# `tasks/todos.json` 模板（JSON）

`tasks/todos.json` 是任务调度的 SSoT（唯一真相来源）。

最小示例：

```json
{
  "updated_at": "2026-02-01 10:00:00",
  "tasks": [
    {
      "id": "001-TASK",
      "title": "任务标题",
      "status": "todo",
      "claimed_by": "",
      "task_file": "tasks/items/001-TASK.md",
      "log_file": "tasks/logs/tasks/001-TASK.log",
      "updated_at": "2026-02-01 10:00:00"
    }
  ]
}
```

字段约定：
- `title`: 必填
- `status`: `todo | doing | blocked | done`
- `claimed_by`: 可为空字符串
- `task_file` / `log_file`: 项目内相对路径
- `updated_at`: 本地时间字符串 `YYYY-MM-DD HH:mm:ss`
