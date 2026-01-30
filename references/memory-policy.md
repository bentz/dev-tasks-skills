# Memory Policy（默认阈值与归档策略）

本文件用于约定项目内 `tasks/` 目录的“体积控制”规则，避免 `todos/info/logs` 随长程协作无限膨胀，从而影响重启恢复与信息检索。

> 默认策略可直接使用；如需覆盖，请在项目的 `tasks/info.md` 的 YAML block 中添加 `memory_policy_override`（见文末）。

---

## 1) 默认阈值（按行数）

- `tasks/todos.md`：超过 **200 行**触发归档
- `tasks/info.md`：超过 **200 行**触发归档

> 说明：任务日志按“每任务一个文件”拆分在 `tasks/logs/tasks/{id}.md` 下，默认不做全局阈值；如单个日志过大，可按同样方式手动归档为 `*.archive-001.md`。

---

## 2) 归档命名

- `tasks/todos.archive-001.md`、`tasks/todos.archive-002.md` …
- `tasks/info.archive-001.md`、`tasks/info.archive-002.md` …

---

## 3) 归档策略（推荐做法）

### 3.1 `tasks/todos.md`（计划板）

目标：保持 `todos.md` 只包含当前活跃任务，历史任务转移到 archive。

- 保留：
  - `status != done` 的任务（todo/doing/blocked）
  - 最近少量 `done`（可选，便于回看近期完成）
- 迁移：
  - 更早的 `done` 任务条目移动到 `tasks/todos.archive-00N.md`
- 在 `tasks/todos.md` 的 YAML block 中追加一个最简 `archives` 指针（YAML 风格即可），例如：

```yaml
archives:
  - file: tasks/todos.archive-001.md
    note: "2026-01-30 归档：迁移早期 done 条目"
```

### 3.2 `tasks/info.md`（项目记忆/索引）

目标：`info.md` 只保留“高复用的稳定信息 + 最简索引”，更细的历史记录迁移到 archive。

- 保留：
  - `index`（最简索引）
  - 项目级稳定事实/约束/常用入口/已知坑（高频复用）
- 迁移：
  - 过旧或低频的 Records/事件记录迁移到 `tasks/info.archive-00N.md`
- `index` 中保留对 archive 的指针，便于反查。

---

## 4) 项目内覆盖（在 `tasks/info.md` 中声明）

在 `tasks/info.md` 的 YAML block 中添加 `memory_policy_override` 即可覆盖默认策略（YAML 风格，非 table）：

```yaml
memory_policy_override:
  max_lines:
    todos: 300
    info: 400
```
