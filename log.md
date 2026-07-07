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

## [2026-04-16] lint | Wiki health check — 3 issues found
- Structural issues:
  - Orphan page: concepts/ralph-loop-philosophy.md (no inbound wikilinks)
  - entities/codex.md has only 1 valid outbound wikilink
  - concepts/harness-engineering.md has only 1 valid outbound wikilink
- No broken wikilinks, no index gaps, no frontmatter issues, no unknown tags, no missing sources, no oversized pages

## [2026-04-16] ingest | Hermes skill implementation analysis
- Source: internal session analysis / code reading
- Session: 20260415_104848_cfaec570
- Raw saved: raw/articles/hermes-skill-implementation-analysis-2026-04-15.md
- Diagram saved: assets/diagrams/hermes-skill-sequence-2026-04-16.svg, assets/diagrams/hermes-skill-sequence-2026-04-16.png
- Concept created: concepts/hermes-skill-implementation.md
- Tags: agent, tools, planning

## [2026-04-16] update | index.md
- Added 1 new concept: hermes-skill-implementation
- Updated total pages count to 10

## [2026-04-16] update | cross-links for hermes-skill-implementation
- Updated concepts/harness-engineering.md to link Hermes skill page
- Updated concepts/anthropic-agent-harness.md to compare against Hermes skill loading model

## [2026-04-16] create | queries/harness-engineering-book-reading-index.md
- Type: query
- Summary: 《驾驭工程》本地阅读索引，汇总源码书稿入口、推荐阅读路径，以及后续学习笔记同步方式
- Sources: raw/books/harness-engineering-from-cc-to-ai-coding/book/src/SUMMARY.md, raw/books/harness-engineering-from-cc-to-ai-coding/book/src/preface.md
- Tags: agent, tools, planning, tutorial

## [2026-04-16] update | index.md
- Added 1 new query: harness-engineering-book-reading-index
- Updated total pages count to 11

## [2026-04-25] ingest | Anthropic code execution with MCP
- Source: https://www.anthropic.com/engineering/code-execution-with-mcp
- Author: Adam Jones, Conor Kelly, 2025-11-04
- Raw saved: raw/articles/anthropic-code-execution-with-mcp-2025-11-04.md (translated to Chinese, with English source backup)
- Concept created: concepts/mcp-code-execution.md
- Entity created: entities/model-context-protocol.md
- Tags: agent, tools, architecture

## [2026-04-25] update | index.md
- Added 1 entity: model-context-protocol
- Added 1 concept: mcp-code-execution
- Updated total pages count to 13

## [2026-04-25] update | cross-links for MCP ingestion
- Updated entities/claude-code.md to link model-context-protocol and mcp-code-execution
- Updated entities/codex.md to add missing outbound wikilinks and relate MCP context-efficiency pattern to coding agents

## [2026-04-28] ingest | 罗福莉访谈：AI范式已然巨变
- Source: https://mp.weixin.qq.com/s/zqnJuv5OVsNGEefM7RguqQ / 张小珺Jùn｜商业访谈录 episode 138
- Author: 张小珺、罗福莉, 2026-04-24
- Raw saved: raw/articles/luofuli-interview-ai-paradigm-shift-2026-04-24.md
- Summary created: summaries/luofuli-interview-ai-paradigm-shift.md
- Web page created: assets/web/luofuli-interview-analysis.html
- Entities created: entities/luo-fuli.md, entities/openclaw.md, entities/mimo.md
- Concepts created: concepts/agent-post-training-paradigm.md, concepts/agent-framework-model-coevolution.md
- Tags: agent, llm, training, tools, memory, multi-agent, evaluation, prediction

## [2026-05-24] ingest | Pi Coding Agent 最全面指南（完美支持/goal）
- Source: https://x.com/i/status/2056235143623495975
- Author: WquGuru (@wquguru), 2026-05-18
- Raw saved: raw/articles/wquguru-pi-coding-agent-guide-2026-05-18.md
- Entity created: entities/pi-coding-agent.md
- Entity created: entities/ring-2.6-1t.md
- Concept created: concepts/plan-first-workflow.md
- Comparison created: comparisons/pi-vs-claude-code.md
- Tags: agent, llm, tools, planning, comparison

## [2026-06-04] ingest | Claude Code Dynamic Workflows
- Source: https://x.com/trq212/status/2061907337154367865
- Author: Thariq Shihipar (@trq212), 2026-06-03
- Raw saved: raw/articles/x-trq212-dynamic-workflows-claude-code-2026-06-03.md
- Concept created: concepts/claude-code-dynamic-workflows.md (Chinese translation)
- Tags: agent, multi-agent, tools, planning
- Key concepts: 动态工作流（Agent 即时自写 harness）、6 种编排模式（Classify-and-act / Fan-out-and-synthesize / Adversarial verification / Generate-and-filter / Tournament / Loop until done）、失败模式（Agent 惰性 / 自我偏好偏差 / 目标漂移）、使用场景（迁移重构、深度研究、排序、分诊等）

## [2026-06-04] update | index.md
- Added 1 new concept: claude-code-dynamic-workflows
- Updated total pages count to 24

## [2026-06-04] create | concepts/claude-code-dynamic-workflows-guide.md
- Type: tutorial
- Source: Claude Code 官方文档 "Orchestrate subagents at scale with dynamic workflows"
- Full Chinese translation of the official Claude Code dynamic workflows documentation page
- Converted React components (<Note>, <Steps>, <Step>) to standard Markdown
- Added wikilinks to [[claude-code-dynamic-workflows]], [[subagent]], [[skill]], [[agent-framework-model-coevolution]], [[plan-first-workflow]]
- Tags: agent, multi-agent, tools, planning, tutorial
- Sources: raw/articles/claude-code-dynamic-workflows-docs-2026-06-04.md

## [2026-06-04] update | index.md
- Added new "Tutorials" section with 1 entry: claude-code-dynamic-workflows-guide
- Updated total pages count to 25

## [2026-07-05] ingest | A Field Guide to Fable: Finding Your Unknowns
- Source: https://x.com/trq212/status/2073100352921215386
- Author: Thariq (@trq212), 2026-07-03
- Raw saved: raw/articles/x-trq212-fable-unknowns-2026-07-03.md
- Concept created: concepts/finding-unknowns.md (Chinese translation)
- Tags: agent, planning, tutorial
- Key concepts: 地图不是领土、四种未知类型（已知的已知/未知、未知的已知/未知）、实现前盲点扫描/头脑风暴/访谈/参考资料/实现计划、实现中实现笔记、实现后推介解释/测验

## [2026-07-08] ingest | ClaudeDevs - Getting started with loops
- Source: https://x.com/i/status/2074208949205881033
- Author: ClaudeDevs (@ClaudeDevs), 2026-07-06
- Concept created: concepts/claude-code-loops.md (Chinese translation)
- Tags: agent, tools, tutorial
- Key concepts: 四种循环类型（基于轮次/基于目标/基于时间/主动循环）、触发方式与停止条件、Token 使用管理、代码质量维护、SKILL.md 验证编码、/goal /loop /schedule 命令
