---
title: "AI 写代码总跑偏？Comet 用五阶段工作流给 Agent 加护栏"
date: "2026-06-10"
category: "GitHub Trending 项目"
source_path: "ai-wechat-writer/daily/2026-06-10/flows/github-trends/03/deep_read_article.md"
source_url: "https://github.com/rpamis/comet"
---
# AI 写代码总跑偏？Comet 用五阶段工作流给 Agent 加护栏

![runner](https://github.com/rpamis/comet/blob/master/img/runner.png)

我最近在看 GitHub 上和 AI coding agent 相关的工具，刷到一个挺有意思的库：**rpamis/comet**。

它不是那种“又套了一个聊天壳”的项目。它想解决的问题很朴素，也很真实：**AI 写代码一开始很猛，但项目一长、上下文一多、几天后再回来，很多决定就断片了。**

这也是我觉得它值得介绍给 AI 工程师的原因。

开源地址：https://github.com/rpamis/comet

## 1. 这个库到底想解决什么问题？

一句话：**它想把 AI coding agent 从“会写代码的聊天窗口”，改造成“按规格、上下文和流程工作的工程小队”。**

![Comet logo](https://github.com/rpamis/comet/blob/master/img/title-log.png)


README 里的描述很直接：它是一个 spec-driven coding harness，面向 Claude Code、Codex、Cursor、Windsurf、OpenCode 等 AI 编程工具；目标是自动生成 PRD、管理 backlog、路由上下文、沉淀知识库，让 agent 在大任务里不要丢状态。

我理解它真正想打的痛点是：

> AI coding agent 做长任务时容易丢上下文：需求、决策、文档、待办、代码结构散在聊天记录里，隔几天再回来就接不上。

如果你只让 AI 改一个函数，这套东西可能显得重。

但如果你让 AI 连续做几天需求、修 bug、写文档、跑审计、维护上下文，你会很快遇到这些问题：

- 需求写在聊天里，过几天没人知道当时为什么这么定。
- Agent 每次都重新读一遍项目，token 很贵，还容易漏。
- 文档、计划、代码、测试不同步。
- 写完代码没有稳定验收流程。
- 一旦失败，只剩一句“模型没做好”，很难复盘。

这个库的价值，就是把这些隐形流程显性化。

## 2. 我为什么注意到它？

几个信号让我停下来多看了一眼：

- GitHub 数据：约 976 stars、118 forks、16 open issues。
- License：MIT。
- 主要语言：TypeScript。
- 最近更新：2026-06-09T10:00:12Z。
- README 明确写了 12 agents、32 skills、7 tools 这类组织方式。
- 文档结构里有 Spec-Driven Development、Context Groups、Quality Pipeline、Built-in Safety Systems。

这些不等于它一定成熟，但说明作者不是只想做一个 demo，而是在尝试回答一个更工程化的问题：**怎么让 AI 编程过程可交接、可审计、可继续。**

## 3. 它的核心能力，我会这样理解

### 1. Agent + Skill 组织

项目里能看到 .agents/.claude/skills 的组织方式，把文档、审计、Chrome DevTools 等能力做成可复用 skill。

## 4. 怎么用？先别急着装进生产项目

README 说它支持快速安装，但我这篇没有假装已经完整实测。所以我更建议按“旁路试用”的方式来用。

### 第一步：拿一个不重要的测试 repo

不要一上来就扔到生产项目。新建一个 toy project，或者复制一个小型开源项目。

### 第二步：按 README 安装 harness

README 里写了 Install 入口，并且提到：

- How to reliably trigger nested Skills — Not letting the Agent rely on document descriptions to perform "look-alike
- How to make combined Skills flow automatically across phases — Not relying on manual intervention. Comet's 5-phase
- How to turn the Spec lifecycle into a resumable workflow — Comet links OpenSpec change/spec artifacts with
- How to turn document synchronization from "user reminders" into automation — Comet puts handoff, state updates,

这里有个细节挺重要：它声称会备份已有 .claude/、CLAUDE.md，并且不直接动已有 process/ 目录。这说明它意识到了“别把用户现有工作台搞乱”这个问题。

### 第三步：让它先做文档/上下文初始化，不要先写代码

我建议第一条任务不要让它实现功能，而是让它做：

~~~text
请先分析这个仓库，生成项目上下文、模块说明、测试入口和下一步建议。
不要修改业务代码。先给我计划，等我确认后再继续。
~~~

为什么？因为这正好能测试它最核心的能力：**能不能先理解项目，再组织上下文。**

### 第四步：用一个小需求测试完整链路

比如：

~~~text
给这个项目增加一个很小的功能：新增一个配置项，并补一条测试。
要求先写计划，再修改代码，最后说明改了哪些文件、如何验证、有什么风险。
~~~

这时候你要观察的不是“它有没有一次写对”，而是：

- 有没有生成清晰计划。
- 有没有复用前面生成的上下文。
- 有没有记录为什么改这些文件。
- 有没有跑测试或说明无法测试的原因。
- 有没有把新经验写回文档/记忆。

### 第五步：故意给它一个脏场景

真正的 Agent 工程不能只看顺风局。你可以故意制造一个失败场景：缺依赖、测试失败、需求冲突、文件不存在。

看它会不会承认卡住，还是继续硬编。

## 5. 我从关键文件里看到的证据

- 当前证据主要来自 README 和 GitHub API，关键工作流文件证据不足，正式使用前需要再拉代码检查。

这些东西比 star 更重要。因为 star 只能说明“有人关注”，而这些文件能说明“作者有没有把流程落到工程 artifact 里”。

## 6. 它适合谁？

我觉得它适合三类人：

- 已经在用 Claude Code / Codex / Cursor 写项目，但经常被上下文丢失折磨的人。
- 想研究 Agent 工作流、skills、长期记忆、文档路由的 AI 工程师。
- 小团队想把 AI 编程从“个人玄学 prompt”变成“团队可 review 的流程”。

不太适合三类人：

- 只想让 AI 写几行脚本的人。
- 没有日志、权限、备份意识，直接把工具塞进生产 repo 的人。
- 期待它装上就自动替代工程管理的人。

## 7. 它解决的是 Agent 的真问题吗？

我的判断是：**方向上是。成熟度还要继续观察。**

Agent 的真问题，不是“模型还能不能再聪明一点”。

真问题是：

- 长任务如何不丢上下文。
- 多轮修改如何留下决策记录。
- 工具和技能如何复用。
- 文档和代码如何同步。
- 失败如何被发现和复盘。
- 权限和外部动作如何被限制。

rpamis/comet 正好在这些问题上动手了。

但它也有明显风险：

- 项目很新，长期维护还要看。
- README 很强，真实复杂项目表现需要实测。
- 依赖 Claude Code/Codex/Cursor 等外部工具，迁移成本要评估。
- 如果团队没有权限边界，agent harness 反而可能放大误操作。
- 我这次没看到 release 样本，issue 样本也不足，生产可用性不能过度承诺。

当前 issue 样本：
- #83 Question: 非自动流转需要comet-open ,　在hotfix和tweat的时候，会产生delta spec，这个有必要吗？（comments=0）
- #82 Question: comet update之后找不到comet命令（comments=0）
- #75 refactor: migrate shell scripts to TypeScript（comments=2）
- #72 chore: add comet-panel as git submodule（comments=5）
- #68 feat: 关于流程自动提交代码的优化建议（comments=1）
- #66 fix: Comet 不会遵循 OpenSpec 的 config.yaml 中针对特定 Artifact 的 Rule（comments=2）

最近 release 样本：
- 0.3.7：2026-06-09T10:00:12Z
- 0.3.6：2026-06-02T16:06:48Z
- 0.3.5：2026-05-29T16:36:55Z

## 8. 如果你只学一个设计，学这个

我最建议从这个库里学的，不是某个命令，而是这个设计思想：

> 不要让 Agent 只记住聊天记录，要让它维护工程上下文。

一个可复制的最小版本是：

~~~text
/project
  process/
    context/
      all-context.md        # 项目总入口
      architecture.md       # 架构和关键模块
      decisions.md          # 已做过的重要决定
      tests.md              # 怎么验证
      risks.md              # 风险和边界
  tasks/
    backlog.md
    current-plan.md
    review-log.md
~~~

然后给 Agent 一条规则：

~~~text
开始实现前，先读 process/context/all-context.md。
改完功能后，更新 decisions/tests/review-log。
如果信息不足，写 blocker，不要编结论。
~~~

这就是 vibecode-pro-max-kit 背后最值得拿走的东西。

## 9. 最后给一个推荐结论

如果你是 AI 工程师，我建议你收藏并旁路试用。

不要把它当“开箱即用的银弹”，而是把它当一个很好的 Agent 工程样本：它把 spec、context、skills、quality pipeline、安全边界这些东西放在了一起。

我会继续观察它后续三个信号：

- 是否有更稳定的 release 节奏。
- issue 是否从安装问题转向真实使用反馈。
- 是否出现更多复杂项目里的实测案例。

如果这些信号继续变好，它可能会成为 AI coding agent 工作流里很值得参考的一套 harness。

评论区可以聊一个问题：你现在用 AI 写代码，最痛的是“它写错”，还是“它忘了前面为什么这么写”？


---

> 本文同步自微信公众号「AltenAI观察」的公开归档。完整每日推送与后续精读欢迎搜索关注。
