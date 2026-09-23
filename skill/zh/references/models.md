# 按模型调校：Opus 5.5 与 Fable 5.1

> 片段编号（如 4.2）指向 `snippets.md`。
>
> 下表"处方"列里的英文块，**全部是官方文档原文**，照抄即可，不要翻译或改写。

## 目录

- Claude Opus 5.5——Opus 5.5 页面明确写了的；只在 Opus 5 页面写了、5.5 页面没有重述的
- Claude Fable 5.1 / Mythos 5.1——Fable 5.1 页面明确写了的；只在 Fable 5 页面写了、5.1 页面没有重述的
- 验证指令：Opus 5.5 与 Fable 5.1 方向相反
- effort 取值参考

## Claude Opus 5.5

2026 年 9 月 22 日发布。默认 effort **`medium`**；adaptive thinking 始终开启且**无法关闭**；1M token 上下文窗口；最大输出 128K；$4 / $20 每 MTok。模型 ID `claude-opus-5-5`。Claude Opus 5 现已列为仍可用的旧模型（legacy），价格 $5 / $25。

**Opus 5 的指引怎么沿用。** Opus 5.5 的 prompting 专页开头原文：`Existing Claude Opus 5 prompts should perform well without changes, and the patterns in Prompting Claude Opus 5 remain a reasonable starting point.` 通用最佳实践页另加了一条适用于所有模型专属技巧的提醒：`Where a technique names a specific model, treat it as measured on that model and re-check it against your own evals before applying it to another.` 所以本文件把两者分开——和处理 Fable 系列的做法完全一致：第一张表是 Opus 5.5 页面明确写了的；第二张表是只在 Opus 5 页面写了、Opus 5.5 页面既没有重述也没有撤回的。官方迁移指南自己的措辞也指向同一个方向：`Instructions tuned for Claude Opus 5's behavior may no longer be needed`。

### Opus 5.5 页面明确写了的

| 已知倾向 | 处方 |
|---|---|
| **默认 effort 是 `medium`**，Opus 5 是 `high`——不传 `effort` 的请求比以前低了一档 | 显式设 `effort` 并重做扫描。官方明确写了 Opus 5.5 在 `medium` **达到或超过 Opus 5 在 `high`** 的编码与知识工作评测成绩，而且若干编码评测上 `low` 已经很接近，成本低得多 |
| **同一档位下每回合思考得比 Opus 5 多**，`xhigh` 和 `max` 尤其明显 | 不要照搬 Opus 5 的值——照搬会让回合更长、输出 token 更多。只把 `xhigh` 和 `max` 留给**实测**到质量提升的工作。想少思考就**先降 effort 档位**：官方说这比用 prompt 指令更可靠 |
| 即使思考文本不返回给你，思考也计入 `max_tokens`；按 Opus 5 关闭 thinking 时的用量设的上限，会把回复截断 | 留足空间。面对 agentic 编码产生的长回合，官方原话是 `a max_tokens of 128,000, the model's maximum, has worked well in Anthropic's testing` |
| 在有多个部分的长任务上它会边做边向用户汇报，而**其中一些汇报是以文字而不是工具调用结束回合的**（`stop_reason: "end_turn"`）。无人值守的循环如果把这种回合当成任务结束，就停在那儿了 | 把「只有文字的回合」当成汇报，不是完成的证据。把任务的各个部分放进一份由模型自己更新的清单，回合结束时若还有未完成项且没说明阻塞原因，就发一条点名这些项的简短 user 消息（4.31 同时给了这条续跑消息和 system prompt 追加段）。自动续跑两三次之后就停，不要无限重复。如果模型启动的东西还在跑（后台命令、子智能体），先等它结束并把输出作为下一条 user 消息回传 |
| **工具调用之间的文字以 `thinking` 块返回**，不是 `text` 块——默认 `thinking.display: "omitted"` 下内容为空，所以只渲染 `text` 块的客户端在整个长 agentic 回合里都会显得没动静 | 先查客户端：设 `display: "updates"`（beta，`thinking-display-updates-2026-08-18`）拿到每条播报的摘要。然后用 4.3 的 Opus 5.5 块调节奏；用 4.23 的工具传递必须逐字送达用户的内容；回合仍然安静就用 4.3 里那条提醒。这是**相对 Opus 5 的响应形态变更**，不会让任何请求失败 |
| 跨多个已连接应用做流程自动化时**上手很快**，任务描述比较松散时会漏掉请求没有点名的信息 | 4.32。官方测得加上这条指令后，多应用自动化任务的正确完成率明显提高（`medium` 和 `max` 两档都是），代价是工具调用和 token 略多。因为它让模型照找到的东西去行动，**要把不可信内容挡在它会搜索的记录之外** |
| **对「已过去多少时间」这类信息非常敏感**——可以拿来给多智能体设置提速 | 4.33。在每条消息末尾给出时间预算（`elapsed 340s / 1200s`）；它会为了在预算内完成而调整节奏，而且通常远早于预算完成，所以预算要设得比你真正希望花的时间略高。预算只是建议性的——需要硬停就自己保留超时 |
| 在 chat 里**回答前会长时间思考**，拖慢首 token；后续轮次里即使是很短的追问，它有时也会回头重新过一遍之前的答案 | 从 system prompt 里删掉「回答前仔细思考」这类句子——官方测得删掉之后回复开始得更早，质量没有明显下降。追问这个问题用 4.34。希望它持续复查早先工作的场景，不要加 4.34 |
| 抵抗间接 prompt 注入的能力强于此前任何 Opus 模型；给对上下文时，对**用户粘贴进来的文本**里的指令同样稳健——但你得告诉它哪段是粘贴来的 | 4.35：给每段粘贴内容套上带同一个短随机 ID 的标签，并加上配套的 system prompt 说明。标签是纯文本、可以被伪造，所以把它当成多道防线中的一道 |
| 要求做前端但没给设计方向时，会落回少数几种默认样式；笼统地说「别一股 AI 味」基本只是把一种默认换成另一种 | 4.36——点名具体要避开的样式。迭代着做：看第一版改用了哪些样式，再把清单加长 |
| **不借助工具读图表、流程图和截图就比 Opus 5 准得多** | 重新检验为更早模型搭的视觉脚手架还需不需要。最密的输入上仍有两件事能加分：更高分辨率的图（工程制图尤其），以及图像处理工具——装有原图和 PIL / OpenCV 的容器，或者单独一个裁剪工具（单独裁剪工具就能拿到大部分提升）。effort 越高它越会用这些工具。不给工具时，提高 effort 能改善工程制图，但对图表帮助不大 |
| 安全分类器覆盖**生物、网络安全、推理抽取**三类。生物和 `reasoning_extraction` **对从 Opus 5 迁移过来的人是新增的** | 处理 `stop_reason: "refusal"` 并配置回落——但注意服务端 fallback **不会重试** `reasoning_extraction`，它会把拒绝直接返回给你。在源码里找漏洞是允许的；高风险两用网络安全活动不允许。日常健康与教育类问题不受生物分类器影响 |
| 要求模型把推理复述进正文的 prompt 可能被 `reasoning_extraction` 拒绝 | 见 SKILL.md 3.9。删掉这类指令，设 `display: "summarized"`，改从 thinking 块里读 |

**如果你的 Opus 5 集成原本关闭了 thinking**：Opus 5.5 不接受 `thinking: {"type": "disabled"}`（SKILL.md 3.2），迁移时还要配套改四件事：

- **从 `low` effort 起步并实测。** `low` 下它的思考很短；具体多大比例会完全跳过思考取决于你的 prompt，所以要在自己的流量上测延迟和质量，质量掉了就升到 `medium`。如果之后首 token 时间仍然要紧，官方说 system prompt 里加一句 `Answer directly without deliberating.` 能进一步减少思考——加了之后要测质量，因为思考变少可能把质量带下来
- **删掉那些「用来替代思考」的指令。** 如果你的 prompt 当初要求模型把推理写进回复里当作思考的替代品，把那条指令删掉，改从 [summarized thinking](https://platform.claude.com/docs/en/build-with-claude/thinking#summarized-thinking) 块读（`display: "summarized"`）；否则这种 prompt 可能被 `reasoning_extraction` 拒绝
- **重新检验关闭 thinking 时的那套缓解措施**，并且**无论如何都把「不要思考」那条规则删掉**（SKILL.md 3.10）
- **按块类型而不是按位置读响应**——回复可能以 `thinking` 块开头，也可能不

**Opus 5.5 页面写明的 harness 层规则**（仅 API；见 `agents.md`）：对话历史只追加不修改，因为改动 thinking 块之前的任何内容都会让它失效；会话中途改 effort 用按消息设置而不是改顶层值；强制 `tool_choice` 返回 400（SKILL.md 3.12）；在 Claude API 和 Google Cloud 上 computer use 必须用 `computer_toolset_20260801` 工具集——旧的 `computer_20251124` 返回 400，但在 Amazon Bedrock 上仍然可用。

### 只在 Opus 5 页面写了、Opus 5.5 页面没有重述的

下表每条处方都是 Opus 5 页面的官方原文，Opus 5.5 页面写明这些做法仍是合理起点；但*倾向本身*在 Opus 5.5 上是否仍然存在，官方两边都没有说。把它们当作在 Opus 5 上测得的，在自己的评测上复核。其中两条与 Opus 5.5 *有明确记载*的行为方向相反，已在行内标出。

| 已知倾向（Opus 5） | 处方 |
|---|---|
| 用户可见回复比更早的 Opus 模型长；**降 effort 不能可靠缩短可见输出** | 显式写简洁指令（4.2）。注意这里有张力：官方说 Opus 5.5 `tends to finish the same task with fewer tokens`，所以先实测，别默认这个长度问题还在 |
| Agent 场景播报频繁、单条消息长 | 显式描述播报节奏（4.3）。在 Opus 5.5 上有记载的问题**正好相反**——播报以默认为空的 `thinking` 块返回，所以回合会显得安静。加任何东西之前先查 `thinking.display` |
| 写到磁盘的文档（报告、Markdown、总结）偏长 | 长度校准指令（4.4） |
| **自我验证充分** | **删除**所有验证类指令——见 SKILL.md 3.4，那里说明了为什么这是 Opus 5 的规则而不是 Opus 5.5 的规则。最佳实践页把它写成「让 Claude 自检」这条通用建议的例外，并且只点名了 Opus 5 |
| 自行扩大任务范围，加上没被要求的步骤 | 范围约束（4.5） |
| 比更早的模型更积极派生子智能体 | 给明确判据或确定性上限（4.10）。在 Claude Code 和 Agent SDK 里，上限是 `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH`、`CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS` 和 SDK 的 `max_budget_usd`（需要 Claude Code 2.1.217 及以上）；Claude Code 只在用它的 `claude_code` system prompt 预设时才会在 Opus 5 上自动加一条委派指令，自定义或不传 system prompt 时要自己加 |
| 更频繁地口头纠正自己此前的说法 | `Only correct an earlier statement when the error would change the user's code, conclusions, or decisions. State corrections plainly and briefly, then continue the task. For slips that change nothing for the user, make the fix and move on without noting it.` |
| 代码审查中"只报高危"会被**字面执行** | 用 4.16；把过滤动作移到独立的下游步骤。官方说 Opus 5.5 的代码审查*更好*——真 bug 更多、误报更少——但「字面执行指令」这个倾向既没被重述也没被撤回，所以这个做法仍然适用 |
| `low` / `medium` 质量强，而 token 与延迟只是高档位的一个零头 | 在 Opus 5.5 上已被上面它自己的 effort 段落取代：默认档本来就是 `medium`，下调目标是 `low` |

**只在 Opus 5 上才可能出现**（Opus 5 在 `high` 及以下 effort 可以关闭 thinking；Opus 5.5 根本不能关，所以这两种现象都不会发生）：关闭 thinking 时 Opus 5 可能把 tool call 写成用户可见的文本而不是发出 `tool_use` 块——该调用不会执行，而且在 agent 循环里泄漏的文本会留在历史里污染后续轮次；也可能把 `<thinking>` 等内部 XML 标签泄漏到可见输出。官方给的是一条合并指令：

```
When you use a tool, you may say a brief sentence first. If no tool can express what the user asked for, say so instead of guessing. Do not include internal or system XML tags in your response.
```

官方说法：多数任务上，开着 thinking 跑 `low` effort，效果好于在相近成本下关掉 thinking——而在 Opus 5.5 上，开着本来也是唯一选项。

## Claude Fable 5.1 / Mythos 5.1（默认 effort: high；thinking 始终开启且不可配置）

thinking 始终开启，adaptive thinking 是唯一模式。Claude Mythos 5.1 是同一个模型，只面向 Project Glasswing 参与者提供，规格、价格和 prompting 指引相同；官方写明的差别里，唯一影响 prompt 或 harness 代码的是 Mythos 5.1 不执行 thinking 块的「历史只追加」检查（另有可用性上的差别）。2026 年 9 月 1 日发布；Fable 5 现已列为仍可用的旧模型（legacy）。在 Claude Code 里，`claude-fable-5-1` 从 2.1.257（2026-09-01）起是默认的 Fable 模型。

定位：官方称其为 **Anthropic 目前最强的公开发布模型**（原文 `Anthropic's most capable widely released model`）。官方选型建议是另一半：**多数负载从 Claude Opus 5.5 起步；需要高强度推理和长周期 agentic 工作、或者 Opus 5.5 在更高 effort 下的评测仍不达标时，用 Claude Fable 5.1。** 官方选型矩阵列出的用例是：连续跑数小时的 agent 会话、多步深度研究、从分析一路做到成品文档 / 表格 / 幻灯片。「最强」和「默认该选哪个」不是同一个问题。它相对 Fable 5 的提升在高 effort 下最大，集中在长会话 agentic 编码、文档 / 表格 / 幻灯片工作、多步研究、视觉、长上下文和 computer use。

**Fable 5 的指引怎么沿用。** Fable 5.1 的 prompting 专页开头原文：`Your existing Claude Fable 5 prompts should perform well on Claude Fable 5.1 without changes, but a handful of behavioral differences are worth knowing about.` 配合上面引过的同一条最佳实践提醒，本文件用同样的方式把 Fable 的材料分成两张表。

### Fable 5.1 页面明确写了的

| 已知倾向 | 处方 |
|---|---|
| **长时间工具调用回合里给用户的播报比 Fable 5 更少**，effort 越高、工具链越长越明显：agent 一连几分钟没动静，或者最后一条消息只讲了最后一步 | 按顺序做：先确认播报到底有没有送到客户端（它们以进度更新型 `thinking` 块返回，在默认的 `thinking.display: "omitted"` 下是空的——设成 `"updates"`，beta）；删掉压制播报的 prompt 句子（"hold all findings for the final response"）；然后才加 4.3 的 Fable 5.1 块。官方给它的相对延迟标注是 "Slower"，超时和进度提示照 Fable 5 那样调整 |
| 在复杂的异步任务上**只描述下一步而不去做**（"Next, I'll …"），或者对请求已经覆盖的步骤停下来请示——用户不得不回 "continue" | 4.18，两段一起用；只用第一段也能保住大部分效果。开头那句承担了大部分效果，照原样保留 |
| 交付所要求的**有时还多给**：顺手修旁边的代码、扩展任务没提的行为、提交超出改动所需的测试文件 | 4.6 的 Fable 5.1 块。官方测得：未被要求的添加和提交的测试代码大幅减少，任务成功率没有可测的变化 |
| 在编码和 computer use 循环里，下一批独立调用只是隐含时，可能**一轮只发一个工具调用**（Fable 5 会打包发）——多花 token 和往返，不影响质量 | 4.9 里那一句提示，每轮以 turn-scoped system message（beta）重发，或放在 `tool_result` 块之后 |
| 有些情况下**文风比 Fable 5 更密**：句子更长、段落更少 | 4.26。管长度的 4.2–4.4 管不到密度 |
| 聊天里**用的格式比旧模型少**——加粗更少、标题和列表更少——所以为旧模型写的反格式化规则会压掉内容需要的结构 | 删掉那些规则，或换成 4.13 里说明「什么时候该用格式」的 Fable 5.1 块 |
| 做文档摘要时更容易**照搬原文而不标为引用** | 4.30——在 system prompt 里放一个完整的正确示例 |
| `low` effort 下**凭记忆作答**而不去调搜索 / 检索工具 | 给受影响的那几轮提高 effort（按消息设 effort，beta），或用 4.27 |
| 小改动也更倾向于**整个文件重写**——结果一样，输出 token 和时间更多 | 4.28 |
| `xhigh`/`max` 下可能**先在思考里把长交付物起草一遍，再作为回复写一遍** | 这类请求跑在 `high`；确实要更高时，`max_tokens` 给思考加回复留空间，并追加 4.29（仅 API） |
| 让它复述内部推理会触发 `reasoning_extraction` 拒绝——thinking 页现在把 Fable 5.1、Opus 5.5 和 Fable 5 并列写明 | 见 SKILL.md 3.9 |
| 安全分类器覆盖的五个类别与 Fable 5 相同（`cyber`、`bio`、`frontier_llm`、`reasoning_extraction`、`general_harms`）。Fable 5.1 迁移指南把这描述为「比 Claude Opus 5 只有网络安全一类的分类器更宽」——注意 **Opus 5.5 此后已经补上了 `bio` 和 `reasoning_extraction`**，所以两个模型现在的差别是 `frontier_llm` 和 `general_harms` 这两类，而不是四类。良性工作也可能触发；在源码里找漏洞是允许的；误报比 Fable 5 发布时少 | 处理 `stop_reason: "refusal"` 并配置回落。减少误报：问 "Are there any bugs in this program?" 而不是 "Does this program compile without errors?"；给小众语言补上下文；别让 base64 数据进工具输出 |
| 强制工具调用（`tool_choice` 的 `any` / `tool`）返回 400 | 见 SKILL.md 3.12——现在 Opus 5.5 上同样如此 |
| **子智能体在跑时主 agent 闲着**（官方症状原句 "Lead agent idles while subagents run"） | 让它继续干活：启动子智能体的工具立即返回，结果之后以 `user` 消息回传，另给一个「想等的时候用来等」的工具。官方测得平均完成时间下降，质量、token 用量和成本相近；模型仍然经常选择等 |
| 密集图表和图像 | 给它裁剪放大工具（或一个装有原图和 PIL / OpenCV 的容器），让它可以反复分析、裁剪、核对——单独一个裁剪工具就能拿到大部分提升 |

Fable 5.1 页面写明的 harness 层规则（仅 API；见 `agents.md`）：对话历史只追加不修改，因为改动历史轮次会让它的 thinking 块失效（Mythos 5.1 不执行这项检查）；会话中途改 effort 用按消息设置而不是改顶层值；即使在 Fable 5 上做过 effort 扫描，也要重新做一遍。

### 只在 Fable 5 页面写了、Fable 5.1 页面没有重述的

下表每条处方都是 Fable 5 页面的官方原文，Fable 5.1 页面写明这些 prompt 可以沿用；但*倾向本身*在 Fable 5.1 上是否仍然存在，官方两边都没有说。把它们当作在 Fable 5 上测得的，在自己的评测上复核。

| 已知倾向（Fable 5） | 处方 |
|---|---|
| **单次请求可能跑很多分钟，自主运行可达数小时** | 迁移前先调整客户端超时、流式和进度提示；考虑改成异步轮询而非阻塞等待。Fable 5.1 页面从侧面印证了这个量级（"quiet for minutes at a time"；相对延迟 "Slower"），但没有重述这句话 |
| 任务模糊时会过度规划 | `When you have enough information to act, act. Do not re-derive facts already established in the conversation, re-litigate a decision the user has already made, or narrate options you will not pursue in user-facing messages. If you are weighing a choice, give a recommendation, not an exhaustive survey. This does not apply to thinking blocks.` |
| 高 effort 下会做未被要求的整理 / 重构 | 用 4.6 的 **Fable 5 变体**（同一条目里的 Fable 5.1 块是相邻倾向——擅自修 bug、多交测试文件——在 5.1 上测得的处方） |
| **指令遵循能力强到一句话就能管住一类行为**，不必逐条枚举 | 用简短指令代替长清单（SKILL.md 3.3） |
| 长自主运行中可能编造进度 | 4.17——「几乎消除了编造的进度汇报」是在 Fable 5 上测得的 |
| 会做未被要求的动作（自动起草邮件、建防御性 git 分支） | 4.24 独立版；在 Fable 5.1 上它的前后两半已并入 4.18 |
| 更主动派并行子智能体 | **鼓励使用**，但给明确的委派判据；优先异步通信而非阻塞等待；长生命周期子智能体能靠缓存读取省时省钱（Fable 5.1 页面关于这件事的 harness 建议在上一张表） |
| 看到剩余 token 倒计时会提议开新会话 | 4.19；更好的做法是别把倒计时暴露给模型（4.18 的 Fable 5.1 自主运行块另加了一句 `Do not stop because the context or session is long.`） |
| **为旧模型写的 skills 往往过于规定化，可能降低输出质量** | 审查并删减；Fable 5 也能根据任务自行更新 skill |

**脚手架建议（Fable 5 页面，Fable 5.1 页面未重述）**：从你难度范围的**上限**开始试（挑一个比你会派给旧模型更难的任务），让它先界定范围、提澄清问题、再执行。

## ⚠️ 验证指令：Opus 5.5 与 Fable 5.1 方向相反

这是最容易搞错的一处，两条官方指引看似矛盾，实则场景不同：

| 场景 | 做法 | 依据 |
|---|---|---|
| **Opus 5，常规任务** | **删除**所有验证/复查指令 | 它自己会验证；保留会造成过度验证。这条只对 Opus 5 写明，没有写给任何其他模型 |
| **Opus 5.5，常规任务** | **不要条件反射式地加；继承来的要先实测再删** | Opus 5.5 的页面既没有重述过度验证这个结论，也没有撤回它；而最佳实践页的通用建议仍然偏向让模型自检。见 SKILL.md 3.4 |
| **Fable 5.1，长时自主运行** | **显式写入**周期性自我检查 | 官方原文发布在 Fable 5 页面，Fable 5.1 页面既没有重述也没有撤回（它写明 Fable 5 prompt 可以原样沿用）：`Establish a method for checking your own work at an interval of [X] as you build. Run this every [X interval], verifying your work with subagents against the specification.` |
| **Opus 5.5，无人值守的长跑** | **在模型之外加一个完成度检查** | Opus 5.5 页面自己的建议：把任务的各部分放进一份由模型更新的清单，或者 `have a separate, smaller model check the conversation against it at each end of turn`。这是外部检查，不是自我复查——正是 3.4 区分的那条界线 |

关键区别：官方（Fable 5 页面同一段）同时指出，**独立的、干净上下文的 verifier 子智能体，效果优于模型自我批评**。所以"删验证"删的是**让模型自己再看一遍**，"加验证"加的是**换一个上下文来查**。两者不冲突。

（注意这与 4.10「不要用子智能体验证自己的工作」也不冲突。4.10 要按同一条「上下文是否干净」的轴来读：它禁的是**把自我复查外包出去**——把模型自己的产出连同它自己的判断标准一起交给子智能体，等于为 Opus 5 本来就会做的判断付两遍钱。**换到干净上下文、带独立判据的 verifier**，在所有适用模型上都是允许的。Fable 系列的长跑加的不是许可，而是**周期性**：把这个检查按间隔反复跑，而不是只在最后跑一次。）

## effort 取值参考

`max` > `xhigh` > `high` > `medium` > `low`

**默认值是按模型定的：Opus 5.5 是 `medium`，其他支持 effort 的模型都是 `high`。** 这是本页最要紧的一件事——同一个不传 `effort` 的请求，在 Opus 5.5 上比在 Opus 5 上低一档。

- **Opus 5.5**：从默认的 `medium` 起步。只有在自己的 evals 上**实测**到质量提升才上 `xhigh` 或 `max`——官方原话是 `Reserve xhigh and max for work where you've measured a quality gain`。评测显示质量不掉的地方降到 `low`；若干编码评测上 `low` 已经接近 `medium`，成本低得多。官方明确写了 **Opus 5.5 在 `medium` 达到或超过 Opus 5 在 `high`**，所以旧的 `high` 设置对应的是新的默认档，而不是 `high`
- **Fable 5.1**：从默认的 `high` 起步。最吃能力的 agentic 与编码工作上 `xhigh` 或 `max`；例行或延迟敏感的工作，在评测显示质量不掉之后降到 `medium`/`low`。官方说它相对 Fable 5 的提升在各档 effort 上都有、高档位最大；`medium` 下大致追平 Fable 5 且成本更低，`low` 下在单任务成本上常能和 Opus、Sonnet 系列竞争而分数更高——凡是你原本会「拿小模型跑高 effort」的地方，都把它纳入比较

官方把 `xhigh` 定位为面向长周期工作的扩展能力，适用范围写的是**超过 30 分钟、token 预算以百万计**的长周期 agentic 与编码任务——这就是「足够重」的具体标准。`low` 被定位为最省的一档，用于需要最快速度与最低成本的简单任务，官方点名的例子是子智能体。

**如果 effort 设置是从别的模型沿用过来的，要在自己的评测上重新做一次 effort 扫描，而不是直接照搬。** Opus 5.5 上写得最重：`test several levels against your own evals rather than carrying over the setting you used on Claude Opus 5`；Fable 5.1 专页则写明即使在 Fable 5 上做过扫描也要重做，因为各档 effort 的名字在不同模型上对应的思考量并不相同。

官方给出的两条先后顺序：

- **调 effort 往往比换模型更有效**（官方原文 `Tuning effort is often a better lever than switching models`）：先在手上这个模型上把档位跑一遍，再考虑换模型
- 如果模型思考的频率超出任务需要，**先降 effort 档位，再考虑用 prompt 去引导**。prompt 措辞是第二顺位的杠杆，不是第一顺位，而且效果对具体措辞很敏感

要点：
- **把 `effort` 设成该模型的默认值，与完全不传该参数行为完全相同**——Opus 5.5 是 `"medium"`，其他模型是 `"high"`
- effort 是**行为信号而非硬预算**，影响正文、工具调用和思考的全部 token
- **低 effort 会直接减少工具调用次数**、合并操作、省去前言；高 effort 则更多工具调用、先讲计划、给详细总结
- 同一档位下 **Opus 5.5 每回合思考得比 Opus 5 多**，`xhigh` 和 `max` 尤其明显
- 在请求之间改**顶层** effort 会让 prompt cache 失效。Opus 5.5、Fable 5.1、Mythos 5.1 和 Opus 5 支持**按消息改 effort**（beta，header `mid-conversation-output-config-2026-07-01`：一条 `content` 为空、带 `output_config.effort` 的 `role: "system"` 消息），从下一条 user 消息起生效且缓存不失效。在 Fable 5.1 上官方还明确更推荐这种形式，因为改顶层值对模型的引导不够可靠——它之前的回复是在旧档位下写的，会倾向于保持一致。Fable 5 不支持（返回 400）
- 要给 `max_tokens` 留足空间——它是思考加正文的**总**上限，而且即使思考文本不返回给你，思考也计入这个上限。Opus 5.5 上官方原话是面对长 agentic 编码回合 `a max_tokens of 128,000, the model's maximum, has worked well`；Fable 5.1 的页面则从 `high` 起就点了这件事。此项仅在直接调 API 时需要关心
- 在 Claude Code 里 `/effort` 是按模型记的，Opus 5.5 这类新发布模型在用户手动选之前走自己的默认档（2.1.280）
