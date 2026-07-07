---
title: Claude Code 循环模式
created: 2026-07-08
updated: 2026-07-08
type: concept
tags: [agent, tools, tutorial]
sources: [raw/articles/claude-devs-loops.md]
---

# Claude Code 循环模式

Claude Code 将"循环"（loop）定义为 agent 重复执行工作周期直到满足停止条件的模式。循环不是单一概念，而是根据触发方式、停止条件、使用的原语和适用任务类型分为多种类型。

## 核心定义

循环是 agent 重复执行工作周期直到满足停止条件的模式。不是所有任务都需要复杂循环，应从最简单的方案开始，按需选择模式。

## 四种循环类型

### 1. 基于轮次的循环（Turn-based loops）

- **触发方式**：用户提示词
- **停止条件**：Claude 判断任务完成或需要额外上下文
- **适用场景**：较短任务，不属于常规流程或计划
- **Token 管理**：编写具体提示词，使用 skills 改进验证以减少轮次

每个提示词启动一个手动循环，用户指导每一轮。Claude 收集上下文、采取行动、检查工作、必要时重复，然后响应。这称为"agentic loop"。

**验证改进**：将手动验证步骤编码为 SKILL.md，让 Claude 能端到端地检查自己的工作。检查越量化，Claude 越容易自我验证。

示例 SKILL.md：
```markdown
---
name: verify-frontend-change
description: Verify any UI change end-to-end before declaring it done.
---

# Verifying frontend changes

Never report a UI change as complete based on a successful edit alone. Verify it the way a human reviewer would:

1. Start the dev server and open the edited page in the browser.
2. Interact with the change directly. For a new control (button, input, toggle): click it, confirm the expected state change, and screenshot before/after.
3. Check the browser console: zero new errors or warnings.
4. Use the Chrome Devtools MCP, run a performance trace and audit Core Web Vitals.

If any step fails, fix the issue and rerun from step 1 — do not hand back partially verified work.
```

### 2. 基于目标的循环（Goal-based loop）

- **触发方式**：实时手动提示
- **停止条件**：目标达成或达到最大轮次
- **适用场景**：有可验证退出标准的任务
- **Token 管理**：设置明确的完成标准和轮次上限（如"5 次尝试后停止"）

使用 `/goal` 命令定义成功标准。Claude 不需要判断什么是"足够好"并提前结束循环。每次 Claude 尝试停止时，评估模型检查条件并让其继续工作，直到目标达成或达到轮次上限。

**确定性标准最有效**：如通过的测试数量或达到特定分数阈值。

示例：
```bash
/goal get the homepage Lighthouse score to 90 or above, stop after 5 tries.
```

### 3. 基于时间的循环（Time-based loop）

- **触发方式**：指定时间间隔
- **停止条件**：用户取消，或工作完成（PR 合并、队列清空）
- **适用场景**：定期工作，或与外部系统/环境交互
- **Token 管理**：设置更长间隔，或基于事件而非时间响应

使用 `/loop` 按间隔重新运行提示词：
```bash
/loop 5m check my PR, address review comments, and fix failing CI
```

`/loop` 在本地运行，关闭电脑则停止。使用 `/schedule` 创建 routine 可将循环移到云端。

### 4. 主动循环（Proactive loops）

- **触发方式**：事件或计划，无实时人工干预
- **停止条件**：每个任务在目标达成时退出。routine 本身运行直到关闭
- **适用场景**：定期、明确定义的工作流：bug 报告、issue 分类、迁移、依赖升级等
- **Token 管理**：将 routine 路由到更小更快的模型，对判断性任务使用最强模型

组合多种原语：
- `/schedule`（研究预览）运行检查新报告的 routine
- `/goal` 定义完成标准，skills 文档化验证方式
- 动态工作流编排 agent 分类、修复、审查
- Auto mode 让 routine 无需请求许可即可运行

示例：
```bash
/schedule every hour: check the project-feedback channel for bug reports.
/goal: don't stop until every report found this run is triaged, actioned, and responded to.
When fixing a bug, use a workflow to explore three solutions in parallel worktrees and have a judge adversarially review them.
```

## 代码质量维护

循环输出质量取决于周围系统：

1. **保持代码库整洁**：Claude 遵循代码库中已有的模式和约定
2. **提供自我验证方式**：用 skills 编码"好"的标准
3. **文档易获取**：框架和库文档应有最新的最佳实践
4. **使用第二个 agent 做代码审查**：拥有新鲜上下文的审查者偏见更少。可使用内置 `/code-review` skill 或 Code Review for Github

当单个结果不达标时，不要只修复单个问题，尝试将其编码以改进所有未来迭代的系统。

## Token 使用管理

循环应有明确边界：

1. **为任务选择合适的原语和模型**：小任务不需要多 agent 或循环，某些任务可使用更便宜更快的模型
2. **定义明确的成功和停止标准**：具体说明"完成"的样子，让 Claude 更快到达解决方案（但不要过早）
3. **大规模运行前先试点**：动态工作流可生成数百个 agent，先在小 slice 上评估使用量
4. **对确定性工作使用脚本**：运行脚本比推理步骤更便宜。例如 PDF skill 可附带表单填充脚本，Claude 每次运行，而不是重新推导代码
5. **不要过于频繁运行 routine**：间隔匹配被观察对象的变化频率
6. **审查使用量**：`/usage` 命令分解最近按 skills、subagents、MCPs 的使用量；`/goal` 无参数显示轮次和 token 使用量；`/workflows` 显示每个 agent 的 token 使用量，可随时停止 agent

## 入门建议

查看已有的工作，找出自己成为瓶颈的一个任务，问自己哪部分可以交出去：能否写验证检查？目标是否足够清晰？工作是否按计划到达？

有了想法后，运行循环，观察结果（在哪里停滞或过度延伸），不要害怕迭代。

## 总结对比

| 循环类型 | 你交出的 | 使用时机 | 使用工具 |
|---------|---------|---------|---------|
| 基于轮次 | 检查 | 探索或决策时 | 自定义验证 skills |
| 基于目标 | 停止条件 | 知道完成的样子 | `/goal` |
| 基于时间 | 触发器 | 工作在项目外按计划发生 | `/loop`, `/schedule` |
| 主动 | 提示词 | 工作定期且明确定义 | 以上全部 + 动态工作流 |

## 相关概念

- [[claude-code]] - Claude Code 整体系统
- [[agent-loop]] - Agent 循环的通用概念
- [[skills]] - Skills 在验证和自动化中的作用
- [[dynamic-workflows]] - 动态工作流编排
