# Hermes skill 实现原理分析（基于 2026-04-15 晚间代码阅读）

**Source:** 内部会话分析 / Hermes 仓库代码阅读
**Session:** 20260415_104848_cfaec570
**Date:** 2026-04-15
**Scope:** `SKILL.md`、`skills_list()`、`skill_view()`、`build_skills_system_prompt()`、skill 发现/过滤/加载/注入链路

---

## 分析背景

这次分析的目标是搞清楚 Hermes 里的 skill 到底是什么、如何被发现、何时被加载，以及它怎样影响模型后续的工具调用策略。

结论上，Hermes 的 skill 不是传统意义上的“代码插件”，而更像是一种**目录化、带元数据的操作手册与工作流规范**。模型并不会在一开始就读取所有 skill 的完整内容，而是先看到一份紧凑索引，再按需逐步加载。

## 核心文件与关键函数

### 关键源码文件

- `/root/.hermes/hermes-agent/agent/skill_utils.py`
- `/root/.hermes/hermes-agent/tools/skills_tool.py`
- `/root/.hermes/hermes-agent/agent/skill_commands.py`
- `/root/.hermes/hermes-agent/agent/prompt_builder.py`
- `/root/.hermes/hermes-agent/tools/skill_manager_tool.py`
- `/root/.hermes/hermes-agent/run_agent.py`

### 关键函数

- `parse_frontmatter(content)`
- `skill_matches_platform(frontmatter)`
- `get_disabled_skill_names(platform=None)`
- `get_external_skills_dirs()`
- `get_all_skills_dirs()`
- `build_skills_system_prompt(available_tools=None, available_toolsets=None)`
- `skills_list(category=None, task_id=None)`
- `skill_view(name, file_path=None, task_id=None)`
- `_load_skill_payload(skill_identifier, task_id=None)`
- `_build_skill_message(...)`
- `skill_manage(...)`

## skill 的核心载体：目录 + SKILL.md

每个 skill 的主体是一个目录，其中主文件为 `SKILL.md`。该文件使用 **YAML frontmatter + Markdown 正文** 组织内容。

常见 frontmatter 字段包括：

- `name`
- `description`
- `platforms`
- `metadata.hermes.tags`
- `metadata.hermes.related_skills`
- `metadata.hermes.config`
- `required_environment_variables`
- `required_credential_files`

一个 skill 目录除了 `SKILL.md` 外，还可以带附属资源：

- `references/`
- `templates/`
- `assets/`
- `scripts/`

这意味着 skill 不只是“一段提示词”，而是一个可扩展的小型知识包。

## 三层机制：索引、按需加载、管理

### 1. 索引层：先给模型一个技能地图

在构建 system prompt 时，`build_skills_system_prompt()` 会扫描本地 skills、external dirs 和 plugin skills，生成一个**紧凑的技能索引**注入 system prompt。

这一层只告诉模型：

- 有哪些 skill
- skill 名称与描述是什么
- 大概适用于什么任务

也就是说，模型先看到的是目录，不是全文。

### 2. 按需加载层：progressive disclosure

Hermes skill 的读取是渐进式的：

- `skills_list()`：只返回摘要元数据（名称、描述、分类等）
- `skill_view(name)`：返回完整 skill 正文、tags、path、linked_files、setup 状态等

因此模型典型的工作方式是：

1. 先从索引里意识到可能有相关 skill
2. 再调用 `skills_list()` 看候选摘要
3. 命中后用 `skill_view()` 加载完整 `SKILL.md`
4. 依据该 skill 中的流程和规则来执行后续动作

这是典型的 **progressive disclosure（渐进式披露）** 设计。

### 3. 管理层：skill 也是可维护对象

`skill_manage()` 提供创建、编辑、patch、删除、写附属文件等管理动作。也就是说，skill 在 Hermes 里不是静态文档，而是一个可以随着实践不断修订的对象。

## system prompt 中对 skill 的强约束

Hermes 的 system prompt 对 skill 的地位设定得非常高。`prompt_builder.py` 中的技能提示明确要求：

- 回复前先扫描可用 skills
- 只要 task 与某个 skill 部分相关，就应该优先 `skill_view(name)`
- 如果 skill 有问题或过时，应使用 `skill_manage(action='patch')` 修补

因此，skill 在 Hermes 里不是“可选参考资料”，而是**强制优先检查的知识与工作流层**。

## 发现与过滤逻辑

Hermes 在发现 skill 时不只是简单遍历目录，还会做多层过滤。

### 1. 平台过滤

`skill_matches_platform(frontmatter)` 根据 frontmatter 中的 `platforms` 字段和当前 `sys.platform` 判断 skill 是否适用。

平台映射包括：

- `macos -> darwin`
- `linux -> linux`
- `windows -> win32`

### 2. 禁用配置

`get_disabled_skill_names()` 会从 `config.yaml` 中读取：

- `skills.disabled`
- `skills.platform_disabled.<platform>`

命中的 skill 不会进入可用技能集。

### 3. 外部目录

`get_external_skills_dirs()` 与 `get_all_skills_dirs()` 让 Hermes 除了读取 `~/.hermes/skills/` 外，也可从额外目录加载 skill。

### 4. 本地优先级

如果本地 skill 与 external dir 中 skill 重名，构建 skills 索引时**本地优先**，external 版本会被跳过。

### 5. 插件技能

`skill_view(name)` 支持 `plugin:skill` 命名空间形式。命中这类技能时，会转给 plugin manager 去解析；若插件被禁用，还会返回重新启用的提示。

## 配置注入与环境准备

skill 不只是说明文字，还是可携带运行前准备信息的元对象。

### 1. 配置注入

`_inject_skill_config()` 可以读取 `metadata.hermes.config`，再把 `~/.hermes/config.yaml` 中的配置值注入 skill 上下文。

### 2. 环境变量与凭证

`skill_view()` 会检查：

- `required_environment_variables`
- `required_credential_files`

返回字段中可包含：

- `setup_needed`
- `setup_skipped`
- `setup_note`
- `gateway_setup_hint`
- `missing_required_environment_variables`
- `missing_credential_files`
- `readiness_status`

这使 skill 成为一种“可带 readiness 状态的知识对象”。

## skill 与 slash command / 会话注入的关系

`agent/skill_commands.py` 中的 `_load_skill_payload()` 和 `_build_skill_message()` 表明，skill 还可以通过 slash command 直接注入会话。

拼装后的消息通常包含：

- skill 主体内容
- setup note
- supporting files
- 用户附加说明

因此 skill 不只是工具面板里一个条目，也能作为上下文构建模块被直接送进当前会话。

## 总结判断

Hermes 的 skill 体系可以概括为：

1. **可索引的知识单元**：system prompt 先注入 skills 索引
2. **按需加载的工作流规范**：`skills_list()` 看摘要，`skill_view()` 读全文
3. **带元数据与运行状态的任务模块**：支持平台过滤、config 注入、env 检查、linked files
4. **可持续维护的程序性知识**：通过 `skill_manage()` 更新与演化

如果把 OpenAI 的 `AGENTS.md + docs/` 看成“仓库级知识地图”，那么 Hermes 的 skill 更像“任务级、工具级、工作流级的模块化操作手册”。

## 与其他概念的关系

- 它和 [[harness-engineering]] 一样，都强调让 Agent 先看到可导航的知识地图，而不是一次性塞满上下文。
- 它和 [[anthropic-agent-harness]] 的共同点在于，都把“上下文管理”和“渐进式读取”当成可靠 Agent 系统的一部分。
- Hermes skill 比单纯提示模板更强，因为它附带 readiness、配置、linked files 和管理能力。
