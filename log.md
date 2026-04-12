# Wiki Log

> 所有 wiki 操作的按时间记录。只追加，不修改。
> 格式：`## [YYYY-MM-DD] action | subject`
> Actions: ingest, update, query, lint, create, archive, delete
> 超过 500 条时轮转：重命名为 log-YYYY.md，重新开始。

## [2026-04-12] create | Wiki initialized
- Domain: AI Agent、大语言模型（LLM）、前端技术
- Structure created with SCHEMA.md, index.md, log.md
- Tag taxonomy: llm, architecture, training, fine-tuning, inference, benchmark, multimodal, agent, planning, tools, memory, multi-agent, evaluation, frontend, framework, build-tool, css, performance, typescript, testing, web-standard, comparison, tutorial, controversy, prediction, timeline

## [2026-04-12] ingest | OpenAI harness engineering blog
- Source: https://openai.com/index/harness-engineering/
- Author: Ryan Lopopolo, 2026-02-11
- Raw saved: raw/articles/openai-harness-engineering-2026-02-11.md
- Concept created: concepts/harness-engineering.md
- Entity created: entities/codex.md
- Tags: agent, tools, evaluation

## [2026-04-12] ingest | Anthropic effective harnesses for long-running agents
- Source: https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
- Author: Justin Young, 2025-11-26
- Raw saved: raw/articles/anthropic-effective-harnesses-for-long-running-agents-2025-11-26.md (translated to Chinese)
- Tags: agent, tools, planning

## [2026-04-12] ingest | Anthropic building C compiler with parallel Claudes
- Source: https://www.anthropic.com/engineering/building-c-compiler
- Author: Nicholas Carlini, 2026-02-05
- Raw saved: raw/articles/anthropic-building-c-compiler-parallel-claudes-2026-02-05.md (translated to Chinese)
- Tags: agent, multi-agent, planning, evaluation

## [2026-04-12] create | concepts/anthropic-agent-harness.md
- Type: concept
- Summary: Anthropic 的 Agent 执行框架方法论总结：初始化+编码双 Agent 模式、Ralph-loop、上下文管理、跨会话持久化、失败模式与解决方案
- Tags: agent, tools, planning
- Sources: raw/articles/anthropic-effective-harnesses-for-long-running-agents-2025-11-26.md, raw/articles/anthropic-building-c-compiler-parallel-claudes-2026-02-05.md

## [2026-04-12] create | concepts/parallel-agent-teams.md
- Type: concept
- Summary: 并行 Agent 协作架构：Docker 容器隔离、git 锁同步机制、GCC 预言机方法实现大型任务并行化、多角色专业化分配
- Tags: agent, multi-agent, planning, evaluation
- Sources: raw/articles/anthropic-building-c-compiler-parallel-claudes-2026-02-05.md

## [2026-04-12] create | entities/claude-code.md
- Type: entity
- Summary: Anthropic 的编码 Agent 产品，支持长期运行与并行协作，基于 Opus 4.x 模型
- Tags: agent, tools
- Sources: raw/articles/anthropic-effective-harnesses-for-long-running-agents-2025-11-26.md, raw/articles/anthropic-building-c-compiler-parallel-claudes-2026-02-05.md

## [2026-04-12] create | entities/anthropic-engineering-blog.md
- Type: entity
- Summary: Anthropic 工程博客，收录 Agent harness、长期运行 Agent、自主软件开发等主题文章
- Tags: agent, evaluation
- Sources: raw/articles/anthropic-effective-harnesses-for-long-running-agents-2025-11-26.md, raw/articles/anthropic-building-c-compiler-parallel-claudes-2026-02-05.md

## [2026-04-12] update | index.md
- Added 3 new entities: claude-code, anthropic-engineering-blog
- Added 2 new concepts: anthropic-agent-harness, parallel-agent-teams
- Updated total pages count to 5

## [2026-04-12] ingest | Anthropic harness design for long-running application development
- Source: https://www.anthropic.com/engineering/harness-design-long-running-apps
- Author: Prithvi Rajasekaran, 2026-03-24
- Raw saved: raw/articles/anthropic-harness-design-long-running-apps-2026-03-24.md (translated to Chinese)
- Concepts created: concepts/generator-evaluator-loop.md, concepts/sprint-contract.md
- Tags: agent, multi-agent, evaluation, planning, tools
- Key concepts: 生成器-评估器循环（GAN 启发架构、四维度评分、Playwright MCP 交互评估）、冲刺契约（Generator/Evaluator 协商"完成"标准）、三 Agent 架构（Planner/Generator/Evaluator）、上下文重置 vs 压缩、评估器调优循环

## [2026-04-12] update | index.md
- Added 2 new concepts: generator-evaluator-loop, sprint-contract
- Updated total pages count to 8

## [2026-04-12] ingest | Geoffrey Huntley "everything is a ralph loop"
- Source: https://ghuntley.com/loop/
- Author: Geoffrey Huntley, 2026-01-17
- Raw saved: raw/articles/geoffrey-huntley-everything-is-ralph-loop-2026-01-17.md (translated to Chinese)
- Concept created: concepts/ralph-loop-philosophy.md
- Tags: agent, architecture, tools

## [2026-04-12] lint | Wiki health check — 0 issues found
- Fixed 8 broken wikilinks:
  - anthropic-agent-harness.md: replaced 2 raw-article wikilinks with markdown links
  - claude-code.md: replaced 2 raw-article wikilinks with markdown links
  - codex.md: removed [[agent]] broken link (page doesn't exist yet)
  - harness-engineering.md: removed [[agent]] and [[evaluation]] broken links
  - parallel-agent-teams.md: removed [[multi-agent]] broken link
- Fixed tag extraction regex (multi-agent now correctly recognized)
- All 8 pages pass frontmatter validation
- All tags in taxonomy
- No orphan pages, no index gaps, no pages over 200 lines
