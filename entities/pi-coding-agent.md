---
title: Pi Coding Agent
created: 2026-05-24
updated: 2026-05-24
type: entity
tags: [agent, tools, agent]
sources: [raw/articles/wquguru-pi-coding-agent-guide-2026-05-18.md]
---

# Pi Coding Agent

## 概述

Pi Coding Agent 是一个 **minimal core** 的 AI 编程代理框架，由 earendil-works 开发。与 Claude Code 等开箱即用的产品化环境不同，Pi 采用组件化设计哲学——核心极简，能力以扩展形式按需安装。

核心理念：用户自主决定装哪些组件、给多少上下文、什么时候使用 high/xhigh 推理档、哪些工具进入 prompt。代价是需要会配置；收益是测试模型时更透明、更可控、更容易复现。

## 关键事实

### 架构组成
- **CLI**: `@earendil-works/pi-coding-agent`
- **核心**: `pi-agent-core`
- **多 Provider 支持**: `pi-ai`
- **内置 Tools**: 极少，基本只有读写文件 + grep/find/ls

### 扩展体系（四类）
1. **TypeScript Extensions**: 用代码挂生命周期事件，对应 Claude Code hooks，但可写逻辑
2. **Skills**: SKILL.md + 脚本，与 Claude Code skills 同类
3. **Prompt Templates**: 对应 Claude Code slash commands
4. **Pi Packages**: 通过 `pi install npm:<pkg>` 或 `pi install git:<repo>` 安装，支持 `-l` 项目级安装

### 配置位置
- 主配置：`~/.pi/agent/`
- 模型配置：`models.json`（自定义 provider 需手填）

### 上下文占用
- 注册的所有 Tools 约只占 7.7k 上下文

### 模型支持
- 内置 Provider：DeepSeek 等（只需给 key，不要手配 models.json）
- 自定义 Provider：任意 OpenAI 兼容端点，需手填 models.json
- 支持 fallback chain

### 与 Claude Code 的差异
| 维度 | Pi | Claude Code |
|------|----|-------------|
| 设计哲学 | minimal core + 按需扩展 | 全功能产品化 |
| 模型开放性 | 任意 OpenAI 兼容 provider | Anthropic 为主 |
| 上下文控制 | 用户自主决定 | 系统管理 |
| 持久记忆 | 无 CLAUDE.md 式持久记忆 | 有 |
| 权限引擎 | 无内置规则引擎 | 有 |
| Plan gate | 无强制只读 gate | 有 |
| Marketplace | 无官方 marketplace | 有 |
| Subagent | 支持（@tintinweb/pi-subagents） | 内置 |
| MCP | 支持（pi-mcp-adapter） | 内置 |
| Context prune | 支持（pi-context-prune） | 内置 |

### 推荐最小可用栈
- pi-mcp-adapter
- pi-web-access
- @tintinweb/pi-subagents
- @ff-labs/pi-fff
- pi-context-prune

## 与其他实体的关系

- [[claude-code]] — 对比对象，Pi 的定位是可替代但更可控的 agent 底座
- [[ring-2.6-1t]] — 开源大模型，Pi 中测试的典型模型
- [[plan-first-workflow]] — Pi 中推荐的工作流模式

## 来源

- GitHub: https://github.com/earendil-works/pi
- NPM: @earendil-works/pi-coding-agent
- 指南文章: WquGuru, 2026-05-18
