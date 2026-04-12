# Wiki Schema

## Domain

AI Agent、大语言模型（LLM）、前端技术。覆盖模型架构、Agent 框架、工具生态、前端框架与工程实践等。

## Conventions

- 文件名：小写 + 连字符，无空格（例：`transformer-architecture.md`）
- 每个知识页必须以 YAML frontmatter 开头（见下方模板）
- 使用 `[[wikilinks]]` 在页面间链接（每页至少 2 个出站链接）
- 更新页面时必须 bump `updated` 日期
- 新建页面必须添加到 `index.md` 对应分类下
- 每次操作必须追加到 `log.md`

## Frontmatter

```yaml
---
title: Page Title
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: entity | concept | comparison | query | summary
tags: [from taxonomy below]
sources: [raw/articles/source-name.md]
---
```

## Tag Taxonomy

> 每个页面的 tag 必须来自此分类体系。新增 tag 需先在此注册。

### LLM / 模型
- `llm` — 大语言模型相关
- `architecture` — 模型架构（Transformer, MoE, RNN 等）
- `training` — 训练方法、数据、分布式
- `fine-tuning` — LoRA, QLoRA, SFT, DPO, RLHF 等
- `inference` — 推理优化、量化、部署
- `benchmark` — 评测基准与结果
- `multimodal` — 多模态（视觉、语音、视频）

### Agent
- `agent` — AI Agent 系统与框架
- `planning` — Agent 规划、推理链、ReAct
- `tools` — 工具调用、Function Calling、MCP
- `memory` — Agent 记忆机制、RAG
- `multi-agent` — 多 Agent 协作
- `evaluation` — Agent 评测

### 前端技术
- `frontend` — 前端技术总类
- `framework` — 框架（React, Vue, Svelte, Angular 等）
- `build-tool` — 构建工具（Vite, Webpack, esbuild 等）
- `css` — CSS、Tailwind、设计系统
- `performance` — 性能优化
- `typescript` — TypeScript 类型系统
- `testing` — 测试（Vitest, Playwright, Cypress）
- `web-standard` — Web 标准、浏览器 API

### 元标签
- `comparison` — 对比分析
- `tutorial` — 教程/指南
- `controversy` — 争议话题
- `prediction` — 趋势预测
- `timeline` — 时间线/历史

## Page Thresholds

- **创建页面**：当实体/概念在 2+ 来源中出现，或是单个来源的核心主题
- **更新已有页面**：当新来源提及已覆盖内容时追加信息
- **不创建页面**：仅被一笔带过或不在领域内的内容
- **拆分页面**：超过 ~200 行时拆分为子主题并交叉链接
- **归档页面**：内容完全被替代时移入 `_archive/`，从 index 移除

## Entity Pages

每个实体一个页面（公司、框架、模型、工具、人物等）。包含：
- 概述 / 是什么
- 关键事实和数据
- 与其他实体的关系（`[[wikilinks]]`）
- 来源引用

## Concept Pages

每个概念一个页面。包含：
- 定义 / 解释
- 当前知识状态
- 开放问题或争议
- 相关概念（`[[wikilinks]]`）

## Comparison Pages

对比分析。包含：
- 对比对象和原因
- 对比维度（推荐表格形式）
- 结论或综合判断
- 来源

## Update Policy

当新信息与已有内容冲突时：
1. 检查日期 — 新来源通常替代旧来源
2. 若确实矛盾，同时记录两个观点并标注日期和来源
3. 在 frontmatter 中标记：`contradictions: [page-name]`
4. 在 lint 报告中提示用户审核
