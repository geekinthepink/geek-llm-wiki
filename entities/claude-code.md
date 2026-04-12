---
title: Claude Code
created: 2026-04-12
updated: 2026-04-12
type: entity
tags: [agent, tools]
sources: [raw/articles/anthropic-effective-harnesses-for-long-running-agents-2025-11-26.md, raw/articles/anthropic-building-c-compiler-parallel-claudes-2026-02-05.md]
---

# Claude Code

## 概述

Claude Code 是 Anthropic 开发的编码 Agent 产品，基于 Claude 大语言模型构建，能够在开发环境中自主执行编码任务。它是 [[anthropic-agent-harness]] 方法论的核心执行引擎。

## 核心能力

### 编码与工具使用

Claude Code 是一个通用的编码 Agent，能够：

- 使用 shell 工具执行命令（`bash`、`pwd`、`git` 等）
- 读写文件
- 操作 git 仓库（commit、push、pull、merge）
- 运行开发服务器
- 通过 MCP（Model Context Protocol）服务器集成浏览器自动化工具（如 Puppeteer）

### 上下文管理

- 支持压缩（compaction）机制，在单个会话中管理大量上下文
- 通过 `claude-progress.txt`、git 历史等机制跨会话保持状态
- 在 [[anthropic-agent-harness]] 框架下，能够快速理解项目状态并继续工作

### 长期运行能力

通过 Ralph-loop 等 harness 模式，Claude Code 可以在没有人类干预的情况下持续运行：

```bash
claude --dangerously-skip-permissions \
       -p "$(cat AGENT_PROMPT.md)" \
       --model claude-opus-X-Y
```

## 模型支持

Claude Code 支持多种 Claude 模型：

| 模型 | 特点 |
|------|------|
| Opus 4.5 | 首个能够通过大型测试套件构建功能性编译器的模型 |
| Opus 4.6 | 能够在 Agent 团队协作下从零构建 C 编译器并编译 Linux 内核 |

## 实际案例

### claude.ai 克隆项目

在 Anthropic [长期运行 Agent 的有效 Harness](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) 中描述的 Web 应用克隆项目：

- 使用初始化 Agent + 编码 Agent 双阶段架构
- 展开超过 200 个功能需求
- 使用 Puppeteer MCP 进行端到端浏览器自动化测试
- 通过增量进展模式避免一步到位的失败

### C 编译器项目

在 Anthropic [用并行 Claudes 构建 C 编译器](https://www.anthropic.com/engineering/building-c-compiler) 中的 C 编译器项目：

- 16 个 Claude 实例并行工作
- ~2,000 个 Claude Code 会话
- 20 亿输入 token，1.4 亿输出 token
- 产出 10 万行代码的 Rust C 编译器
- 能够在 x86、ARM、RISC-V 上编译 Linux 6.9

## 技术架构

### Agent Harness 集成

Claude Code 作为 Agent 的执行引擎，与以下组件集成：

- **Claude Agent SDK**：提供工具调用、上下文管理等核心能力
- **Git**：版本控制、任务同步（在 [[parallel-agent-teams]] 中用作锁机制）
- **Docker**：隔离运行环境（并行场景）
- **MCP 服务器**：扩展工具能力（浏览器自动化等）

### 权限模式

- 标准模式：需要人类确认每次工具调用
- `--dangerously-skip-permissions`：跳过权限确认，用于自主运行场景

## 与相关实体的关系

- [[anthropic-engineering-blog]] — Anthropic 工程博客，发布 Claude Code 的实践经验
- [[anthropic-agent-harness]] — 使 Claude Code 能够长期运行的方法论
- [[parallel-agent-teams]] — 使多个 Claude Code 实例并行协作的架构
- [[codex]] — OpenAI 的编码 Agent，与 Claude Code 是竞争关系
- [[harness-engineering]] — OpenAI 的 Agent 工程实践，与 Anthropic 的方法论形成对比

## 参考来源

- [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) — Anthropic Engineering Blog, 2025-11-26
- [Building a C compiler with a team of parallel Claudes](https://www.anthropic.com/engineering/building-c-compiler) — Anthropic Engineering Blog, 2026-02-05
- [GitHub: Claude's C Compiler](https://github.com/anthropics/claudes-c-compiler)
