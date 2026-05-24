---
title: Plan-First + Skill 工作流
created: 2026-05-24
updated: 2026-05-24
type: concept
tags: [agent, planning, tools]
sources: [raw/articles/wquguru-pi-coding-agent-guide-2026-05-18.md]
---

# Plan-First + Skill 工作流

## 定义

Plan-First + Skill 是一种 AI Agent 工作流模式，核心原则是：在让模型执行代码之前，先要求它产出计划，经过评审后再执行；同时用 Skill 将领域经验固化成可复用的工作流说明。

## 核心理念

任何模型发挥最大功效都需要遵循：
1. **目标明确**、上下文给足、流程和边界写清楚
2. **先 plan，再 execute**
3. **用 skill 把领域经验固化进工作流**

## 关键实践

### Plan-First 工作流

复杂任务不直接开干，先让模型输出：
- 要读哪些文件
- 风险点是什么
- 验收标准是什么
- 分几步做
- 哪些地方需要测试覆盖

然后人工评审 plan，再让模型执行。

### Skill 的定位

Skill 不是给模型外挂能力，而是把**隐性经验显式化**：
- 测试应该怎么写
- UI 应该怎么验收
- repo bug fix 应该先看哪些文件
- 什么情况下要停止旧上下文重开

推荐的 Skill 类别：
- repo-level debugging
- 前端设计 polish
- 测试规范
- Deep Research
- 财务/数据分析
- 项目 SOP / 会议材料整理

### 搭配组件

在 Pi 中，需要以扩展形式安装以下组件来实现 plan-first + skill 工作流：
- **plan-first**: 复杂任务先写计划，再评审，再执行
- **subagent**: planner / executor / reviewer 分工
- **skill**: 把测试规范、设计规范、项目 SOP、调试流程写成可复用说明
- **context prune**: 长任务中途清理上下文，保留目标和关键状态
- **usage/cache**: 成本可见性

## 成本工程

一种现实的分工策略：
- 贵的深推理模型（如 Ring xhigh）：复杂 plan / 核心逻辑 / 最终 review
- 中等模型（如 Ring high）：普通工程推进
- 便宜快的模型（如 DeepSeek）：测试、样板代码、局部 review

## 观察与教训

在实际工程任务中，如果缺少 plan-first + skill 工作流，容易出现的问题：
- UI 状态契约没有在 plan 阶段锁死，导致控件交互出问题
- 边界情况（如 symbol 级 override）没有被测试明确覆盖
- 测试有同义反复倾向，没有真正跑状态链
- 前端 polish 缺少 design/UI skill 指导，能出但不够可交付

## 开放问题

- Plan-first 工作流在简单任务中是否会过度工程化？
- Skill 的维护成本如何平衡？
- 不同模型对 plan-first 的响应质量差异如何量化？

## 相关概念

- [[pi-coding-agent]]
- [[claude-code]]
- [[ring-2.6-1t]]
