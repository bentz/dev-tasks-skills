# 任务模板示例

> 用于创建 `tasks/items/{id}.md` 的参考结构。按需删改字段即可。

## Meta

- id: IM-XXX
- status: pending
- priority: medium
- module: node/main/xxx

## Description

- 背景：简述现状/痛点
- 目标：清晰可验证
- 范围：本次会修改/影响的模块
- 非范围：明确不做的内容
- 约束/风险：版本、兼容性、性能、边界

## Files

- path/to/file-a.ts
- path/to/file-b.ts

## Commands

- pnpm lint
- pnpm test:node -- tests/node/main/xxx.test.ts

## Acceptance

- 不引入新的 TypeScript / ESLint error
- 功能行为与需求一致（含正常与边界）
- 关键路径具备测试覆盖

## Notes

- 2026-01-12: 任务创建，待确认。
