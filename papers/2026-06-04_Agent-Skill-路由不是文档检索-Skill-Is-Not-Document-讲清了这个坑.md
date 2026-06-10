---
title: "Agent Skill 路由不是文档检索：Skill Is Not Document 讲清了这个坑"
date: "2026-06-04"
category: "论文精读"
source_path: "ai-wechat-writer/daily/2026-06-04/flows/paper-deep-read/manual-002-outline_paper_arxiv_2606.03565/deep_read_article.md"
source_url: "https://arxiv.org/abs/2606.03565"
---
# Agent Skill 路由不是文档检索：Skill Is Not Document 讲清了这个坑

我最近读到一篇很适合 Agent 工程师看的论文：**Skill Is Not Document: A Query-Conditional Benchmark and Two-Stage Retriever for LLM Agent Skill Routing**。

它讲的不是“怎么让 Agent 多装几个工具”，而是一个更容易被低估的问题：**Agent 有很多 skill 时，怎么把正确的一组 skill 找出来？**

这件事看起来像检索，但论文标题已经把结论写出来了：**Skill 不是普通文档。**

## 1. 为什么这篇论文值得读？

很多 Agent 系统都会从“几个工具”长成“一堆工具”：搜索、浏览器、代码执行、数据库、文档、日历、消息、内部 API、各种自定义 skill。

当 skill 数量变多，真正的问题就来了：

- 用户一句话可能需要多个 skill 配合，而不是一个最相关 skill。
- 单个 skill 看起来相关，不代表和其他 skill 组合起来能完成任务。
- 错误 skill 被召回后，后面的规划和执行都会被带偏。
- 传统文档检索只看 query-document 相关性，不太关心“这组结果能不能一起完成任务”。

这篇论文正好把这个问题单独拎出来：**Agent Skill Routing 应该有自己的 benchmark 和训练信号。**

## 2. 论文信息卡

- 论文：Skill Is Not Document: A Query-Conditional Benchmark and Two-Stage Retriever for LLM Agent Skill Routing
- arXiv：2606.03565
- 方向：Information Retrieval / Agent Skill Routing
- 作者：Zifei Wang、Wei Wen、Qiang Ji、Ruizhi Qiao、Xing Sun
- 提交时间：2026-06-02
- 篇幅：19 pages, 8 figures
- 核心产物：R3-Skill 双语 benchmark；R3-Embedding + R3-Reranker 两阶段检索系统
- 论文报告指标：Hit@1 = 0.7714，NDCG@10 = 0.8327，Set-Compat = 0.3525

这些指标来自 arXiv 摘要。正式使用前仍要看全文里的数据集构造、baseline、消融和开源仓库状态。

## 3. 它的核心问题：skill 组合正确，不等于每个 skill 单独相关

普通文档检索里，我们通常问：这个文档和 query 相关吗？

但 Agent skill routing 里，更关键的问题是：

> 这组 skill 放在一起，能不能完成当前任务？

比如用户说：

“帮我读这个网页，整理重点，然后发到 Discord。”

单看相关性，网页读取、摘要、Discord 发送都可能相关。但如果召回了“发邮件”而不是“发 Discord”，或者漏掉“网页读取”，整个任务就会失败。

论文把这种组合层面的正确性称为 skill compatibility。它不是独立相关性简单相加能得到的。

## 4. 方法拆解：R3 把“拒绝信号”变成资源

这篇论文最有意思的点，是 Reject-as-Resource Retriever，也就是 R3。

很多 LLM 数据合成流程里，模型会产生一类信号：**哪些 skill 不应该在当前 query 下被一起召回。**

通常这些 rejection decision 会被当成低质量数据扔掉。但论文认为，这恰恰是训练 skill routing 的有用监督信号。

它的系统是两阶段：

第一阶段，R3-Embedding 负责粗召回。它像传统 bi-encoder，适合快速从大量 skill 里捞候选。

第二阶段，R3-Reranker 负责重排。论文的分析认为，“push-away” 这类拒绝信号在 bi-encoder 里容易被双边平衡稀释，但在 cross-encoder reranker 里可以作为更细的排序监督。

工程上可以理解为：

- embedding 负责“先别漏太多”；
- reranker 负责“这组 skill 到底该不该一起上”。

## 5. 对 Agent 工程有什么启发？

我觉得最值得带走的是三点。

第一，skill 路由要评测 top-K 集合，而不是只评测 top-1。

Agent 任务经常需要工具组合，所以只看第一个召回是否命中，会低估组合错误。

第二，负样本不是垃圾。

“这个 skill 不该被选中”在 skill routing 里非常有价值。以后做内部 Agent skill 数据集时，不要只存成功轨迹，也要存错误召回和人工驳回原因。

第三，skill 描述不要写成普通文档。

一个好 skill card 应该至少包含：适用任务、输入约束、输出格式、权限边界、不能做什么、常见误用。否则检索器只能按语义相似度猜。

## 6. 如果要落地，可以怎么试？

不要一上来训练模型。可以先做一个轻量版本：

1. 把现有 tools/skills 整理成 skill card。
2. 每张 card 写清楚：用途、输入、输出、权限、禁用场景。
3. 收集 50-100 条真实用户请求。
4. 人工标注“应该召回哪些 skill”和“不应该召回哪些 skill”。
5. 先用 embedding 做候选召回，再用 LLM/reranker 做重排。
6. 评测 top-K skill set 是否能完成任务，而不是只看文本相关性。

最小 checklist：

- 是否存在多个 skill 单独相关但组合错误？
- 是否记录了被拒绝 skill 的原因？
- 是否区分“语义相关”和“任务可执行”？
- 是否有 Set-Compat 这类集合级指标？
- 是否能回放一次错误路由的完整链路？

## 7. 局限和边界

这篇论文解决的是 skill routing，不是完整 Agent 可靠性。

即使 skill 选对了，后面仍然可能在参数生成、权限确认、工具执行、结果校验、外部发送等环节出错。

另外，论文指标不能直接等价于线上收益。线上系统还要看 latency、成本、权限隔离、日志可观测性，以及 skill 版本变化后的回归测试。

## 8. 我的判断

这篇论文真正有价值的地方，是把 Agent 里一个很具体的基础设施问题讲清楚了：**工具多了以后，路由本身就是一个需要评测和训练的系统。**

如果你在做 Agent、OpenClaw、MCP、企业内部工具调用，这篇值得收藏。哪怕暂时不用 R3，也应该把“skill 不是文档”这句话写进你的工具治理规范里。

最后留个问题：你现在的 Agent 系统里，工具调用失败更多是因为“工具不会用”，还是一开始就“选错了工具”？


---

> 本文同步自微信公众号「AltenAI观察」的公开归档。完整每日推送与后续精读欢迎搜索关注。
