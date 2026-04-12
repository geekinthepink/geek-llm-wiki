---
title: Anthropic Engineering Blog
created: 2026-04-12
updated: 2026-04-12
type: entity
tags: [agent, evaluation]
sources: [raw/articles/anthropic-effective-harnesses-for-long-running-agents-2025-11-26.md, raw/articles/anthropic-building-c-compiler-parallel-claudes-2026-02-05.md]
---

# Anthropic Engineering Blog

## 概述

Anthropic Engineering Blog（Anthropic 工程博客）是 Anthropic 公司发布工程实践、技术洞察和产品经验的官方平台。地址：[anthropic.com/engineering](https://www.anthropic.com/engineering)

该博客主要涵盖 Anthropic 工程师在使用和开发 AI 系统过程中的实践经验，特别是关于 Agent harness 设计、长期运行 Agent、自主软件开发等前沿主题。

## 收录文章

本文档收录了以下两篇重要文章：

### 1. Effective harnesses for long-running agents（2025-11-26）

- **作者**：Justin Young
- **主题**：长期运行 Agent 的 harness 设计方法论
- **核心贡献**：
  - 初始化 Agent + 编码 Agent 双阶段架构
  - 功能列表驱动的增量进展模式
  - 跨会话上下文管理策略
  - 4 种常见失败模式及解决方案
- **关键词**：context window, compaction, initializer agent, coding agent, feature list

### 2. Building a C compiler with a team of parallel Claudes（2026-02-05）

- **作者**：Nicholas Carlini（Safeguards 团队研究员）
- **主题**：并行 Agent 团队构建 C 编译器的实践
- **核心贡献**：
  - Ralph-loop 无限循环 harness
  - 基于 git 锁的多 Agent 同步机制
  - GCC 预言机方法实现大型任务并行化
  - 多角色专业化 Agent 分配
  - 10 万行 C 编译器的完整评估
- **关键词**：agent teams, parallelism, lock mechanism, GCC oracle, delta debugging

## 与其他工程博客的对比

| 维度 | Anthropic Engineering | OpenAI Engineering |
|------|----------------------|-------------------|
| 关注点 | Agent harness、长期运行、自主开发 | Agent 脚手架、仓库知识管理 |
| 方法论 | 双 Agent 模式、并行团队 | Harness Engineering、渐进式披露 |
| 案例 | Web 应用克隆、C 编译器 | ~100 万行代码的 Agent 生成项目 |
| 核心产品 | [[claude-code]] | [[codex]] |

## 相关实体

- [[claude-code]] — Anthropic 的编码 Agent 产品
- [[anthropic-agent-harness]] — 从博客文章中总结的 Agent 执行框架方法论
- [[parallel-agent-teams]] — 从博客文章中总结的并行 Agent 协作架构
- [[harness-engineering]] — OpenAI 的 Agent 工程实践，与 Anthropic 方法论形成对比

## 参考来源

- [Engineering at Anthropic](https://www.anthropic.com/engineering) — 官方工程博客首页
- [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) — 2025-11-26
- [Building a C compiler with a team of parallel Claudes](https://www.anthropic.com/engineering/building-c-compiler) — 2026-02-05
