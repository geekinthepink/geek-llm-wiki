---
title: Model Context Protocol
created: 2026-04-25
updated: 2026-04-25
type: entity
tags: [agent, tools]
sources: [raw/articles/anthropic-code-execution-with-mcp-2025-11-04.md]
---

# Model Context Protocol

## 概述

Model Context Protocol（MCP）是 Anthropic 于 2024 年 11 月推出的开放标准，用来让 AI Agent 连接外部工具、数据源和业务系统。它的目标是减少“每个 Agent 与每个工具都要单独集成”的碎片化问题，让开发者在 Agent 中实现一次 MCP 后，就能接入更广泛的 server 生态。

在 Anthropic 2025 年 11 月的文章中，MCP 已被描述为连接 Agent 与工具/数据的事实标准，并拥有大量社区 MCP server 与多语言 SDK。随着生态扩大，MCP 的核心挑战从“能否连接工具”转向“连接大量工具后如何保持 Agent 高效”。

## 关键事实

| 项目 | 内容 |
|---|---|
| 发布方 | Anthropic |
| 发布时间 | 2024-11 |
| 类型 | 开放协议 / Agent 工具连接标准 |
| 主要用途 | 连接内容仓库、业务工具、开发环境、数据库、自动化系统等 |
| 生态组件 | MCP servers、SDK、clients、host applications |
| 规模化挑战 | 工具定义和中间结果占用模型上下文 |

## 在 Agent 架构中的位置

MCP 负责“连接外部世界”，但不自动解决上下文管理问题。一个 Agent 可以通过 MCP 获得大量工具能力；如果 client 把全部工具定义直接塞进模型上下文，就会造成 token 浪费和延迟上升。[[mcp-code-execution]] 是 Anthropic 对这一问题的工程化回应：把 MCP server 暴露为代码 API，让工具发现和大结果处理尽量留在执行环境。

## 与其他实体/概念的关系

- [[mcp-code-execution]] — MCP 规模化后的上下文效率方案。
- [[claude-code]] — Anthropic 编码 Agent，可通过 MCP 扩展浏览器自动化等工具能力。
- [[anthropic-agent-harness]] — 长期运行 Agent 实践中经常使用 MCP 作为外部工具接口。
- [[hermes-skill-implementation]] — 与 MCP 代码执行一样强调渐进式披露和可复用程序性知识。

## 学习要点

MCP 的第一层价值是标准化连接；第二层挑战是连接太多之后，Agent 如何选择工具、控制上下文、处理大结果和保持安全。真正要学的是 MCP 与代码执行、沙箱、权限、日志、skill/函数复用之间的组合，而不是只把 MCP 理解成“工具调用协议”。

## 参考来源

- [Code execution with MCP: Building more efficient agents](https://www.anthropic.com/engineering/code-execution-with-mcp) — Anthropic Engineering, 2025-11-04
- [Model Context Protocol](https://modelcontextprotocol.io/) — MCP 官方站点
