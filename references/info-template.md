# `tasks/info.md` 模板（项目索引 + 可选覆盖配置）

`tasks/info.md` 用于沉淀项目级信息与最简索引，并可在其中覆盖默认的 memory-policy。

建议结构：

```yaml
updated_at: "2026-01-30 10:00"

# 可选：覆盖默认 memory-policy（不写则使用 skill 默认值）
memory_policy_override:
  max_lines:
    todos: 200
    info: 200

# 最简索引（YAML 风格，不要 table）
index:
  - id: 001-TASK
    title: "实现登录页 i18n"
    status: doing
    task_file: tasks/items/001-TASK.md
    log_file: tasks/logs/tasks/001-TASK.md
    note: "依赖 server: /auth/loginStatus 接口返回语言列表"

facts:
  - "Server 入口：apps/server/src/index.ts"
  - "Plugin 入口：apps/plugin/src"
  - "Admin 入口：apps/admin/src/App.tsx"
```
