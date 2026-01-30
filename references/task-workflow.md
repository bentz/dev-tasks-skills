# Workflow Specification

本文件定义了在任意项目中使用 `dev-task` skill 进行“任务化开发”时的行为规范。
AI **默认不得修改本文件内容**；仅在**用户明确授权**时才允许修改。

---

## 0. 模式（simple / long）

- `simple`：依然使用 `tasks/` 做可审计记录，但不引入“调度器（scheduler）”角色；默认单 agent 串行推进。
- `long`：引入调度器（scheduler），允许并行 agent 执行；支持重启恢复；可选启用 “Doctor（健康检查）”。

> 说明：无论 simple/long，项目内都应存在 `tasks/` 目录与核心文件；`long` 只是增加“调度”与“恢复/并行”能力。

## 1. 相关文件约定（全部位于项目仓库的 `tasks/` 下）

- 任务计划板：`tasks/todos.md`
  - 用于记录“有哪些任务、当前状态、由哪个 agent 认领”。
  - **格式必须固定**：推荐使用一个 `yaml` code block 存放结构化数据（不要用 Markdown table）。
  - 推荐 YAML 结构：
    - `updated_at`：更新时间
    - `mode`：`simple | long`
    - `archives`：归档指针（可选）
    - `tasks`：任务数组
  - 最小示例（可直接复制；完整模板见 `references/todos-template.md`，非必读）：

```yaml
updated_at: "2026-01-30 10:00"
mode: long # simple | long
archives: []
tasks:
  - id: 001-TASK
    title: "..."
    status: doing # todo | doing | blocked | done
    claimed_by: agent-1
    task_file: tasks/items/001-TASK.md
    log_file: tasks/logs/tasks/001-TASK.md
    updated_at: "2026-01-30 10:00"
```
  - 字段建议包含：
    - `status`：`todo | doing | blocked | done`
    - `claimed_by`：agent id；未认领可为空
    - `task_file`：例如 `tasks/items/001-TASK.md`
    - `log_file`：例如 `tasks/logs/tasks/001-TASK.md`
    - `updated_at`：建议 `"YYYY-MM-DD HH:mm"`（本地时间字符串）
  - **调度真相来源（SSoT）**：`tasks/todos.md` 作为调度与分配的唯一真相；任务文件中的 `Meta.status/claimed_by` 仅作为镜像反查。

- 任务文件：`tasks/items/{id}.md`（后续统称为 {任务文件md}）
  - 一个文件承载：规划（Plan）/ 约束（Constraints）/ 验收（Acceptance）/ 已做内容（Notes/Progress）/ 结果（Result）。
  - 任务文件必须包含 `Meta.status`（与 `tasks/todos.md` 镜像，用于反查）。
  - 任务文件可以包含 `Meta.claimed_by`（镜像反查用；以 `tasks/todos.md` 为准）。
  - 任务文件不应作为“调度面板”使用：避免在任务文件里维护额外的队列/分配状态。

- 项目信息与索引：`tasks/info.md`
  - 用于沉淀项目级“稳定事实/约束/常用入口/已知坑”等，并维护最简索引（YAML 风格，不用 table）。
  - 可在其中声明非默认的 memory-policy（见下文）。
  - 推荐 YAML 结构：
    - `updated_at`
    - `index`
    - `facts`（可选）
    - `memory_policy_override`（可选）
  - 最小示例（完整模板见 `references/info-template.md`，非必读）：

```yaml
updated_at: "2026-01-30 10:00"
memory_policy_override:
  max_lines:
    todos: 200
    info: 200
index:
  - id: 001-TASK
    title: "..."
    status: doing
    task_file: tasks/items/001-TASK.md
    log_file: tasks/logs/tasks/001-TASK.md
facts:
  - "..."
```

- 日志目录：`tasks/logs/`
  - 调度器日志：`tasks/logs/scheduler.md`（仅 long 模式强制；simple 模式可选）
  - 任务执行日志：`tasks/logs/tasks/{id}.md`（执行期间必须存在；记录命令与关键信息摘要）
  - 最小示例（完整模板见 `references/scheduler-log-template.md` / `references/task-log-template.md`，非必读）：

```md
# Scheduler Log
- 2026-01-30 10:00: mode=long
```

```md
# Task Log: 001-TASK
- 2026-01-30 10:00: Started
- 2026-01-30 10:05: Ran `...`
```

---

## 2. 角色与职责（scheduler / planner / executor）

### 2.1 scheduler（调度器，long 模式启用）

- 负责：任务拆分与编排、认领与分配、并行冲突处理、blocked 解除路径、重启恢复。
- 负责维护的一致性：
  - `tasks/todos.md` 的条目与 `tasks/items/{id}.md` 的 `Meta.status` 同步（镜像反查）。
  - `tasks/todos.md` 的 `claimed_by` 与实际执行者一致（使用 agent id）。
- 写入范围：
  - 允许修改：`tasks/todos.md`、`tasks/info.md`
  - 允许追加：`tasks/logs/scheduler.md`、`tasks/logs/tasks/{id}.md`
  - 允许更新：`tasks/items/{id}.md` 的 `Meta.status` / `Meta.claimed_by`（仅镜像字段）
  - 不应修改：{任务文件md} 的 `Plan/Constraints/Acceptance/Files`（这些由 planner 负责）

#### agent id 规则（claimed_by）

- 优先使用系统提供的稳定 id（如对话上下文里可获得）。
- 如果无法获得稳定 id，由 scheduler 分配：`agent-1`、`agent-2`、`agent-3`… 并在 `tasks/logs/scheduler.md` 记录映射（便于重启恢复与反查）。

### 2.2 planner（规划器）

- 负责：在 {任务文件md} 内产出“可执行计划”与“约束/验收”。
- 输出必须尽量具体（文件/模块/关键函数/边界/测试点），并能被 executor 顺序执行。
- 写入范围（建议）：
  - 允许修改：{任务文件md} 的 `Plan/Constraints/Acceptance/Files`
  - 允许更新：{任务文件md} 的 `Meta`（除 status/claimed_by 这类镜像字段外，需与 scheduler 协调）
  - 不建议直接改：`tasks/todos.md`

### 2.3 executor（执行器，可并行）

- 负责：按 {任务文件md} 的计划实现、验证、回填结果。
- 写入范围（建议）：
  - 允许追加：`tasks/logs/tasks/{id}.md`（命令、输出摘要、失败原因）
  - 允许追加：{任务文件md} 的 `Notes/Progress/Result`（已做内容与结论）
  - 不建议在 long 模式直接改 `tasks/todos.md`（避免并发冲突）；由 scheduler 统一更新状态
  - 不建议在执行阶段重写 `Plan/Constraints`；如需调整先与 scheduler/planner 对齐，再变更并记录原因

> 注：simple 模式下同一个 agent 可以同时扮演 planner+executor（并承担最小化的调度工作）。

---

## 3. 工作循环（单任务生命周期）

### 3.1 创建任务（Create）

1. 创建 `tasks/items/{id}.md`（参考 `references/000-task-example.md`）
2. 创建 `tasks/logs/tasks/{id}.md`（参考 `references/task-log-template.md`）
3. 在 `tasks/todos.md` 中新增条目（参考 `references/todos-template.md`）：
   - `status: todo`
   - `claimed_by: ""`
   - `task_file`、`log_file`、`updated_at`
4. 请求用户确认任务内容；用户确认后才进入执行。

### 3.2 认领任务（Claim）

- long 模式：由 scheduler 执行
  - 在 `tasks/todos.md` 中写入 `claimed_by: <agent-id>`，并将 `status` 切到 `doing`
  - 同步更新 {任务文件md} 的 `Meta.status: doing`
  - 在 `tasks/logs/scheduler.md` 追加“认领记录”

- simple 模式：当前 agent 自行认领并更新上述文件。

### 3.3 执行与记录（Execute）

- executor 按计划修改代码；执行中必须记录：
  - 在 `tasks/logs/tasks/{id}.md` 追加：
    - 时间戳
    - 运行的命令（如有）
    - 关键输出摘要/错误摘要
    - 自检结论/下一步
  - 在 {任务文件md} 追加“Progress/Notes/Result”：
    - 已做内容（高层摘要）
    - 关键决策与约束变化（如有）

### 3.4 完成或阻塞（Close）

- 完成（done）条件：
  - 通过自检认为满足 `Acceptance`
  - 任务文件与日志均已补齐（Result/验证方式/已知风险）
- 阻塞（blocked）条件：
  - 无法继续推进，或需要用户输入/外部资源/权限

更新要求（建议由 scheduler 执行，避免并发冲突）：
- `tasks/todos.md`：更新 `status: done|blocked`、`updated_at`
- {任务文件md}：同步更新 `Meta.status`，并在 `Notes` 末尾追加完成/阻塞原因
- `tasks/logs/tasks/{id}.md`：追加最终摘要与建议下一步（blocked 必须包含“如何解除阻塞”）

---

## 4. 编辑与并发规则（避免“状态漂移”）

- `tasks/todos.md` 是调度视图；{任务文件md} 是任务视图；两者都包含 `status` 用于反查。
- long 模式建议：
  - 只有 scheduler 负责更新 `tasks/todos.md` 与 {任务文件md}.`Meta.status`
  - executor 只追加日志与结果，不直接改“官方状态”
- simple 模式：
  - 当前 agent 作为“最小化 scheduler”，允许直接更新两边，但必须保持同步。

### 4.1 状态机（必须遵守）

- 允许的状态值：`todo | doing | blocked | done`
- 允许的状态迁移：
  - `todo → doing`
  - `doing → blocked | done`
  - `blocked → doing`
- 不变量（Doctor 必查）：
  - `status: doing` ⇒ `claimed_by` 必须非空
  - `status: todo` ⇒ `claimed_by` 应为空（推荐）

### 4.2 冲突处理（todos vs Meta）

- 若 `tasks/todos.md` 与 {任务文件md}.`Meta.status/claimed_by` 不一致：
  - Doctor 需要报告不一致；
  - 修复时以 `tasks/todos.md` 为准，同步更新任务文件的 `Meta.*`；
  - 自动修复前必须征询用户确认，并在 `tasks/logs/scheduler.md` 记录“修复前/修复后摘要”。

---

## 5. Doctor（健康检查，仅 long 模式启用）

在 long 模式启动或重启恢复时，scheduler 应先做一次只读检查，并把发现的问题写入 `tasks/logs/scheduler.md`：
- `tasks/todos.md` 与 `tasks/items/*.md` 的 `Meta.status` 是否一致
- 是否存在 `claimed_by` 但长时间无更新的任务（可能卡死）
- 是否存在 `doing` 但缺少 `tasks/logs/tasks/{id}.md` 的任务
- 标题/描述/验收是否互相矛盾（会导致 executor 误解）
- 是否存在 `doing` 但 `claimed_by` 为空（违反状态机不变量）

> Doctor 默认只报告；如需自动修复（例如同步 status），应先征询用户确认。

---

## 6. 安全与范围限制

- 命令执行：
  - 不再要求“仅运行声明过的命令”，但**必须**把实际运行的命令记录到 `tasks/logs/tasks/{id}.md`。
  - 任何明显破坏性命令（例如 `rm -rf`、`git reset --hard`）必须先征询用户确认。

- 代码范围：
  - 优先修改 {任务文件md} 中 `Files` 列出文件；
  - 如需修改其他文件，必须在任务日志中说明原因；
  - 除非用户明确要求，避免对 `.env*`、CI/CD、部署脚本做改动。

- 文件结构：
  - 不得重命名或移动 `tasks/todos.md`、`tasks/items/`、`tasks/info.md`、`tasks/logs/`。

---

## 7. Memory Policy（默认策略在 skill 内，可在项目中覆盖）

- 默认策略见：`references/memory-policy.md`
- 项目覆盖方式：在 `tasks/info.md` 的 YAML block 中增加 `memory_policy_override`（参考：`references/info-template.md`）
- scheduler 应优先读取 override；无 override 则使用默认策略。
