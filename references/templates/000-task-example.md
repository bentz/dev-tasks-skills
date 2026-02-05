# 任务模板示例

> 用于创建 `tasks/items/{id}.md` 的参考结构。按需删改字段即可。
> Plan 阶段对话仅输出简化方案，本模板用于“创建/落盘”后的详细计划文件。

## Meta

- id [必填]: 001-TASK
- status [必填]: todo # todo | doing | blocked | done
- priority [可选]: medium
- module [可选]: node/main/xxx
- claimed_by [可选]: ""
- log_file [必填]: tasks/logs/tasks/001-TASK.log

## Description

- 背景 [必填]: 简述现状/痛点
- 目标 [必填]: 清晰可验证
- 范围 [必填]: 本次会修改/影响的模块
- 非范围 [可选]: 明确不做的内容
- 约束/风险 [可选]: 版本、兼容性、性能、边界

## Constraints

- [可选] 关键约束（变更需记录原因）

## Plan

- [必填] 1) 改动的文件/模块/函数
- [必填] 2) 边界与验证方式

## Files

- [可选] 可能涉及的目录范围（非硬约束）
- path/to/dir-a/
- path/to/dir-b/

## Acceptance

- [必填] 不引入新的 TypeScript / ESLint error
- [必填] 功能行为与需求一致（含正常与边界）
- [可选] 关键路径具备测试覆盖

## Notes

- 2026-01-12: 任务创建，待确认。

## Result

- [必填] 完成/阻塞说明（做了什么、结论、如何验证、已知风险/下一步）
