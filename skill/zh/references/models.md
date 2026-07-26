# 按模型调校：Opus 5 与 Fable 5

> 片段编号（如 4.2）指向 `snippets.md`。

### Claude Opus 5（默认 effort: high；thinking 默认开启）

| 已知倾向 | 处方 |
|---|---|
| 用户可见回复默认偏长；**降 effort 不能可靠缩短可见输出** | 显式写简洁指令（4.2） |
| Agent 场景播报频繁、单条消息长 | 显式描述播报节奏（4.3） |
| 写到磁盘的文档偏长 | 长度校准指令（4.4） |
| **自我验证充分** | **删除**所有验证类指令（见 SKILL.md 3.4） |
| 自行扩大任务范围 | 范围约束（4.5） |
| 更积极派生子智能体 | 显式设限（4.10） |
| 更频繁地口头纠正自己此前的说法 | `Only correct an earlier statement when the error would change the user's code, conclusions, or decisions. State corrections plainly and briefly, then continue the task. For slips that change nothing for the user, make the fix and move on without noting it.` |
| 代码审查中"只报高危"会被字面执行 | 用 4.16 |
| `low` / `medium` effort 质量强，而 token 与延迟只是高档位的一个零头 | **从默认 `high` 起步**，只有足够重的编码与 agent 工作才上 `xhigh`。评测显示质量不掉的地方放开用 `low`/`medium`，把它当成控成本和控延迟的主要手段。设置若从更早的模型沿用而来，重新做一次 effort 扫描 |
| 关闭 thinking 时可能把 tool call 写成文本（该调用不会执行，且泄漏文本会留在历史里污染后续轮次） | **首选保持 thinking 开启**，用低 effort 控成本。必须关闭时加：`You may say a brief sentence before using a tool.` |
| 关闭 thinking 时可能把 `<thinking>` 等内部标签泄漏到可见输出 | 删掉 system prompt 里"不要思考/不要推理"的规则（它会加剧泄漏），改用通用形式：`Do not include internal or system XML tags in your response.` 不要点名 thinking 标签，点名反而更差 |

其他：1M token 上下文为默认且最大；`xhigh`/`max` 下不能关闭 thinking——在这两档设 `thinking: {"type": "disabled"}` 会返回 400。多数任务上，**开着 thinking 跑 `low` effort，效果好于在相近成本下关掉 thinking**。**在 `xhigh` 或 `max` effort 下**运行时要给 `max_tokens` 留足空间，官方建议以 64k 为合理起点。

### Claude Fable 5 / Mythos 5（默认 effort: high；thinking 始终开启且不可关闭）

定位：处理此前"太复杂、太长、太模糊"的问题，擅长人类需要数小时到数周的端到端工作。

| 已知倾向 | 处方 |
|---|---|
| **单次请求可能跑很多分钟，自主运行可达数小时** | 迁移前先调整客户端超时、流式和进度提示；考虑改成异步轮询而非阻塞等待 |
| 任务模糊时会过度规划 | `When you have enough information to act, act. Do not re-derive facts already established in the conversation, re-litigate a decision the user has already made, or narrate options you will not pursue in user-facing messages. If you are weighing a choice, give a recommendation, not an exhaustive survey. This does not apply to thinking blocks.` |
| 高 effort 下会做未被要求的整理/重构 | 用 4.6 的 **Fable 5 专用加强版**减法指令 |
| **指令遵循能力强到可以一句话概括**，不必逐条枚举 | 用简短指令代替长清单 |
| 长自主运行中可能编造进度 | **4.17（这是 Fable 5 最值钱的一条）** |
| 会做未被要求的动作（自动起草邮件、建防御性 git 分支） | 明确划边界，用 4.24 |
| 更主动派并行子智能体 | **鼓励使用**，但给明确的委派判据；优先异步通信而非阻塞等待；长生命周期子智能体能靠缓存读取省时省钱 |
| 深入长会话后偶尔只说"我这就去做 X"却不发工具调用 | 一句 "continue" 即可；自主管线用 4.18 |
| 看到剩余 token 倒计时会提议开新会话 | 4.19 |
| **为旧模型写的 skills 往往过于规定化，会降低 Fable 5 的输出质量** | 审查并删减；Fable 5 也能根据任务自行更新 skill |
| 让它复述内部推理会触发拒绝 | 见 SKILL.md 3.9 |

**脚手架建议**：从你难度范围的**上限**开始试（挑一个比你会派给旧模型更难的任务），让它先界定范围、提澄清问题、再执行。

### ⚠️ 验证指令：Opus 5 与 Fable 5 方向相反

这是最容易搞错的一处，两条官方指引看似矛盾，实则场景不同：

| 场景 | 做法 | 依据 |
|---|---|---|
| **Opus 5，常规任务** | **删除**所有验证/复查指令 | 它自己会验证；保留会造成过度验证 |
| **Fable 5，长时自主运行** | **显式写入**周期性自我检查 | 官方原文：`Establish a method for checking your own work at an interval of [X] as you build. Run this every [X interval], verifying your work with subagents against the specification.` |

关键区别：官方同时指出，**独立的、干净上下文的 verifier 子智能体，效果优于模型自我批评**。所以"删验证"删的是**让模型自己再看一遍**，"加验证"加的是**换一个上下文来查**。两者不冲突。

（注意这与 4.10「不要用子智能体验证自己的工作」也不冲突——4.10 针对 Opus 5 的常规任务，此处针对 Fable 5 的多小时自主运行。）

### effort 取值参考

`max`（无上限）> `xhigh`（长周期 agent/编码）> `high`（默认）> `medium`（成本敏感）> `low`（短任务、子智能体、低延迟）

**两个模型都从默认 `high` 起步，区别在什么时候上调：**

- **Opus 5**：**足够重**的编码与 agent 工作上 `xhigh`；值得不设上限烧 token 的任务上 `max`。评测显示质量不掉的地方，放开用 `low`/`medium`，把它当成控制 token 成本和响应时间的主要手段
- **Fable 5**：只有最吃能力的任务才上 `xhigh`，日常工作降到 `medium`/`low`。它的低 effort 常常还优于旧模型的 `xhigh`

官方对 `xhigh` 的定位是"面向长周期工作的扩展能力"：**超过 30 分钟、token 预算以百万计**的长周期 agentic 与编码任务——这就是"足够重"的具体标准。`low` 的定位是"最省"，用于需要最快速度与最低成本的简单任务，官方点名的例子是子智能体。

**如果 effort 设置是从更早的模型沿用过来的，要在自己的评测上重新做一次 effort 扫描，而不是直接照搬。** 两个模型的页面都写了这一条。

要点：
- `effort: "high"` 与完全不传该参数**行为完全相同**
- effort 是**行为信号而非硬预算**，影响正文、工具调用和思考的全部 token
- **低 effort 会直接减少工具调用次数**、合并操作、省去前言；高 effort 则更多工具调用、先讲计划、给详细总结
- 启用 prompt caching 的会话中不要中途改 effort，会导致缓存失效
- 要给 `max_tokens` 留足空间——它是思考加正文的**总**上限。Opus 5 在 `xhigh`/`max` 下要注意（官方建议 64k 起）；Fable 5 的页面则在 `high` 和 `xhigh` 两档都点了这件事。此项仅在直接调 API 时需要关心
