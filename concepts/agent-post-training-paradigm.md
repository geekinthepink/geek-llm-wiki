---
title: Agent 后训练范式
created: 2026-04-28
updated: 2026-04-28
type: concept
tags: [agent, training, fine-tuning, evaluation, planning]
sources: [raw/articles/luofuli-interview-ai-paradigm-shift-2026-04-24.md]
---

# Agent 后训练范式

## 定义

Agent 后训练范式指大模型竞争从“预训练规模 + Chat 对齐”转向“围绕 Agent 长程任务进行 SFT/RL、环境复现、工具调用、trajectory 评估和框架适配”的训练范式。罗福莉在访谈中把它视为 2026 年大模型竞争的核心赛点。

## 当前知识状态

传统 Chat 后训练主要优化单轮或短多轮问答体验；Agent 后训练要处理长期目标、多工具调用、跨 session memory、代码执行、消息通道、多 Agent 协作和任务验收。它的样本不只是 prompt-response，而是完整轨迹：目标、计划、执行、失败、修复、评估和环境状态。

访谈中的关键判断是：国内团队在 Pre-train 上的代差缩小后，下一阶段要 all in Agent 的 Post-train，尤其是 Agent 上的 RL scaling。难点不是“有没有更多数据”，而是能不能构造足够真实、足够长程、reward 可定义的环境。

## 关键实践

- 构造长程任务，而不是只优化短 benchmark。
- 还原真实环境，使模型能在接近生产场景的状态中 rollout。
- 设计 reward，区分“形式完成”和“真实完成”。
- 让 [[openclaw]]、[[claude-code]] 等框架产生的使用轨迹进入训练闭环。
- 把组织经验沉淀为 Skills，补足预训练不可见的业务逻辑。

## 开放问题

- 如何低成本重跑 128K 到 1M 上下文的 trajectory？
- 如何为复杂任务设计可泛化评估，而不是只防止致命错误？
- 如何避免模型学会迎合 reward，而不真正提升任务完成质量？
- 如何让 [[mimo]] 这类模型在多种 Agent 框架中稳定迁移？

## 相关概念

- [[agent-framework-model-coevolution]]
- [[harness-engineering]]
