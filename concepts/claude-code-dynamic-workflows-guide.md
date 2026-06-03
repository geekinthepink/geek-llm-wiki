---
title: "大规模动态工作流：编排子代理实战指南"
created: 2026-06-04
updated: 2026-06-04
type: tutorial
tags: [agent, multi-agent, tools, planning, tutorial]
sources: [raw/articles/claude-code-dynamic-workflows-docs-2026-06-04.md]
---

# 大规模动态工作流：编排子代理实战指南

> 动态工作流（Dynamic Workflows）通过 Claude 编写、你可重复运行的脚本来大规模编排众多 subagent（子代理）。适用于代码库审计、大规模迁移、以及需要交叉验证的研究任务。

> **注意**：动态工作流目前处于研究预览阶段（research preview）。需要 Claude Code v2.1.154 或更高版本，所有付费套餐均可使用，同时也支持 Anthropic API、Amazon Bedrock、Google Cloud Vertex AI 和 Microsoft Foundry。在 Pro 套餐中，需要在 `/config` 的 "Dynamic workflows" 行手动开启。

动态工作流（Dynamic Workflow）是一个编排 [[claude-code-dynamic-workflows]] 的 JavaScript 脚本。Claude 根据你描述的任务自动编写该脚本，运行时（runtime）在后台执行它，而你的会话保持响应状态。

当任务需要的 agent（代理）数量超过单次对话能协调的范围，或者你希望将编排逻辑固化为可读、可重跑的脚本时，就应该使用工作流。典型场景包括：全代码库的 bug 排查、500 个文件的迁移、需要来源交叉核对的研究问题，以及在做出决定前从多个独立角度起草方案。

本指南涵盖以下内容：

* 判断何时使用 [工作流](#when-to-use-a-workflow) 而非子代理或 skill（技能）
* 使用 `/deep-research` [运行内置工作流](#run-a-bundled-workflow)
* 让 [Claude 为你的任务编写工作流](#have-claude-write-a-workflow) 并保存
* 理解 [工作流的运行机制](#how-a-workflow-runs) 以及 [管理工作流运行](#manage-runs)

## 何时使用工作流 {#when-to-use-a-workflow}

[[subagent]]、[[skill]]、agent team（代理团队）和工作流都能执行多步骤任务。区别在于**谁持有计划**：

|                                 | Subagent（子代理）                | Skill（技能）                      | Agent Team（代理团队）                        | Workflow（工作流）                    |
| :------------------------------ | :-------------------------------- | :--------------------------------- | :------------------------------------------- | :----------------------------------- |
| 是什么                          | Claude 派生的工作单元             | Claude 遵循的指令集               | 一个 lead agent（主导代理）监督对等会话      | 运行时执行的脚本                     |
| 谁决定下一步运行什么            | Claude，逐轮决策                  | Claude，遵循提示                  | 主导代理，逐轮决策                           | 脚本本身                             |
| 中间结果存放在哪里              | Claude 的上下文窗口（context window） | Claude 的上下文窗口           | 共享任务列表                                 | 脚本变量                             |
| 什么可重复                      | 工作单元定义                      | 指令                               | 团队定义                                     | 编排逻辑本身                         |
| 规模                            | 每轮少量委派任务                  | 与子代理相同                       | 少量长期运行的对等代理                       | 每次运行数十到数百个代理             |
| 中断行为                        | 重启当前轮次                      | 重启当前轮次                       | 队友继续运行                                 | 在同一会话内可恢复                   |

工作流将计划移入代码。使用子代理、技能或代理团队时，Claude 是编排者：它逐轮决定要派生或分配什么，每个结果都进入上下文窗口。而工作流脚本自身持有循环、分支和中间结果，因此 Claude 的上下文中只保留最终答案。

将计划移入代码还让工作流可以应用可重复的质量模式，而不仅仅是运行更多代理：它可以让独立的代理在报告前对抗性地审查彼此的发现，或者从多个角度起草方案并相互权衡，从而获得比单次通过更可靠的结果。

## 运行内置工作流 {#run-a-bundled-workflow}

了解工作流最快的方式是运行 `/deep-research`，这是 Claude Code 内置的 [工作流](#bundled-workflows)，用于从多个来源调查问题。你将看到代理在后台完成多个阶段的工作，而你的会话保持空闲，最终获得一份完整报告而非逐轮对话记录。

**1. 运行工作流**

运行 `/deep-research` 并附带你想调查的问题。它从多个角度发起网页搜索（web search），抓取并交叉核对找到的来源，最终合成一份带引用的报告。

```text
/deep-research What changed in the Node.js permission model between v20 and v22?
```

**2. 授权工作流**

Claude Code 会询问是否允许该工作流。选择 **Yes** 继续。具体提示取决于你的权限模式（permission mode）。参见 [运行前审批计划](#approve-the-plan-before-it-runs) 了解各模式的选项。

**3. 观察进度**

运行在后台启动。运行 `/workflows`，用箭头键选择该运行，然后按 Enter 打开其进度视图：

```text
/workflows
```

该视图显示每个阶段及其代理数量、总 token 数和耗时。可以深入任意阶段查看其代理及各自的发现。参见 [观察运行](#watch-the-run) 了解全部控制操作。

你也可以从输入框下方的任务面板观察：运行期间会在那里显示一行进度摘要。按向下箭头聚焦它，然后按 Enter 展开。

**4. 阅读报告**

运行完成后，报告将出现在你的会话中。报告引用了每个主张（claim）的来源，未能通过交叉核对的主张已被过滤掉。

完成上述步骤后，若要为你自己的任务运行工作流，可以 [让 Claude 编写一个](#have-claude-write-a-workflow)，当运行结果符合预期时，你可以 [将其保存](#save-the-workflow-for-reuse) 为你自己的命令。

### 内置工作流

Claude Code 内置了 `/deep-research` 作为默认工作流：

| 命令                        | 功能说明                                                                                                                                                                                                                                                                                                        |
| :-------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/deep-research <question>` | 从多个角度对一个问题发起网页搜索，抓取并交叉核对找到的来源，对每个主张投票，最终返回一份带引用的报告，未通过交叉核对的主张已被过滤。需要启用 [WebSearch 工具](/en/tools-reference#websearch-tool-behavior) 方可使用                                                                                        |

你 [自己保存的工作流](#save-the-workflow-for-reuse) 也会以同样的方式成为命令，并出现在 `/` 自动补全中，与内置工作流并列。

### 观察运行 {#watch-the-run}

工作流在后台运行，因此代理工作时会话保持响应。随时运行 `/workflows` 可以列出正在运行和已完成的工作流，然后选择一个打开其进度视图。

```text
/workflows
```

进度视图显示每个阶段的代理数量、token 总计和耗时。底部列出了每个操作的快捷键：

| 快捷键         | 操作                                                                                              |
| :------------- | :------------------------------------------------------------------------------------------------ |
| `↑` / `↓`      | 选择一个阶段或代理                                                                                |
| `Enter` 或 `→` | 深入选中的阶段，然后深入代理以阅读其提示（prompt）、最近的工具调用（tool calls）和结果            |
| `Esc`          | 返回上一级                                                                                        |
| `j` / `k`      | 在代理详情溢出时滚动查看                                                                          |
| `p`            | 暂停或恢复运行                                                                                    |
| `x`            | 停止选中的代理；当焦点在运行上时停止整个工作流                                                    |
| `r`            | 重启选中的正在运行的代理                                                                          |
| `s`            | [保存](#save-the-workflow-for-reuse) 运行的脚本为命令                                             |

## 让 Claude 编写工作流 {#have-claude-write-a-workflow}

你可以通过两种方式让 Claude 为你的任务编写工作流：

* 在提示（prompt）中 [请求工作流](#ask-for-a-workflow-in-your-prompt)，可以使用自己的措辞或包含关键字 `ultracode`，Claude 会为该任务编写工作流。
* [让 Claude 自行决定——使用 ultracode](#let-claude-decide-with-ultracode)：设置 `/effort ultracode`，Claude 会为会话中的每个实质性任务自动规划工作流。

你也可以运行已经存在的工作流命令：[内置工作流](#bundled-workflows) 如 `/deep-research`，或者你已 [保存](#save-the-workflow-for-reuse) 的工作流。

### 在提示中请求工作流 {#ask-for-a-workflow-in-your-prompt}

要在不改变会话 effort level（努力级别）的情况下以工作流运行单个任务，在提示中包含关键字 `ultracode`。使用自己的语言请求也可以，例如 "use a workflow" 或 "run a workflow"：Claude 将直接请求视为同样的 opt-in（授权）。在 v2.1.160 之前，字面触发关键字是 `workflow`；自然语言请求在两个版本中均有效。

```text
ultracode: audit every API endpoint under src/routes/ for missing auth checks
```

Claude Code 会在你的输入中高亮显示该关键字，Claude 会为该任务编写工作流脚本，而非逐轮处理。如果你无意启动工作流，在 macOS 上按 `Option+W`，在 Windows 和 Linux 上按 `Alt+W` 来取消本次提示的高亮，或者当光标在高亮关键字后面时按退格键（backspace）。要让关键字完全不再触发，可以在 `/config` 中关闭 Ultracode 关键字触发器。

如果运行结果符合预期，你可以事后 [将其保存为命令](#save-the-workflow-for-reuse)。

如果你已经用其他方式构建了编排器，例如一组子代理提示文件或一个扇出工作的 skill，你可以将该编排器指向 Claude，并要求它编写一个做同样事情的工作流。

### 让 Claude 自行决定——使用 ultracode {#let-claude-decide-with-ultracode}

Ultracode 是 Claude Code 的一个设置，它将 `xhigh` [推理努力级别](/en/model-config#adjust-effort-level) 与自动工作流编排相结合。开启后，Claude 会为每个实质性任务规划工作流，而无需你主动请求。

```text
/effort ultracode
```

开启 ultracode 后，Claude 自行判断任务何时需要工作流。单个请求可以连续触发多个工作流：一个用于理解代码，一个用于执行更改，一个用于验证。这适用于会话中的每个任务，因此每个请求比低努力级别消耗更多 token，耗时也更长。

Ultracode 仅在当前会话中有效，新建会话时会重置。回到常规工作时用 `/effort high` 降级。它仅在支持 `xhigh` [effort](/en/model-config#adjust-effort-level) 的模型上可用；其他模型的 `/effort` 菜单不提供此选项。

### 运行前审批计划 {#approve-the-plan-before-it-runs}

在 CLI 中，每次运行的提示会显示计划的阶段以及以下选项：

* **Yes, run it**：开始运行
* **Yes, and don't ask again for `<name>` in `<path>`**：开始运行，且从此以后在此项目中对该工作流跳过此提示
* **View raw script**：在决定之前阅读脚本原文
* **No**：取消

`Ctrl+G` 在编辑器中打开脚本。`Tab` 让你在运行开始前调整提示。

你是否看到此提示取决于你的 [权限模式](/en/permission-modes)：

| 权限模式                                    | 提示时机                                                                                                                                         |
| :------------------------------------------ | :----------------------------------------------------------------------------------------------------------------------------------------------- |
| Default, accept edits（默认，接受编辑）     | 每次运行，除非你已在此项目中对该工作流选择了 **Yes, and don't ask again**                                                                        |
| Auto（自动）                                | 仅首次启动时。任何 **Yes** 会在用户设置中记录同意，后续启动不再提示。开启 ultracode 时完全跳过                                                   |
| Bypass permissions（绕过权限），`claude -p`，Agent SDK | 从不。运行立即开始                                                                                                                          |

在桌面应用（Desktop app）中，审批卡片显示工作流名称、阶段列表和 token 用量警告，以及 **Once**（一次）、**Always**（始终）和 **Deny**（拒绝）操作。进度视图出现在 "Background tasks" 侧边面板中。

你的权限模式仅控制上述启动提示。工作流派生的子代理始终以 `acceptEdits` 模式运行，并继承你的 [工具白名单](/en/settings#permission-settings)，无论你的会话处于何种模式。文件编辑自动批准。

不在白名单中的 shell 命令、网页抓取和 MCP 工具仍可能在运行中途向你提示。要避免在长时间运行中出现这种情况，请在启动前将代理需要的命令添加到白名单中。

在 `claude -p` 和 Agent SDK 中无人可提示，因此工具调用遵循你配置的权限规则，无需交互式确认。

### 保存工作流以便重用 {#save-the-workflow-for-reuse}

当 Claude 为你会重复的任务编写了工作流，你可以将该运行的脚本保存为命令。像每次在分支上运行的审查流程这样的过程，每次都会执行相同的编排逻辑。

运行 `/workflows`，选择你想保留的运行，然后按 `s`。在保存对话框中，Tab 键可在两个保存位置之间切换：

* 项目中的 `.claude/workflows/`：所有克隆该仓库的人共享
* 家目录中的 `~/.claude/workflows/`：在每个项目中可用，仅你自己可见

按 Enter 保存。无论从哪个位置保存，工作流在未来会话中以 `/<name>` 方式运行。

如果项目工作流和个人工作流同名，则运行项目工作流。

### 向已保存的工作流传递输入 {#pass-input-to-a-saved-workflow}

已保存的工作流可以通过 `args` 参数接收输入。脚本将其作为名为 `args` 的全局变量读取。你可以在调用时提供研究问题、目标路径列表或配置对象，而无需为每次运行编辑脚本。

以下提示使用 issue 编号列表运行已保存的工作流：

```text
> Run /triage-issues on issues 1024, 1025, and 1030
```

Claude 将列表作为结构化数据传递，因此脚本可以直接在 `args` 上调用数组和对象方法，无需先解析。如果省略 `args`，该全局变量在脚本中为 `undefined`。

## 工作流如何运行 {#how-a-workflow-runs}

工作流运行时（workflow runtime）在与你的对话隔离的环境中执行脚本。中间结果保存在脚本变量中，而非进入 Claude 的上下文。

每次运行将脚本写入会话目录下的 `~/.claude/projects/` 中的文件。Claude 在运行开始时收到该路径，因此你可以主动索取。你可以打开该文件阅读 Claude 编写的编排逻辑，与之前运行的脚本做 diff（差异比较），或者编辑后要求 Claude 从编辑后的版本重新启动。

运行时在运行进展中跟踪每个代理的结果，这使得运行在同一会话内 [可恢复](#resume-after-a-pause)。

### 行为与限制 {#behavior-and-limits}

运行时应用以下约束：

| 约束条件                                                         | 原因                               |
| :--------------------------------------------------------------- | :--------------------------------- |
| 运行中途不接受用户输入                                           | 只有代理权限提示可以暂停运行。若需在阶段之间获得确认，将每个阶段作为独立工作流运行 |
| 工作流本身无法直接访问文件系统或 shell                           | 代理负责读取、写入和运行命令。脚本负责协调代理         |
| 最多 16 个并发代理，CPU 核心数有限的机器上更少                   | 限制本地资源使用                     |
| 每次运行最多 1,000 个代理总数                                    | 防止失控循环                         |

## 管理运行 {#manage-runs}

运行开始后，你可以从 `/workflows` 视图管理它，或者通过展开输入框下方任务面板中的进度行来管理。

### 暂停后恢复 {#resume-after-a-pause}

如果你停止了一个运行，可以恢复它：已完成的代理返回缓存的结果，其余代理继续实时运行。从 `/workflows` 恢复暂停的运行：选择它并按 `p`，或者要求 Claude 使用相同的脚本重新启动工作流。

恢复功能仅在同一 Claude Code 会话内有效。如果你在工作流运行期间退出了 Claude Code，下次会话将重新开始该工作流。

### 成本 {#cost}

工作流会派生许多代理，因此单次运行的 token 消耗可能明显高于在对话中处理同一任务。运行会像其他会话一样计入你套餐的使用量和速率限制（rate limits）。

要在投入大型任务前预估消耗，可以先在一小部分上运行工作流：一个目录而非整个仓库，或者一个狭窄的问题而非宽泛的问题。`/workflows` 视图在运行期间显示每个代理的 token 用量，你可以随时在那里停止运行而不会丢失已完成的工作。运行时的 [代理上限](#behavior-and-limits) 限制了单次运行能派生的代理数量，从而约束了失控脚本的成本。

工作流中的每个代理使用你会话的模型，除非脚本将某个阶段路由到其他模型。要控制模型成本：

* 在大型运行前检查 `/model`，如果你通常为常规工作切换到较小的模型
* 在描述任务时，要求 Claude 对不需要最强模型的阶段使用较小的模型

### 关闭工作流 {#turn-workflows-off}

工作流在 CLI、桌面应用、IDE 扩展、[非交互模式](/en/headless)（`claude -p`）以及 [Agent SDK](/en/agent-sdk/overview) 中均可用。所有界面上的禁用设置是相同的。

要为自己关闭工作流：

* 在 `/config` 中关闭 Dynamic workflows 开关。跨会话持久化。
* 在 `~/.claude/settings.json` 中设置 `"disableWorkflows": true`。跨会话持久化。
* 设置环境变量 `CLAUDE_CODE_DISABLE_WORKFLOWS=1`。启动时读取，因此在你设置的任何位置均生效。

要为整个组织关闭工作流，在 [托管设置](/en/server-managed-settings) 中设置 `"disableWorkflows": true`，或在 [Claude Code 管理员设置](https://claude.ai/admin-settings/claude-code) 页面上使用开关。

关闭工作流后，内置工作流命令不可用，`ultracode` 关键字不再触发工作流创建，`/effort ultracode` 降级为 `xhigh`。

## 相关资源

* [[subagent]] — 将任务委派给隔离的 Claude 会话
* [[skill]] — 领域特定的指令集
* [[agent-framework-model-coevolution]] — Agent 框架与模型协同进化
* [[plan-first-workflow]] — AI Agent 工作流模式：先产计划再评审后执行
* [Tools reference](/en/tools-reference) — 可用工具及其行为
