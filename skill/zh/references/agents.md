# Agent 与长任务：上下文工程

> 本文件中的英文块均为 Anthropic 官方**原文**（出处见 `SOURCES.md`）。文中的 API 细节只在自己拼请求时才用得上：用 Claude Code、claude.ai、Claude Managed Agents 或 Claude Agent SDK 时，对话历史由它们管理。

## 目录

- 上下文是有限资源
- 跨上下文窗口的工作流
- 状态管理
- 上下文压缩感知
- 复杂研究任务
- 让 Claude 自己编排，而不是把一切塞进上下文
- 设计上下文以最大化缓存命中
- 什么时候该把 bash 动作提升为专用工具
- 工具设计

## 上下文是有限资源
性能随上下文填充而下降（context rot）。目标不是把你知道的都告诉它，而是**能最大化达成目标概率的最小高信噪比 token 集**。

## 跨上下文窗口的工作流
1. **第一个上下文窗口用不同的 prompt**：先搭框架（写测试、建 setup 脚本），后续窗口再迭代 todo list
2. **让模型用结构化格式写测试**（如 `tests.json`），并提醒：`It is unacceptable to remove or edit tests because this could lead to missing or buggy functionality.`
3. **建 `init.sh` 之类的脚本**：起服务、跑测试、跑 lint——避免每个新窗口重复劳动
4. **上下文窗口被清空时，可以考虑用「全新窗口」代替 compaction**：现行模型极擅长从文件系统恢复状态，**某些场景下**可以利用这一点来代替压缩（官方措辞是 "consider" / "in some cases"，不是一律如此）。这时开场要具体：
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
- 版本与检查点 → git。现行模型尤其擅长用 git 跨会话追踪状态
- 明确要求增量推进

## 上下文压缩感知

如果你的 agent harness 会压缩上下文，或者允许把上下文存到外部文件（比如 Claude Code），最佳实践页建议在 prompt 里加上这段，让 Claude 据此行事。这段放在该页的 context awareness 一节下，它列出具备这项能力的是 Sonnet 5、Sonnet 4.6、Sonnet 4.5 和 Haiku 4.5。Context windows 页写明，Opus 4.7 及之后的 Opus 模型、Sonnet 5.5、Fable 5.2 和 Haiku 5.5 收不到那些注入的预算标签，在这些模型上要给模型一个明确的预算，用 task budgets（beta）。所以这段是在那几个更早的模型上测得的。给 Fable 5.2 用之前另看一条：Fable 5.2 专页说，harness 写进 prompt 的 token 预算会改变它花多少，建议去掉预算试一下：
```
Your context window will be automatically compacted as it approaches its limit, allowing you to continue working indefinitely from where you left off. Therefore, do not stop tasks early due to token budget concerns. As you approach your token budget limit, save your current progress and state to memory before the context window refreshes. Always be as persistent and autonomous as possible and complete tasks fully, even if the end of your budget is approaching. Never artificially stop any task early regardless of the context remaining.
```

**怎么压缩才不触发 prefix check**（仅 API；preserved thinking 页）。在客户端裁剪——丢掉或摘要最老的几轮、原样保留最近的——是一种对历史的改动：被保留轮次里的 thinking 块是在被删内容还在的时候产生的，会通不过检查。不算改动的做法有三种：

- **按需压缩**（compaction on demand，beta，`compact-2026-09-04`）：由一个单独的请求返回摘要，你把返回的块放在它概括的那些消息的位置上；被保留的轮次连同它们的 thinking 可以继续有效
- **到 token 阈值时压缩**：上下文接近你设的阈值时，把更早的轮次摘要进一个 compaction 块；它的 `instructions` 参数可以放你自己的摘要 prompt
- **context editing**：按规则清掉旧的工具结果或旧的 thinking 块，从最老的开始

压缩之后有两样东西不会自己延续（「Compaction and preserved thinking」页）：被摘要掉的那些轮次里的会话中 system 消息也被摘要了，它们的文字指令在替换之后不再生效；它们设的按消息 effort 也不延续，在后面的消息重新设档之前，轮次按请求顶层的 `output_config.effort` 跑，没设就是模型的默认档。要让某条指令或某个档位继续有效，在保留轮次之后的第一个新 `user` 回合后面，用一条 `role: "system"` 消息再说一遍。

自己在客户端写摘要时，官方推荐的是 **simple compaction**：把整个会话摘要成一条 user 消息，只发这条消息加下一条指令，更早的什么都不回放——没有旧的 thinking 被发回去，也就没有东西会通不过检查。官方原话：`Claude models are trained on long-horizon tasks with this scheme, and for most workloads it performs well.` 不要在一个工具回合中间压缩，也不要从中间挖掉几轮。

自己做摘要那一步用的指令，只发布在 Fable 5.1 专页上（在 Fable 5.1 上测得；Fable 5.2 的页面没有重刊）。同一页还说，Fable 5.1 的缓存读取是输入价的 2.5%，为省钱而提早压缩 `can be the wrong trade-off between cost and intelligence`，建议试试更晚的压缩点；Fable 5.2 的缓存读取是同样的倍率，但它的页面没有重述这条建议：

```
Summarize the transcript inside <summary></summary> tags. Include relevant information in the summary such that this conversation will be continued by a new context window without needing to redo work or be reprovided with relevant constraints or context. Be sure to preserve: (1) any difficulties or problems that came up, and how they were handled or resolved; (2) any possibilities, options, or approaches that were raised, tried, or set aside, and why; (3) anything that was asked for, decided, agreed, ruled out, or established as a preference, constraint, or boundary — stated exactly; (4) exactly where things stand now — what has been covered, settled, or completed so far; (5) anything still open, unresolved, promised, or expected to happen next; (6) specific details that would be hard to reconstruct — names, numbers, dates, exact wording, links or references — kept exactly. Be complete on these even at the cost of length; keep everything else concise. Weight the two voices differently: keep what the user said, asked for, shared, or established carefully and close to their own words; your own explanations and reasoning can be condensed much further, to what they concluded or produced — as long as nothing in the six items above is dropped.
```

## 复杂研究任务
```
Search for this information in a structured way. As you gather data, develop several competing hypotheses. Track your confidence levels in your progress notes to improve calibration. Regularly self-critique your approach and plan. Update a hypothesis tree or research notes file to persist information and provide transparency. Break down this complex research task systematically.
```

## 让 Claude 自己编排，而不是把一切塞进上下文

**不要把每个工具结果都当 token 塞回上下文窗口。** 文章的说法：在工具设计里用写死的过滤器也能处理，但那样仍是 harness 在替 Claude 做一个它自己更适合做的编排决定。

给 Claude 代码执行能力（bash、语言 REPL），让它自己写逻辑来表达工具调用、过滤输出——**只有执行结果进入它的注意力**。官方数据：在 BrowseComp 网页浏览任务上，给 Opus 4.6 输出过滤能力使准确率从 **45.3% 提升到 61.6%**。

同一思路的另外两条：
- **让 Claude 自己管上下文**：用 skills 做渐进式披露（frontmatter 常驻、正文按需），用 context editing 清掉过期的工具结果，用子智能体隔离上下文（BrowseComp 比最好的单智能体结果高 2.8%，Opus 4.6）
- **让 Claude 自己持久化上下文**：compaction（在 BrowseComp 上，同样设置下 Sonnet 4.5 停在 43%，Opus 4.6 到了 84%）+ memory 文件夹（给 Sonnet 4.5 一个 memory 文件夹，BrowseComp-Plus 准确率从 60.4% 升到 67.2%）

> 出处说明：以上数字来自 Anthropic 在**更早一代模型**上的实测（Opus 4.6 和 Sonnet 4.5）。文章的论点是模型越强，越可以把决策交还给它，同时 `Assumptions about what Claude can't do need to be re-tested with each step change in its capability`——所以要在现行模型上重新验证，而不是假定这些数字能直接沿用。

## 设计上下文以最大化缓存命中

**缓存读取的成本是基础输入 token 的 10%**（Opus 5.5 和 Sonnet 5.5 上是 5%，Fable 5.2 和 Mythos 5.2 上是 2.5%），所以这是纯粹的成本杠杆。缓存写入要多付：5 分钟 TTL 的写入是基础输入价的 1.25 倍，1 小时 TTL 是 2 倍。并发请求时，缓存条目要等第一个响应开始之后才可用，需要并行请求命中缓存就先等第一个响应。五条原则：

| 做法 | 要点 |
|---|---|
| **静态在前** | 稳定内容（系统提示词、工具定义）放在动态内容之前 |
| **用消息追加而非改提示词** | 需要注入提醒时，追加到 messages 里，不要去改 system prompt |
| **不要中途换模型** | 缓存是按模型隔离的 |
| **工具增删会作废缓存** | 需要动态发现工具时用 tool search，或用会话中工具变更（见下文），而不是增删工具定义 |
| **移动断点** | 多轮应用里把 cache breakpoint 移到最新消息 |

> 直接后果：**在请求之间改顶层 effort 会作废缓存**，同理。四个现行模型和 Mythos 5.2 支持按消息改 effort（beta，header `mid-conversation-output-config-2026-07-01`），缓存不失效——见 `models.md`「effort 取值参考」。

**第二条的机制。** prompt caching 按 `tools` → `system` → `messages` 的顺序对请求前缀做哈希，命中要求前缀逐字节一致。所以往 system prompt 末尾追加一句话，作废的不只是 system prompt，还包括它之后所有已缓存的消息。如果某条指令是跑到中途才变得相关，就在 `messages` 里那个位置追加一条 `{"role": "system"}` 消息，而不是去改顶层 `system` 字段：缓存前缀保持不变，指令仍然以系统指令的身份生效，并且对其后的轮次优先于顶层字段。四个现行模型和 Mythos 5.2 支持，不需要 beta header。位置有限制：带内容的 system 消息不能是 `messages` 的第一条，必须紧跟在一个 `user` 回合（包括带 `tool_result` 的）或以 server tool 结果结尾的 assistant 回合之后，不能夹在 `tool_use` 和它的 `tool_result` 之间，放错位置返回 400。也不要把对话之外的文字（原始工具输出、检索到的文档、网页内容）直接放进 system 消息：Claude 把 system 内容当作 operator 的指令来执行，这样做等于给了那段文字 operator 级别的权威，这类数据留在 `tool_result` 块里。措辞上，同一页建议把 system 内容写成陈述情况，而不是推翻用户的命令：说出事实（它的例子是 `new input arrived from the user: X`），让 Claude 据此行动；`ignore what the user said` 这类话不如说明什么变了有效。在 Sonnet 5.5 和 Haiku 5.5 上，工具循环中途转达用户的话要放进 user 回合：这两个模型可能把紧跟在工具结果之后的 system 消息当成不可信文本而忽略，所以把用户的话作为 `text` 块接在同一条 `user` 消息里最后一个 `tool_result` 之后，harness 的提醒仍放在其后单独的 system 消息里。同一机制也管工具（beta，`inline-tools-2026-09-15`）：在 `tools` 里一开始就声明全套工具，之后用 `role: "system"` 消息里的 `tool_addition` / `tool_removal` 块，从对话的某一点起提供或撤回某个工具——`tool_addition` 还可以直接带上一个首个请求时还不知道的工具的完整定义。`tools` 数组本身始终不变，缓存前缀保持完整。`role: "system"` 消息还可以用 `clear_at: "next_user_message"` 限定只对本轮生效（beta，header `mid-conversation-system-clear-at-2026-08-21`）：当前轮里它有 system prompt 的权威，出现更新的 user 消息后就不再渲染，本身仍留在数组里所以前面什么都没变，清掉后也不计输入 token——这是官方给工具循环里每轮一条提醒安排的位置。

**历史只追加不修改（四个现行模型都是）。** 每个 assistant 回合原样追加（含 thinking 块，空的也要），请求之间不要改动历史轮次。thinking 块只对产生它时前面的那些内容有效：`system` prompt、`tools`、或任何更早的消息变了之后再回传它，就通不过 prefix check。最佳实践页的原话：`editing earlier messages, rebuilding system or tools, or summarizing older turns in place between requests invalidates every later thinking block`。通不过时会怎样，按模型和账号不同（Messages API，`prefix_mismatch_behavior` 没设时；在 Message Batches API 里，表中写 400 的地方改为丢掉通不过的块）：

| 模型 | 2026-08-31 及之后创建的账号 | 更早创建的账号 |
|---|---|---|
| Fable 5.2、Mythos 5.2 | 400 | 400 |
| Opus 5.5、Sonnet 5.5、Haiku 5.5（Fable 5.1 也在这一行，从它迁移时用得上） | 400 | 不报错，失效的块照常到达模型 |

设了 `thinking.block_binding.prefix_mismatch_behavior: "drop_block"`（beta，`thinking-binding-controls-2026-08-01`）则改为丢弃失效的块并继续，响应里的 `input_transformations` 会列出被丢的块。Fable 5.2 的迁移指南把它定位成修代码期间的权宜之计：丢块可能让输出 token 变多；Fable 5.2 专页说，在 Anthropic 的测试里每一轮都丢掉之前的思考时，模型会重新思考、多用输出 token，那是最坏的情况。对 Sonnet 5.5、Haiku 5.5、Fable 5.2 和 Mythos 5.2 的块来说，通过 MCP connector 连接的服务器在下一个请求返回了不同的工具列表、而工具集又没有固定时，也算一次改动；在 Claude API 上的修法是把服务器的工具列表固定下来。

**所以 prompt 上的改动只在新会话里做**（Fable 5.2 和 Haiku 5.5 专页都这样写）：往 system prompt 里加文字、删掉 agent 带着的笔记和规则、改 tool description，都会改变前缀。会话中途要改指令，放进最新一轮，或追加一条会话中 system 消息。

**thinking 块还和产生它的模型绑定。** 换模型之后推理能不能跟过去，看目标模型读不读得了来源模型的块（Claude API；preserved thinking 页的表）：

| 从哪个模型换走 | 换到 Fable 5.2 / Mythos 5.2 | 换到 Opus 5.5 | 换到 Sonnet 5.5 | 换到 Haiku 5.5 |
|---|---|---|---|---|
| Fable 5.2 / Mythos 5.2 | 保留 | 丢失 | 丢失 | 丢失 |
| Opus 5.5 | 保留 | 保留 | 丢失 | 丢失 |
| Sonnet 5.5 | 保留 | 保留 | 保留 | 丢失 |
| Haiku 5.5 | 保留 | 保留 | 保留 | 保留 |

要点是只有 Fable 5.2 和 Mythos 5.2 读得了 Fable 5.2 的块：路由、客户端重试或拒绝后的 fallback 把对话从 Fable 5.2 换到别的模型——包括它自己的 fallback 目标 Opus 4.8 和 Opus 5——之后的轮次就没有它的推理了。请求里带了目标模型读不了的块时，API 会在模型看到之前丢掉它，请求照常成功，被丢掉的块不计费；切换后的第一轮成本和延迟可能更高。Fable 5.2、Mythos 5.2、Sonnet 5.5、Haiku 5.5 产生的块还和产生它的账号绑定，只在那个账号或与它关联的账号里有效，换到别的账号回放时同样会被丢掉。

会踩到 prefix check 的改动，大多也会重置缓存——注入又删除每轮提醒、就地摘要旧轮次、会话中途改 system prompt——修法也一样：提醒用 turn-scoped system message，改指令和工具用会话中 system 消息和工具变更而不是重写 `system` 或 `tools`，裁剪上下文交给上面那三种不算改动的做法。

## 什么时候该把 bash 动作提升为专用工具

bash 杠杆大，但 harness 只能看到一串命令字符串——每种动作都是同一个形状；做成专用工具，harness 才有针对这个动作、带类型化参数的钩子，可以拦截、把关、渲染或审计。以下四种情况应当做成专用的声明式工具：

1. **安全边界**：难以撤销的动作（如调用外部 API），可以用用户确认来把关
2. **写操作需要陈旧性检查**：防止覆盖已被改动的文件
3. **需要面向用户呈现**：弹窗、多选项展示
4. **需要结构化日志、追踪和回放**

（还有另一条路，但只应该用在用户信任大方向的任务上：用第二个 Claude 实例判断 bash 命令是否安全，这是 Claude Code auto 模式的做法，可以减少对专用工具的需要；某些高风险动作仍值得用专用工具。）

## 工具设计

出自 Anthropic 的「Writing effective tools for agents」和「Effective context engineering for AI agents」两篇文章：
- 每个工具有清楚、彼此不同的用途（`Make sure each tool you build has a clear, distinct purpose`）。判断法：`If a human engineer can't definitively say which tool should be used in a given situation, an AI agent can't be expected to do better`
- 参数名不要有歧义（`user_id` 而非 `user`）
- 工具返回有意义的标识符：把无语义的 UUID 解析成可读名称（甚至用从 0 开始的编号），能减少幻觉、显著提升 Claude 在检索任务上的精度
- 按服务和资源类型做命名空间（`asana_projects_search`、`asana_users_search`）
- 返回值 token 高效：分页、范围选择、过滤、截断，并给合理默认上限（Claude Code 的默认是 25000 token）
- 提供 `ResponseFormat` 枚举（`concise` / `detailed`）——官方给的例子里详细版 206 token、精简版 72 token，约为三分之一
- 错误信息要可执行：说清怎么改，附正确格式示例，而不是抛栈
- 工具描述当作给新同事的入职说明来写，把隐含前提写明——专有查询格式、术语、资源之间的关系
