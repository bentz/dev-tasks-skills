---
name: dev-task
description: "该技能用于指导“复杂需求的开发流程”，强调抽象步骤与可复用的落地方式，而不是针对某个具体需求的复盘总结。它帮助在多轮对话中澄清目标、在执行阶段组织任务与 todos、在收尾阶段补齐测试与边界处理。"
---

## 重要
- 永远使用中文进行对话

## 核心规则（MUST/SHOULD/MAY）

MUST
- 未指定任务 id 或任务文件路径 → 进入 Plan 阶段；指定了则进入 执行阶段（Runner）
- 用户未确认前不得落盘 `tasks/`
- 破坏性命令必须先征询用户确认

SHOULD
- 按需读取 references，默认不全量加载
- 执行时记录关键命令与结论到任务日志
- 变更范围优先遵循 `Files` 列表，超出需说明原因

MAY
- 触发并发或归档场景时加载扩展文档

## 读取策略（节省 token）

- Plan 阶段：读取 `references/plan-workflow.md`
- 执行阶段：读取 `references/task-workflow.md`
- 模板/策略/扩展：仅在需要时读取（并发/归档场景再读 `references/extensions/*.md`）

## 阶段说明

- Plan 阶段对话仅输出简化方案；详细计划仅在“创建/落盘/立项”等落盘后写入 `tasks/items/{id}.md`
- Plan/执行的详细流程、输出契约与触发词见对应 workflow 文档
- 用户明确只分析/建议时：不创建任务、不写 `tasks/`
