---
title: 《驾驭工程》本地阅读索引
created: 2026-04-16
updated: 2026-04-16
type: query
tags: [agent, tools, planning, tutorial]
sources: [raw/books/harness-engineering-from-cc-to-ai-coding/book/src/SUMMARY.md, raw/books/harness-engineering-from-cc-to-ai-coding/book/src/preface.md]
---

# 《驾驭工程》本地阅读索引

这是一页给后续系统学习用的本地索引，方便从 wiki 直接跳到《驾驭工程 — 从 Claude Code 源码到 AI 编码最佳实践》的源码书稿，而不必每次都回到 GitHub 或在线站点。

本地源码目录：`/root/wiki/raw/books/harness-engineering-from-cc-to-ai-coding`

关键入口：
- [中文目录 SUMMARY](../raw/books/harness-engineering-from-cc-to-ai-coding/book/src/SUMMARY.md)
- [中文前言 preface](../raw/books/harness-engineering-from-cc-to-ai-coding/book/src/preface.md)
- [第一篇目录](../raw/books/harness-engineering-from-cc-to-ai-coding/book/src/part1/)
- [第六篇目录](../raw/books/harness-engineering-from-cc-to-ai-coding/book/src/part6/)
- [附录目录](../raw/books/harness-engineering-from-cc-to-ai-coding/book/src/appendix/)

## 全书结构

- 第一篇：架构 — Claude Code 如何运作
- 第二篇：提示工程 — 系统提示词作为控制面
- 第三篇：上下文管理 — 200K Token 竞技场
- 第四篇：提示词缓存 — 隐藏的成本优化器
- 第五篇：安全与权限 — 纵深防御
- 第六篇：高级子系统
- 第七篇：AI Agent 构建者的经验教训
- 附录：文件索引、环境变量、术语表、Feature Flag、版本演化、端到端案例、认证与订阅系统

## 建议阅读路径

### 路径 A：先抓主骨架

按下面顺序读，先建立 Claude Code 的整体心智模型：
1. [前言](../raw/books/harness-engineering-from-cc-to-ai-coding/book/src/preface.md)
2. [第1章：AI 编码 Agent 的完整技术栈](../raw/books/harness-engineering-from-cc-to-ai-coding/book/src/part1/ch01.md)
3. [第3章：Agent Loop](../raw/books/harness-engineering-from-cc-to-ai-coding/book/src/part1/ch03.md)
4. [第5章：系统提示词架构](../raw/books/harness-engineering-from-cc-to-ai-coding/book/src/part2/ch05.md)
5. [第9章：自动压缩](../raw/books/harness-engineering-from-cc-to-ai-coding/book/src/part3/ch09.md)
6. [第20章：Agent 派生与编排](../raw/books/harness-engineering-from-cc-to-ai-coding/book/src/part6/ch20.md)
7. [第25-27章：经验与模式总结](../raw/books/harness-engineering-from-cc-to-ai-coding/book/src/part7/)
8. [第30章：构建你自己的 AI Agent](../raw/books/harness-engineering-from-cc-to-ai-coding/book/src/part7/ch30.md)

### 路径 B：围绕你当前 wiki 已有主题读

如果想把阅读与现有知识页联动，优先读这些章节：
- [[claude-code]] → 第1章、第20章、第20b章、第21章
- [[anthropic-agent-harness]] → 第3章、第4章、第18章、第20章
- [[parallel-agent-teams]] → 第20b章、第20c章
- [[harness-engineering]] → 第25章、第26章、第27章
- [[ralph-loop-philosophy]] → 第25章、第28章

### 路径 C：偏工程实现

如果你更关心自己怎么做 Agent，优先盯这几块：
- 工具系统与执行编排
- 系统提示词与工具提示词
- 自动压缩 / 微压缩 / Token 预算
- Hooks / 权限 / 沙箱
- Teams / Ultraplan / Skills / Memory

## 做笔记时建议固定记录四类信息

每章都可以只记四件事，后面我就能稳定帮你同步进 wiki：
1. 这章讲了什么
2. 最重要的 3 个机制
3. 它和现有页面的关系（例如 [[claude-code]]、[[anthropic-agent-harness]]、[[harness-engineering]]）
4. 你自己的判断：可借鉴 / 可怀疑 / 需要实测

## 后续同步方式

你后面把章节笔记直接发我，我可以继续把内容沉淀到现有页面，或者拆成新的概念页、对比页、查询页。这个入口页主要承担“导航”和“学习节奏锚点”的作用。
