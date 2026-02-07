# 归档策略（扩展）

## 触发

- `tasks/todos.json` 或任务日志接近容量上限时触发归档评估。
- 用户明确提出 `archive`/`归档` 时触发归档流程。

## 硬规则

- 只归档 `status=done` 的任务。
- 保留最近 3 条已完成任务。
- 归档前必须征询用户确认。

## 参考

- 详细阈值、命名与执行步骤见 `references/archive-policy.md`。
