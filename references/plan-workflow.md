# Plan Workflow

本文件用于 planner 角色的工作流。

---

## 1. 触发条件

- 未指定任务 id 或任务文件路径时，进入 planner 流程。

---

## 2. 输出与确认

### 2.1 产出计划
- 产出可执行计划与约束/验收
- 使用 `references/000-task-example.md` 作为模板
- 仅生成计划与任务内容，不直接执行

### 2.2 用户确认
- 生成任务内容后必须请求用户确认
- 用户确认前不写入 `tasks/todos.json`

---

## 3. 落盘（确认后）

1) 创建 `tasks/items/{id}.md`
2) 创建 `tasks/logs/tasks/{id}.log`
3) 在 `tasks/todos.json` 中新增条目（`status: todo`）
4) 提示用户任务已落盘，可进入 runner 执行

---

## 4. 写入范围

- 允许写入：
  - `tasks/items/{id}.md`
  - `tasks/logs/tasks/{id}.log`
  - `tasks/todos.json`

- 不允许写入：
  - 业务代码
  - 其他任务文件
