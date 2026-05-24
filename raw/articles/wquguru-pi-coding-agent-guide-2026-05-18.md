# Pi Coding Agent 最全面指南（完美支持/goal）

**作者**: WquGuru (@wquguru)
**日期**: 2026-05-18
**来源**: https://x.com/i/status/2056235143623495975

如果你已经习惯 Claude Code，Pi 第一眼不会显得更省心。Claude Code 的优势是把 subagents、Plan Mode、MCP、权限、上下文压缩、skills、commands 这些工程特性都焊进产品里，开箱即用。

但你如果想认真测试非 Anthropic 模型时，Claude Code 未必是最干净的环境。模型能力、Claude Code 自身的工作流假设、非 Anthropic 模型的适配摩擦，会混在一起，很难拆开判断。

Pi 的意义就在这里：

> Pi 不是另一个 Claude Code，而是一个更可拆的 agent 底座。用户自主决定装哪些组件、给多少上下文、什么时候 high / xhigh、哪些工具进入 prompt。代价是要会配置；收益是测试模型时更透明、更可控、更容易复现。

所以我写这篇文章只做一件事：给 Claude Code 用户一张 Pi 迁移地图。看完应该知道 Pi 和 Claude Code 怎么对应，模型在 Pi 里怎么配，最小可用栈是什么，哪些坑会影响效果，以及为什么模型特别需要 plan-first + skill 工作流。

## 一、先给 Claude Code 用户一张地图

Claude Code 用户理解 Pi，最重要的是先换一个心智模型。

Claude Code 是产品化的一整套工程环境。用户不太需要关心哪些工具被注入上下文、哪些生命周期事件触发、权限和 Plan Mode 怎么拼起来。它默认就给你一套强约束工作流。

Pi 是 minimal core：CLI（@earendil-works/pi-coding-agent）+ pi-agent-core + 多 provider 的 pi-ai。核心内置 Tools 极少，基本只有读写文件加 grep/find/ls。你熟悉的那些能力，很多都不是 core，而是扩展，大概分四类：

- TypeScript Extensions：用代码挂生命周期事件，对应 Claude Code hooks，但不是声明式 JSON，而是可以写逻辑的扩展代码
- Skills：SKILL.md + 脚本，和 Claude Code skills 是同一类东西
- Prompt Templates：对应 Claude Code slash commands
- Pi Packages：通过 pi install npm:<pkg> 或 pi install git:<repo> 安装，也可以 -l 装到项目级

配置主要在 ~/.pi/agent/

Pi 注册的所有 Tools 大约只占 7.7k 上下文。Pi 不默认给你全家桶，它的设计哲学决定了装什么的决定权在于用户：

> 如果要开箱即用，继续用 Claude Code 或 Codex
> 如果要更干净地测试 Ring、控制上下文预算、接任意 OpenAI 兼容模型，Pi 更适合

## 二、Pi<->Claude Code 对照表

- 第一，Pi 的能力很完备，而不过把能力拆开了，subagent、plan、MCP、web、context prune 这些东西都能支持；
- 第二，Pi 当然也有缺口，CLAUDE.md 式持久记忆、权限规则引擎、强制只读 Plan gate、官方 marketplace 和发现 UI，都没有 Claude Code 那么完整；
- 第三，Pi 在模型层更开放（毋庸置疑的优势）。它能接任意 OpenAI 兼容 provider，也能显式做 fallback chain，这正是我们接 Ring-2.6-1T 的入口。

## 三、开源模型 Ring-2.6-1T为例，在 Pi 里的推荐模型配置

为什么选 Ring-2.6-1T，很简单，它是HuggingFace上最近开源的模型，参数量达到惊人的1T（1000B），很适合用来测试最新模型的能力。

provider 选择：
- DeepSeek = 内置 provider：只给 key，绝不手配 models.json
- Ring-2.6-1T = 自定义 provider：需要手填 models.json

接 Ring 时，选 OpenAI 兼容端点 /v1，而不是 Anthropic 兼容层。

原因：Anthropic 兼容层通常是薄包装，cache_control、thinking blocks、tool schema 这些特性容易在转换里丢；OpenAI shape 是 vLLM/SGLang 常见导出路径，国内厂商对这个形态调 bug 最多，兼容性更稳。

models.json 关键配置要点：
1. compat.supportsDeveloperRole: false — 不写的话部分端点会拒 developer 角色
2. thinkingLevelMap 必须含 "xhigh" : "xhigh" — 否则 Pi 可能隐藏 xhigh 档
3. contextWindow / maxTokens 不要填小 — Ring-2.6-1T 是长上下文模型

high 和 xhigh 分工：
- Ring-2.6-1T high：默认工程执行档，适合高频交互
- Ring-2.6-1T xhigh：复杂规划、核心逻辑、最终审查
- DeepSeek-V4-Pro / Flash：测试、review、非核心代码、快速修补

## 四、推荐的 Pi 最小可用栈

核心 5 个：
- pi-mcp-adapter
- pi-web-access
- @tintinweb/pi-subagents
- @ff-labs/pi-fff
- pi-context-prune

额外推荐 5 个增强组件。

## 五、7 个最容易踩的坑

排查 provider 问题时，Pi 往往会把错误压成一行。经验是：不要只看 Pi 输出，直接 curl 端点，把变量逐个二分。

## 六、为什么特别需要 plan-first + skill

任何模型发挥出最大功效都需要遵循：
1. 目标明确、上下文给足、流程和边界写清楚
2. 先 plan，再 execute
3. 用 skill 把领域经验固化进工作流

Pi 默认没有这些功能，需要以扩展形式安装：
- plan-first：复杂任务先让模型写计划，再评审计划，再执行
- subagent：planner / executor / reviewer 分工
- skill：把测试规范、设计规范、项目 SOP、调试流程写成可复用说明
- context prune：长任务中途清理上下文
- usage/cache 可见性：知道成本花在哪

## 七、一个真实工程任务里的观察

用现货-永续资金费率监控设计文档，观察 Pi + Ring-2.6-1T 在完整工程任务里的表现。

暴露的问题：
- UI 状态契约没有在 plan 阶段锁死
- funding interval 的 symbol 级 override 没有被测试明确覆盖
- 测试有同义反复倾向
- 前端 polish 没有接 design / UI skill

结论：
> Ring 不应该被当成"一次性生成完整系统"的黑盒。
> 它更适合被放进一个 plan-first、skill-amplified、review-driven 的工作流里。

## 八、推荐工作流

1. 先只装核心栈（5个包），上下文保持干净
2. 默认 high，复杂点切 xhigh
3. 强制 plan-first：复杂任务先输出计划再执行
4. 用 skill 固化工作流
5. 让更贵的脑子做 plan，让便宜快的模型做执行

## 附：链接与参考

- skill 仓库：https://github.com/wquguru/skills/tree/main/skills/pi-setup
- Pi 官方仓库：https://github.com/earendil-works/pi
- Pi CLI：@earendil-works/pi-coding-agent
- Ring-2.6-1T Hugging Face：https://huggingface.co/inclusionAI/Ring-2.6-1T
- Ling Studio：https://ling.tbox.cn/chat
