---
title: Harness Engineering
created: 2026-04-12
updated: 2026-04-12
type: concept
tags: [agent, tools, evaluation]
sources: [raw/articles/openai-harness-engineering-2026-02-11.md]
---

# Harness Engineering

## 定义

Harness Engineering 是一种以 AI Agent 为核心的软件工程范式。人类不再直接编写代码，而是设计开发环境、构建反馈循环、指定任务意图，让编码 Agent（如 [[codex]]）完成所有代码生成工作。

核心理念：**Humans steer. Agents execute.**

## 关键实践

### 1. 仓库知识即系统记录 (Repository Knowledge as System of Record)

- AGENTS.md 不是百科全书，而是**目录/地图**（约 100 行）
- 知识库存放在结构化的 `docs/` 目录中，包括设计文档、执行计划、产品规格、架构文档等
- 采用**渐进式披露**（progressive disclosure）：Agent 从小的稳定入口开始，按需深入
- 通过 linter 和 CI 机械验证知识库的时效性、交叉链接和结构正确性
- 专门的 "doc-gardening" Agent 定期扫描过时文档并开修复 PR

### 2. Agent 可读性优先 (Agent Legibility is the Goal)

- 代码仓库首先为 Agent 的可读性优化，而非人类偏好
- 所有知识必须存在于仓库内的版本化文件中（代码、Markdown、schema、计划）
- Agent 看不到的知识（Google Docs、Slack、人脑）对它来说**不存在**
- 倾向使用 "boring" 技术：组合性好、API 稳定、训练数据中常见
- 有时让 Agent 重新实现子集比依赖不透明的公共库更划算

### 3. 严格架构约束 (Enforcing Architecture)

- 分层架构：**Types → Config → Repo → Service → Runtime → UI**
- 每个业务领域分为固定层集，严格验证依赖方向
- 跨领域关注点（auth、遥测、feature flags）通过单一 Providers 接口进入
- 自定义 linter 机械执行规则（结构化日志、命名约定、文件大小限制等）
- linter 错误消息包含修复指引，注入到 Agent 上下文中

### 4. 高吞吐量合并哲学 (Throughput-Driven Merge Philosophy)

- 最小化阻塞性合并门
- PR 生命周期短
- 测试不稳定通常通过重跑而非无限阻塞解决
- 在 Agent 吞吐量远超人类注意力的系统中：**修正是廉价的，等待是昂贵的**

### 5. 垃圾回收式技术债管理 (Entropy and Garbage Collection)

- Agent 会复制仓库中已有的模式（包括次优模式），导致漂移
- 手工清理不可扩展（曾花费 20% 周五时间清理 "AI slop"）
- 编码 "golden principles" 到仓库，后台 Agent 定期扫描并开重构 PR
- 技术债像高息贷款：**持续小步偿还优于累积后痛苦处理**
- 人类品味被捕获一次，然后持续强制应用于每一行代码

### 6. 应用可观测性 (Application Legibility)

- 让 Agent 直接访问 UI、日志、指标
- 每个 git worktree 可独立启动应用实例
- 接入 Chrome DevTools Protocol：Agent 可操作 DOM、截图、导航
- 本地可观测栈（Vector → Victoria Logs/Metrics/Traces）
- Agent 可用 LogQL 查询日志、PromQL 查询指标
- 单次 Agent 运行可持续 6+ 小时（人类睡觉时）

## Agent 自主性演进

成熟后，Agent 可从单个 prompt 端到端完成：

1. 验证代码库当前状态
2. 复现报告的 bug
3. 录屏演示失败
4. 实现修复
5. 通过驱动应用验证修复
6. 录屏演示修复结果
7. 打开 PR
8. 回应 Agent 和人类反馈
9. 检测并修复构建失败
10. 仅在需要判断时升级给人类
11. 合并变更

## 数据

| 指标 | 数值 |
|------|------|
| 开发周期 | 5 个月 |
| 代码量 | ~100 万行 |
| 初始团队 | 3 名工程师 |
| 当前团队 | 7 名工程师 |
| PR 总量 | ~1,500 |
| 人均吞吐 | 3.5 PR/人/天 |
| 开发速度 | 约为手写的 1/10 时间 |
| 手动代码 | 0 行 |

## 关键教训

- **纪律体现在脚手架而非代码中**：工具、抽象、反馈循环比代码本身更重要
- **上下文是稀缺资源**：给 Agent 地图而非千页手册
- **一切重要 = 一切不重要**：过度指导导致 Agent 局部模式匹配
- **约束是乘数**：对 Agent 来说，编码的规则一旦设定就处处适用
- **人类工作升维**：从写代码变为设计环境、指定意图、构建反馈循环

## 开放问题

- 完全 Agent 生成的系统中，架构连贯性如何随时间（年维度）演化？
- 人类判断在何处提供最大杠杆？如何编码这种判断使其复利增长？
- 随着模型能力持续提升，此系统将如何演化？

## 相关概念

- [[codex]] — OpenAI 的编码 Agent
- [[agent]] — AI Agent 系统总览
- [[evaluation]] — Agent 评测方法
