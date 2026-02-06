# 归档与容量策略（tasks/）

本文件约定 `tasks/` 的体积控制与归档流程，保证可审计且长期可维护。

---

## 1) 归档触发（默认阈值）

满足任一条件即可归档：
- `tasks/todos.json` 超过 **200 行** 或 **20 条任务**
- `tasks/logs/tasks/` 中单个日志超过 **400 行**

---

## 2) 归档对象与命名

- `tasks/archive/todos/archive-{task_name}-todos-YYYYMM.json`：历史任务清单（按月归档）
- `tasks/archive/logs/archive-{task_name}-{id}-YYYYMMDD.log`：超大日志拆分归档（按日归档）

> 命名包含任务名、任务 id 与日期，便于追溯与避免覆盖。

---

## 3) 归档策略（推荐做法）

### 3.1 `tasks/todos.json`

目标：保持仅包含“活跃 + 近期完成”任务。

- 保留：
  - `status != done` 的任务（todo/doing/blocked）
- 最近完成的少量任务（默认 3 条）
- 迁移：
- 其余 `done` 任务移动到 `tasks/archive/todos/archive-{task_name}-todos-YYYYMM.json`

### 3.2 任务日志

目标：避免单日志过大影响阅读与检索。

- 单个日志超过阈值时，将早期内容移动到
  `tasks/archive/logs/archive-{task_name}-{id}-YYYYMMDD.log`
- 当前日志保留最新内容（默认 200 行）

---

## 4) 手动归档与覆盖

- 允许项目按需覆盖阈值与保留数量
- 覆盖项应记录在项目文档或任务说明中（注明原因与有效期）
