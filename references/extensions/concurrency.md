# 并发控制（扩展）

## 适用场景
- 多 agent 并行认领/更新任务
- 自动化流程同时写入 `tasks/todos.json`

## 目标
- 避免并发覆盖与状态错乱

## 规则（MUST）
- 写入前校验 `version` 或 `updated_at`
- 冲突时禁止覆盖，必须重新读取后再写
- `status=doing` 时 `claimed_by` 必须非空（与核心状态机一致）

## 推荐字段（MAY）
- `version`: number
- `last_modified_by`: string

## 冲突处理（SHOULD）
- 记录冲突到 `tasks/logs/tasks/{id}.log`
- 如冲突频繁，改为串行执行或人工协调
