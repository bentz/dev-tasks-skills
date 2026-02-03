# Plan Workflow

本文件用于 planner 角色的工作流。

---

## 1. 触发条件

- 未指定任务 id 或任务文件路径时，进入 planner 流程。

---

## 2. 输出与确认

### 2.1 方案阶段（不落盘）
- 产出方案摘要（目标/范围/非范围/关键风险/验收要点）
- 列出可能涉及的文件
- 仅对话输出，不写入 `tasks/`

### 2.2 用户确认（落盘）
- 用户明确“确认创建/落盘/立项”后才允许落盘
- 用户确认前不写入 `tasks/todos.json`

### 2.3 补充与最终确认
- 落盘后允许补充/调整当前任务文档
- 用户明确“最终确认/开始执行”后进入 runner

---

## 3. 落盘（确认后）

1) 创建 `tasks/items/{id}.md`（结构使用 `references/templates/000-task-example.md`）
2) 创建 `tasks/logs/tasks/{id}.log`
3) 在 `tasks/todos.json` 中新增条目（`status: todo`）
4) 提示用户任务已落盘，可继续补充/调整

---

## 4. 写入范围

- 允许写入：
  - `tasks/items/{id}.md`
  - `tasks/logs/tasks/{id}.log`
  - `tasks/todos.json`

- 不允许写入：
  - 业务代码
  - 其他任务文件
