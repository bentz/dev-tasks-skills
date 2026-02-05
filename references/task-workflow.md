# Runner Workflow

本文件定义执行阶段（Runner）任务化开发规范。
AI 默认不得修改本文件内容；仅在用户明确授权时才允许修改。

---

## 1. 相关文件约定（全部位于项目仓库的 `tasks/` 下）

- 任务计划板：`tasks/todos.json`
  - SSoT（唯一真相来源）
  - 字段：`updated_at`、`tasks[]`
  - 任务条目字段：
    - `id` / `title`
    - `status`: `todo | doing | blocked | done`
    - `priority`（可选）
    - `claimed_by`（可选，空字符串表示未认领）
    - `task_file`: `tasks/items/{id}.md`
    - `log_file`: `tasks/logs/tasks/{id}.log`
    - `updated_at`

- 任务文件：`tasks/items/{id}.md`
  - 包含 Plan/Constraints/Files/Acceptance/Notes/Result
  - `Meta.status` 与 `tasks/todos.json` 镜像，仅用于反查
  - `Meta.claimed_by` 可选镜像字段

- 日志：
  - 任务执行日志：`tasks/logs/tasks/{id}.log`
  - 默认不读取日志进上下文，除非用户明确要求排查/核对

---

## 2. 运行方式（执行阶段）

触发条件：指定任务 id/任务文件路径。

前置条件（必须满足）：
- 执行前必须读取 `tasks/items/{id}.md`，且至少包含 Plan / Acceptance；缺失则返回 Plan 阶段补齐，不得执行（Files 为可选）

执行原则：
- 严格按 Plan / Acceptance 执行并记录日志；Files 仅作为目录范围参考，非硬约束
- 允许直接更新 `tasks/todos.json` 的当前任务 `status/updated_at`
- 回填 `tasks/items/{id}.md` 的 Result/Notes

### 2.1 对话输出（执行阶段）
- 输出字段：执行摘要/结果状态/验证方式/关键变更/阻塞说明

---

## 3. 工作循环（单任务生命周期）

### 3.1 认领任务
- 认领时将 `status` 改为 `doing`，写入 `claimed_by`，更新 `updated_at`

### 3.2 执行与记录
- 记录命令与关键输出到 `tasks/logs/tasks/{id}.log`
- 回填 `tasks/items/{id}.md` 的 Notes/Result

### 3.3 完成或阻塞
- 完成：`status=done`
- 阻塞：`status=blocked`
- 更新 `tasks/todos.json` 当前任务状态与 `updated_at`
- 在日志和任务文件中记录结论/阻塞原因

---

## 4. 状态机（必须遵守）

- 允许状态：`todo | doing | blocked | done`
- 允许迁移：
  - `todo → doing`
  - `doing → blocked | done`
  - `blocked → doing`
- 不变量：
  - `status: doing` ⇒ `claimed_by` 必须非空
  - `status: todo` ⇒ `claimed_by` 应为空

---

## 5. 安全与范围限制

- 任何破坏性命令（如 `rm -rf`、`git reset --hard`）必须先征询用户确认
- 变更范围优先遵循 `Files` 列表，超出必须在日志说明原因

---

## 6. 验收底线（简版）

- 符合需求（含关键边界），不引入新的 TypeScript / ESLint error，任务文件补齐“如何验证”

---

## 7. 错误处理（最小版）

- `error_type`: `TRANSIENT | PERMISSION | REQUIREMENT | ENVIRONMENT`
- 失败时：对话与任务日志记录 `error_type`、摘要与建议动作；按类型处理（TRANSIENT 重试、PERMISSION 确认、REQUIREMENT 回 Plan、ENVIRONMENT 修复）
