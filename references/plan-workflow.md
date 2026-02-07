# Plan Workflow

本文件定义 Plan 阶段规则。

## 1. 触发条件

- 未指定任务 id 或任务文件路径时进入 Plan 阶段。

## 2. 对话输出与确认

- 多轮澄清后，必须先询问“是否可以出方案”；未获确认不得输出方案草稿。
- 方案内容保持简化：目标范围、风险约束、验收要点。
- 澄清阶段若用户提出补充/调整，后续方案输出仅包含与上一版相比的差异点。
- Plan 阶段只做对话，不写入 `tasks/`。

## 3. 落盘触发与动作

- 仅当用户明确提出“创建/立项/落盘”等意图时才允许落盘。
- 生成 `id`：最大序号 + 1（不足 3 位补零）+ `-<kebab-case>` 任务简写。
- 创建 `tasks/items/{id}.md`（模板：`references/templates/000-task-example.md`）。
- 创建 `tasks/logs/tasks/{id}.log`（模板：`references/templates/task-log-template.md`）。
- 在 `tasks/todos.json` 新增任务（模板：`references/templates/todos-template.md`，初始 `status: todo`）。
- 用户明确“开始执行/执行”后，切换到 Runner 阶段。

## 4. 写入边界

- 允许：`tasks/items/{id}.md`、`tasks/logs/tasks/{id}.log`、`tasks/todos.json`。
- 禁止：业务代码、其他任务文件。
