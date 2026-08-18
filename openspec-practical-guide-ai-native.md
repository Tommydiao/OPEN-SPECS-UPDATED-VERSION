# OpenSpec AI-Native 实战指南

> 本文是对 `docs/openspec-practical-guide.md` 的 AI-Native 协作升级稿。它保留 OpenSpec 的 Proposal、Design、Delta Specs、Tasks、Validate 与 Archive 机制，并增加 Intent、项目状态、风险分级、决策日志、自动修复循环、独立审查与交付证据。
>
> 本文所述 `intent.md`、`state.yaml`、`risk.yaml`、`decision-log.md`、`evidence/` 等内容属于团队扩展约定，不代表 OpenSpec 官方强制目录。团队可以在不破坏现有 OpenSpec 兼容性的前提下逐步采用。

## 1. 引言

OpenSpec 是一种以规格（Spec）为中心的工程方法与工具链，通过结构化文档连接需求、设计、代码和测试，降低“产品理解、开发实现、测试验收”之间的偏差。

传统的 OpenSpec 落地通常仍由人驱动：

```
人提出需求
→ 人触发 Explore
→ 人生成或修改 Proposal / Design / Specs / Tasks
→ 人触发 Apply
→ AI 编写代码
→ 人触发 Verify
→ 人决定 Archive

```

这种方式已经比“PRD + 聊天记录 + AI 临时写代码”更可靠，但 AI 仍主要是被调用的执行者。项目下一步做什么、失败后如何调整、什么时候需要审批、什么证据足以证明完成，仍然高度依赖人逐步推动。

本文提出一种 **AI-Native OpenSpec 协作模式**：

> **Human-Governed, AI-Operated, Evidence-Gated**
> 人负责目标、边界和责任；AI 负责探索、规划、执行与修正；工程证据决定流程是否可以继续。

本文结合“经销商登录后提醒即将超过缴费期限的 ARC Basic 待支付订单”这一完整案例，演示从 Intent、AI 探索、规格生成、风险审批、自动实现、独立审查，到验收、归档的全流程。

核心目标不是让 AI 取代产品经理或开发人员，而是重新分配职责：

- **产品经理**：定义业务目标、用户价值、业务规则、成功标准与不可由 AI 决定的事项。
- **开发人员**：定义技术宪法、架构边界、工具权限、风险规则和生产责任。
- **AI Agent**：持续读取项目状态，自主规划、实现、测试、修复、重规划并整理交付证据。
- **测试与 CI**：提供可重复、可审计的质量证据。
- **人类审批者**：只处理业务歧义、高风险技术变更和最终发布决策。

---

## 2. 方法论与 AI-Native OpenSpec 核心概念

AI-Native OpenSpec 不是取消 Spec，也不是让 AI 无限制自主开发，而是把 OpenSpec 从“人工操作的流程文档”升级为“AI 持续维护的工程合同”。

### 2.1 核心哲学

AI-Native OpenSpec 遵循以下原则：

- **目标优先于任务（Goal over Tasks）**：人提供目标、约束和成功标准，AI 根据代码库实际情况动态形成任务。
- **治理而非微操（Governance not Micromanagement）**：人定义 AI 能做什么、不能做什么，以及何时必须暂停。
- **运行而非调用（Operated not Prompted）**：AI 根据项目状态自动进入下一动作，而不是等待人逐条输入命令。
- **证据而非自述（Evidence not Claims）**：AI 不能仅凭“已完成”宣告交付，必须提供测试、截图、日志、Diff、性能与安全检查结果。
- **动态而非静态计划（Dynamic not Static）**：`tasks.md` 是可调整的运行计划。实现中发现新事实时，AI 必须更新 Design、Specs、Tasks 和风险判断。
- **风险决定自主权（Risk-Proportional Autonomy）**：低风险改动可以自动执行；权限、支付、数据删除、不可逆迁移等高风险动作必须人工批准。
- **独立审查（Independent Review）**：实现 Agent 不能成为唯一裁判。审查应由独立上下文或独立 Agent 对照 Intent、Spec、Diff 与证据完成。
- **存量优先（Brownfield-first）**：AI 必须先理解既有代码、主 Spec、测试和历史决策，再提出修改方案。

### 2.2 目录结构与多层事实来源

传统 OpenSpec 主要区分主规范 `openspec/specs/` 与进行中变更 `openspec/changes/`。AI-Native 模式在此基础上增加“人类意图、运行状态、决策与证据”四类资产。

```
project/
├── AGENTS.md                         <-- 项目宪法：AI 长期遵守的工程规则
├── openspec/
│   ├── config.yaml                  <-- OpenSpec 项目配置
│   ├── specs/                       <-- 已发布系统行为的事实来源
│   └── changes/
│       └── add-arc-payment-reminder/
│           ├── .openspec.yaml       <-- OpenSpec 元数据
│           ├── intent.md            <-- 人类拥有：目标、约束、成功标准
│           ├── proposal.md          <-- AI 起草：Why & What
│           ├── design.md            <-- AI 起草：How
│           ├── specs/               <-- AI 起草：Delta Requirements
│           ├── tasks.md             <-- AI 维护：动态实施计划
│           ├── state.yaml           <-- 系统维护：当前项目状态
│           ├── risk.yaml            <-- AI 评估、人审批：风险与权限
│           ├── decision-log.md      <-- AI 记录：方案、理由、审批结果
│           └── evidence/
│               ├── test-report.md
│               ├── review-report.md
│               ├── security-report.md
│               ├── performance-report.md
│               └── screenshots/
└── src/

```

各资产的所有权不同：

| 资产主要所有者AI 是否可自主修改作用 |                 |                 |                            |
| ------------------- | --------------- | --------------- | -------------------------- |
| `intent.md`         | 产品经理 / 业务负责人    | 不可修改业务结论，只能提出建议 | 定义目标、规则、成功标准和非目标           |
| `AGENTS.md`         | 开发负责人           | 不可绕过            | 定义架构、测试、权限和安全边界            |
| `proposal.md`       | AI 起草，产品审核      | 可在范围内更新         | 描述 Why、What、Scope 与 Impact |
| `design.md`         | AI 起草，开发审核      | 可在低风险范围内更新      | 描述技术方案、数据流和权衡              |
| `specs/`            | AI 起草，产品与开发共同审核 | 可根据已批准决策更新      | 定义系统必须满足的行为                |
| `tasks.md`          | AI              | 可动态修改           | 记录当前可执行计划，不作为永久事实          |
| `state.yaml`        | Orchestrator    | 自动修改            | 决定下一步动作                    |
| `risk.yaml`         | AI 评估，人审批       | 不可降低已确认风险等级     | 控制自主权限                     |
| `decision-log.md`   | AI 记录，人确认关键决策   | 只能追加，不应静默覆盖     | 保存决策理由和责任链                 |
| `evidence/`         | CI、测试和审查系统      | 自动生成            | 证明实现是否满足交付条件               |

这里不再只有一个 Source of Truth，而是四个互补事实来源：

1. **Intent Source of Truth**：人真正要解决的问题。
2. **Behavior Source of Truth**：系统当前和目标行为的 Spec。
3. **Execution Source of Truth**：当前状态、任务与风险。
4. **Evidence Source of Truth**：测试、审查和验收结果。

### 2.3 状态驱动工作流与 AI 协作指令

OpenSpec 的 CLI 与 Slash Commands 仍然可以使用，但在 AI-Native 模式中，命令不再是项目的中心，**项目状态才是中心**。

推荐状态如下：

```
INTAKE
  ↓
AI_EXPLORING
  ↓
NEEDS_DECISION ──→ AI_EXPLORING
  ↓
SPEC_READY
  ↓
READY_TO_BUILD
  ↓
AI_BUILDING
  ↓
AI_VERIFYING
  ↓
NEEDS_REVIEW
  ↓
READY_TO_MERGE
  ↓
MERGED
  ↓
ARCHIVED

```

状态转换由事件触发：

| 当前状态触发条件下一状态主要执行者 |                                 |                  |                  |
| ----------------- | ------------------------------- | ---------------- | ---------------- |
| `INTAKE`          | Issue 或 Intent 标记为 `ai-ready`   | `AI_EXPLORING`   | Orchestrator     |
| `AI_EXPLORING`    | 发现关键业务歧义                        | `NEEDS_DECISION` | AI               |
| `NEEDS_DECISION`  | 产品或开发给出决定                       | `AI_EXPLORING`   | 人                |
| `AI_EXPLORING`    | Proposal、Design、Specs、Tasks 已生成 | `SPEC_READY`     | AI               |
| `SPEC_READY`      | 必要审批通过                          | `READY_TO_BUILD` | 人 / 策略引擎         |
| `READY_TO_BUILD`  | 创建隔离分支或 Worktree                | `AI_BUILDING`    | AI               |
| `AI_BUILDING`     | 实现批次完成                          | `AI_VERIFYING`   | AI               |
| `AI_VERIFYING`    | 所有自动门禁通过                        | `NEEDS_REVIEW`   | CI / AI          |
| `NEEDS_REVIEW`    | 独立审查通过                          | `READY_TO_MERGE` | Review Agent / 人 |
| `READY_TO_MERGE`  | 最终审批通过                          | `MERGED`         | 开发负责人            |
| `MERGED`          | 主分支验证通过                         | `ARCHIVED`       | 自动化流程            |

OpenSpec 命令可以作为底层动作映射：

```
AI_EXPLORING   -> /opsx:explore
SPEC_READY     -> /opsx:propose 或 /opsx:ff
AI_BUILDING    -> /opsx:apply
AI_VERIFYING   -> /opsx:verify
ARCHIVED       -> /opsx:archive

```

关键区别是：人不需要每天告诉 AI “现在 Explore、现在 Apply、现在 Verify”。Orchestrator 根据 `state.yaml` 和门禁结果自动选择下一动作。

### 2.4 验证、证据与可观测性

AI-Native 项目必须同时验证五个层次：

1. **结构验证**：OpenSpec 文件格式、Delta Header、Requirement 与 Scenario 是否完整。
2. **行为验证**：代码是否满足每条 Requirement 和 Scenario。
3. **目标验证**：实现是否真的满足 `intent.md` 中的业务目标，而不是只满足局部测试。
4. **风险验证**：权限、安全、隐私、迁移、支付等风险是否经过正确审批。
5. **过程验证**：AI 做过什么、为什么这么做、调用了哪些工具、失败后如何修复，是否可以追踪。

每个交付必须生成 Evidence Pack：

```
Evidence Pack
├── OpenSpec validate 结果
├── Type Check / Lint
├── Unit Test
├── Integration Test
├── E2E Test
├── Security / Permission Test
├── Performance Baseline
├── UI Screenshots
├── Review Report
├── Decision Log
└── Rollback Plan

```

AI 的“完成”定义必须是：

```
所有必需 Scenario 有实现
+
所有必需 Scenario 有验证
+
所有质量门禁通过
+
所有高风险动作有审批
+
独立审查通过
+
交付证据完整

```

---

## 3. AI-Native 迭代流程总览

AI-Native OpenSpec 仍然遵循规格优先，但把“人工逐步驱动”改为“Intent 启动、状态推进、证据放行”。

### 3.1 目标与边界（Intent）

在任何探索、设计或编码开始前，产品经理先提供 `intent.md`。它不需要拆前端、后端或数据库任务，而是回答：

- 为什么做？
- 为谁做？
- 当前问题是什么？
- 期望业务结果是什么？
- 哪些规则不可改变？
- 哪些内容明确不做？
- 什么结果算成功？
- 哪些事项必须由人决定？

产品输入的是“目标卡”，不是“技术任务书”。

### 3.2 变更初始化

已有 OpenSpec 项目可以继续使用原有初始化方式：

```
openspec init --tools none

```

AI Orchestrator 接收到 `ai-ready` 事件后创建变更：

```
openspec/changes/add-arc-payment-reminder/

```

并初始化：

```
intent.md
proposal.md
design.md
specs/
tasks.md
state.yaml
risk.yaml
decision-log.md
evidence/

```

#### 3.2.1 `intent.md`（人类意图）

下面是本案例的 Intent 示例：

```
# Intent: ARC Basic 即将到期订单提醒

## Business Goal

降低 ARC Basic 待支付订单超过缴费时限而失效的风险。

## Target Users

BMW 经销商系统中负责订单处理和缴费的用户。

## Current Problem

用户登录系统后，无法及时发现即将超过缴费期限的待支付订单，
可能导致订单失效并影响客户权益。

## Expected Outcome

用户登录后可以立即发现当前经销商需要尽快处理的订单，
并能够进入相关订单页面继续处理。

## Business Rules

- 仅查询当前登录用户所属经销商的订单。
- 仅统计 ARC Basic 待支付订单。
- 剩余缴费时间小于或等于 30 天时提醒。
- 已支付、已取消、已失效订单不提醒。
- 本功能不得修改订单状态或缴费截止时间。

## Success Criteria

- 剩余 29 天和 30 天的待支付订单会触发提醒。
- 剩余 31 天的待支付订单不会触发提醒。
- 已支付订单不会触发提醒。
- 当前经销商不能看到其他经销商的订单信息。
- 登录主流程没有明显性能退化。

## Out of Scope

- 短信、微信和邮件提醒。
- 自动缴费。
- 修改订单有效期。
- 新增跨经销商管理功能。

## Human Decisions Required

- 同一登录会话中，关闭弹窗后是否再次展示。
- 多笔订单是展示总数、摘要还是完整明细。
- 是否允许用户选择“今日不再提醒”。

## Design Reference

- Figma: <node-url>

```

AI 不得擅自把 `Out of Scope` 内容加入实现，也不得静默改变 Business Rules。若 AI 认为 Intent 有问题，应提出建议并等待明确决定。

#### 3.2.2 其他核心文件

- `proposal.md`：AI 根据 Intent 与代码库探索结果描述 Why、What、Scope 与 Impact。
- `design.md`：AI 提出技术方案、架构图、数据流、备选方案与权衡。
- `specs/`：将业务规则转换为 SHALL / MUST Requirements 与 Given / When / Then Scenarios。
- `tasks.md`：AI 维护的动态实施计划。
- `risk.yaml`：风险等级、受限工具和审批要求。
- `state.yaml`：当前状态、下一动作、重试次数和阻塞原因。
- `decision-log.md`：记录关键问题、方案、推荐、决定人和决定时间。
- `evidence/`：保存实现和验收证据。

### 3.3 AI 探索与规划

接收 Intent 后，AI 首先进入 Explore，而不是直接写代码。

AI 应主动读取：

- `AGENTS.md` 与 `openspec/config.yaml`；
- 现有 `openspec/specs/`；
- 登录流程与首页加载逻辑；
- 订单模型、状态枚举和截止日期计算方式；
- 经销商权限与数据隔离逻辑；
- 现有 API、组件和测试；
- Figma 或页面截图；
- 历史相似变更与 Decision Log；
- CI、部署和回滚规则。

探索输出至少包含：

```
## AI 对需求的理解

## 现有系统发现

## 受影响的 Capabilities

## 关键假设

## 业务歧义

## 技术风险

## 可选方案

## AI 推荐方案

## 预计验证方式

## 需要人工决定的问题

```

AI 不应把所有问题都上报给人。以下问题通常由 AI 自主决定：

- 变量名和普通函数拆分；
- 已有架构规则明确覆盖的目录选择；
- 测试文件命名；
- 低风险重构；
- 格式、Lint 和类型问题。

以下问题必须交给人：

- 会改变用户行为的业务歧义；
- Scope 扩大；
- 权限、支付、敏感数据和合规；
- 不可逆数据库变更；
- 重大架构偏离；
- 无法同时满足的业务规则；
- 成本、性能或上线风险显著增加。

### 3.4 AI 自主实现循环

规格获准后，AI 进入小步执行循环：

```
Observe
  ↓
Plan
  ↓
Act
  ↓
Test
  ↓
Evaluate
  ↓
Reflect
  ↓
Replan

```

每次只执行一个可验证批次，例如：

```
Batch 1：后端查询与经销商权限测试
Batch 2：提醒 API 与集成测试
Batch 3：首页弹窗与交互
Batch 4：E2E、视觉与性能验证

```

批次执行规则：

1. 读取最新 Intent、Spec、Design、Tasks 和 Decision Log。
2. 选择下一项未完成任务。
3. 在隔离分支或 Worktree 中修改代码。
4. 运行与本批次最相关的最小测试集。
5. 通过后再运行更大范围回归测试。
6. 如果失败，分析是代码问题、测试问题、环境问题还是规格问题。
7. 对可恢复问题自动修复。
8. 发现新事实时更新 Design、Tasks 和风险评估。
9. 若触发停止条件，进入 `NEEDS_DECISION` 或 `BLOCKED`。
10. 每个批次完成后写入 Evidence 和 Decision Log。

`tasks.md` 不是不可改变的承诺。例如 AI 发现当前数据模型没有可靠的缴费截止时间字段，应当：

```
暂停当前批次
→ 更新系统发现
→ 调整 Design
→ 增加数据迁移任务
→ 重新评估风险
→ 判断是否需要开发审批
→ 获批后继续

```

而不是在错误假设上继续写代码。

### 3.5 风险分级与证据门禁

建议将变更分成三级：

| 风险级别示例AI 权限 |                                    |                         |
| ----------- | ---------------------------------- | ----------------------- |
| **Green**   | 文案、样式、非关键组件、小范围测试补充                | AI 可自主实现、测试并创建 Draft PR |
| **Yellow**  | 新接口、普通业务逻辑、可逆表字段、依赖升级              | AI 可实现；合并前必须由开发审核       |
| **Red**     | 权限、支付、敏感数据、删除数据、不可逆 Migration、生产配置 | 执行前必须审批；完成后再次审批         |

工具权限也应按风险控制：

```
读取仓库                -> 自动允许
修改功能分支            -> 自动允许
运行测试                -> 自动允许
创建 Draft PR           -> 自动允许
修改测试断言            -> 允许，但必须解释原因并接受独立审查
执行可逆本地 Migration -> Yellow 审批
删除数据                -> Red 审批
访问生产数据库          -> 禁止
合并主分支              -> 人工审批
发布生产环境            -> 人工审批

```

AI 不得通过删除测试、降低断言、跳过权限校验等方式让门禁“变绿”。

### 3.6 独立审查、归档与沉淀

实现 Agent 完成后，不得直接宣布交付。独立 Review Agent 应使用干净上下文读取：

- `intent.md`；
- 主 Spec 与 Delta Specs；
- Design 与 Decision Log；
- 代码 Diff；
- 测试和 Evidence；
- 风险与审批记录。

审查结果分为：

```
PASS
PASS_WITH_NOTES
CHANGES_REQUIRED
BLOCKED

```

所有门禁通过后：

1. AI 创建或更新 Draft PR。
2. 产品根据 Scenario、截图和用户路径进行业务验收。
3. 开发根据 Diff、架构、安全、性能和回滚方案进行技术验收。
4. 合并后在主分支再次运行关键验证。
5. 执行 `/opsx:archive` 或 `openspec archive`。
6. Delta Spec 合并到 `openspec/specs/`。
7. Decision Log 与 Evidence 随变更一起归档。

---

## 4. 案例背景：ARC Basic 即将到期订单提醒

### 4.1 核心域与协作上下文

本案例涉及六个业务与工程上下文：

- **Authentication（认证）**：用户登录与会话建立。
- **Dealer Scope（经销商权限）**：限制用户只能访问所属经销商数据。
- **ARC Basic Order（订单）**：订单状态、产品类型、缴费截止时间。
- **Reminder（提醒）**：判断是否需要提醒及返回提醒摘要。
- **Dashboard UI（首页界面）**：登录后弹窗、关闭和跳转交互。
- **Audit & Evidence（审计与证据）**：记录触发逻辑、测试结果和审批。

协作角色包括：

| 角色核心职责             |                                  |
| ------------------ | -------------------------------- |
| 产品经理               | Intent、业务规则、成功标准、业务验收            |
| 开发负责人              | AGENTS.md、架构规则、风险边界、技术审批         |
| Orchestrator Agent | 读取状态、规划下一步、协调执行与审查               |
| Builder Agent      | 修改代码、生成测试、自动修复                   |
| Review Agent       | 独立检查 Intent、Spec、Diff 和 Evidence |
| CI / Test Runner   | 提供可重复的客观验证结果                     |

初期可以由一个 Agent 通过不同模式承担 Orchestrator、Builder 与 Reviewer，但 Reviewer 必须使用独立上下文，避免“自己实现、自己证明正确”。

### 4.2 简化假设

为了聚焦 AI-Native 协作，本案例做以下假设：

- 系统已经存在登录页、首页、订单列表和经销商权限模型。
- 订单中已经存在可计算剩余缴费天数的数据。
- AI 只在隔离分支或 Worktree 中修改代码。
- AI 不持有生产环境写权限。
- Figma 设计可以读取，或至少提供截图和尺寸说明。
- CI 已能运行 Type Check、Unit Test、Integration Test 与 E2E。
- 产品与开发可以在 Issue 或 PR 中完成审批。

如果探索结果证明某个假设不成立，AI 必须更新 Design、Tasks 和 Risk，而不是继续假设。

### 4.3 非功能性目标

- **数据隔离**：任何响应都不得包含其他经销商的订单信息。
- **性能**：提醒检查不得明显拖慢登录和首页首屏加载。
- **可靠性**：提醒服务异常时不得阻塞用户登录。
- **质量**：所有 P0/P1 Scenario 必须有自动化测试。
- **可观测性**：应能判断提醒查询是否成功、耗时多少、是否降级。
- **可回滚性**：功能可以通过 Feature Flag 或代码回滚关闭。
- **可审计性**：关键业务和技术决策必须记录在 Decision Log。

---

## 5. 架构设计

### 5.1 分层架构

AI-Native 协作采用五层架构：

| 层级主要资产职责依赖方向  |                                      |                 |                   |
| ------------- | ------------------------------------ | --------------- | ----------------- |
| **人类治理层**     | `intent.md`、审批记录                     | 定义目标、业务边界、风险责任  | -> Contract       |
| **合同与上下文层**   | `AGENTS.md`、`openspec/specs/`、Design | 定义系统行为、架构和约束    | -> Orchestration  |
| **Agent 编排层** | `state.yaml`、Planner、Orchestrator    | 选择动作、动态规划、停止与升级 | -> Tools          |
| **工具执行层**     | Git、测试、浏览器、数据库沙箱                     | 修改、运行、检查和收集结果   | -> Evidence       |
| **证据与交付层**    | CI、`evidence/`、PR                    | 判断是否达到交付标准      | -> Human Approval |

依赖方向必须单向：

```
Human Intent
    ↓
Spec & Constraints
    ↓
Agent Decisions
    ↓
Tool Actions
    ↓
Evidence
    ↓
Human Approval

```

AI 不能反向通过修改 Intent 来合理化已经写出的代码，也不能通过修改质量门禁来掩盖实现问题。

### 5.2 边界与依赖规则

- **Intent 不可静默修改**：AI 只能提出变更建议，由产品明确确认。
- **Spec 必须可追踪**：每条 Requirement 必须映射到实现和验证。
- **Builder 与 Reviewer 隔离**：审查阶段不得仅依赖 Builder 的总结。
- **最小权限**：Agent 只获得完成当前任务所需的最小工具权限。
- **生产隔离**：默认禁止写生产数据库、修改生产 Secret 和直接部署。
- **范围控制**：AI 不得为了“顺便优化”扩大 Scope。
- **测试保护**：修改既有测试必须说明是需求变化、错误修复还是测试重构。
- **失败有上限**：连续自动修复达到阈值后必须停止，不能无限消耗资源。
- **高风险双门禁**：Red 变更在执行前和合并前都需要人工批准。

### 5.3 数据流概览

以“ARC Basic 即将到期订单提醒”为例：

```
flowchart TD
    I[产品提交 intent.md] --> O[Orchestrator 读取项目上下文]
    O --> E[Explore 代码 Spec 测试与设计]
    E --> Q{是否存在关键歧义或高风险}
    Q -- 是 --> H[请求产品或开发决策]
    H --> E
    Q -- 否 --> P[生成 Proposal Design Specs Tasks]
    P --> A{是否需要人工批准}
    A -- 是 --> R[审批]
    R --> B[Builder 小批次实现]
    A -- 否 --> B
    B --> T[自动测试与证据收集]
    T --> V{门禁是否通过}
    V -- 否且可恢复 --> F[分析 修复 重规划]
    F --> B
    V -- 否且不可恢复 --> H
    V -- 是 --> C[独立 Review Agent]
    C --> D{审查是否通过}
    D -- 否 --> B
    D -- 是 --> U[产品与开发最终验收]
    U --> M[合并与归档]

```

---

## 6. 系统设计

### 6.1 项目状态协议

`state.yaml` 描述当前变更的运行状态：

```
change_id: add-arc-payment-reminder
status: AI_BUILDING
current_batch: backend-reminder-query
next_action: run_integration_tests
retry_count: 1
max_auto_retries: 3
blocked_by: null
required_approvals:
  - type: technical
    status: approved
  - type: product
    status: approved
last_evidence:
  - evidence/test-report.md
updated_at: 2026-08-18T10:30:00+08:00

```

状态机必须满足：

- 不能从 `INTAKE` 直接跳到 `AI_BUILDING`。
- `SPEC_READY` 前必须完成 Explore 和 Spec Validate。
- Yellow / Red 风险未获批准时不能进入受限动作。
- `AI_VERIFYING` 未通过时不能进入 `NEEDS_REVIEW`。
- 独立审查未通过时不能进入 `READY_TO_MERGE`。
- 合并后的主分支验证失败时不能 Archive。

### 6.2 运行数据模型

建议定义以下核心实体：

```
# AI Change Run Specification

## Overview

定义一次 AI-Native 变更运行所需的状态、决策、风险、证据和审批。

## ADDED Requirements

### Requirement: 变更运行状态

系统 SHALL 为每个 OpenSpec Change 保存唯一的运行状态。

**Priority**: P0 (Critical)

**Rationale**: Agent 必须知道当前项目位置，才能自动选择下一动作并支持中断恢复。

#### Scenario: 从探索恢复

Given 变更状态为 AI_EXPLORING
And 上次运行已保存代码库分析结果
When Orchestrator 恢复运行
Then 系统继续未完成的探索任务
And 不重复已完成且仍有效的分析

---

### Requirement: 决策记录不可静默覆盖

系统 MUST 以追加方式保存关键业务与技术决策。

**Priority**: P0 (Critical)

**Rationale**: 决策链是审计、回滚和责任确认的基础。

#### Scenario: 产品修改提醒范围

Given 原决策为剩余时间小于 30 天时提醒
When 产品确认修改为小于或等于 30 天
Then Decision Log 新增一条决定
And 旧决定被标记为 Superseded
And Spec 与测试根据新决定更新

```

建议的数据对象包括：

| 对象关键字段      |                                                                        |
| ----------- | ---------------------------------------------------------------------- |
| `ChangeRun` | `changeId`, `status`, `currentBatch`, `retryCount`, `nextAction`       |
| `Decision`  | `id`, `question`, `options`, `recommendation`, `decision`, `decidedBy` |
| `Risk`      | `category`, `level`, `reason`, `restrictedActions`, `approvalRequired` |
| `Evidence`  | `type`, `source`, `result`, `artifactPath`, `commitSha`                |
| `Approval`  | `type`, `status`, `approver`, `scope`, `timestamp`                     |

### 6.3 异常处理与停止条件

AI 必须区分四类失败：

| 失败类型示例默认处理                 |                     |                     |
| -------------------------- | ------------------- | ------------------- |
| **Implementation Failure** | 类型错误、测试失败、接口实现错误    | 自动分析与修复             |
| **Environment Failure**    | 测试服务不可用、依赖下载失败      | 重试、降级或报告环境阻塞        |
| **Specification Failure**  | Scenario 冲突、业务规则不完整 | 进入 `NEEDS_DECISION` |
| **Governance Failure**     | 需要生产权限、不可逆迁移、越权操作   | 立即停止并请求审批           |

强制停止条件：

- 连续自动修复达到 `max_auto_retries`；
- 发现 Intent、Spec 或 Figma 相互冲突；
- 必须扩大已批准 Scope；
- 需要删除数据或执行不可逆迁移；
- 涉及权限、支付、隐私或合规且没有审批；
- 为通过测试必须弱化既有安全规则；
- 无法获得可信的验收证据；
- 预估成本或执行范围显著超过批准阈值。

阻塞报告必须包含：

```
# Blocked Report

## Current State

## Completed Work

## Blocking Reason

## Evidence

## Attempts Made

## Available Options

## AI Recommendation

## Required Decision

## Impact of No Decision

```

---

## 7. 模块详细设计

### 7.1 Intent Intake（需求接入）

**Capabilities**：

- `parseIntent()`：读取业务目标、用户、规则、成功标准和非目标。
- `validateIntent()`：检查是否缺少关键字段。
- `protectHumanDecisions()`：防止 AI 静默修改人类决定。
- `classifyDecisionOwners()`：判断问题应由产品、开发还是安全负责人决定。

Intent Intake 不负责生成技术任务，只负责形成可治理的目标输入。

### 7.2 Explore & Plan（探索与规划）

**Capabilities**：

- `inspectRepository()`：读取代码结构、技术栈和运行入口。
- `inspectSpecs()`：查找现有 Capability 与相似 Requirement。
- `inspectTests()`：判断现有测试覆盖和可执行性。
- `inspectDataFlow()`：追踪登录、订单查询与权限过滤。
- `identifyGaps()`：识别假设、冲突、风险和缺失信息。
- `generateProposal()`：根据 Intent 生成 Why、What 与 Scope。
- `generateDesign()`：生成技术方案、替代方案与权衡。
- `generateDeltaSpecs()`：生成 Requirement 与 Scenario。
- `generateInitialTasks()`：形成可调整的初始计划。

### 7.3 Build & Repair（实现与修复）

**Capabilities**：

- `createIsolatedWorkspace()`：创建分支或 Worktree。
- `selectNextBatch()`：选择下一个最小可验证批次。
- `implementChange()`：修改代码与配置。
- `generateTests()`：根据 Scenario 生成或补充测试。
- `runFocusedTests()`：先运行局部测试。
- `runRegressionTests()`：批次通过后运行回归测试。
- `diagnoseFailure()`：判断失败根因。
- `repairOrReplan()`：自动修复或更新 Design、Tasks。
- `recordExecution()`：写入 Decision Log 与 Evidence。

### 7.4 Verify & Review（验证与审查）

**Capabilities**：

- `validateOpenSpec()`：检查 Spec 结构与 Scenario 覆盖。
- `checkTraceability()`：检查 Intent → Spec → Code → Test → Evidence。
- `reviewDiff()`：检查代码质量、范围和架构一致性。
- `reviewTestIntegrity()`：识别删除断言、跳过测试等行为。
- `runPermissionReview()`：验证经销商数据隔离。
- `runProductE2E()`：从真实用户路径验证结果。
- `compareVisualEvidence()`：检查页面与 Figma 或基线截图。
- `produceReviewReport()`：输出独立审查结论。

### 7.5 Governance & Approval（治理与审批）

**Capabilities**：

- `classifyRisk()`：计算 Green / Yellow / Red 风险。
- `authorizeToolCall()`：在敏感操作前判断是否允许。
- `requestApproval()`：把问题、选项、推荐和影响提交给责任人。
- `resumeAfterApproval()`：批准后从保存状态继续。
- `rejectScopeCreep()`：阻止未经批准的范围扩大。
- `recordApproval()`：保存批准人、范围、时间和条件。
- `enforceMergePolicy()`：未满足门禁时禁止合并。

---

## 8. 协作接口设计

AI-Native OpenSpec 的“接口”不只包括 REST API，也包括产品、开发、Agent、CI 和审批者之间的结构化协作协议。

建议按 Capability 创建：

```
openspec/changes/add-arc-payment-reminder/specs/
├── arc-payment-reminder/spec.md
├── dealer-data-isolation/spec.md
├── ai-change-orchestration/spec.md
├── human-approval/spec.md
└── evidence-gate/spec.md

```

以下是核心 Spec 示例。

```
# AI Change Orchestration Specification

## Overview

定义 AI Agent 如何从人类 Intent 出发，自主完成探索、规划、实现、验证与交付。

## ADDED Requirements

### Requirement: 编码前完成上下文探索

AI Agent MUST 在修改业务代码前读取 Intent、项目宪法、现有 Spec、相关代码和测试。

**Priority**: P0 (Critical)

**Rationale**: 未理解存量系统就直接编码会放大错误假设并破坏既有行为。

#### Scenario: 发现已有提醒能力

Given Intent 要求新增登录提醒
And 系统已存在通用首页提醒框架
When AI 进入 Explore
Then AI 应识别并复用现有提醒框架
And Design 应说明复用方式
And AI 不应无理由创建第二套平行框架

---

### Requirement: 可恢复失败自动修复

AI Agent SHALL 对低风险、可恢复的实现失败进行有限次数的自动分析和修复。

**Priority**: P1 (High)

**Rationale**: AI-Native 流程应减少人对普通开发错误的逐次干预。

#### Scenario: 新增接口出现类型错误

Given AI 已完成一个后端实现批次
And Type Check 返回类型不匹配
When 错误不涉及业务规则变化
Then AI 分析错误根因
And 在最大重试次数内自动修复
And 重新运行 Type Check
And 将修复结果写入 Evidence

---

### Requirement: 关键歧义必须升级给人

AI Agent MUST NOT 自主决定会改变用户行为或业务责任的关键歧义。

**Priority**: P0 (Critical)

**Rationale**: AI 可以推荐方案，但业务责任必须由授权人员承担。

#### Scenario: 关闭弹窗后的重复提醒规则缺失

Given Intent 未定义用户关闭弹窗后本次会话是否再次展示
When 不同选择会改变用户体验
Then AI 进入 NEEDS_DECISION
And 提供可选方案与推荐
And 在产品确认前不实现该行为

```

```
# Human Approval Specification

## Overview

定义不同风险等级下必须获得的人工批准。

## ADDED Requirements

### Requirement: Red 风险动作双重审批

系统 MUST 对 Red 风险动作执行前审批和合并前审批。

**Priority**: P0 (Critical)

**Rationale**: 高风险操作一旦执行可能造成数据、安全或业务损失。

#### Scenario: 需要不可逆数据库迁移

Given AI 发现实现需要删除或不可逆转换生产数据
When AI 更新 Risk 为 Red
Then AI 停止执行 Migration
And 提交影响、替代方案和回滚限制
And 仅在开发负责人批准后继续
And 合并前再次检查审批范围

```

```
# Evidence Gate Specification

## Overview

定义 AI 变更进入审查、合并和归档前必须具备的证据。

## ADDED Requirements

### Requirement: 无证据不得声明完成

AI Agent MUST NOT 在缺少必需验证结果时将变更标记为完成。

**Priority**: P0 (Critical)

**Rationale**: 自然语言自述不能替代可重复的工程验证。

#### Scenario: 前端弹窗代码已完成但 E2E 未运行

Given Builder 已完成前端弹窗实现
And Unit Test 与 Type Check 已通过
But 登录 E2E 尚未运行
When AI 评估完成状态
Then 状态保持 AI_VERIFYING
And 不得进入 NEEDS_REVIEW
And Evidence Report 标记 E2E 为 Missing

```

业务 Spec 示例：

```
# ARC Payment Reminder Specification

## Overview

提醒当前经销商处理即将超过缴费期限的 ARC Basic 待支付订单。

## ADDED Requirements

### Requirement: 登录后展示即将到期订单提醒

系统 SHALL 在当前经销商存在剩余缴费时间小于或等于 30 天的 ARC Basic 待支付订单时展示提醒。

**Priority**: P0 (Critical)

**Rationale**: 及时提醒可以降低订单超过缴费期限后失效的风险。

#### Scenario: 剩余 30 天时提醒

Given 当前用户属于经销商 A
And 经销商 A 存在一笔 ARC Basic 待支付订单
And 订单剩余缴费时间为 30 天
When 用户登录并进入首页
Then 系统展示提醒弹窗
And 提醒仅包含经销商 A 的订单摘要

#### Scenario: 剩余 31 天时不提醒

Given 当前用户属于经销商 A
And 经销商 A 存在一笔 ARC Basic 待支付订单
And 订单剩余缴费时间为 31 天
When 用户登录并进入首页
Then 系统不展示该订单提醒

#### Scenario: 已支付订单不提醒

Given 当前用户属于经销商 A
And 经销商 A 存在一笔剩余 10 天的已支付订单
When 用户登录并进入首页
Then 系统不展示该订单提醒

#### Scenario: 不泄露其他经销商订单

Given 当前用户属于经销商 A
And 经销商 B 存在即将到期的待支付订单
When 当前用户登录并进入首页
Then 系统不返回经销商 B 的订单数据
And 系统不因经销商 B 的订单展示提醒

```

> **格式要求**：
>
> - 使用 `## ADDED/MODIFIED/REMOVED Requirements` 作为 Delta Header。
> - 每个需求使用 `### Requirement: <标题>`。
> - Requirement 描述包含 `SHALL` 或 `MUST`。
> - 每个 Requirement 包含 `Priority` 与 `Rationale`。
> - 每个 Requirement 至少包含一个 Given / When / Then Scenario。
> - 编写后运行 `openspec validate <change-name>`。
> - AI-Native 扩展还应检查 Scenario 是否映射到测试和 Evidence。

---

## 9. AI-Native 规范驱动实现

AI-Native 模式的核心不是“AI 写代码”，而是 **AI 在治理与证据约束下维护 Intent、Spec、实现和验证之间的闭环**。

### 9.1 全链路追踪矩阵

建立以下映射：

```
Intent
  ↕
Decision
  ↕
Requirement / Scenario
  ↕
Design
  ↕
Task Batch
  ↕
Code
  ↕
Test
  ↕
Evidence
  ↕
Approval

```

示例：

| Intent / Requirement决策与设计代码实现验证方式Evidence |                         |                            |                      |                                  |
| ----------------------------------------- | ----------------------- | -------------------------- | -------------------- | -------------------------------- |
| `≤30 天待支付订单提醒`                            | Decision D-003：包含第 30 天 | `reminder.service.ts`      | Unit + Integration   | `test-report.md#threshold`       |
| `仅当前经销商数据`                                | 复用 Dealer Scope Filter  | `order.repository.ts`      | Permission Test      | `security-report.md`             |
| `登录不被提醒服务阻塞`                              | 首页异步加载并可降级              | `dashboard.loader.ts`      | Failure Injection    | `resilience-report.md`           |
| `展示提醒弹窗`                                  | 复用现有 Modal 组件           | `PaymentReminderModal.tsx` | Playwright E2E       | `screenshots/login-reminder.png` |
| `登录性能无明显退化`                               | 查询索引 + 延迟加载             | Query / API                | Performance Baseline | `performance-report.md`          |

### 9.2 目录结构映射

```
project/
├── AGENTS.md
├── docs/
│   └── openspec-ai-native-practical-guide.md
├── openspec/
│   ├── config.yaml
│   ├── specs/
│   │   ├── dealer-data-isolation/spec.md
│   │   ├── arc-order-management/spec.md
│   │   └── dashboard-reminder/spec.md
│   └── changes/
│       └── add-arc-payment-reminder/
│           ├── .openspec.yaml
│           ├── intent.md
│           ├── proposal.md
│           ├── design.md
│           ├── tasks.md
│           ├── state.yaml
│           ├── risk.yaml
│           ├── decision-log.md
│           ├── specs/
│           │   ├── arc-payment-reminder/spec.md
│           │   ├── dealer-data-isolation/spec.md
│           │   ├── ai-change-orchestration/spec.md
│           │   ├── human-approval/spec.md
│           │   └── evidence-gate/spec.md
│           └── evidence/
│               ├── test-report.md
│               ├── security-report.md
│               ├── performance-report.md
│               ├── review-report.md
│               └── screenshots/
├── src/
│   ├── orders/
│   ├── reminders/
│   └── dashboard/
└── tests/
    ├── unit/
    ├── integration/
    └── e2e/

```

### 9.3 Agent Orchestrator 实现示例

以下为流程伪代码，重点展示状态、风险与证据，而不是限定具体框架：

```
async function runChange(changeId: string): Promise<void> {
  const context = await loadChangeContext(changeId);
  const policy = await loadProjectPolicy("AGENTS.md");

  while (!isTerminal(context.state.status)) {
    switch (context.state.status) {
      case "INTAKE": {
        validateIntent(context.intent);
        await transition(context, "AI_EXPLORING");
        break;
      }

      case "AI_EXPLORING": {
        const findings = await exploreRepository(context, policy);
        const decisions = identifyHumanDecisions(findings);

        if (decisions.length > 0) {
          await writeDecisionRequests(context, decisions);
          await transition(context, "NEEDS_DECISION");
          break;
        }

        await generateProposalDesignSpecsAndTasks(context, findings);
        await runOpenSpecValidate(changeId);
        await classifyRisk(context);
        await transition(context, "SPEC_READY");
        break;
      }

      case "SPEC_READY": {
        if (await approvalsSatisfied(context)) {
          await transition(context, "READY_TO_BUILD");
        } else {
          await requestRequiredApprovals(context);
          return;
        }
        break;
      }

      case "READY_TO_BUILD": {
        await createIsolatedWorkspace(changeId);
        await transition(context, "AI_BUILDING");
        break;
      }

      case "AI_BUILDING": {
        const batch = selectNextBatch(context.tasks);
        const result = await implementAndRunFocusedTests(batch, context);

        if (result.passed) {
          await recordEvidence(context, result);
          markBatchComplete(context.tasks, batch);
        } else if (result.recoverable && context.state.retryCount < 3) {
          await diagnoseRepairAndReplan(result, context);
          context.state.retryCount += 1;
        } else {
          await writeBlockedReport(context, result);
          await transition(context, "NEEDS_DECISION");
          return;
        }

        if (allBatchesComplete(context.tasks)) {
          await transition(context, "AI_VERIFYING");
        }
        break;
      }

      case "AI_VERIFYING": {
        const evidence = await runAllRequiredGates(context);

        if (!evidence.allPassed) {
          await diagnoseRepairAndReplan(evidence, context);
          await transition(context, "AI_BUILDING");
          break;
        }

        await transition(context, "NEEDS_REVIEW");
        break;
      }

      case "NEEDS_REVIEW": {
        const review = await runIndependentReview(context);

        if (review.status === "PASS") {
          await createOrUpdateDraftPullRequest(context);
          await transition(context, "READY_TO_MERGE");
        } else {
          await addReviewTasks(context.tasks, review.findings);
          await transition(context, "AI_BUILDING");
        }
        break;
      }

      case "READY_TO_MERGE": {
        await requestFinalHumanApproval(context);
        return;
      }

      default:
        return;
    }

    await persistContext(context);
  }
}

```

这段流程体现四条规则：

1. 状态决定动作。
2. 风险决定审批。
3. 测试失败触发修复或重规划。
4. 证据和独立审查决定能否交付。

---

## 10. 验证与评估：确保实现符合目标与规格

AI-Native 验证不仅检查“代码是否符合 Spec”，还检查“Spec 是否符合 Intent、过程是否符合治理规则”。

### 10.1 追踪矩阵（Traceability Matrix）

每次变更至少回答：

- 每个 Success Criterion 对应哪些 Requirement？
- 每个 Requirement 对应哪些 Scenario？
- 每个 Scenario 对应哪些测试？
- 每个测试结果保存在哪里？
- 每个高风险决策由谁批准？
- 每个代码变更为何属于当前 Scope？
- 是否存在没有 Spec 支撑的新增行为？
- 是否存在有 Spec 但没有实现或证据的行为？

AI 可以自动生成追踪报告，但独立 Reviewer 必须验证其真实性。

### 10.2 从 Scenario 到测试与证据

Spec 中的场景：

```
#### Scenario: 不泄露其他经销商订单

Given 当前用户属于经销商 A
And 经销商 B 存在即将到期的待支付订单
When 当前用户登录并进入首页
Then 系统不返回经销商 B 的订单数据
And 系统不因经销商 B 的订单展示提醒

```

对应的集成测试示例：

```
test("dealer A cannot see dealer B payment reminder", async ({ request }) => {
  await seedPendingOrder({
    dealerId: "dealer-b",
    remainingDays: 10,
    status: "PENDING_PAYMENT",
    productType: "ARC_BASIC",
  });

  const tokenA = await loginAsDealer("dealer-a");
  const response = await request.get("/api/dashboard/payment-reminders", {
    headers: { Authorization: `Bearer ${tokenA}` },
  });

  expect(response.status()).toBe(200);

  const body = await response.json();
  expect(body.orders).toEqual([]);
  expect(body.shouldShowReminder).toBe(false);
});

```

对应的 Evidence：

```
evidence/security-report.md
- Test: dealer A cannot see dealer B payment reminder
- Commit: <sha>
- Result: PASS
- Executed by: CI
- Timestamp: <time>
- Log artifact: <path>

```

对于 UI 场景，还应生成：

- 登录后的页面截图；
- 30 天、31 天、已支付和无订单状态截图；
- 浏览器 Console 检查；
- 网络请求摘要；
- Figma 对比结果。

### 10.3 与 TDD 和传统 OpenSpec 的关系

```
传统 TDD:
需求 → 测试 → 代码 → 重构

传统 OpenSpec:
需求 → Spec → 验证 Spec → 测试 → 代码 → 人工验收

AI-Native OpenSpec:
Intent
→ AI Explore
→ Proposal / Design / Spec
→ 风险审批
→ 测试与实现小循环
→ 自动评估
→ 自动修复或重规划
→ 独立审查
→ Evidence Gate
→ 人工最终验收
→ Archive

```

关键区别：

- **Intent 先于 Spec**：防止 AI 只优化局部描述而偏离业务目标。
- **测试先后可动态调整**：AI 可以根据 Scenario 先写测试，也可以先建立最小骨架，但必须最终形成可追踪映射。
- **失败会回流到计划**：测试失败不只是修代码，也可能要求修改 Design 或补充决策。
- **完成由证据决定**：不是由 Agent 的语言判断决定。
- **人只处理关键问题**：普通实现错误由 AI 自主闭环。

### 10.4 PR 与独立审查清单

-  `intent.md` 的业务目标和非目标没有被静默改变。
-  每个 Requirement 有对应实现。
-  每个 Scenario 有对应自动化测试或明确人工验证理由。
-  29、30、31 天边界均被覆盖。
-  已支付、已取消、已失效状态均被覆盖。
-  经销商数据隔离有独立安全测试。
-  AI 没有删除、跳过或弱化关键测试来获得通过。
-  Design 与实际实现一致。
-  `tasks.md` 中的动态调整有原因记录。
-  所有 Yellow / Red 风险有正确审批。
-  UI 截图、E2E、Console 和接口结果齐全。
-  性能与降级行为有证据。
-  变更没有超出 Scope。
-  回滚方案可执行。
-  Review Agent 使用了独立上下文。
-  PR 中包含完整 Evidence Pack。

---

## 11. 测试设计：验证业务、工程与 Agent 流程

测试不只是代码的可执行规格，也是约束 AI 自主行为的基础设施。

### 11.1 单元测试

针对确定性业务规则：

- 剩余天数计算；
- `≤30` 阈值判断；
- 订单状态过滤；
- ARC Basic 产品类型过滤；
- 提醒摘要生成；
- 同一会话重复提醒策略。

示例：

```
describe("shouldShowArcPaymentReminder", () => {
  it("returns true for a pending ARC Basic order with 30 days remaining", () => {
    expect(
      shouldShowArcPaymentReminder({
        productType: "ARC_BASIC",
        status: "PENDING_PAYMENT",
        remainingDays: 30,
      }),
    ).toBe(true);
  });

  it("returns false for 31 days remaining", () => {
    expect(
      shouldShowArcPaymentReminder({
        productType: "ARC_BASIC",
        status: "PENDING_PAYMENT",
        remainingDays: 31,
      }),
    ).toBe(false);
  });
});

```

### 11.2 集成测试

验证模块间协作：

- 登录身份能否正确传入提醒查询；
- API 是否只查询当前经销商；
- 状态和日期过滤是否在数据库层或服务层正确执行；
- 多笔订单返回数量和摘要是否符合 Spec；
- 查询异常时是否按 Design 降级；
- 接口错误格式是否统一。

### 11.3 E2E 与视觉测试

模拟真实用户路径：

```
登录
→ 进入首页
→ 加载提醒
→ 展示弹窗
→ 查看文案和订单数量
→ 点击处理
→ 跳转目标订单页面
→ 返回首页
→ 验证重复展示策略

```

E2E 至少覆盖：

- 有 30 天待支付订单；
- 只有 31 天订单；
- 只有已支付订单；
- 其他经销商存在订单；
- 提醒 API 失败；
- 多笔订单；
- 不同屏幕尺寸；
- 关闭和跳转交互。

### 11.4 权限、安全与性能测试

- **权限测试**：Dealer A 无法看到 Dealer B 的数据。
- **越权测试**：修改请求参数不能访问其他经销商。
- **敏感信息测试**：提醒摘要不返回不必要的客户隐私字段。
- **性能测试**：比较变更前后首页首屏与提醒 API 延迟。
- **压力测试**：大量待支付订单下查询仍满足目标。
- **降级测试**：提醒服务失败不阻塞登录。
- **日志测试**：日志不记录敏感数据。

### 11.5 Agent 工作流测试

AI-Native 系统本身也需要测试：

- 未读取 Intent 时禁止进入 Build；
- Spec Validate 失败时禁止执行代码；
- Yellow / Red 审批缺失时工具调用被阻止；
- 连续失败达到上限后进入 `NEEDS_DECISION`；
- Reviewer 不得复用 Builder 的结论作为唯一证据；
- Evidence 缺失时不能进入 `READY_TO_MERGE`；
- 中断后可以从 `state.yaml` 恢复；
- 重复事件不会创建多个相同变更或 PR。

---

## 12. AI-Native 协作操作手册

### 12.1 准备环境

基础环境：

- Git 与受保护的主分支；
- OpenSpec CLI；
- 可运行的本地或容器化开发环境；
- Unit、Integration、E2E 与性能测试入口；
- CI Pipeline；
- 可隔离的分支或 Worktree；
- AI Agent 可读取的仓库和文档；
- 审批与审计渠道。

首先创建 `AGENTS.md`：

```
# Project Constitution

## Product Context

本项目是 BMW 经销商保险业务系统。

## Technology Stack

- Frontend: Next.js
- Backend: NestJS
- Database: PostgreSQL / Prisma

## Architecture Rules

- 所有订单查询必须包含 Dealer Scope。
- 业务逻辑不得直接写在 Controller。
- 所有数据库结构变化必须提供 Migration。
- 所有新增业务规则必须更新 OpenSpec。
- 所有新增接口必须包含 Integration Test。

## AI Autonomous Actions

- 读取仓库和文档。
- 创建功能分支或 Worktree。
- 修改功能分支代码。
- 生成和运行测试。
- 修复 Lint、Type Check 和普通测试错误。
- 创建 Draft PR。
- 更新 Proposal、Design、Specs、Tasks 和 Evidence。

## Human Approval Required

- 修改权限模型。
- 修改支付逻辑。
- 删除或不可逆迁移数据。
- 访问生产数据。
- 修改生产 Secret 或基础设施。
- 合并主分支。
- 发布生产环境。

## Definition of Done

- OpenSpec Validate 通过。
- Type Check 和 Lint 通过。
- Unit、Integration、E2E 通过。
- 权限与安全检查通过。
- 性能目标达成。
- 独立 Review 通过。
- Evidence Pack 完整。

```

### 12.2 启动第一个 AI-Native 变更

推荐入口是 GitHub Issue、产品需求系统或 `intent.md`。

人工备用流程：

```
# 1. 初始化 OpenSpec（尚未初始化时）
openspec init --tools none

# 2. 创建变更目录
mkdir -p openspec/changes/add-arc-payment-reminder

# 3. 产品填写 intent.md
# 4. 将变更状态设为 INTAKE
# 5. 添加 ai-ready 标签或触发 Orchestrator

```

Orchestrator 自动执行：

```
读取 Intent
→ Explore
→ 输出关键问题
→ 获取必要决策
→ 生成 Proposal / Design / Specs / Tasks
→ Validate
→ 风险评估
→ 进入 Build

```

如果团队暂时没有自动 Orchestrator，可以用现有 Slash Commands 作为过渡：

```
/opsx:explore
/opsx:propose add ARC Basic payment reminder
/opsx:apply
/opsx:verify
/opsx:archive

```

但每一步都应读取和更新 `state.yaml`、`risk.yaml`、Decision Log 与 Evidence，避免退回纯人工驱动模式。

### 12.3 Review、合并与归档

AI 创建的 PR 应包含：

```
## Business Goal

## Scope

## Implementation Summary

## OpenSpec Changes

## Decisions Made

## Risk Classification

## Scenario Verification

## Automated Test Results

## Permission and Security Results

## UI Evidence

## Performance Evidence

## Known Limitations

## Rollback Plan

## Required Human Approvals

```

产品重点审查：

- 业务目标；
- Scenario；
- 文案、交互和页面；
- 用户路径；
- 是否出现未经确认的新行为。

开发重点审查：

- 架构和 Diff；
- 数据权限；
- 测试完整性；
- 数据库与性能；
- 风险和回滚；
- 是否满足 AGENTS.md。

合并后：

```
# 主分支重新验证
openspec validate add-arc-payment-reminder

# 验收通过后归档
openspec archive add-arc-payment-reminder

```

具体命令参数以项目实际 OpenSpec 版本为准。关键要求是：只有主分支验证和必要审批通过后，Delta Specs 才能成为正式主规范。

### 12.4 阻塞与人工接管

人工接管不意味着 AI-Native 失败，而是治理机制正常工作。

收到 `NEEDS_DECISION` 或 `BLOCKED` 时，人只需要处理结构化问题：

```
问题是什么
→ 为什么 AI 不能自行决定
→ 有哪些可选方案
→ AI 推荐哪一个
→ 每个方案的影响
→ 谁拥有决定权

```

人给出决定后，AI 从保存状态继续，而不是重新开始整个项目。

---

## 13. 生产级扩展实践

### 13.1 持久化状态与中断恢复

生产级 Orchestrator 不能只依赖一次对话上下文。至少需要持久化：

- Change 状态；
- 当前批次；
- 已完成任务；
- 工具调用结果；
- 重试次数；
- 决策与审批；
- Evidence 索引；
- 当前 Commit SHA；
- 下一动作。

恢复时必须检查代码、Spec 与保存状态是否仍然一致，不能盲目沿用过期计划。

### 13.2 权限、安全与沙箱

建议采用：

- 只读仓库权限作为默认值；
- 仅对功能分支开放写权限；
- 工具级 Allowlist；
- Secret 隔离；
- 网络访问白名单；
- 数据库使用测试副本；
- 生产环境零默认权限；
- 高风险工具调用前审批；
- 所有操作写入审计日志。

AI 不应获得“为了方便”而开放的全局管理员权限。

### 13.3 幂等性与并发控制

自动化事件可能重复触发。系统必须保证：

- 同一个 Issue 不重复创建 Change；
- 同一个 Change 不重复创建工作分支；
- 同一个批次不被多个 Agent 同时修改；
- 同一个审批不会被重复消费；
- 同一个 PR 不被重复创建；
- Archive 不重复合并 Delta Spec。

可使用：

```
change_id + action + commit_sha

```

作为幂等键，并为 Change 增加 Lease 或 Lock。

### 13.4 可观测性

至少记录：

- 每个状态停留时间；
- Agent 每次动作与工具调用；
- 自动修复次数；
- 人工决策次数和等待位置；
- 测试通过率；
- Review 退回原因；
- Scope 变更次数；
- Token、时间和计算成本；
- 从 Intent 到 Merge 的 Lead Time；
- 上线后缺陷与回滚情况。

重点不是监控“AI 调用了多少次”，而是判断：

- AI 在哪里频繁失败？
- 哪些需求经常缺少关键信息？
- 哪些风险规则过松或过严？
- 哪些测试不足以支撑自动化？
- 人工时间真正花在了哪里？
- AI 是否提高了交付速度，同时保持或提升质量？

### 13.5 多 Agent 与跨仓库扩展

只有当单 Agent 明显成为瓶颈时，才引入多 Agent：

```
Orchestrator
├── Product Analysis Agent
├── Architecture Agent
├── Backend Builder
├── Frontend Builder
├── Test Agent
├── Security Reviewer
└── Product Review Agent

```

多 Agent 必须共享同一 Intent、Spec、Decision Log 和 Evidence 索引，避免每个 Agent 使用不同事实。

跨仓库项目可以集中维护 OpenSpec Store，并为前端、后端、数据和基础设施仓库建立只读引用。每个仓库拥有自己的实现任务和测试，但共享同一个 Change ID 与最终 Evidence Pack。

---

## 14. 结语

通过 ARC Basic 即将到期订单提醒案例，本文展示了 OpenSpec 如何从“AI 友好的规格驱动开发”升级为“AI-Native 的项目运行系统”。

传统 OpenSpec 的核心关系是：

```
Spec ↔ Code ↔ Test

```

AI-Native OpenSpec 扩展为：

```
Intent
↔ Decision
↔ Spec
↔ Plan
↔ Code
↔ Test
↔ Evidence
↔ Approval

```

最终应形成以下组织关系：

- **Intent 即方向**：人定义为什么做、什么不能做。
- **Spec 即合同**：产品、开发和 AI 共享同一行为定义。
- **Agent 即运行者**：AI 自主探索、规划、实现、修复和重规划。
- **测试即证据**：所有结论必须可重复验证。
- **风险即权限**：AI 的自主范围随风险变化。
- **人即治理者**：人处理价值判断、高风险决策和最终责任。

AI-Native 并不意味着“完全无人参与”，而是把人从重复的任务拆解、流程催促和普通错误处理里释放出来，使产品经理聚焦用户价值，使开发人员聚焦架构、安全与工程系统，让 AI 在明确边界内持续运行项目。

这套模式的最终目标可以概括为：

> **人给目标，Spec 定合同，AI 跑项目，证据做裁判，人负最终责任。**