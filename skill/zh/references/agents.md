# Agent 与长任务：上下文工程

> 片段编号（如 4.17）指向 `snippets.md`。
>
> 本文件中的英文块均为 Anthropic 官方长任务指引**原文**（见 `SOURCES.md`），照抄即可，不要翻译或改写。

## 目录

- 上下文是第一约束
- 跨上下文窗口的工作流
- 状态管理
- 上下文压缩感知
- 复杂研究任务
- 让 Claude 自己编排，而不是把一切塞进上下文
- 设计上下文以最大化缓存命中
- 什么时候该把 bash 动作提升为专用工具
- 工具设计优先于提示词

## 上下文是第一约束
性能随上下文填充而下降（context rot）。目标是**能最大化达成目标概率的最小高信噪比 token 集**。

## 跨上下文窗口的工作流
1. **第一个上下文窗口用不同的 prompt**：先搭框架（写测试、建 setup 脚本），后续窗口再迭代 todo list
2. **让模型用结构化格式写测试**（如 `tests.json`），并提醒：`It is unacceptable to remove or edit tests because this could lead to missing or buggy functionality.`
3. **建 `init.sh` 之类的脚本**：起服务、跑测试、跑 lint——避免每个新窗口重复劳动
4. **可以考虑用「全新窗口」代替 compaction**：现行模型极擅长从文件系统恢复状态，**某些场景下**这比压缩更划算（官方措辞是 "consider" / "in some cases"，不是一律如此）。这时开场要具体：
   - `Call pwd; you can only read and write files in this directory.`
   - `Review progress.txt, tests.json, and the git logs.`
   - `Manually run through a fundamental integration test before moving on to implementing new features.`
5. **提供验证工具**（浏览器自动化、computer use 等）——任务越长、越自主，模型就越需要在没有持续人类反馈的情况下自行验证正确性
6. **鼓励用满上下文**：
```
This is a very long task, so it may be beneficial to plan out your work clearly. It's encouraged to spend your entire output context working on the task - just make sure you don't run out of context with significant uncommitted work. Continue working systematically until you have completed this task.
```

## 状态管理
- 结构化数据（测试结果、任务状态）→ JSON
- 进度笔记 → 自由文本
- 版本与检查点 → git（现行模型用得很好）
- 明确要求增量推进

## 上下文压缩感知
```
Your context window will be automatically compacted as it approaches its limit, allowing you to continue working indefinitely from where you left off. Therefore, do not stop tasks early due to token budget concerns. As you approach your token budget limit, save your current progress and state to memory before the context window refreshes. Always be as persistent and autonomous as possible and complete tasks fully, even if the end of your budget is approaching. Never artificially stop any task early regardless of the context remaining.
```

**如果压缩是在客户端做的**（Fable 5.1 专页）：服务端压缩已经会告诉模型该保留什么；自己做摘要那一步，官方给的指令在下面。两个适用模型上现在还多了第三种选择——**按需压缩**（compaction on demand，beta，`compact-2026-09-04`）：传顶层 `compaction` 参数，API 返回一个带签名的 `compaction` 块来概括整轮对话，之后把它放在最前面替换掉被概括的那些消息。压缩时机由你决定，而且被保留下来的那些轮次里的 thinking 块在替换之后仍可能有效——这一点在 Opus 5.5 上尤其要紧，正因为它的 thinking 块和对话绑定。在 Fable 5.1 上最简单的形式是：用一条摘要消息加新的 user 消息替换整个历史，其他什么都不回放——没有 thinking 块被带过去，就不会撞上「历史只追加不修改」的检查。另外 Fable 5.1 的缓存读取更便宜，为省钱而提早压缩可能已经不划算，试试更晚的压缩点。

```
Summarize the transcript inside <summary></summary> tags. Include relevant information in the summary such that this conversation will be continued by a new context window without needing to redo work or be reprovided with relevant constraints or context. Be sure to preserve: (1) any difficulties or problems that came up, and how they were handled or resolved; (2) any possibilities, options, or approaches that were raised, tried, or set aside, and why; (3) anything that was asked for, decided, agreed, ruled out, or established as a preference, constraint, or boundary — stated exactly; (4) exactly where things stand now — what has been covered, settled, or completed so far; (5) anything still open, unresolved, promised, or expected to happen next; (6) specific details that would be hard to reconstruct — names, numbers, dates, exact wording, links or references — kept exactly. Be complete on these even at the cost of length; keep everything else concise. Weight the two voices differently: keep what the user said, asked for, shared, or established carefully and close to their own words; your own explanations and reasoning can be condensed much further, to what they concluded or produced — as long as nothing in the six items above is dropped.
```

## 复杂研究任务
```
Search for this information in a structured way. As you gather data, develop several competing hypotheses. Track your confidence levels in your progress notes to improve calibration. Regularly self-critique your approach and plan. Update a hypothesis tree or research notes file to persist information and provide transparency. Break down this complex research task systematically.
```

## 让 Claude 自己编排，而不是把一切塞进上下文

这是 harness 层面收益最大的一条。**不要把每个工具结果都当 token 塞回上下文窗口**，也不要把过滤逻辑硬编码进工具设计。

给 Claude 代码执行能力（bash、语言 REPL），让它自己写逻辑来表达工具调用、过滤输出——**只有执行结果进入它的注意力**。官方数据：在 BrowseComp 网页浏览任务上，给模型输出过滤能力使准确率从 **45.3% 提升到 61.6%**。

同一思路的另外两条：
- **让 Claude 自己管上下文**：用 skills 做渐进式披露（frontmatter 常驻、正文按需），用 context editing 清掉过期的工具结果，用子智能体隔离上下文（BrowseComp +2.8%）
- **让 Claude 自己持久化上下文**：compaction + memory 文件夹（agentic search 任务 60.4% → 67.2%）

> 出处说明：以上三组数字来自 Anthropic 在**更早一代模型**上的实测。文章本身的论点是这个趋势会随模型变强而延续——模型越强，越该把决策交还给它。

## 设计上下文以最大化缓存命中

**缓存 token 的成本只有基础输入 token 的 10%**（Opus 5.5 上是 5%，Fable 5.1 和 Mythos 5.1 上是 2.5%），所以这是纯粹的成本杠杆。五条原则：

| 做法 | 要点 |
|---|---|
| **静态在前** | 稳定内容（系统提示词、工具定义）放在动态内容之前 |
| **用消息追加而非改提示词** | 需要注入提醒时，追加到 messages 里，不要去改 system prompt |
| **不要中途换模型** | 缓存是按模型隔离的 |
| **工具增删会作废缓存** | 需要动态发现工具时用 tool search，而不是增删工具定义 |
| **移动断点** | 多轮应用里把 cache breakpoint 移到最新消息 |

> 直接后果：**在请求之间改顶层 effort 会作废缓存**，同理（见 SKILL.md 第 2 节）。Opus 5.5、Fable 5.1、Mythos 5.1 和 Opus 5 支持按消息改 effort（beta），缓存不失效——见 `models.md`。

**第二条的机制。** prompt caching 按 `tools` → `system` → `messages` 的顺序对请求前缀做哈希，命中要求前缀逐字节一致。所以往 system prompt 末尾追加一句话，作废的不只是 system prompt，还包括它之后所有已缓存的消息。如果某条指令是跑到中途才变得相关，就在 `messages` 里那个位置追加一条 `{"role": "system"}` 消息，而不是去改顶层 `system` 字段：缓存前缀保持不变，指令仍然以系统指令的身份生效，并且对其后的轮次优先于顶层字段。Fable 5.1、Mythos 5.1、Fable 5、Mythos 5、Opus 5.5、Opus 4.8、Opus 5 支持且不需要 beta header；**Sonnet 5 不支持**，那边只能用顶层 `system` 字段。`role: "system"` 消息还可以用 `clear_at: "next_user_message"` 限定只对本轮生效（beta，header `mid-conversation-system-clear-at-2026-08-21`）：当前轮里它有 system prompt 的权威，出现更新的 user 消息后就不再渲染，本身仍留在数组里所以前面什么都没变，清掉后也不计输入 token——这是官方给「工具循环里每轮一条提醒」安排的位置。

**历史只追加不修改（Fable 5.1 和 Opus 5.5）。** 每个 assistant 回合原样追加（含 thinking 块），请求之间不要改动历史轮次。在这两个模型上，thinking 块都只在产生它的那个对话里有效：2026 年 8 月 31 日及之后创建的账号，一旦它前面的内容（system prompt、工具列表或任何更早的消息）变了再回放它，就返回 400；设了 `thinking.block_binding.prefix_mismatch_behavior: "drop_block"`（beta，`thinking-binding-controls-2026-08-01`）则改为丢弃该块；更早创建的账号把这个字段设成任一个值都等于主动加入这项检查。最佳实践页对两者都写明了：`On Claude Fable 5.1 and Claude Opus 5.5, modifying the conversation before a thinking block results in an error, or in the block being dropped if you opt into that`。这是从 Opus 5 迁移过来的**四项破坏性变更**之一。

**thinking 块还和产生它的模型绑定。** Opus 5.5 能读 Opus 5 以及更早的 Opus、Sonnet、Haiku 模型的 thinking 块，但读不了 Claude Fable 或 Claude Mythos 系列的；在 Claude API 上，能读 Opus 5.5 的块的只有 Fable 5.1 和 Mythos 5.1。所以从 Opus 5 换到 Opus 5.5、或从 Opus 5.5 换到 Claude API 上的 Fable 5.1 / Mythos 5.1，推理会保留；换成其他任何模型，切换之后的轮次就没有前一个模型的推理了。请求里带了目标模型读不了的块时，API 会在模型看到之前丢掉它——请求照常成功，被丢掉的块不计费。所以如果你的路由或 fallback 可能把一轮对话在不同模型族之间搬动，就要预期推理不会跟着过去。会踩到它的改动和会重置缓存的改动是同一批——注入又删除每轮提醒、就地摘要旧轮次、会话中途改 system prompt——修法也一样：提醒用 turn-scoped system message，改指令和工具用会话中 system 消息和工具变更而不是重写 `system` 或 `tools`，裁剪上下文交给服务端压缩或 context editing。从最老的一端连续移除若干 thinking 块、移动 `cache_control` 标记、在请求之间改 `effort`，都不会让后面的块失效。

## 什么时候该把 bash 动作提升为专用工具

bash 杠杆大但缺乏可观测性，harness 无法拦截、把关或审计。以下四种情况应当做成专用的声明式工具：

1. **安全边界**：难以撤销的动作，需要用户确认（如调用外部 API）
2. **写操作需要陈旧性检查**：防止覆盖已被改动的文件
3. **需要面向用户呈现**：弹窗、多选项展示
4. **需要结构化日志、追踪和回放**

（低风险工作可以走另一条路：用第二个 Claude 实例判断 bash 命令是否安全，这是 Claude Code auto 模式的做法，可以减少专用工具的数量。）

## 工具设计优先于提示词
有工具的场景下，**改工具比改提示词更有效**：
- 每个工具功能互不重叠，模型能一眼分清何时调用哪个。判断法很直接——**如果工程师自己都说不清某个场景该用哪个工具，agent 更不可能分得清**
- 参数名语义明确（`user_id` 而非 `user`）——把无语义的 UUID 解析成可读名称能显著提升精度
- 按服务和资源类型做命名空间（`asana_projects_search`）
- 返回值 token 高效：分页、范围选择、过滤、截断，并给合理默认上限（Claude Code 的默认是 25000 token）
- 提供 `ResponseFormat` 枚举（`concise` / `detailed`）——官方给的例子里详细版 206 token、精简版 72 token，省 65%
- 错误信息要可执行：说清怎么改，附正确格式示例，而不是抛栈
- 工具描述当作给新同事的入职说明来写，把隐含前提写明
