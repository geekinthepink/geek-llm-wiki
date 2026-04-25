---
title: Codex (OpenAI)
created: 2026-04-12
updated: 2026-04-25
type: entity
tags: [agent, tools]
sources: [raw/articles/openai-harness-engineering-2026-02-11.md, raw/articles/anthropic-code-execution-with-mcp-2025-11-04.md]
---

# Codex (OpenAI)

## 概述

Codex 是 OpenAI 的 AI 编码 Agent，基于 GPT-5 等模型驱动。能够端到端完成软件开发任务，包括代码生成、测试编写、CI 配置、文档撰写、PR 评审和合并。

## 关键能力

- **代码生成**：应用逻辑、测试、基础设施、工具链
- **自主评审**：本地 + 云端多 Agent 互审，Ralph Wiggum Loop
- **工具调用**：直接使用 gh CLI、本地脚本、仓库嵌入的 skills
- **应用驱动**：通过 Chrome DevTools MCP 操作 UI、验证修复
- **可观测性**：查询日志（LogQL）、指标（PromQL）、追踪（TraceQL）
- **自主合并**：检测构建失败、回应反馈、squash & merge

## Harness Engineering 数据

| 指标 | 数值 |
|------|------|
| 生成代码量 | ~100 万行 |
| PR 处理量 | ~1,500 |
| 人工代码行 | 0 |
| 单次运行时长 | 6+ 小时 |
| 团队规模 | 3 → 7 人 |
| 人均吞吐 | 3.5 PR/人/天 |

## 架构

- 通过 Codex CLI 驱动
- 使用 GPT-5 作为基础模型
- 仓库嵌入 skills 提供上下文
- 与 Chrome DevTools Protocol 集成进行 UI 验证
- 接入本地可观测栈进行调试

## 相关实体

- [[harness-engineering]] — OpenAI 的 Agent 优先工程方法论
- [[model-context-protocol]] — Codex 这类编码 Agent 也会通过 MCP/浏览器工具接口扩展外部操作能力
- [[mcp-code-execution]] — 当工具数量和中间结果扩大时，可借鉴的上下文效率模式
