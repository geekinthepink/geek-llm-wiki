---
title: Agent 框架与模型协同进化
created: 2026-04-28
updated: 2026-04-28
type: concept
tags: [agent, tools, memory, multi-agent, evaluation]
sources: [raw/articles/luofuli-interview-ai-paradigm-shift-2026-04-24.md]
---

# Agent 框架与模型协同进化

## 定义

Agent 框架与模型协同进化，是指模型能力和 Agent 框架设计互相增强：强模型可以重构框架、生成 Skills、改进 memory 和 workflow；改进后的框架又能放大中层模型能力，并产生新的轨迹数据用于 [[agent-post-training-paradigm]]。

## 访谈中的表现

罗福莉描述了一个典型循环：先用 Claude Opus 4.6 高强度使用和改造 [[openclaw]]，重写 memory 系统与 multi-agent 逻辑；随后把改造后的框架接入 Sonnet、国内模型和 [[mimo]]，发现中层模型也能在许多任务上表现接近顶尖模型。

这说明 Agent 框架不是静态壳，而是模型能力的一部分。框架中的 memory、Skills、工具编排、消息通道、主动任务和评估机制，都会改变模型实际可完成任务的边界。

## 当前知识状态

在 Chat 时代，模型能力主要由参数、数据和对齐决定；在 Agent 时代，实际能力由“模型 + 框架 + 工具 + 记忆 + 环境 + 评估”共同决定。一个较弱模型接入强框架，可能超过强模型在弱交互界面中的实际生产力。

## 开放问题

- 哪些框架能力应固化为模型内在能力，哪些应保留在外部系统？
- 多 Agent workflow 如何自我迭代，而不引入不可控复杂度？
- memory 分层如何避免噪声积累和隐私风险？
- [[claude-code]] 这类黑盒框架与开源框架的能力差距会如何演化？

## 相关概念

- [[harness-engineering]]
- [[parallel-agent-teams]]
