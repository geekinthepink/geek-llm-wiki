---
title: Anthropic Agent Harness
created: 2026-04-12
updated: 2026-04-12
type: concept
tags: [agent, tools, planning]
sources: [raw/articles/anthropic-effective-harnesses-for-long-running-agents-2025-11-26.md, raw/articles/anthropic-building-c-compiler-parallel-claudes-2026-02-05.md]
---

# Anthropic Agent Harness

## 概述

Anthropic Agent Harness 是 Anthropic 工程团队在长期运行 AI Agent 的实践中总结出的一系列方法论和架构模式。这些实践使 [[claude-code]] 等编码 Agent 能够跨多个上下文窗口持续工作，并在没有人类主动干预的情况下完成复杂项目。

核心挑战：Agent 必须在离散会话中工作，每个新会话开始时都没有之前会话的记忆。如何桥接会话之间的差距，使 Agent 能够持续、增量地取得进展？

## 核心架构：初始化 + 编码双 Agent 模式

来自 [[anthropic-effective-harnesses-for-long-running-agents-2025-11-26]] 的两阶段设计：

### 1. 初始化 Agent（Initializer Agent）

首个 Agent 会话使用专门的提示，负责搭建整个项目的基础环境：

- **`init.sh` 脚本**：可运行开发服务器并在实现新功能前执行基本端到端测试
- **`claude-progress.txt` 进度文件**：记录 Agent 行为的日志，使后续会话快速了解工作状态
- **功能列表文件（`feature_list.json`）**：将用户需求展开为结构化的功能描述，每个功能初始标记为 `passes: false`
- **初始 git 提交**：展示添加了哪些文件，建立版本控制历史

```json
{
    "category": "functional",
    "description": "New chat button creates a fresh conversation",
    "steps": ["...", "..."],
    "passes": false
}
```

关键设计决策：使用 JSON 而非 Markdown 存储功能列表，因为模型不太可能不当地更改 JSON 文件。

### 2. 编码 Agent（Coding Agent）

每个后续会话遵循固定的启动流程：

1. 运行 `pwd` 确认工作目录
2. 阅读 git 日志和进度文件了解最近工作
3. 阅读功能列表，选择最高优先级的未完成功能
4. 启动开发服务器并验证基本功能是否正常
5. 实现新功能
6. 提交 git 并更新进度文件

编码 Agent 每次只处理**一个功能**，这是防止 Agent 试图一步到位完成整个应用的关键约束。

## Ralph-loop：无限循环模式

来自 [[anthropic-building-c-compiler-parallel-claudes-2026-02-05]] 的简单循环 harness：

```bash
#!/bin/bash
while true; do
    COMMIT=$(git rev-parse --short=6 HEAD)
    LOGFILE="agent_logs/agent_${COMMIT}.log"
    claude --dangerously-skip-permissions \
           -p "$(cat AGENT_PROMPT.md)" \
           --model claude-opus-X-Y &> "$LOGFILE"
done
```

关键特征：
- Agent 完成任务后自动开始下一个，无需人类干预
- 每次会话使用当前 git commit hash 作为日志文件名
- 配合 AGENT_PROMPT.md 中的指示：分解问题为小块、跟踪工作、确定下一步

## 上下文管理策略

### 避免上下文窗口污染

- 测试 harness 最多打印几行输出，将详细信息记录到文件
- 错误日志格式化为 `ERROR: 原因` 在同一行，便于 grep
- 预先计算汇总统计数据，避免 Agent 重新计算

### 克服时间盲症（Time blindness）

- Agent 无法感知时间，可能花数小时运行测试
- 使用 `--fast` 选项运行 1%-10% 的随机测试样本
- 子样本对每个 Agent 是确定性的，但在虚拟机之间随机

### 跨会话持久化

Agent 通过以下机制在会话间传递状态：

| 机制 | 用途 |
|------|------|
| Git 历史 | 记录代码变更、支持回退 |
| `claude-progress.txt` | 进度摘要和当前状态 |
| `feature_list.json` | 功能完成状态跟踪 |
| README 和进度文件 | 项目上下文和架构说明 |

## 测试驱动开发

### 端到端测试

- 使用浏览器自动化工具（如 Puppeteer MCP）以人类用户方式验证功能
- 每个会话开始时必须验证基本功能是否正常工作
- 只有经过仔细测试后才能将功能标记为"通过"

### 持续集成

- 构建 CI 管道防止新提交破坏现有代码
- 更严格的执行机制确保 Agent 正确测试其工作

## 失败模式与解决方案

| 失败模式 | 解决方案 |
|----------|----------|
| Agent 过早宣布项目完成 | 功能列表文件 + 逐个功能处理 |
| 环境留有 bug 或未记录进展 | Git 提交 + 进度文件 + 基本测试 |
| 功能未充分测试即标记完成 | 端到端测试要求 + 浏览器自动化 |
| Agent 不知道如何运行应用 | `init.sh` 启动脚本 |
| 合并冲突 | Agent 自行处理 git merge，通常能正确解决 |

## 与 OpenAI Harness Engineering 的对比

Anthropic 的方法与 OpenAI 的 [[harness-engineering]] 有显著差异：

| 维度 | Anthropic | OpenAI |
|------|-----------|--------|
| 知识管理 | `claude-progress.txt` + git 历史 | AGENTS.md + `docs/` 目录 |
| 上下文策略 | 每次新会话从零开始 | 渐进式披露（progressive disclosure） |
| 架构约束 | 通过提示词和文件结构 | 自定义 linter + 分层架构验证 |
| 并行化 | Docker 容器 + git 锁机制 | 多 worktree |
| 测试 | 端到端浏览器自动化 | CI/CD + linter 验证 |

## 开放问题

- 单一通用编码 Agent 与多 Agent 架构哪个更优？
- 如何将 Web 应用开发的经验推广到科学研究、金融建模等其他领域？
- 完全自主开发的质量保证策略是什么？
- 随着模型能力提升，harness 设计将如何演化？

## 相关概念

- [[claude-code]] — Anthropic 的编码 Agent
- [[harness-engineering]] — 以 Agent 为核心的软件工程范式
- [[parallel-agent-teams]] — 并行 Agent 协作架构
- [[anthropic-engineering-blog]] — Anthropic 工程博客
