# 任务模板示例

> 用于创建 `tasks/items/{id}.md` 的参考结构。按需删改字段即可。
> Plan 阶段对话仅输出简化方案，本模板用于“创建/落盘”后的详细计划文件。

## Meta

- id: 001-TASK
- status: todo # todo | doing | blocked | done（与 tasks/todos.json 镜像，用于反查）
- priority: medium
- module: node/main/xxx
- claimed_by: "" # 可选：agent id（反查用；以 tasks/todos.json 为准）
- log_file: tasks/logs/tasks/001-TASK.log

## Description

- 背景：简述现状/痛点
- 目标：清晰可验证
- 范围：本次会修改/影响的模块
- 非范围：明确不做的内容
- 约束/风险：版本、兼容性、性能、边界

## Constraints

- （可选：把关键约束单独列出来，执行中如有变更需记录原因）

## Plan

- 1) （写清楚要改哪些文件/模块/函数）
- 2) （写清楚边界与验证方式）

## Files

- （可选：可能涉及的目录范围，非硬约束）
- path/to/dir-a/
- path/to/dir-b/

## Acceptance

- 不引入新的 TypeScript / ESLint error
- 功能行为与需求一致（含正常与边界）
- 关键路径具备测试覆盖

## Notes

- 2026-01-12: 任务创建，待确认。

## Result

- （完成/阻塞时填写：做了什么、结论、如何验证、已知风险/下一步）
