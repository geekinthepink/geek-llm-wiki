---
title: Parallel Agent Teams
created: 2026-04-12
updated: 2026-04-12
type: concept
tags: [agent, multi-agent, planning, evaluation]
sources: [raw/articles/anthropic-building-c-compiler-parallel-claudes-2026-02-05.md]
---

# Parallel Agent Teams（并行 Agent 团队）

## 定义

Parallel Agent Teams 是一种多 Agent 协作架构，其中多个 AI Agent 实例在**没有人类主动干预**的情况下，并行地在同一个共享代码库上工作。这种方法极大地扩展了 LLM Agent 能够完成的项目规模和复杂度。

典型案例：使用 16 个 Claude Agent 实例并行工作，在两周内、消耗约 2,000 个会话和 20,000 美元 API 成本，从零构建了一个 10 万行代码的 C 编译器，能够在 x86、ARM 和 RISC-V 上编译 Linux 6.9。

## 架构设计

### 基础设施

```
                    ┌─────────────┐
                    │ Bare Git    │
                    │ Upstream    │
                    │ Repository  │
                    └──────┬──────┘
                           │ push/pull
            ┌──────────────┼──────────────┐
            │              │              │
     ┌──────▼──────┐ ┌────▼─────┐ ┌──────▼──────┐
     │ Docker      │ │ Docker   │ │ Docker      │
     │ Container 1 │ │ Cont. 2  │ │ Container N │
     │ /workspace  │ │          │ │             │
     │ Agent A     │ │ Agent B  │ │ Agent N     │
     └─────────────┘ └──────────┘ └─────────────┘
```

- **裸 git 仓库（bare git repo）**作为上游（upstream），挂载到每个容器的 `/upstream`
- 每个 Agent 在独立的 Docker 容器中运行，克隆本地副本到 `/workspace`
- 完成后从本地容器推送到上游

### 任务同步：基于 Git 的锁机制

为了防止多个 Agent 同时处理相同的任务，使用简单的文件系统锁：

1. **加锁**：Agent 通过在 `current_tasks/` 目录下写入文本文件来声明任务（如 `current_tasks/parse_if_statement.txt`）
2. **冲突解决**：如果两个 Agent 尝试认领同一任务，git 的同步机制强制第二个 Agent 选择另一个
3. **解锁**：Agent 完成工作后，拉取上游更改、合并、推送、然后移除锁文件

### 无编排 Agent（No Orchestration Agent）

当前实现的一个关键特征是**不使用编排 Agent**。每个 Agent 自行决定下一步做什么：

- 在大多数情况下，Agent 选取"下一个最明显"的问题
- 当卡在 bug 上时，Agent 维护一个记录失败方法和剩余任务的文档
- Agent 自行处理合并冲突

## 并行化策略

### 测试驱动的并行化

当测试套件中有许多失败的测试时，并行化是自然的：每个 Agent 选择不同的失败测试来处理。

### 大型任务的并行化挑战

编译 Linux 内核是一个单一的巨型任务，所有 Agent 会碰到相同的 bug。解决方案：

**GCC 预言机（Oracle）方法**：
1. 随机选择大部分文件用 GCC 编译
2. 仅用 Claude 的编译器编译剩余文件
3. 如果内核正常工作 → 问题不在 Claude 的子集中
4. 如果内核失败 → 进一步细化，用 GCC 重新编译部分文件
5. 结合增量调试（delta debugging）定位同时失败的文件对

### 专业化角色分配

并行化使 Agent 专业化成为可能：

| 角色 | 职责 |
|------|------|
| 代码整合 Agent | 合并发现的重复代码 |
| 性能优化 Agent | 提高编译器本身性能 |
| 代码生成优化 Agent | 输出更高效的编译代码 |
| 架构审查 Agent | 从 Rust 开发者角度批判设计并做结构性改进 |
| 文档 Agent | 维护和更新文档 |

## 关键设计原则

### 1. 极高质量的测试

- 任务验证器必须几乎完美，否则 Agent 会解决错误的问题
- 持续集成管道防止新提交破坏现有代码
- 测试需要覆盖：编译器测试套件、开源软件包编译验证、回归测试

### 2. 上下文窗口友好

- 测试输出应最小化，详细信息写入日志文件
- 错误信息格式化为一行，便于 grep
- 预计算汇总统计，避免 Agent 重复计算

### 3. 时间管理

- Agent 无法感知时间流逝
- 使用 `--fast` 选项运行测试子样本（1%-10%）
- 确定性子样本确保每个 Agent 能精确识别回归

### 4. 自描述环境

- 维护详细的 README 和进度文件
- 频繁更新当前状态
- 帮助新 Agent 在没有上下文的情况下快速定位

## 评估数据

| 指标 | 数值 |
|------|------|
| Agent 数量 | 16 |
| Claude Code 会话数 | ~2,000 |
| 开发周期 | 2 周 |
| 输入 Token | 20 亿 |
| 输出 Token | 1.4 亿 |
| API 成本 | ~$20,000 |
| 代码行数 | ~100,000 行 |
| 目标模型 | Opus 4.6 |

### 成果

- 在 x86、ARM、RISC-V 上构建可启动的 Linux 6.9
- 编译 QEMU、FFmpeg、SQLite、PostgreSQL、Redis
- GCC torture test suite 99% 通过率
- 可以编译并运行 Doom

### 局限性

- 缺少 16 位 x86 代码生成器（调用 GCC）
- 没有自己的汇编器和链接器
- 生成的代码效率低于禁用优化的 GCC
- 代码质量不及专业 Rust 程序员

## 与 [[anthropic-agent-harness]] 的关系

Parallel Agent Teams 建立在 [[anthropic-agent-harness]] 的核心方法之上，增加了以下能力：

| 能力 | Anthropic Agent Harness | Parallel Agent Teams |
|------|------------------------|---------------------|
| 会话管理 | 初始化 + 编码双 Agent | 多 Agent 并行循环 |
| 任务协调 | 功能列表驱动 | Git 锁机制 + 自选择 |
| 并发 | 单 Agent 顺序执行 | 多容器并行执行 |
| 专业化 | 单一通用编码 Agent | 多角色专业化 Agent |
| 编排 | 无编排 Agent | 无编排 Agent（自组织） |

## 开放问题

- 无编排 Agent 的自组织策略在更大规模（100+ Agent）下是否仍然有效？
- 如何平衡专业化与通用化？
- Agent 间的隐式协调（通过 git）是否足够，还是需要显式通信机制？
- 如何将此方法应用于非编译类项目？

## 相关概念

- [[anthropic-agent-harness]] — Anthropic 的 Agent 执行框架方法论
- [[claude-code]] — Anthropic 的编码 Agent
- [[harness-engineering]] — 以 Agent 为核心的软件工程范式
- [[multi-agent]] — 多 Agent 协作系统
