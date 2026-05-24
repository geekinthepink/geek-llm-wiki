---
title: Pi vs Claude Code
created: 2026-05-24
updated: 2026-05-24
type: comparison
tags: [agent, comparison]
sources: [raw/articles/wquguru-pi-coding-agent-guide-2026-05-18.md]
---

# Pi vs Claude Code 对比

## 对比概述

Pi Coding Agent 和 Claude Code 都是 AI 编程代理工具，但设计理念和使用场景有显著差异。Pi 定位为一个更可拆的 agent 底座，Claude Code 则是产品化的一整套工程环境。

## 对比维度

| 维度 | Pi | Claude Code |
|------|----|-------------|
| 设计哲学 | minimal core + 按需扩展 | 全功能产品化，开箱即用 |
| 核心内置工具 | 极少（读写文件 + grep/find/ls） | 丰富 |
| 上下文占用（Tools） | ~7.7k | 更高 |
| 模型开放性 | 任意 OpenAI 兼容 provider | 以 Anthropic 为主 |
| Provider 配置 | 内置只需 key，自定义需手配 models.json | 系统管理 |
| Fallback chain | 支持 | 不支持 |
| Subagent | 扩展支持（@tintinweb/pi-subagents） | 内置 |
| MCP | 扩展支持（pi-mcp-adapter） | 内置 |
| Context prune | 扩展支持（pi-context-prune） | 内置 |
| Skills | SKILL.md + 脚本 | SKILL.md + 脚本 |
| 生命周期事件 | TypeScript Extensions（可写逻辑） | Hooks（声明式 JSON） |
| Prompt Templates | 扩展 | Slash commands |
| 包安装 | `pi install npm:<pkg>` / `git:<repo>` | 内置 marketplace |
| 持久记忆 | 无 CLAUDE.md 式 | CLAUDE.md 支持 |
| 权限规则引擎 | 无 | 有 |
| Plan gate | 无强制只读 gate | 有 |
| 上下文控制 | 用户自主决定装什么、给多少 | 系统管理 |
| 发现 UI / Marketplace | 无 | 有 |

## 能力完备性

Pi 具备的能力（以扩展形式）：
- subagent ✓
- plan ✓
- MCP ✓
- web access ✓
- context prune ✓

Pi 缺失的能力：
- CLAUDE.md 式持久记忆 ✗
- 权限规则引擎 ✗
- 强制只读 Plan gate ✗
- 官方 marketplace 和发现 UI ✗

## 适用场景

### 适合用 Pi 的场景
- 需要干净地测试非 Anthropic 模型
- 需要控制上下文预算
- 需要接任意 OpenAI 兼容模型
- 需要更透明、更可控、更容易复现的 agent 环境
- 愿意花时间配置

### 适合用 Claude Code 的场景
- 开箱即用，不想折腾配置
- 需要完整的权限/sandbox 体系
- 需要强安全边界的场景
- 不想关心工具注入、生命周期事件等底层细节

## 结论

Pi 的价值不是让 Claude Code 用户换一个更省心的工具，而是给开源模型一个更干净、更透明、更可控的 Agent 运行环境。真正的关键不是"模型是否无所不能"，而是"目标、上下文、工具、plan、skill 和验收标准有没有配好"。

## 相关

- [[pi-coding-agent]]
- [[claude-code]]
- [[plan-first-workflow]]
