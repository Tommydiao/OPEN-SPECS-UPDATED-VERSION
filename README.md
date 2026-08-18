# OPEN-SPECS-UPDATED-VERSION

> **OpenSpec 的 AI-Native 协作模式升级方案**

本仓库基于 OpenSpec 的 **Spec-Driven Development（规格驱动开发）** 思想，进一步探索一种更适合 AI Coding / AI Agent 时代的软件研发协作模式。

核心目标是将传统：

```text
人提出需求
→ 人驱动流程
→ AI 执行任务
→ 人验证结果
```

升级为：

```text
人定义目标与边界
        ↓
AI 理解项目并自主规划
        ↓
AI 实现 / 测试 / 修复 / 重规划
        ↓
工程证据自动验证
        ↓
人处理关键决策与最终审批
```

即：

> **Human-Governed, AI-Operated, Evidence-Gated**  
> 人负责目标、边界和责任；AI 负责探索、规划、执行与修正；工程证据决定流程是否可以继续。

---

## 为什么要做这个版本？

传统 OpenSpec 已经很好地解决了：

```text
需求
↓
Spec
↓
Code
↓
Test
```

之间的一致性问题。

但在实际 AI 开发过程中，整个研发流程仍然较依赖人来驱动：

```text
人触发 Explore
→ 人确认 Proposal / Design / Specs / Tasks
→ 人触发 Apply
→ AI 编写代码
→ 人触发 Verify
→ 人决定 Archive
```

这种模式更接近：

> **AI-Assisted Development**

而不是完整的：

> **AI-Native Development**

本项目尝试进一步解决的问题是：

> **能否让 AI 不只是“执行开发任务”，而是成为项目日常运行的主体？**

---

# 核心设计

在保留 OpenSpec：

- `proposal.md`
- `design.md`
- `specs/`
- `tasks.md`
- `validate`
- `archive`

等核心机制的基础上，引入以下 AI-Native 扩展。

## 1. Intent

新增：

```text
intent.md
```

由产品 / 业务负责人维护。

Intent 不描述具体怎么开发，而是定义：

- 为什么做
- 为谁做
- 当前问题
- 业务目标
- Business Rules
- Success Criteria
- Out of Scope
- 哪些问题必须由人决定

产品不再负责拆：

```text
前端任务
后端任务
数据库任务
接口任务
```

而是把重点放在：

> **Goal + Rules + Constraints + Success Criteria**

---

## 2. AI Orchestrator

AI 不再等待人逐步执行：

```text
Explore
Apply
Verify
Archive
```

而是根据项目状态自动决定下一步。

推荐状态：

```text
INTAKE
  ↓
AI_EXPLORING
  ↓
NEEDS_DECISION
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

项目由：

```text
state.yaml
```

驱动，而不是由人不断输入下一条 Prompt 驱动。

---

## 3. Dynamic Planning

传统 `tasks.md` 更接近静态任务列表。

本方案中：

```text
tasks.md
```

变成 AI 持续维护的 **动态计划**。

AI 的工作循环变为：

```text
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

当 AI 在开发过程中发现：

- 原设计无法实现
- 数据结构不支持
- Spec 存在冲突
- 测试失败
- Scope 发生变化

AI 可以重新调整：

```text
Design
Specs
Tasks
Risk
```

而不是停下来等待人重新拆任务。

---

## 4. Risk-Based Autonomy

AI 的自主权限根据风险等级决定。

### Green

例如：

- 文案
- UI 样式
- 小范围 Bug
- 测试补充

AI 可以自主：

```text
实现
→ 测试
→ 修复
→ 创建 Draft PR
```

### Yellow

例如：

- 新 API
- 普通业务逻辑
- 可逆数据库字段
- 依赖升级

AI 可以自主实现，但需要开发人员 Review 后合并。

### Red

例如：

- 权限
- 支付
- 敏感数据
- 删除数据
- 不可逆 Migration
- 生产环境配置

必须：

```text
执行前人工审批
+
合并前再次审批
```

---

## 5. Evidence Gate

AI 不能仅仅通过一句：

> 已完成。

来结束任务。

每次交付都需要 Evidence Pack：

```text
OpenSpec Validate
+
Type Check
+
Lint
+
Unit Test
+
Integration Test
+
E2E Test
+
Permission / Security Test
+
Performance Test
+
UI Screenshots
+
Review Report
+
Decision Log
+
Rollback Plan
```

只有证据满足要求后，项目状态才能继续推进。

---

## 6. Independent Review

Builder Agent 完成开发之后，不能自己直接宣布：

```text
PASS
```

需要独立 Review Agent 重新读取：

```text
Intent
+
Spec
+
Design
+
Decision Log
+
Code Diff
+
Tests
+
Evidence
```

审查：

- 是否满足业务目标
- 是否满足所有 Scenario
- 是否扩大 Scope
- 是否违反架构规则
- 是否绕过权限
- 是否通过修改测试来“制造通过”
- 是否存在安全和性能问题

最终输出：

```text
PASS
PASS_WITH_NOTES
CHANGES_REQUIRED
BLOCKED
```

---

# AI-Native OpenSpec 架构

```text
┌─────────────────────────────────┐
│ Human Governance                │
│ Intent / Rules / Approval       │
└────────────────┬────────────────┘
                 ↓
┌─────────────────────────────────┐
│ Contract & Context              │
│ OpenSpec / AGENTS.md / Design   │
└────────────────┬────────────────┘
                 ↓
┌─────────────────────────────────┐
│ AI Orchestrator                 │
│ Explore / Plan / Build / Replan │
└────────────────┬────────────────┘
                 ↓
┌─────────────────────────────────┐
│ Agent Execution                 │
│ Code / Test / Repair / Review   │
└────────────────┬────────────────┘
                 ↓
┌─────────────────────────────────┐
│ Evidence Gate                   │
│ CI / Test / Security / E2E      │
└────────────────┬────────────────┘
                 ↓
┌─────────────────────────────────┐
│ Human Approval                  │
│ Product / Engineering / Release │
└─────────────────────────────────┘
```

---

# 推荐目录结构

```text
project/
├── AGENTS.md
│
├── openspec/
│   ├── config.yaml
│   │
│   ├── specs/
│   │
│   └── changes/
│       └── change-name/
│
│           ├── intent.md
│           ├── proposal.md
│           ├── design.md
│           ├── tasks.md
│
│           ├── state.yaml
│           ├── risk.yaml
│           ├── decision-log.md
│
│           ├── specs/
│           │
│           └── evidence/
│               ├── test-report.md
│               ├── security-report.md
│               ├── performance-report.md
│               ├── review-report.md
│               └── screenshots/
│
├── src/
│
└── tests/
```

其中：

```text
intent.md
state.yaml
risk.yaml
decision-log.md
evidence/
```

属于本项目提出的 **AI-Native 扩展约定**，并非 OpenSpec 官方强制目录。

---

# Product / Engineering / AI 的职责变化

## Product

主要负责：

```text
Business Goal
User Value
Business Rules
Success Criteria
Out of Scope
Human Decisions
```

产品不再承担大量技术任务拆解工作。

## Engineering

主要负责：

```text
Architecture
Engineering Rules
Security Boundaries
Agent Permissions
CI / Testing Infrastructure
Production Approval
```

开发人员逐渐从：

> 主要代码生产者

转变为：

> **AI Engineering System 的设计者和治理者**

## AI Agent

主要负责：

```text
Explore
↓
Plan
↓
Generate Spec
↓
Generate Tasks
↓
Build
↓
Test
↓
Repair
↓
Replan
↓
Review
↓
Generate Evidence
↓
Create PR
```

---

# 与传统 OpenSpec 的区别

### Traditional OpenSpec

```text
Human
  ↓
Proposal
  ↓
Design
  ↓
Specs
  ↓
Tasks
  ↓
Human triggers Apply
  ↓
AI / Developer
  ↓
Human triggers Verify
```

### AI-Native OpenSpec

```text
Human Intent
      ↓
AI Explore
      ↓
AI Proposal / Design / Specs
      ↓
Risk Gate
      ↓
AI Build
      ↓
AI Test
      ↓
AI Repair / Replan
      ↑             ↓
      └─────────────┘
             ↓
Independent Review
             ↓
Evidence Gate
             ↓
Human Approval
             ↓
Archive
```

---

# 完整实战指南

仓库中的完整设计和案例请阅读：

## [OpenSpec AI-Native 实战指南](./openspec-practical-guide-ai-native.md)

其中包含：

- AI-Native OpenSpec 方法论
- Intent 设计
- Agent Orchestrator
- 状态机设计
- 风险分级
- Dynamic Tasks
- Human-in-the-loop
- Evidence Gate
- Independent Review
- Spec 示例
- Agent Orchestrator 伪代码
- Unit / Integration / E2E
- 权限测试
- AI Workflow Test
- 生产级扩展方案

---

# 当前状态

当前仓库主要用于：

> **讨论、验证和迭代 AI-Native 软件研发协作方式。**

目前重点仍然是：

```text
Methodology
+
Architecture
+
Workflow
+
Governance
```

后续可以继续扩展：

```text
GitHub Issue
      ↓
AI Orchestrator
      ↓
OpenSpec
      ↓
Codex / Claude Code
      ↓
GitHub Actions
      ↓
Independent Review Agent
      ↓
Evidence Gate
      ↓
Pull Request
```

将方法论进一步实现为可运行的 AI-Native Software Development Workflow。

---

# 核心思想

传统软件研发：

> **人管理项目，人写代码，工具负责辅助。**

AI-Assisted 研发：

> **人管理项目，AI 帮助写代码。**

AI-Native 研发：

> **人治理项目，AI 运行项目，工程系统验证 AI。**

最终目标：

> **人给目标，Spec 定合同，AI 跑项目，证据做裁判，人负最终责任。**

---

## Disclaimer

本仓库是对 OpenSpec 思想的 AI-Native 扩展探索。

其中部分目录、状态机、Risk Model、Evidence Gate、Agent Orchestration 等设计属于本仓库提出的实践方案，并非 OpenSpec 官方规范。

本项目的目标不是替代 OpenSpec，而是在其 Spec-Driven Development 基础上探索更加适合 AI Agent 时代的软件工程协作方式。
