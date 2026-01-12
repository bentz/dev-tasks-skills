# Workflow Specification

本文件定义了在本仓库中使用 Cursor 处理任务队列时的行为规范。
AI **默认不得修改本文件内容**；仅在**用户明确授权**时才允许修改。

---

## 1. 相关文件约定

- 任务队列文件：`tasks/queue.md`
  - 记录所有任务的索引信息（id、文件路径、状态、优先级、占用 worker 等）。
  - 每一行代表一个任务，至少包含：
    - id
    - file（对应任务描述 md 的路径，例如 `tasks/items/001-useConversations.md`）
    - status（pending / claimed / in-progress / done / blocked）
    - priority（high / medium / low）
    - worker（当前占用该任务的 worker-id，可为空）

- 任务描述文件：`tasks/items/{id}.md`（后续统称为 {任务描述md}）
  - 保存单个任务的详细信息：Meta / Description / Files / Commands / Acceptance / Notes。
  - 由人类维护为主，AI 仅允许：
    - 更新 `Meta` 中的 `status` 字段；
    - 在 `Notes` 区域末尾追加简短的完成/阻塞/补充/变更记录；
    - 允许在 `Notes` 下新增小节（如“补充/变更记录”）。

- Worker 状态文件：`tasks/workers/worker-1.md`、`tasks/workers/worker-2.md`、`tasks/workers/worker-3.md`
  - 每个文件代表一个 worker 的执行状态（可理解为一个“并行执行槽位”）。
  - 包含 Meta / Current Task / Plan / Progress Log / Result 等区域。
  - AI 只允许修改当前 worker 对应的状态文件，不得改动其他 worker 的文件。

---

## 2. Worker 身份与任务选择策略

### 2.1 Worker 身份

- 本仓库最多同时开启 3 个并行 worker，对应状态文件：
  - `worker-1` → `tasks/workers/worker-1.md`
  - `worker-2` → `tasks/workers/worker-2.md`
  - `worker-3` → `tasks/workers/worker-3.md`
- 人类会在对话中通过自然语言明确指定当前 worker，例如：
  - “你现在是 worker-1，请按 workflow 执行任务”
- 如果不指定worker，则从当前可用的worker中选则一个进行，并回复到对话中，如：“我现在是worker-1，现在开始执行”
- AI 必须：
  - 将当前 worker-id 视为固定值（如 `worker-1`），只操作对应的 worker 状态文件；
  - 不得修改其他 worker 的状态文件
  - 如当前worker都被占用，则回复到对话中，如：“当前无可用worker，请确认”，并终止执行。

### 2.2 任务选择策略（单 worker 视角）

当 AI 收到“按工作流执行一个任务”的指令时，应按以下顺序处理：

1. 读取当前 worker 的状态文件（例如 `tasks/workers/worker-1.md`）：
   - 如果其中 `Current Task.status` 为 `in-progress` 或 `claimed`：
     - 优先继续当前任务；
     - 不从队列中选择新任务。
2. 如果当前 worker 没有进行中的任务：
   - 读取 `tasks/queue.md`；
   - 从中选择一条适合当前 worker 执行的任务，规则：
     - `status = pending`；
     - `worker` 字段为空（表示尚未被其他 worker 占用）；
     - `priority` 越高优先级越高（high > medium > low）；
     - 多个任务满足条件时，选择队列中靠前的一条。
3. 对选中的任务进行 **认领（claim）**：
   - 将 `tasks/queue.md` 中该任务一行的：
     - `status` 改为 `claimed`；
     - `worker` 改为当前 worker-id（例如 `worker-1`）。
   - 将当前任务信息写入对应的 worker 状态文件。

**禁止行为：**

- 不得认领已经 `status != pending` 的任务；
- 不得将当前 worker-id 写入与本次任务无关的队列行；
- 不得改动其他任务的 id / file / title 等静态字段。

---

## 3. 工作循环（单任务执行流程）

当一个任务已经被当前 worker 认领（queue 中为 `claimed`，worker 为当前 worker-id）后，AI 必须按照以下流程执行：

1. **更新当前 worker 状态文件**
   - 在 `Meta` 中：
     - `mode` 设为 `working`
     - 更新 `last_updated` 为当前时间（UTC 或本地时间字符串均可）
   - 在 `Current Task` 中：
     - 填写 `id`、`title`、`file`、`status`（设为 `in-progress`）
     - 可选填写任务在 queue 中的位置说明（如 `queue_row_hint`）

2. **生成执行计划**
   - 读取对应的 {任务描述md}：
     - 解析 `Description` / `Files` / `Commands` / `Acceptance` 等内容；
   - 在当前 worker 状态文件的 `Plan` 区域写出 3–7 个具体步骤：
     - 每步尽量说明要修改的文件、模块、函数或组件；
     - 步骤应可以被顺序执行。

3. **标记任务为 in-progress**
   - 将 `tasks/queue.md` 中该任务行的 `status` 从 `claimed` 改为 `in-progress`；
   - 保持 `worker` 字段为当前 worker-id 不变。

4. **按计划修改代码**
   - 仅在 {任务描述md} 的 `Files` 部分列出的文件，以及与其强相关的文件中修改代码；
   - 如需修改额外文件，须在 `Plan` 或 `Progress Log` 中说明原因。

5. **运行命令**
   - 根据 {任务描述md} 的 `Commands` 列表，依次运行命令（例如 `pnpm test xxx`、`pnpm lint`）；
   - 在当前 worker 状态文件的 `Progress Log` 和 `Result` 区域记录每条命令及通过/失败情况；
   - 不得运行未在 `Commands` 中声明的其他命令；
   - 允许重复执行已声明命令；允许为已声明命令附加参数（例如 `--reporter dot`），但必须记录到 `Progress Log`。

6. **验收与状态更新**
   - 若所有命令执行成功，且自检认为满足 `Acceptance` 条款：
     - 将 `tasks/queue.md` 中该任务的 `status` 改为 `done`，并删除当前行；
     - 在 {任务描述md} 的 `Meta.status` 中改为 `done`（如存在该字段）；
     - 在 {任务描述md} 的 `Notes` 区域末尾追加一条“完成说明”；
     - 在当前 worker 状态文件中：
       - `Current Task.status` 设为 `completed`；
       - 在 `Result` 区域写出本次任务的总结；
       - `Meta.mode` 设为 `waiting-review`。
   - 若命令失败或 AI 无法判断是否满足验收条件：
     - 将 `tasks/queue.md` 中该任务的 `status` 改为 `blocked`；
     - 在 {任务描述md} 的 `Meta.status` 中改为 `blocked`（如存在该字段）；
     - 在 {任务描述md} 的 `Notes` 区域末尾追加一条“阻塞原因说明”；
     - 在当前 worker 状态文件中：
       - `Current Task.status` 设为 `blocked`；
       - 在 `Result` 区域详细记录失败原因 / 日志摘要 / 建议的下一步；
       - `Meta.mode` 设为 `waiting-review`。

7. **结束本次执行**
   - 完成上述更新后，停止执行，不得自动开始下一个任务；
   - 等待人类 review 或再次指令继续。

---

## 4. Worker 状态文件的编辑规则

以任一 worker 状态文件（例如 `tasks/workers/worker-1.md`）为例，AI 仅允许如下修改：

- `Meta` 区域：
  - 允许修改：
    - `mode`
    - `last_updated`
  - 不得修改：
    - `worker_id`、文件标题等固定标识信息。

- `Current Task` 区域：
  - 允许修改：
    - `id`
    - `title`
    - `status`
    - `file`
    - `queue_row_hint`（如存在）
  - 不得删除整个区域或改为其他含义。

- `Plan` 区域：
  - 每次执行任务时可以整体覆盖重写；
  - 计划应对应当前 `Current Task.id`。

- `Progress Log` 区域：
  - 仅允许在末尾追加新条目；
  - 不得删除或修改历史记录。

- `Result` 区域：
  - 允许追加新的结果小节；
  - 不要求保留历史结果的唯一性，但不得删除已有结果。

AI 不得修改其他 worker 的状态文件，也不得在当前执行任务之外的状态文件中写入任何内容。

---

## 5. 安全与范围限制

- 命令执行：
  - 仅允许运行 {任务描述md} 的 `Commands` 中明确列出的命令；
  - 不得发明新的 shell 命令；
  - 不得执行可能删除文件、修改 git 历史或访问远程网络的命令（例如 `rm -rf`、`git reset --hard` 等）。

- 代码范围：
  - 优先修改 {任务描述md} 中 `Files` 列出文件；
  - 如需修改其他文件，必须在 `Plan` 或 `Progress Log` 中说明原因；
  - 禁止对 `.env*`、CI/CD 配置、生产部署相关脚本进行任何修改。

- 文件结构：
  - 不得重命名或移动 `tasks/queue.md`、`tasks/items/`、`tasks/workers/` 目录；
  - 不得在本 workflow 描述之外随意新增/删除任务队列行。

