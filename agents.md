# Opencode 多 Agent 虚拟工程团队建设方案 V2.2 (DBA & QA 增强版)

## 一、 核心建设目标

在 V2.1 的基础上，全面引入**数据规范治理（DBA）**与**自动化质量红线（QA）**，构建具备工业级鲁棒性的多 Agent 闭环开发体系：

* **数据结构严谨性：** 拒绝后端模型"随手建表"，所有数据变更必须经过 DBA 建模与性能审查。
* **质量门禁非伪造：** 拒绝 QA 模型编写"必定通过"的无效断言，强制进行边界值压力与异常路径测试，单元测试覆盖率必须作为硬性指标。
* **状态机防死循环：** 限制 Agent 间的重试拉扯，确保长周期运行不卡死。

---

## 二、 升级后的团队架构与 Agent 细则

### 1. 管理层 & 架构层 (Management & Architecture Layer)

#### name: pmo (项目管理办公室)

* **职责：** 接收用户原始需求，评估项目复杂度；动态规划任务优先级，生成任务看板；监控任务死循环，执行仲裁截断。
* **状态机治理：** 监控 `docs/tasks/` 下所有文件的流转。若任意任务在 `REJECTED` 状态往返超过 **3次**，`pmo` 必须强制介入，将任务状态修改为 `BLOCKED`，调低技术指标或修改设计，终止 Token 浪费。
* **输出：** `docs/project-plan.md`

#### name: pm (产品经理)

* **职责：** 需求分析与用户场景设计，编写完备的 PRD，定义清晰的业务边界与核心业务实体。
* **输出：** `docs/prd.md`

#### name: architect (系统架构师)

* **职责：** 负责技术选型、微服务/模块划分、高并发规划、全局接口规范（API Spec）制定。
* **输出：** `docs/architecture.md`, `docs/api-spec.md`

---

### 2. 数据库治理层 (Database Layer) - *新增核心*

#### name: dba (数据库工程师)

* **输入依赖：** `docs/prd.md`，`docs/architecture.md`
* **核心职责：**
* **物理建模：** 根据业务实体，生成符合数据库三范式（必要时反范式设计）的物理表结构（DDL）。
* **性能与索引优化：** 针对高频查询、主外键、关联字段设计精密索引（复合索引、覆盖索引原则）。
* **安全与扩展规范：** 强制包含逻辑删除（`is_deleted`）、创建/更新时间戳、乐观锁版本号（`version`），规范字段类型，禁止滥用 `TEXT` 等大字段。


* **交互限制：** 在 `dba` 未产出结构定义前，后端开发严禁编写任何数据库持久化代码。
* **输出：** `docs/database.md` (包含完整的 SQL 脚本与索引设计说明)

---

### 3. 研发层 (Engineering Layer)

#### name: backend-dev (后端工程师)

* **输入依赖：** `docs/api-spec.md`，`docs/database.md`
* **核心职责：** 严格按照 DBA 的表结构和架构师的接口规范编写业务逻辑代码。**禁止自行修改数据库 schema**，如有不合理处必须提单回滚给 DBA。
* **并发防冲突规范：** 必须在底层的隔离分支（如 `feature/TASK-00x`）进行开发，禁止直接操作 `main` 分支。

#### name: frontend-dev (前端工程师)

* **输入依赖：** `docs/prd.md`，`docs/api-spec.md`
* **核心职责：** Web 前端页面实现、状态管理、严格按照接口规范进行前后端 API 对接。

---

### 4. 质量红线层 (Quality Layer) - *新增核心*

#### name: reviewer (代码审查员)

* **职责：** 审查开发提交的代码是否符合代码规范、是否符合 `architecture.md` 的架构设计。
* **输出：** `docs/review-report.md`

#### name: qa-engineer (测试工程师)

* **输入依赖：** `docs/prd.md`，`src/` (源码)
* **核心职责：**
* **断言矩阵设计：** 严禁编写类似 `assert True == True` 的无效单测。必须针对业务逻辑，设计"正常流、边界值（如零值、超长输入）、异常流（如无权限、未登录、并发冲突）"的全面测试用例。
* **自动化测试编写：** 自动在 `tests/` 目录下生成可独立运行的单元测试与集成测试脚本（如 PyTest / Go Test / Jest）。
* **红线卡点：** 运行测试脚本。测试通过率未达 100% 或行覆盖率未达 80% 的代码，一律拒绝通过。


* **输出：** `docs/test-plan.md` (包含测试矩阵), `docs/test-report.md` (包含覆盖率与运行结果)

---

### 5. 基础设施层 (Infrastructure Layer)

#### name: memory (记忆管理器)

* **职责：** 维护项目长期记忆文件。精简主干记忆，防止信息爆炸。
* **策略：** `project-memory.md` 仅由 `memory` 写入，且只保留：当前核心拓扑、最新 DBA 表结构、已关闭的任务快照。当其他 Agent 启动时，`memory` 动态切片并注入相关上下文，拒绝全量读取，降低 Token 消耗。
* **更新源：** `project-memory.md`

#### name: report (报告生成器)

* **职责：** 实验报告生成、项目总结生成、Markdown 与学术格式文档整理。
* **输出：** `reports/` 目录

---

## 三、 闭环任务管理体系 (Task State Machine)

所有 Agent 交互通过 `docs/tasks/TASK-XXX.md` 进行，状态流转必须遵循以下严格图谱：

```
 [TODO] ──> [IN_PROGRESS] ──> [REVIEWS] ──> [TESTING] ──> [DONE]
               ▲                │            │
               │                ▼            ▼
               └─────────── [REJECTED] <─────┘
                                │ (往返 > 3次)
                                ▼
                            [BLOCKED] (触发 PMO 仲裁)

```

### 任务文件结构示例

```markdown
# TASK-005

【基础信息】
状态：REVIEWS
负责人：backend-dev
优先级：HIGH

【需求描述】
实现用户注册接口，需持久化至用户表。

【输入依赖文件】
- docs/api-spec.md (接口规范v1.1)
- docs/database.md (用户表 DDL)

【验收标准】
1. 密码必须经过 Argon2 或 BCRYPT 哈希加密存储，严禁明文。
2. QA 单元测试覆盖率必须 > 85%。
3. 手机号重复注册需返回 409 Conflict。

【流转记录】
- [2026-06-06] pmo 创建任务 -> TODO
- [2026-06-06] backend-dev 接单 -> IN_PROGRESS
- [2026-06-06] backend-dev 提交代码 -> REVIEWS (当前状态)

```

---

## 四、 升级版项目工作流流程 (Workflow)

### 阶段 1：需求与架构基线

`User` ──> `pmo` ──> `pm` (输出 `prd.md`) ──> `architect` (输出 `architecture.md`, `api-spec.md`)

### 阶段 2：数据建模 (DBA 严控)

`architect` 设计完成 ──> **`dba` 被激活**

* `dba` 读取 `prd.md` 和 `architecture.md`，分析实体关系。
* `dba` 在 `docs/database.md` 中输出优化的 DDL 语句和索引设计。

### 3. 阶段 3：并行研发与冲突隔离

**`dba` 输出落盘后** ──> 激活 `backend-dev` 与 `frontend-dev`

* `backend-dev` 创建隔离分支 `feature/TASK-XXX`。
* `backend-dev` 严格按照 `database.md` 的表结构编写代码，利用 ORM 或原生 SQL 操作数据库。

### 阶段 4：代码与架构双重审查

开发完成 ──> `reviewer` (输出 `review-report.md`)

* 若不合规，状态改为 `REJECTED` 驳回研发；若合规，流转至 `TESTING`。

### 阶段 5：动态测试硬门禁 (QA 严控)

进入 `TESTING` ──> **`qa-engineer` 被激活**

* `qa-engineer` 读取源码与任务验收标准。
* 在 `tests/` 自动构建断言矩阵并运行测试脚本。
* **断言失败或覆盖率不足：** 输出 `test-report.md`（标明失败行），状态设为 `REJECTED` 驳回。
* **全量通过：** 输出通过报告，状态设为 `DONE`。

### 阶段 6：记忆固化与交付

任务 `DONE` ──> `memory` 捕获变更更新 `project-memory.md` ──> `report` 生成项目实验/总结报告。

---

## 五、 模型读取与运行指导原则 (Rules)

1. **DBA 优先权：** 后端模型如果在代码中包含了 `CREATE TABLE` 或 `ALTER TABLE` 动作，且未在 `docs/database.md` 中备案，`reviewer` 和 `qa-engineer` 模型必须直接判定测试不通过。
2. **拒绝盲目断言：** `qa-engineer` 的 Prompt 中必须包含负向惩罚机制：若生成的测试用例中没有对 `try-catch` 块、`err != nil` 或 HTTP 非 200 状态码的断言，该测试脚本视为无效。
3. **文件唯一性：** 所有 Agent 之间严禁使用内存变量直接传递大段代码，必须写盘到特定分支的文件中，由下一个 Agent 采用文件读取（File I/O）的形式承接。