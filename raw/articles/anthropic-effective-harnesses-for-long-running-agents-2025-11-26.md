# Effective harnesses for long-running agents

**Published**: Nov 26, 2025

随着 AI Agent 能力的提升，开发者越来越多地要求它们承担复杂的任务，这些任务需要数小时甚至数天的工作。然而，让 Agent 在多个上下文窗口（context windows）之间持续取得进展仍然是一个未解决的问题。

长期运行的 Agent 面临的核心挑战是，它们必须在离散的会话（sessions）中工作，而每个新会话开始时都不记得之前发生了什么。想象一个由轮班工程师组成的软件项目，每个新来的工程师对上一班发生的事情毫无记忆。由于上下文窗口有限，而且大多数复杂项目无法在单个窗口内完成，Agent 需要一种方法来弥合编码会话之间的差距。

我们开发了一个两方面的解决方案，使 [Claude Agent SDK](https://platform.claude.com/docs/en/agent-sdk/overview) 能够跨多个上下文窗口高效工作：一个负责首次运行时设置环境的 **初始化 Agent（initializer agent）**，以及一个负责在每个会话中取得增量进展、同时为下一个会话留下清晰产物的 **编码 Agent（coding agent）**。你可以在配套的 [快速入门指南](https://github.com/anthropics/claude-quickstarts/tree/main/autonomous-coding) 中找到代码示例。

## 长期运行 Agent 的问题

Claude Agent SDK 是一个强大的通用 Agent harness（执行框架），擅长编码以及其他需要模型使用工具来收集上下文、规划和执行的任务。它具有上下文管理能力，例如压缩（compaction），使 Agent 能够在不耗尽上下文窗口的情况下处理任务。理论上，有了这个设置，Agent 应该可以无限期地持续执行有用工作。

然而，压缩（compaction）并不足够。开箱即用，即使是像 Opus 4.5 这样的前沿编码模型，在 Claude Agent SDK 上跨多个上下文窗口循环运行，如果仅给出高层提示（如"构建一个 [claude.ai](http://claude.ai/redirect/website.v1.c8eaab82-e5f2-49d3-87b0-1a155b2c5f96) 的克隆"），也无法构建出生产质量的 Web 应用。

Claude 的失败表现为两种模式。首先，Agent 倾向于一次性做太多事情——本质上试图一步到位地完成整个应用。这通常导致模型在实现过程中耗尽上下文，留下一个半实现且未记录的功能，让下一个会话的 Agent 去猜测发生了什么，并花费大量时间尝试让基本应用重新运行。即使有压缩机制，这种情况也会发生，因为压缩并不总是能将清晰的指令传递给下一个 Agent。

第二种失败模式通常发生在项目的后期。在已经构建了一些功能之后，后续的 Agent 实例会环顾四周，看到已经取得了一些进展，就会宣布任务完成。

这将问题分解为两个部分。首先，我们需要设置一个初始环境，为给定提示要求的 _所有_ 功能奠定基础，使 Agent 能够逐步、逐个功能地工作。其次，我们应该提示每个 Agent 朝着目标取得增量进展，同时在会话结束时将环境保持在干净状态。所谓"干净状态"，我们指的是适合合并到主分支的代码：没有重大 bug，代码井然有序且文档完善，总体而言，开发者可以轻松地开始新功能的开发，而不必先清理无关的烂摊子。

在内部实验中，我们使用了两部分解决方案来解决这些问题：

1. **初始化 Agent（Initializer agent）**：首个 Agent 会话使用专门的提示，要求模型设置初始环境：一个 `init.sh` 脚本、一个记录 Agent 行为的 `claude-progress.txt` 文件，以及一个展示添加了哪些文件的初始 git 提交。
2. **编码 Agent（Coding agent）**：每个后续会话都要求模型取得增量进展，然后留下结构化的更新。[^1]

这里的关键洞察是找到一种方法，让 Agent 在使用新的上下文窗口启动时能够快速理解工作状态，这通过 `claude-progress.txt` 文件和 git 历史记录来实现。这些实践的灵感来源于了解高效的软件工程师每天在做什么。

## 环境管理

在更新的 [Claude 4 提示指南](https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/claude-4-best-practices#multi-context-window-workflows) 中，我们分享了一些多上下文窗口工作流的最佳实践，包括使用"为第一个上下文窗口使用不同提示"的 harness 结构。这个"不同的提示"要求初始化 Agent 设置所有未来编码 Agent 高效工作所需的上下文环境。在这里，我们对这种环境的一些关键组件进行深入探讨。

### 功能列表（Feature list）

为了解决 Agent 一步到位完成应用或过早认为项目完成的问题，我们提示初始化 Agent 编写一个全面的功能需求文件，对用户初始提示进行展开。在 [claude.ai](http://claude.ai/redirect/website.v1.c8eaab82-e5f2-49d3-87b0-1a155b2c5f96) 克隆示例中，这意味着超过 200 个功能，例如"用户可以打开新聊天、输入查询、按回车键并看到 AI 回复"。这些功能最初都被标记为"失败（failing）"状态，以便后续的编码 Agent 对完整功能有清晰的概述。

```json
{
    "category": "functional",
    "description": "New chat button creates a fresh conversation",
    "steps": [
      "Navigate to main interface",
      "Click the 'New Chat' button",
      "Verify a new conversation is created",
      "Check that chat area shows welcome state",
      "Verify conversation appears in sidebar"
    ],
    "passes": false
}
```

我们提示编码 Agent 只能通过更改 `passes` 字段的状态来编辑此文件，并且我们使用了措辞强烈的指令，如"删除或编辑测试是不可接受的，因为这可能导致功能缺失或出现 bug。" 经过一些实验，我们最终选择了 JSON 格式，因为模型不太可能像对待 Markdown 文件那样不当地更改或覆盖 JSON 文件。

### 增量进展（Incremental progress）

有了初始环境脚手架后，编码 Agent 的下一个迭代版本被要求一次只处理一个功能。这种增量方法被证明对解决 Agent 试图一次做太多事情的倾向至关重要。

在实现增量工作后，模型在代码变更后仍将环境保持在干净状态仍然至关重要。在实验中，我们发现引发这种行为的最佳方式是要求模型将其进展提交到 git，附上描述性的提交信息，并在进度文件中撰写进展摘要。这使模型能够使用 git 回退不良的代码更改并恢复代码库的工作状态。

这些方法也提高了效率，因为它们消除了 Agent 不得不猜测发生了什么并花时间尝试让基本应用重新运行的需求。

### 测试（Testing）

我们观察到的最后一个主要失败模式是 Claude 倾向于在没有适当测试的情况下将功能标记为完成。在没有明确提示的情况下，Claude 倾向于进行代码更改，甚至使用单元测试或针对开发服务器的 `curl` 命令进行测试，但无法认识到功能未能端到端地正常工作。

在构建 Web 应用的情况下，一旦明确提示 Claude 使用浏览器自动化工具并以人类用户的方式进行所有测试，它在端到端验证功能方面做得相当好。

![Image 2: Claude 通过 Puppeteer MCP 服务器截取的 claude.ai 克隆测试截图](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2Ff94c2257964fb2d623f1e81f874977ebfc0986bc-1920x1080.gif&w=3840&q=75)

_Claude 通过 Puppeteer MCP 服务器在测试 claude.ai 克隆时截取的截图。_

为 Claude 提供这类测试工具大幅提高了性能，因为 Agent 能够识别和修复仅从代码中不明显的 bug。

仍然存在一些问题，例如 Claude 视觉能力的局限性以及浏览器自动化工具的限制，使其难以识别所有类型的 bug。例如，Claude 无法通过 Puppeteer MCP 看到浏览器原生的 alert 模态框，因此依赖这些模态框的功能往往更容易出现 bug。

## 快速上手

有了以上所有设置，每个编码 Agent 都被提示执行一系列步骤来确定自己的位置，有些步骤非常基础但仍然有帮助：

1. _运行 `pwd` 查看你正在工作的目录。你只能编辑此目录中的文件。_
2. _阅读 git 日志和进度文件，了解最近的工作内容。_
3. _阅读功能列表文件，选择尚未完成的最高优先级功能来工作。_

这种方法在每次会话中为 Claude 节省了一些 token，因为它不需要弄清楚如何测试代码。此外，要求初始化 Agent 编写一个 `init.sh` 脚本也很有帮助，该脚本可以运行开发服务器，然后在实现新功能之前运行基本的端到端测试。

在 claude.ai 克隆的情况下，这意味着 Agent 总是启动本地开发服务器，并使用 Puppeteer MCP 启动新聊天、发送消息并接收回复。这确保 Claude 能够快速识别应用是否处于损坏状态，并立即修复任何现有 bug。如果 Agent 转而开始实现新功能，问题可能会变得更糟。

鉴于所有这些，典型的会话以以下助手消息开始：

```
[Assistant] I'll start by getting my bearings and understanding the current state of the project.
[Tool Use] <bash - pwd>
[Tool Use] <read - claude-progress.txt>
[Tool Use] <read - feature_list.json>
[Assistant] Let me check the git log to see recent work.
[Tool Use] <bash - git log --oneline -20>
[Assistant] Now let me check if there's an init.sh script to restart the servers.
<Starts the development server>
[Assistant] Excellent! Now let me navigate to the application and verify that some fundamental features are still working.
<Tests basic functionality>
[Assistant] Based on my verification testing, I can see that the fundamental functionality is working well. The core chat features, theme switching, conversation loading, and error handling are all functioning correctly. Now let me review the tests.json file more comprehensively to understand what needs to be implemented next.
<Starts work on a new feature>
```

### Agent 失败模式与解决方案

| **问题** | **初始化 Agent 行为** | **编码 Agent 行为** |
| --- | --- | --- |
| Claude 过早宣布整个项目完成。 | 设置功能列表文件：基于输入规范，设置一个结构化的 JSON 文件，包含端到端功能描述的列表。 | 在会话开始时阅读功能列表文件。选择一个功能开始工作。 |
| Claude 将环境留在有 bug 或未记录进展的状态。 | 编写初始 git 仓库和进度笔记文件。 | 在会话开始时阅读进度笔记文件和 git 提交日志，并在开发服务器上运行基本测试以捕获任何未记录的 bug。在会话结束时编写 git 提交和进度更新。 |
| Claude 过早将功能标记为完成。 | 设置功能列表文件。 | 自验证所有功能。只有在仔细测试后才将功能标记为"通过（passing）"。 |
| Claude 需要花时间弄清楚如何运行应用。 | 编写一个可以运行开发服务器的 `init.sh` 脚本。 | 在会话开始时阅读 `init.sh`。 |

_总结了长期运行的 AI Agent 中 4 种常见失败模式及解决方案。_

## 未来工作

本研究展示了长期运行 Agent harness 中一套可能的解决方案，使模型能够跨多个上下文窗口取得增量进展。然而，仍然存在未解决的问题。

最值得注意的是，目前仍不清楚单一通用编码 Agent 在跨上下文工作时是否表现最佳，还是通过多 Agent 架构可以获得更好的性能。专门的 Agent（如测试 Agent、质量保证 Agent 或代码清理 Agent）在软件开发生命周期的子任务中可能表现得更好，这似乎是合理的。

此外，此演示针对全栈 Web 应用开发进行了优化。未来的方向是将这些发现推广到其他领域。这些经验教训中的一些或全部很可能可以应用于其他类型的长期 Agent 任务，例如科学研究或金融建模。

### 致谢

由 Justin Young 撰写。特别感谢 David Hershey、Prithvi Rajasakeran、Jeremy Hadfield、Naia Bouscal、Michael Tingley、Jesse Mu、Jake Eaton、Marius Buleandara、Maggie Vo、Pedram Navid、Nadine Yasser 和 Alex Notov 的贡献。

这项工作反映了 Anthropic 多个团队的集体努力，使得 Claude 能够安全地进行长期自主软件工程，尤其是代码 RL 和 Claude Code 团队。有意贡献的求职者欢迎访问 [anthropic.com/careers](http://anthropic.com/careers) 申请。

### 脚注

[^1]: 我们在本语境中将这些称为不同的 Agent，仅仅是因为它们具有不同的初始用户提示。系统提示、工具集和整体 Agent harness 在其他方面是相同的。
