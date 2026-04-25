---
title: Code execution with MCP: Building more efficient agents
created: 2026-04-25
updated: 2026-04-25
type: article
source: https://www.anthropic.com/engineering/code-execution-with-mcp
author: Adam Jones, Conor Kelly
published: 2025-11-04
language: zh-CN
translation_note: 中文学习版，基于 Anthropic 原文整理翻译；保留关键代码片段与术语。
---

# 使用 MCP 进行代码执行：构建更高效的 Agent

原文：https://www.anthropic.com/engineering/code-execution-with-mcp
作者：Adam Jones、Conor Kelly
发布时间：2025-11-04

## 核心摘要

MCP（Model Context Protocol）已经成为 Agent 连接外部工具和数据系统的事实标准。问题是，当一个 Agent 连接几十个 MCP server、暴露成百上千个工具时，传统“直接工具调用”会把大量工具定义和中间结果塞进模型上下文，带来高 token 成本、高延迟和更差的工具组合能力。Anthropic 这篇文章提出的方向是：不要让模型直接调用所有 MCP 工具，而是把 MCP server 暴露成代码执行环境里的 API，让 Agent 写代码去调用工具。这样工具发现、数据过滤、循环控制、中间状态和隐私处理都可以尽量留在执行环境里，只有必要结果再回到模型上下文。

## MCP 为什么重要

MCP 是一个开放标准，用来把 AI Agent 连接到外部系统。传统集成方式需要为每一组“Agent × 工具/数据源”做定制连接，容易碎片化且重复劳动高。MCP 的价值在于：开发者只要在 Agent 里实现一次 MCP，就能接入一个不断扩张的 server 生态。

自 2024 年 11 月发布以来，MCP 社区已经构建了大量 MCP server，主要编程语言也有 SDK。随着采用增长，开发者开始让 Agent 同时连接数十个 MCP server、数百甚至上千个工具。扩展性问题由此出现：工具定义和工具结果本身会消耗上下文窗口。

## 问题一：工具定义淹没上下文窗口

很多 MCP client 会在会话开始时把所有工具定义直接加载进模型上下文，例如 Google Drive 的 `getDocument`、Salesforce 的 `updateRecord` 等。每个工具都需要描述用途、参数、返回值。如果连接了几千个工具，模型可能在真正阅读用户请求前就要处理几十万 token。这不只是成本问题，也会增加响应时间，并稀释模型对当前任务的注意力。

## 问题二：中间工具结果反复穿过模型

传统直接工具调用还有另一个浪费：每个中间结果都要进入模型上下文。文章用一个例子说明：用户要求“从 Google Drive 下载会议记录，并附加到 Salesforce lead”。直接工具调用路径是：

```text
TOOL CALL: gdrive.getDocument(documentId: "abc123")
→ 返回完整会议记录，加载进模型上下文

TOOL CALL: salesforce.updateRecord(... data: { Notes: "完整会议记录" })
→ 模型还要把完整会议记录再写入下一次调用
```

如果是一场两小时销售会议，这可能额外处理 5 万 token，而且完整文本至少经过模型两次。对 Agent 来说，这些数据并不一定需要“被理解”，很多时候只是要从一个系统搬到另一个系统。

## Anthropic 的方案：通过代码执行访问 MCP

文章提出的核心思路是：把 MCP server 映射为代码 API，而不是全部暴露为模型直接可调用工具。Agent 先在代码执行环境中按需导入 server 或函数，然后用普通程序控制流完成任务。

示意上，原来的模式是：

```text
模型上下文 ← 所有工具定义 + 每次工具调用结果
```

新的模式是：

```text
模型上下文 ← 少量任务指令 + 必要结果摘要
代码执行环境 ← MCP server、工具定义、完整中间数据、循环、过滤、状态
```

这样，模型不需要预先读取全部工具定义，也不需要看见每一份完整中间数据。它可以写一段代码完成检索、过滤、转换、搬运，最后只把结果或异常摘要交还给模型。

## 好处一：渐进式披露（Progressive disclosure）

代码执行环境让工具可以按需加载，而不是 upfront 全量加载。Agent 可以先列出可用 server，再根据任务导入相关 API。这和软件工程里的模块导入很像：程序不需要启动时把所有库的全部文档都读进内存。

这点对拥有大量工具的 Agent 特别重要。工具生态越大，直接工具调用越像“把整个 API 文档都塞给模型”；代码执行方式则更像“让模型知道去哪查，需要时再 import”。

## 好处二：工具结果更节省上下文

当 Agent 通过代码调用 MCP 工具时，大型中间数据可以留在执行环境中。比如会议转录全文可以从 Google Drive 取出后直接写入 Salesforce，不必完整进入模型上下文。模型只需要知道任务完成、写入了哪个记录、是否出现异常。

这不只是省 token。它还减少了模型在长文本中迷失或误改数据的机会。对“搬运、筛选、聚合、批处理”类任务，代码执行往往比逐步直接工具调用更稳。

## 好处三：更强的控制流和组合能力

直接工具调用适合少量线性步骤，但复杂任务经常需要循环、条件分支、重试、批处理、分页、局部过滤和错误处理。让 Agent 写代码调用 MCP server 后，它可以使用成熟的编程结构，例如：

- 遍历多个文档、issue、记录或网页
- 在本地过滤大结果集，只保留相关字段
- 对失败请求做重试或降级
- 把多个工具组合成一个小型流水线
- 对中间状态做缓存，避免重复请求

这相当于把 Agent 从“每一步都必须请求模型决策”升级为“模型写程序，程序在沙盒里批量执行”。

## 好处四：隐私保护

如果中间数据不必进入模型上下文，就可以减少敏感信息暴露。文章指出，执行环境可以在把结果返回模型之前先做过滤、脱敏或聚合。例如只返回统计值、匹配行、摘要字段，而不是完整原始记录。

这不是自动解决所有安全问题，但为隐私边界提供了更细粒度的控制：哪些数据只在执行环境里处理，哪些数据允许进入模型上下文，可以由代码和权限策略共同约束。

## 好处五：状态持久化与 Skills

代码执行还支持保存可复用函数、脚本和资源。Anthropic 把这和 Skills 联系起来：如果某段工具组合逻辑经常复用，可以把它保存为函数；再加上 `SKILL.md` 这样的结构化说明，模型之后就能把它当成高层能力调用。

这和 [[hermes-skill-implementation]] 里的思想很接近：把程序性知识从一次性上下文中抽出来，变成可索引、可复用、可演进的能力包。

## 代价与风险

代码执行不是白给的。它需要安全执行环境，包括沙箱、资源限制、权限控制和监控。Agent 生成代码后运行，天然会带来更高的安全和运维复杂度。直接工具调用虽然 token 成本高，但实现和安全边界相对简单。

因此，是否采用“代码执行 + MCP”取决于权衡：当工具数量多、中间数据大、任务需要复杂组合时，收益明显；当只连接少量工具、任务简单、环境无法安全运行代码时，直接工具调用可能更合适。

## 学习判断

这篇文章的重点不是“MCP 本身是什么”，而是 MCP 规模化后的第二阶段问题：工具生态变大以后，Agent 如何不被工具定义和工具结果拖垮。Anthropic 给出的答案很工程化：把 LLM 从数据搬运和控制流细节中解放出来，让代码执行环境承担可编程、可过滤、可持久化的部分。

可以把它理解为 Agent 工程的一条趋势：模型负责意图理解、计划和关键判断；代码负责确定性控制流、批处理和状态管理；MCP 负责标准化连接外部世界。

## 原文内容备份（英文）

> 以下为 Tavily 抽取的原文 Markdown，作为溯源备份。

[Skip to footer](#footer)

[Try Claude](https://claude.ai/)



[Engineering at Anthropic](/engineering)

![](https://www-cdn.anthropic.com/images/4zrzovbb/website/42f40f6fae9ec2d7cf2e5a98908a16d0216b91be-1000x1000.svg)

# Code execution with MCP: Building more efficient agents

Published Nov 04, 2025

Direct tool calls consume context for each definition and result. Agents scale better by writing code to call tools instead. Here's how it works with MCP.

[The Model Context Protocol (MCP)](https://modelcontextprotocol.io/) is an open standard for connecting AI agents to external systems. Connecting agents to tools and data traditionally requires a custom integration for each pairing, creating fragmentation and duplicated effort that makes it difficult to scale truly connected systems. MCP provides a universal protocol—developers implement MCP once in their agent and it unlocks an entire ecosystem of integrations.

Since launching MCP in November 2024, adoption has been rapid: the community has built thousands of [MCP servers](https://github.com/modelcontextprotocol/servers), [SDKs](https://modelcontextprotocol.io/docs/sdk) are available for all major programming languages, and the industry has adopted MCP as the de-facto standard for connecting agents to tools and data.

Today developers routinely build agents with access to hundreds or thousands of tools across dozens of MCP servers. However, as the number of connected tools grows, loading all tool definitions upfront and passing intermediate results through the context window slows down agents and increases costs.

In this blog we'll explore how code execution can enable agents to interact with MCP servers more efficiently, handling more tools while using fewer tokens.

## **Excessive token consumption from tools makes agents less efficient**

As MCP usage scales, there are two common patterns that can increase agent cost and latency:

1. Tool definitions overload the context window;
2. Intermediate tool results consume additional tokens.

### **1. Tool definitions overload the context window**

Most MCP clients load all tool definitions upfront directly into context, exposing them to the model using a direct tool-calling syntax. These tool definitions might look like:

```
gdrive.getDocument Description: Retrieves a document from Google Drive Parameters: documentId (required, string): The ID of the document to retrieve fields (optional, string): Specific fields to return Returns: Document object with title, body content, metadata, permissions, etc.
```

```
salesforce.updateRecord Description: Updates a record in Salesforce Parameters: objectType (required, string): Type of Salesforce object (Lead, Contact, Account, etc.) recordId (required, string): The ID of the record to update data (required, object): Fields to update with their new values Returns: Updated record object with confirmation
```

Tool descriptions occupy more context window space, increasing response time and costs. In cases where agents are connected to thousands of tools, they’ll need to process hundreds of thousands of tokens before reading a request.

### **2. Intermediate tool results consume additional tokens**

Most MCP clients allow models to directly call MCP tools. For example, you might ask your agent: "Download my meeting transcript from Google Drive and attach it to the Salesforce lead."

The model will make calls like:

```
TOOL CALL: gdrive.getDocument(documentId: "abc123") → returns "Discussed Q4 goals...\n[full transcript text]" (loaded into model context) TOOL CALL: salesforce.updateRecord( objectType: "SalesMeeting", recordId: "00Q5f000001abcXYZ", data: { "Notes": "Discussed Q4 goals...\n[full transcript text written out]" } ) (model needs to write entire transcript into context again)
```

Every intermediate result must pass through the model. In this example, the full call transcript flows through twice. For a 2-hour sales meeting, that could mean processing an additional 50,000 tokens. Even larger documents may exceed context window limits, breaking the workflow.

With large documents or complex data structures, models may be more likely to make mistakes when copying data between tool calls.

![Image of how the MCP client works with the MCP server and LLM.](/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F9ecf165020005c09a22a9472cee6309555485619-1920x1080.png&w=3840&q=75)

The MCP client loads tool definitions into the model's context window and orchestrates a message loop where each tool call and result passes through the model between operations.

## **Code execution with MCP improves context efficiency**

With code execution environments becoming more common for agents, a solution is to present MCP servers as code APIs rather than direct tool calls. The agent can then write code to interact with MCP servers. This approach addresses both challenges: agents can load only the tools they need and process data in the execution environment before passing results back to the model.

There are a number of ways to do this. One approach is to generate a file tree of all available tools from connected MCP servers. Here's an implementation using TypeScript:

```
servers ├── google-drive │ ├── getDocument.ts │ ├── ... (other tools) │ └── index.ts ├── salesforce │ ├── updateRecord.ts │ ├── ... (other tools) │ └── index.ts └── ... (other servers)
```

Then each tool corresponds to a file, something like:

```
// ./servers/google-drive/getDocument.ts import { callMCPTool } from "../../../client.js"; interface GetDocumentInput { documentId: string; } interface GetDocumentResponse { content: string; } /* Read a document from Google Drive */ export async function getDocument(input: GetDocumentInput): Promise { return callMCPTool('google_drive__get_document', input); } 
```

Our Google Drive to Salesforce example above becomes the code:

```
// Read transcript from Google Docs and add to Salesforce prospect import * as gdrive from './servers/google-drive'; import * as salesforce from './servers/salesforce'; const transcript = (await gdrive.getDocument({ documentId: 'abc123' })).content; await salesforce.updateRecord({ objectType: 'SalesMeeting', recordId: '00Q5f000001abcXYZ', data: { Notes: transcript } }); 
```

The agent discovers tools by exploring the filesystem: listing the `./servers/` directory to find available servers (like `google-drive` and `salesforce`), then reading the specific tool files it needs (like `getDocument.ts` and `updateRecord.ts`) to understand each tool's interface. This lets the agent load only the definitions it needs for the current task. This reduces the token usage from 150,000 tokens to 2,000 tokens—a time and cost saving of 98.7%**.**

Cloudflare [published similar findings](https://blog.cloudflare.com/code-mode/), referring to code execution with MCP as “Code Mode." The core insight is the same: LLMs are adept at writing code and developers should take advantage of this strength to build agents that interact with MCP servers more efficiently.

## **Benefits of code execution with MCP**

Code execution with MCP enables agents to use context more efficiently by loading tools on demand, filtering data before it reaches the model, and executing complex logic in a single step. There are also security and state management benefits to using this approach.

### Progressive disclosure

Models are great at navigating filesystems. Presenting tools as code on a filesystem allows models to read tool definitions on-demand, rather than reading them all up-front.

Alternatively, a `search_tools` tool can be added to the server to find relevant definitions. For example, when working with the hypothetical Salesforce server used above, the agent searches for "salesforce" and loads only those tools that it needs for the current task. Including a detail level parameter in the `search_tools` tool that allows the agent to select the level of detail required (such as name only, name and description, or the full definition with schemas) also helps the agent conserve context and find tools efficiently.

### Context efficient tool results

When working with large datasets, agents can filter and transform results in code before returning them. Consider fetching a 10,000-row spreadsheet:

```
// Without code execution - all rows flow through context TOOL CALL: gdrive.getSheet(sheetId: 'abc123') → returns 10,000 rows in context to filter manually // With code execution - filter in the execution environment const allRows = await gdrive.getSheet({ sheetId: 'abc123' }); const pendingOrders = allRows.filter(row => row["Status"] === 'pending' ); console.log(`Found ${pendingOrders.length} pending orders`); console.log(pendingOrders.slice(0, 5)); // Only log first 5 for review
```

The agent sees five rows instead of 10,000. Similar patterns work for aggregations, joins across multiple data sources, or extracting specific fields—all without bloating the context window.

#### **More powerful and context-efficient control flow**

Loops, conditionals, and error handling can be done with familiar code patterns rather than chaining individual tool calls. For example, if you need a deployment notification in Slack, the agent can write:

```
let found = false; while (!found) { const messages = await slack.getChannelHistory({ channel: 'C123456' }); found = messages.some(m => m.text.includes('deployment complete')); if (!found) await new Promise(r => setTimeout(r, 5000)); } console.log('Deployment notification received');
```

This approach is more efficient than alternating between MCP tool calls and sleep commands through the agent loop.

Additionally, being able to write out a conditional tree that gets executed also saves on “time to first token” latency: rather than having to wait for a model to evaluate an if-statement, the agent can let the code execution environment do this.

### Privacy-preserving operations

When agents use code execution with MCP, intermediate results stay in the execution environment by default. This way, the agent only sees what you explicitly log or return, meaning data you don’t wish to share with the model can flow through your workflow without ever entering the model's context.

For even more sensitive workloads, the agent harness can tokenize sensitive data automatically. For example, imagine you need to import customer contact details from a spreadsheet into Salesforce. The agent writes:

```
const sheet = await gdrive.getSheet({ sheetId: 'abc123' }); for (const row of sheet.rows) { await salesforce.updateRecord({ objectType: 'Lead', recordId: row.salesforceId, data: { Email: row.email, Phone: row.phone, Name: row.name } }); } console.log(`Updated ${sheet.rows.length} leads`);
```

The MCP client intercepts the data and tokenizes PII before it reaches the model:

```
// What the agent would see, if it logged the sheet.rows: [ { salesforceId: '00Q...', email: '[EMAIL_1]', phone: '[PHONE_1]', name: '[NAME_1]' }, { salesforceId: '00Q...', email: '[EMAIL_2]', phone: '[PHONE_2]', name: '[NAME_2]' }, ... ]
```

Then, when the data is shared in another MCP tool call, it is untokenized via a lookup in the MCP client. The real email addresses, phone numbers, and names flow from Google Sheets to Salesforce, but never through the model. This prevents the agent from accidentally logging or processing sensitive data. You can also use this to define deterministic security rules, choosing where data can flow to and from.

### State persistence and skills

Code execution with filesystem access allows agents to maintain state across operations. Agents can write intermediate results to files, enabling them to resume work and track progress:

```
const leads = await salesforce.query({ query: 'SELECT Id, Email FROM Lead LIMIT 1000' }); const csvData = leads.map(l => `${l.Id},${l.Email}`).join('\n'); await fs.writeFile('./workspace/leads.csv', csvData); // Later execution picks up where it left off const saved = await fs.readFile('./workspace/leads.csv', 'utf-8');
```

Agents can also persist their own code as reusable functions. Once an agent develops working code for a task, it can save that implementation for future use:

```
// In ./skills/save-sheet-as-csv.ts import * as gdrive from './servers/google-drive'; export async function saveSheetAsCsv(sheetId: string) { const data = await gdrive.getSheet({ sheetId }); const csv = data.map(row => row.join(',')).join('\n'); await fs.writeFile(`./workspace/sheet-${sheetId}.csv`, csv); return `./workspace/sheet-${sheetId}.csv`; } // Later, in any agent execution: import { saveSheetAsCsv } from './skills/save-sheet-as-csv'; const csvPath = await saveSheetAsCsv('abc123');
```

This ties in closely to the concept of [Skills](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview), folders of reusable instructions, scripts, and resources for models to improve performance on specialized tasks. Adding a SKILL.md file to these saved functions creates a structured skill that models can reference and use. Over time, this allows your agent to build a toolbox of higher-level capabilities, evolving the scaffolding that it needs to work most effectively.

Note that code execution introduces its own complexity. Running agent-generated code requires a secure execution environment with appropriate [sandboxing](https://www.anthropic.com/engineering/claude-code-sandboxing), resource limits, and monitoring. These infrastructure requirements add operational overhead and security considerations that direct tool calls avoid. The benefits of code execution—reduced token costs, lower latency, and improved tool composition—should be weighed against these implementation costs.

## **Summary**

MCP provides a foundational protocol for agents to connect to many tools and systems. However, once too many servers are connected, tool definitions and results can consume excessive tokens, reducing agent efficiency.

Although many of the problems here feel novel—context management, tool composition, state persistence—they have known solutions from software engineering. Code execution applies these established patterns to agents, letting them use familiar programming constructs to interact with MCP servers more efficiently. If you implement this approach, we encourage you to share your findings with the [MCP community](https://modelcontextprotocol.io/community/communication).

### Acknowledgments

*This article was written by Adam Jones and Conor Kelly. Thanks to Jeremy Fox, Jerome Swannack, Stuart Ritchie, Molly Vorwerck, Matt Samuels, and Maggie Vo for feedback on drafts of this post.*

## Get the developer newsletter

Product updates, how-tos, community spotlights, and more. Delivered monthly to your inbox.

Code execution with MCP: building more efficient AI agents \ Anthropic
