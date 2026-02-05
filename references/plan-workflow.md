# Plan Workflow

本文件用于 Plan 阶段的工作流。

---

## 1. 触发条件

- 未指定任务 id 或任务文件路径时，进入 Plan 阶段。

---

## 2. 输出与确认

- 出方案许可：多轮澄清后必须询问“是否可以出方案”，未获确认不得输出方案草稿
- 简化方案：输出目标/范围/非范围/风险与约束/验收要点/可能涉及目录范围，仅对话输出，不写入 `tasks/`
- 迭代补充：用户提出“补充/调整/追加”时，仅提示与上一版方案的差异点，不新建任务、不落盘
- 落盘触发：用户明确“创建/立项/落盘”等关键字后才允许落盘；确认前不写入 `tasks/todos.json`
- 最终确认：落盘后允许补充/调整当前任务文档（仍需提示差异点）；用户明确“开始执行/执行”后进入执行阶段（Runner）

---

## 3. 落盘（确认后）

0) 若 `tasks/todos.json` 为空，提示用户选择起始序号（默认 001）
1) 生成 `id`：取最大序号 +1（不足 3 位补零）+ `-<kebab-case>` 任务简写
2) 创建 `tasks/items/{id}.md`（文件名规则：`tasks/items/{id}.md`，示例：`tasks/items/001-verify-task.md`；结构使用技能目录下的 `references/templates/000-task-example.md`，生成详细版本计划）
   - 详细计划至少包含 Description/Plan/Acceptance；Files 为可选目录范围
3) 创建 `tasks/logs/tasks/{id}.log`（结构使用技能目录下的 `references/templates/task-log-template.md`）
4) 在 `tasks/todos.json` 中新增条目（`status: todo`，结构使用技能目录下的 `references/templates/todos-template.md`）
4) 提示用户任务已落盘（含路径），可继续补充/调整

---

## 4. 写入范围

- 允许写入：
  - `tasks/items/{id}.md`
  - `tasks/logs/tasks/{id}.log`
  - `tasks/todos.json`

- 不允许写入：
  - 业务代码
  - 其他任务文件
