---
title: Ralph Loop (Everything is a Ralph Loop)
created: 2026-04-12
updated: 2026-04-12
type: concept
tags: [agent, architecture, tools]
sources: [raw/articles/geoffrey-huntley-everything-is-ralph-loop-2026-01-17.md, raw/articles/anthropic-building-c-compiler-parallel-claudes-2026-02-05.md]
---

# Ralph Loop：一种进化的软件构建哲学

## 定义

Ralph Loop 是由 Geoffrey Huntley 提出的一种**根本性的软件构建方法论**。它将软件开发视为一个**编程循环**的过程，而非传统的逐行编写（Brick by Brick）。核心隐喻是“陶轮上的粘土”——将软件需求放入循环中，如果结果不完美，就扔回去重新塑形，直到满意为止。

这一概念命名自《辛普森一家》中的 Ralph Wiggum，象征着**执着迭代**的精神。

## 核心原则

### 1. 单体架构优于微服务 (Monolithic over Microservices)
在当前阶段，多 Agent 协作（Multi-agent）往往带来复杂的非确定性问题。Ralph Loop 提倡：
- **单一进程、单一仓库**：像单体应用一样运行，垂直扩展。
- **一次一事**：每个循环专注完成一个具体任务。
- **避免 Agent 间通信开销**：直接通过文件系统（Git 历史、进度文件）传递状态。

### 2. 上下文工程与分配 (Context Allocation)
- **分配数组**：预先分配好所需的底层规格（如 `init.sh`, 功能列表）。
- **循环目标**：给 Agent 一个目标，不断循环执行，直到目标达成。
- **失败即学习**：观察循环中的失败域，修复它，让其在未来的循环中永不再发生。

### 3. 软件开发已死，软件工程永生
- **传统编程已死**：像搭积木（Jenga）一样手动写代码的时代结束了。
- **软件工程师崛起**：我们需要的是懂得编程“新计算机”（即 LLM Loop）的工程师，而不是只会写语法的打字员。
- **进化软件 (Evolutionary Software)**：软件应像生物一样，具备自动修复、自动部署、自动优化的能力。

### 4. 通用模式 (Generic Pattern)
Ralph 不仅适用于写代码，它适用于所有任务。无论是通过提示手动循环，还是自动化脚本循环（如 `while true; do ...`），核心都是**利用上下文工程榨取模型的最大价值**。

## 与 Anthropic Harness 的对比

| 维度 | Geoffrey Huntley 的 Ralph Loop | Anthropic Harness |
| :--- | :--- | :--- |
| **哲学** | 进化软件、自动修复、软件工厂 | 长期运行 Agent 的工程规范、双 Agent 模式 |
| **架构** | 强调单体、反对过早微服务化 | Docker 容器隔离、并行 Agent 团队 |
| **自动化** | 强调全自动 AFK（Away From Keyboard）运行 | 强调人类通过提示词 Steering |
| **状态传递** | Git + 循环内的上下文重置 | Git + `claude-progress.txt` + 显式功能列表 |

## 相关概念

- [[harness-engineering]] — OpenAI 的 Agent 工程实践，与 Ralph Loop 殊途同归
- [[anthropic-agent-harness]] — Anthropic 的长期运行 Agent 框架
- [[claude-code]] — 常见的 Ralph Loop 执行载体
