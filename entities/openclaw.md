---
title: OpenClaw
created: 2026-04-28
updated: 2026-04-28
type: entity
tags: [agent, tools, memory, multi-agent]
sources: [raw/articles/luofuli-interview-ai-paradigm-shift-2026-04-24.md]
---

# OpenClaw

## 概述

OpenClaw 是访谈中反复讨论的开源 Agent 框架。罗福莉认为它不是简单的 Claude Code 外壳，而是通过持久 memory、消息通道、模型编排、主动任务、Skills 和可修改源码，补偿模型在行动层面的缺陷，并激发中层模型的上限。

## 关键机制

- **持久化 memory**：对记忆进行分层和分级，使跨 session 的上下文共享更稳定。
- **多模型编排**：用户不必手动指定视频理解等专用模型，框架可自行调用更合适模型。
- **主动性设计**：包括定时任务、心跳任务和消息通道，更适合日常任务而非纯代码任务。
- **Skills 生态**：把人类经验和组织规范转化为可复用执行单元。
- **开源可修改性**：相比黑盒 Agent，OpenClaw 允许使用者改 memory、multi-agent workflow 和整体框架。

## 分析判断

OpenClaw 的真正意义在于推动 [[agent-framework-model-coevolution]]：高端模型可以改造框架，改造后的框架再放大中层模型，群体使用又产生新数据和新 Skills。它与 [[claude-code]] 的差异，不只是 UI 或交互方式，而是通用任务框架和可修改性的差异。

## 相关概念

- [[agent-post-training-paradigm]]
- [[harness-engineering]]
