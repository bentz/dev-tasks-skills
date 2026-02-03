# 归档策略（扩展）

## 适用场景
- `tasks/todos.json` 条目数或行数接近阈值
- 单个任务日志过大影响阅读与检索

## 规则（MUST）
- 只归档 `status=done` 的任务
- 保留最近 20 条已完成任务
- 归档前需征询用户确认

## 推荐做法（SHOULD）
- 归档文件命名包含任务名、任务 id 与日期
- 归档后更新 `tasks/todos.json` 的 `updated_at`

## 参考
- 详细阈值与路径规则见 `references/archive-policy.md`
