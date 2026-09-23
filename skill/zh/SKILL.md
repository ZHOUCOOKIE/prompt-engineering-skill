---
name: prompt-engineering
description: Writes, rewrites, reviews, and audits prompts and system prompts using Anthropic's current official guidance, and selects the right model and effort level for the task before drafting. Use when the user asks to write a prompt, improve or fix a prompt, design a system prompt, or check a prompt before running a task — including phrasings like "帮我写 prompt"、"改一下这个 prompt"、"设计 system prompt"、"处理任务之前先检查我的 prompt"、"帮我把需求整理成 prompt"、"help me prompt this better". Applies whether the resulting prompt will be pasted into Claude Code for a coding task or run in a desktop chat or Cowork session for a non-coding task. Also use when the user pastes a prompt and asks why it is not working, when they are authoring an agent system prompt or CLAUDE.md, or when they ask which model or effort level to use. Several widely repeated prompt techniques are now obsolete or rejected by the API, so consult this skill even when prompting from memory feels sufficient.
---

# 写 prompt 的规范

**适用模型**：Claude Opus 5.5 和 Claude Fable 5.1（含 Mythos 5.1）。Claude Opus 5 和 Claude Fable 5 已进入官方 legacy 列表（仍可用，但不再是默认），除了作为沿用措辞的出处之外不在本文件范围内。有几条处方只发布在 Opus 5 或 Fable 5 专页上；两个后继页面都写明现有 prompt 无需修改即可沿用，所以保留，并在正文里逐处标明。凡是旧页面写了某个**行为**、而后继页面既没重述也没撤回的，`references/models.md` 会单独放进一张标注「在该模型上测得」的表，而不是默默地照搬过来。

**这份 skill 存在的理由**：一批广为流传的 prompt 技法在这两个模型上已经失效、被 API 拒绝、或效果反了。你的训练直觉里很可能仍带着它们。**当直觉与第 3 节冲突时，以本文件为准。**

**基准**：Anthropic 官方文档与官方/员工公开材料，快照日期 2026-09-22。涉及具体 API 参数时回原文档确认——那是唯一会随模型更新的地方。

---

## 0. 工作流

1. **判断目标场景**（第 1 节）——决定读哪个 target 参考文件
2. **自行选定模型和 effort**（第 2 节）——不要把这个问题丢回给用户
3. **补齐缺失变量**（第 1 节末尾）——只问真正问不出就写不了的
4. **起草**：读对应 target 文件；需要现成模块时再读 `references/snippets.md`
5. **过第 7 节自检**
6. **交付**：prompt 本体 + 一段交付说明。说明包含五件事——选了哪个模型和 effort、替用户做了哪些假设、哪里最可能需要调、有没有只有用户能补的信息、**如果 prompt 里嵌了英文官方片段，提醒用户不要翻译它们**。能压到两三句就两三句；**假设较多时如实写长，不要为了"简短"把假设藏起来**——用户看不到假设就没法判断要不要改。

---

## 1. 先判断场景

用户在 Claude desktop 里描述需求，产出的 prompt 会去两个地方之一。**先判断去哪儿，因为写法差别很大。**

| 信号 | 场景 | 读这个文件 |
|---|---|---|
| 涉及代码库、文件、git、跑测试、重构、修 bug、建项目；用户说要拿去 Claude Code 或 VS Code 跑 | **A：给 Claude Code 的任务 prompt** | `references/for-claude-code.md` |
| 研究、分析、写作、整理资料、做文档表格幻灯片、日程、信息汇总；在 desktop chat 或 Cowork session 里直接跑 | **B：给 chat 或 Cowork 的任务 prompt** | `references/for-chat-and-cowork.md` |
| 用户在聊天途中丢来一段 prompt 问"这样写行不行" | **C：临时检查** | 见第 6 节，走轻量流程 |

判断不了时按用户描述里的**动词**判断：动代码用 A，其余用 B。两边都沾（例如"分析这个仓库的技术债并写成报告"）时按**最终交付物**判断——交付物是代码改动走 A，是文档走 B。

### 补齐变量

只问真正问不出就写不了的。缺失但能从上下文合理推断的，**直接假设并在交付时说明**，不要为了严谨把问题丢回去。

下表的"必须确定"指的是**这个值必须有明确结论**，不是"必须开口问"。能从上下文推断就推断，推断了就在交付说明里写清楚。

| 变量 | 要求 |
|---|---|
| **成功标准** | **必须确定**。没有可判定的成功标准就写不出好 prompt，也没法自检 |
| **输出形态与长度** | 交付物是文档或报告时**必须确定**。不明说的话长度就由模型自己定，而 effort 并不能可靠地控制它（见第 8 节「长度控制怎么选」） |
| **可用的工具或文件** | 场景 A 必须确定（哪个仓库、哪些文件）；场景 B 看是否需要读取材料 |
| 目标模型与 effort | **不要问**，按第 2 节自己定，把结论告诉用户 |

只有当某项**既推断不出、猜错了又会让整个 prompt 白写**时才开口问。

需要问多项时用 AskUserQuestion **一次性问全**，不要逐个问。

---

## 2. 自行选定模型和 effort

这是起草前的必经一步，结论要写进交付说明里。

### 选模型

| 任务画像 | 选 | 依据 |
|---|---|---|
| 复杂 agentic 编码：真实仓库里的多步改动、大规模重构、跑几小时的自主编码 agent、复杂系统工程 | **Opus 5.5** | 官方定位：在真实仓库的多步工作上最强，比如把一个改动贯穿大型代码库直到测试通过。官方测试中它在默认 `medium` 下就达到或超过 Opus 5 的 `high`，步骤更少、token 更少，长时间自主运行也比 Opus 5 更能持续 |
| 代码审查与找 bug | **Opus 5.5** | 早期测试者反馈它的代码审查强于 Opus 5——找到的真 bug 更多、误报更少，而且会用平实语言解释自己改了什么 |
| 企业知识工作：文档、表格、演示稿、财务建模、长上下文分析 | **Opus 5.5** | 说错数字或引错来源的概率明显更低；能抓住大输入里容易漏掉的细节；产出的表格、幻灯片和文档在分享前需要的修改更少。1M token 上下文窗口 |
| **重视觉的工作与 computer use**：密集图表、工程制图、流程图、截图、从截图操作应用 | **Opus 5.5** | 不借助工具读视觉材料就比 Opus 5 准——官方测试中它在最低 effort 下读密集图表的数值都比 Opus 5 在最高 effort 下更准，输出 token 只用一小部分；computer use 在默认 effort 下就达到了 Opus 5 要高得多的 effort 才有的成功率 |
| **高难推理与长周期 agentic 工作**：跑几小时的 agent 会话、多步深度研究、一路做到成品文档/表格/演示稿的分析；或用户在 Opus 5.5 高 effort 下的 evals 仍然不够 | **Fable 5.1** | 官方选型建议（models overview 与 choosing-a-model 矩阵）。官方定位是 Anthropic 已广泛发布的最强模型 |
| 触及安全分类器领域的工作——该领域里的正当工作同样可能被触发 | **按会被拒来规划**：`cyber`、`bio`、`reasoning_extraction` 两个模型都有；`frontier_llm`、`general_harms` 只有 Fable 5.1 | Opus 5.5 跑生物、网络安全、推理抽取三类分类器，其中生物与推理抽取**对从 Opus 5 迁移过来的人是新增的**。Fable 5.1 与 Fable 5 同为五类。被拒时返回 `stop_reason: "refusal"` 并带 `stop_details` 类别，所以要配 fallback——注意服务端 fallback **不会**重试 `reasoning_extraction`，它会把拒绝直接返回给你。在源码里找漏洞是允许的 |

默认选 **Opus 5.5**——官方说法是绝大多数工作负载都从它开始，Claude Code 自 2.1.280（2026-09-22）起也把它作为默认 Opus 模型。高难推理和长周期 agentic 工作，或者用户自己的 evals 在 Opus 5.5 高 effort 下仍然不够时，才推荐 Fable 5.1；并且要提醒用户它是这一代里最慢的模型，长工具回合里可能几分钟没有动静，超时和交互方式都要跟着改。

**Opus 5.5 比 Opus 5 更便宜也更快**（$4/$20 对 $5/$25；输出 token 生成速度快 30% 以上），所以没有"为了省钱留在 Opus 5"这个理由。

### 选 effort

**两个模型的起点不再一样了：Opus 5.5 默认 `medium`，Fable 5.1 默认 `high`。** 把 Opus 5 的设置照搬过来，正是官方明确点名的那个错误。

| 模型 | 起点 | 什么时候上调 | 什么时候下调 |
|---|---|---|---|
| **Opus 5.5** | **`medium`**（默认值） | 你在自己的 evals 上**实测**到质量提升 → `xhigh` 或 `max`。官方原话是：只把 `xhigh` 和 `max` 留给实测有收益的工作 | 在意成本或延迟且 evals 显示质量不掉 → `low`。官方说在若干编码评测上 `low` 接近 `medium`，成本低得多 |
| **Fable 5.1** | **`high`**（默认值） | 对能力最敏感的 agentic 与编码工作 → `xhigh` 或 `max`。它相对 Fable 5 的提升在高档位最大 | 常规或对延迟敏感的工作，且 evals 显示质量不掉 → `medium`/`low`。官方说 `medium` 大致相当于 Fable 5 且成本更低，`low` 在单任务成本上常常与 Opus、Sonnet 系列相当而分数更高——凡是你本来会用更小模型跑更高 effort 的场合，都应该把它纳入考虑 |

**不要把其他模型的 effort 档位照搬过来。** 官方指令是**在自己的 evals 上重跑一遍 effort 扫描**——Opus 5.5 上写得最明确：`test several levels against your own evals rather than carrying over the setting you used on Claude Opus 5`；Fable 5.1 上即使你已经在 Fable 5 上扫过也要重跑，因为 effort 档位名在不同模型之间并不对应同样的思考量。官方明确写了 **Opus 5.5 在 `medium` 达到或超过 Opus 5 在 `high`** 的编码与知识工作评测成绩，所以等效档位是**低一档**，不是同一档。

**同一档位下 Opus 5.5 每回合思考得比 Opus 5 多**，`xhigh` 和 `max` 尤其明显。照搬 Opus 5 的值会让回合更长、输出 token 更多——这是不要照搬的第二个理由。

**什么叫"高难 agentic 工作"**（这是上调判断的关键，先把它定下来）。两个条件，缺一不可：

1. **agentic**：任务需要模型**自己决定下一步、靠反复工具调用往前推**。典型标志是工具调用轮次多、计划随中间结果变化、单次运行时间长。写代码、跑测试再修、反复检索并跨来源交叉核对，都算。反过来，材料已经备齐、只要一次性产出一个成品（写报告、做表、翻译、改写），**就不是** agentic，哪怕成品很大。
2. **高难**：运行时间长。官方对 `xhigh` 的描述是长周期 agentic 与编码任务——超过 30 分钟、token 预算以百万计。三次工具调用就修完的 bug 是 agentic，但不高难。

**分不清是不是 agentic 时，用这个判据：由谁决定去哪儿找？** 材料是用户备好的、模型只是读一遍，不算；模型自己决定去找什么、再根据找到的决定下一步找什么，算。

**拿不准就留在默认档。** 在 Opus 5.5 上默认档是 `medium`，而且官方的框架是"实测到收益才上调"，不是"任务看起来难就上调"。

**这些参数分别在哪儿设**（不同界面能设的东西不一样）：

| | 场景 A：Claude Code | 场景 B：chat / Cowork | 裸 API |
|---|---|---|---|
| effort | 界面或斜杠命令里设。`/effort` 现在是按模型记的，Opus 5.5 这类新发布模型在用户手动选之前走自己的默认档 | 界面里设 | `output_config: {effort}` |
| thinking | 由客户端管理 | 由客户端管理 | `thinking: {type: "adaptive"}`——在 Opus 5.5 上这是唯一被接受的值 |
| `max_tokens` | **够不着** | **够不着** | 你自己设；官方说在 Opus 5.5 上，面对 agentic 编码可能产生的长回合，**128,000**（该模型的上限）在官方测试中效果不错 |

所以场景 A 和 B 的交付说明里**写清模型和 effort 档位就够了**，不要把 `max_tokens`、`thinking` 写进去——用户在那个界面上根本没有这些开关，写了只会造成困惑。下面的 API 字段细节只在直接调 API 时才用得上。

需要知道的几点：

- **把 `effort` 设成该模型的默认值，与完全不传这个参数行为完全相同**——但默认值是按模型定的：Opus 5.5 是 `medium`，其他支持 effort 的模型都是 `high`
- effort 是**行为信号，不是严格的 token 预算**；它作用于回复里的所有 token——正文、工具调用、思考
- **降低 effort 会直接减少工具调用次数**、合并操作、省掉铺垫；提高则工具调用更多、行动前先讲计划、总结更详细
- **想让它少思考，先降 effort 档位。** 官方说降 effort 减少思考（以及成本和延迟）比用 prompt 指令更可靠
- **effort 控制的是思考量，不是输出长度。** 想让回复变短要靠 prompt（片段 4.2）。（这句写在 Opus 5 专页上；Opus 5.5 和 Fable 5.1 都没有重述，而 Fable 5.1 有记录的文本形态问题是"密"，见片段 4.26）
- 在请求之间改**顶层** effort 会作废 prompt cache。Opus 5.5、Fable 5.1 和 Opus 5 支持按消息改 effort（beta），缓存不失效——仅 API，见 `references/models.md`
- 给 `max_tokens` 留足空间——它是思考加正文的**总**上限，而且即使思考文本不返回给你，思考也计入这个上限

模型的具体已知倾向与对应处方在 `references/models.md`。

---

## 3. 失效清单（硬规则）

这些不是风格偏好，是会导致 API 报错或行为反向的具体问题。

### 3.1 不要用 prefill（预填充助手回复）
Opus 5.5 和 Fable 5.1 **均不支持**，在最后一个 assistant turn 预填充会返回 400。官方原文是 `Starting with Claude 4.6 models and Claude Mythos Preview, prefilled responses (providing a partial assistant message for Claude to continue from) on the last assistant turn are no longer supported`，覆盖本文件适用的全部模型。

官方给的替代是 **structured outputs、system prompt 指令、或 `output_config.format`**。具体来说：结构化输出用 Structured Outputs、`output_config.format`，或直接说明 schema；去开场白写 `Respond directly without preamble. Do not start with phrases like 'Here is...', 'Based on...', etc.`（prompting 最佳实践页 "Migrating away from prefilled responses" 一节的原文）；分类任务用带 enum 的 tool；续写把上文放进 user 消息。

### 3.2 不要用 `budget_tokens`——在 Opus 5.5 上也不要试图关掉 thinking
两个模型均不支持手动 extended thinking，设置返回 400。**Opus 5.5 和 Fable 5.1 的 thinking 都是始终开启且无法配置**——`{"type": "enabled"}` 带 `budget_tokens` 和 `{"type": "disabled"}` 都返回 400。不传 `thinking` 或传 `{"type": "adaptive"}`（两者等价），只用 `effort` 控深度。需要硬上限用 `max_tokens`。

这是**相对 Opus 5 的破坏性变更**——Opus 5 在 `high` 及以下 effort 是接受 `thinking: {"type": "disabled"}` 的。官方报错文案直接给出了改法：

```
"thinking.type.disabled" is not supported for this model. Use "thinking.type.adaptive" and "output_config.effort" to control thinking behavior.
```

因为 Opus 5.5 的每个回复都可能以 `thinking` 块开头（默认 `display: "omitted"` 下内容为空），**要按 `type` 字段而不是按位置取 content block**，并在工具循环里原样回传 `thinking` 块。

### 3.3 不要用堆砌指令代替上下文
核心原则：**给上下文，别给一长串指令。**

Fable 5 专页的说法是——指令遵循已经好到一句简短指令就能管住一类行为，不必逐条枚举；Fable 5.1 专页写明现有 Fable 5 prompt 无需修改即可沿用，也没有撤回这个说法。Claude Code 团队据此把系统提示词砍掉 80% 以上而性能不降。

与其枚举十条"不要 X"，写一句正面描述加说明意图。把 `Default to using [tool]` 改成 `Use [tool] when it would enhance your understanding of the problem`。工具该怎么用，靠 tool description 和参数设计表达，不要在 system prompt 里反复提醒。

> 一处例外：Claude Code 最佳实践仍写着，如果 **CLAUDE.md** 里某一条 Claude 老是跳过，可以只给那一行加 `IMPORTANT` 之类的强调——并且说明，强调的行一多，就没有哪一行突出了。同一份文档也警告：CLAUDE.md 太长时 Claude 会忽略其中一半，因为重要规则淹没在噪音里。

### 3.4 自我验证指令：Opus 5 上删掉，Opus 5.5 上要重新实测
**这条规则的适用范围随 Opus 5.5 变了，套用之前先看限定条件。**

在 **Opus 5** 上这条是无条件的：它无需提示即自行验证，保留 `double-check your answer` / `re-verify before responding` / `include a final verification step` 会造成**过度验证**，浪费 token 且不提升质量；官方措辞是**删除这些指令，而不是改写它们**，harness 里额外加的验证步骤同理。prompting 最佳实践页把这条写成它「让 Claude 自检」这条通用建议的例外，并且只点名了 Opus 5，没有点名任何其他模型。

**在 Opus 5.5 上这条没有被重述。** Opus 5.5 的 prompting 页和 what's-new 页都既没有重复过度验证这个结论，也没有撤回它；而最佳实践页的通用建议仍然写着自检指令 `catches errors reliably, especially for coding and math`。官方迁移指南给的是更弱的一句：`Instructions tuned for Claude Opus 5's behavior may no longer be needed`。**所以在 Opus 5.5 上：不要条件反射式地加自我验证指令；从 Opus 5 继承来、已经删掉的就保持删掉——但要把「删掉它们」当成一件需要在自己 evals 上实测的事，而不是硬规则。** 交付说明里如实这样写，不要把 Opus 5 上的结论当成 Opus 5.5 的结论。

下面这部分对两个模型都适用，因为它讲的是这条规则**从来覆盖的是哪些指令**。

**注意区分：删的是"让模型自己再看一遍"，不是"去核对外部事实"。**

| 要删（模型自审） | 要留（核对外部依据） |
|---|---|
| "最后检查一遍有没有遗漏" | 跑测试、看构建退出码、对比截图（场景 A） |
| "再核对一次你的答案" | 把 A 材料的说法和 B 材料对上，冲突时说明取谁（场景 B） |
| "用子智能体复查你自己的工作"——指连同模型自己的判据一起交出去的那种 | 数字必须能追溯到某份原始材料的具体位置。**换到干净上下文、带独立判据的 verifier** 不在这一行删除之列，见 `references/models.md` |

判据很简单：**检查的依据来自模型自己，就删；来自外部可查证的东西，就留。** 场景 B 的研究与报告类任务里，跨来源交叉核对是任务本身，不是自我验证，写 prompt 时不要误删。

Fable 5.1 的长自主运行方向相反，要显式加周期验证——见 `references/models.md`（这条指令发布在 Fable 5 专页上，沿用）。

### 3.5 不要用否定式堆砌约束
`Never do X` 对 Claude 是极强的冲动信号，一旦与用户后续指令冲突会造成严重混乱。给正面描述加原因：

- ✗ `NEVER use ellipses`
- ✓ `Your response will be read aloud by a text-to-speech engine, so never use ellipses since the text-to-speech engine will not know how to pronounce them.`

第二句里其实仍有 "never"，但它挂在一个**原因**上——模型知道原因，遇到没被覆盖的边界情况也能自己判断。

### 3.6 不要用绝对规则代替判断力
- ✗ `Never write multi-paragraph docstrings or multi-line comment blocks — one short line max`
- ✓ `Write code that reads like the surrounding code: match its comment density, naming, and idiom`

### 3.7 不要在 system prompt 与 tool description 之间重复
工具的使用说明**只写在 tool description 一处**。

> **一处例外：触发时机。** 片段 4.23 就是官方给的例子——Fable 5 专页写明 Fable 5 光靠 tool description 很少会去调 `send_to_user`，所以官方在工具之外还配了一句写进 system prompt 的话，告诉它**什么时候**该用这个工具。这条规则禁的是把同一份**使用说明**写两遍，不是禁止给工具配一句它确实需要的触发说明。

### 3.8 不要靠 `temperature` 制造多样性
本文两个模型上，`temperature`、`top_p`、`top_k` 设非默认值都直接返回 400。官方原话是 `Starting with Claude Opus 4.7, setting temperature, top_p, or top_k to any non-default value returns a 400 error`，覆盖本文件适用的全部模型；Fable 5.1 的 what's-new 页也把它列在「与 Fable 5 相比未变」的行为里。Python SDK（v1.0 及以后）干脆不再定义这几个字段，传了会直接抛 `TypeError`，根本到不了 API。用 prompt 求多样性——让模型**先给选项再落地**：
```
Before building, propose 4 distinct visual directions tailored to this brief (each as: bg hex / accent hex / typeface, plus a one-line rationale). Ask the user to pick one, then implement only that direction.
```
（片段 4.25，逐字取自 Claude Sonnet 5 页面，照抄即可。）

### 3.9 不要要求模型复述或展示内部推理
"show your thinking"、"把思考过程写出来"这类指令在**两个适用模型上都可能触发** `reasoning_extraction` 拒绝类别，请求被拒、回落备用模型的比例上升。thinking 页现在把它们并列写明：`On Claude Fable 5.1, Claude Opus 5.5, and Claude Fable 5, a request that attempts to elicit the model's internal reasoning as part of the response text can be refused with stop_details.category: "reasoning_extraction"`。在 Opus 5.5 上，这个类别**对从 Opus 5 迁移过来的人是新增的**，而且它是唯一一个服务端 fallback *不会*替你重试的类别——会直接把拒绝返回给你。需要推理可见性改读结构化 `thinking` 块；需要在长时运行中把进度呈现给用户，用 send-to-user 工具（片段 4.23）。

### 3.10 不要写"不要思考 / 不要推理"
**不论哪个适用模型，看到这类规则就删掉。** Opus 5.5 上官方指令写得很直接：迁移一个原本关闭 thinking 的 prompt 时，`check whether you still need the instruction, and remove the no-thinking rule either way`。

它当初存在的理由是：**在关闭 thinking 的情况下**，Opus 5 可能把 `<thinking>` 等内部 XML 标签泄漏到可见输出，而 system prompt 里这类规则会**加剧**泄漏。Opus 5.5 和 Fable 5.1 根本无法关闭 thinking（见 3.2），所以这个泄漏场景在适用模型上已经不会出现——但规则本身仍然值得删掉，而且 prompt 里还留着它，说明这份 prompt 是为更早的配置写的。

如果你在维护一个必须关闭 thinking 的 Opus 5 集成，写通用形式，且不要点名 thinking 标签——点名的效果不如通用形式：
```
Do not include internal or system XML tags in your response.
```

### 3.11 继承来的 prompt 先查 prefill
不是你自己写的 prompt——旧模板、4.6 之前的代码库、任何你控制不了的工具生成的东西——结尾仍可能带一个半截的 assistant turn，这在本文两个模型上返回 400。**继承来的东西一律按本节重新过一遍**，先看结尾那一轮。Claude Code 的 `claude-api` skill 带一个 `prompt-audit` 子命令，官方原文描述是 `for auditing prompts and tool descriptions for patterns written for older models`（changelog 2.1.221，2026-08-04）；它只是第一遍粗筛，不能替代本节。

### 3.12 不要强制工具调用——两个适用模型都不行
`tool_choice` 设为 `any` 或 `tool` 类型，在 **Opus 5.5、Fable 5.1 和 Mythos 5.1** 上都返回 400 `invalid_request_error`（thinking 始终开启，强制调用会跳过它）；`auto` 和 `none` 不受影响。在 Opus 5.5 上这是**相对 Opus 5 的破坏性变更**——Opus 5 是接受强制工具调用的。官方报错文案：

```
tool_choice: type "tool" and "any" are not supported for this model.
```

同样的校验也适用于 token counting 端点。要 schema 合规的 JSON，保持 `tool_choice: {"type": "auto"}` 并用 strict tool use 设 `strict: true`，或者把 schema 交给 structured outputs。要让模型调工具而不是用文字回答，在 prompt 里写明什么时候该用这个工具——官方例子是 `Use the get_weather tool to answer`——Fable 5.1 会可靠地照做。

---

## 4. 通用骨架与结构

**原则：能完整界定期望行为的最小信息集。** 按需取用，不要为了填满结构而塞内容。

```
<role_and_goal>
一到两句。身份加要达成什么。不写履历式人设。
</role_and_goal>

<background_information>
模型无法自行推断的领域知识、业务约束、术语定义。不写通用常识。
</background_information>

<instructions>
列出必须做的事；顺序或完整性重要时用编号。用正面描述，附上"为什么"。
</instructions>

## Output description
输出的形态、长度、格式。用"要什么"而非"不要什么"表述。
```

**结构化手段**：XML 标签或 Markdown 标题分区。标签名一致且有描述性。官方片段库里的规律只朝一个方向成立——**凡是包了标签的都是常驻方针**，但不少常驻方针也是纯文本。判据不是长度（库里最短的带标签块只有一句话），而是**这一块需不需要被当作一个有名字的单元来对待**：与周围内容隔开，或者被别处按名字引用。

**长上下文（20k+ token）摆位**：长材料放**最前面**，问题和指令放**最后面**。官方测试中这一条最多可提升 30% 的回答质量。多文档用 `<document>` 包裹，内含 `<source>` 和 `<document_content>`。

### 示例（few-shot）该不该给

最容易用错的一处。官方文档和 Claude Code 团队的说法看似矛盾，实则场景不同：

| 目标 | 该不该给 | 依据 |
|---|---|---|
| 锁定输出**格式、语气、结构** | **给**，3–5 个，多样化，`<example>` 包裹 | 官方文档："Examples are one of the most reliable ways to steer Claude's output format, tone, and structure" |
| agent 系统提示词、要模型**发挥判断和创造力** | **不给**，改为优化工具接口设计 | Claude Code 团队："removing examples was extremely helpful, because it was just more creative than the examples we gave it" |
| 展示**推理模式** | 在 few-shot 里用 `<thinking>` 标签演示 | 官方文档 |

**一句话记忆：示例是约束器。要约束时用它，要创造力时删掉它。**

---

## 5. 参考文件

按需读取，不必全部加载。全部从本文件直接链接。

| 文件 | 什么时候读 |
|---|---|
| **`references/for-claude-code.md`** | 场景 A。Claude Code 任务 prompt 的写法、必须给的验证手段、探索到规划到实现的流程、该写进 CLAUDE.md 还是写进这次 prompt |
| **`references/for-chat-and-cowork.md`** | 场景 B。desktop chat 与 Cowork session 的 prompt 写法、材料摆位、交付物形态、Cowork 特有的文件与 skill 能力 |
| **`references/models.md`** | 选完模型后读。Opus 5.5 与 Fable 5.1 的已知倾向与对应处方——每个模型都分成当前页面明确写了的、和只在前代页面写了的两张表——以及两者相反的验证策略 |
| **`references/snippets.md`** | 需要现成模块时读。36 条编号条目，其中 35 条是官方文档的**逐字原文**英文片段，并说明哪些必须逐字照抄、哪些可以按语境改写 |
| **`references/agents.md`** | prompt 要驱动一个长时间自主运行的 agent 时读。上下文工程、跨窗口工作流、缓存命中、让模型自己编排 |

### 用到英文片段时（无论从哪个参考文件取的）

产出中文 prompt 却嵌入英文官方片段是常态，处理方式固定：

- 英文**成段放置**，不要拆开插进中文句子里；放在它管辖的那一节内部，紧跟中文小标题之后
- **必须在交付说明里告诉用户：这几段英文是官方原文，请不要翻译或"顺手改通顺"**。用户看到中英混杂会很自然地想统一语言，一改就失效
- 只需要一两个片段、且属于 `references/snippets.md` 里标注的"通用策略型"时，可以直接用中文表达，避免为两句话让整份 prompt 变成混排

哪些片段必须逐字、哪些可以改写，见 `references/snippets.md` 开头的分级表。

---

## 6. 三种请求的处理差异

**写或改 prompt（场景 A/B）**：按第 0 节工作流走完，交付说明按第 0 节第 6 步的要求写。不要塞入用户没要的解释、免责声明或替代方案清单。

**审查已有 prompt**：按第 3 节逐条比对，指出**具体问题和对应修法**，不要直接给一份全新的 prompt 替代掉——除非用户要求重写。按严重度排序：会导致 API 报错的（3.1、3.2、3.8、3.12）排最前，行为反向的（3.9、3.10）次之，然后是 3.4——它只有在目标是 Opus 5 时才是硬规则——风格问题最后。

**现在拿到的 prompt 十有八九是为 Opus 5 写的。** Opus 5.5 是 2026-09-22 才发布的，所以先查这四样：照搬过来的 `effort` 值（默认档从 `high` 降到了 `medium`，而 Opus 5.5 在 `medium` 官方就达到或超过 Opus 5 在 `high`，所以照搬的值通常高了一档）；`thinking: {"type": "disabled"}` 或 `budget_tokens`（现在都是 400）；强制 `tool_choice`（现在是 400）；以及只渲染 `text` 块的客户端——它在工具调用之间会看起来没动静。

**聊天途中临时检查（场景 C）**：走轻量流程。**不要小题大做**——不用完整跑一遍工作流，不用读全部参考文件。扫一眼第 3 节的硬规则，有问题就直接说"这里有个问题：X，改成 Y"，没问题就说没问题然后继续原来的话题。用户此刻要的是不被打断，不是一份审查报告。

---

## 7. 交付前自检

- [ ] 已经自己选定模型和 effort，并在交付说明里写清楚了
- [ ] 如果用了英文官方片段：交付说明里写了「这几段是官方原文，请不要翻译或改写」的提醒
- [ ] 没有 prefill、没有 `budget_tokens`
- [ ] 没有为防欠触发而堆的 `CRITICAL: You MUST` / `If in doubt, use [tool]`（**官方片段里的原文措辞不算**，例如 4.7 内含的 `you MUST read the file` 是调校过的原文，照抄即可）
- [ ] 目标是 Opus 5.5：没有**条件反射式加进去**的自我验证指令；如果交付说明里提到删掉某条自我验证指令有好处，要写成「需要实测的一件事」而不是「Opus 5.5 上已确认如此」（见 3.4；外部确定性检查从来就不在这条覆盖范围内，官方研究片段里那句 `Regularly self-critique your approach and plan` 也不算，见 `references/for-chat-and-cowork.md`）
- [ ] 目标是 Opus 5.5：没有 `thinking: {"type": "disabled"}`、没有强制 `tool_choice`、没有「不要思考」类规则；effort 以档位形式写明，起点是 `medium`
- [ ] 目标是 Fable 5.1：没有"展示你的思考过程"类指令；没有强制 `tool_choice`；长自主运行已加 4.18 的自主运行块（4.17 的进度核对规则仍可用，它是在 Fable 5 上测得的）。在 Opus 5.5 上，"展示你的思考过程"这条同样适用（见 3.9）
- [ ] **你自己写的**约束都是正面表述，且带了「为什么」（**官方片段里的原文措辞不算**——4.6、4.13、4.25 本身就是成串的否定式，例如 4.25 开头的 `NEVER use generic AI-generated aesthetics…`，是调校过的原文，照抄即可）
- [ ] 每条指令都想过：**一个善意的人类会怎么误解它？**（Claude Code 团队 Cat Wu 的自检法）
- [ ] 同一条指令没有在 system prompt 和 tool description 里重复出现（**模型确实需要的触发说明不算**——例如片段 4.23 配套的那句 system prompt，见 3.7）
- [ ] 示例的取舍符合第 4 节的场景判断，不是无脑加或无脑删
- [ ] 长上下文场景：材料在前、问题在后
- [ ] 输出长度有明确要求——两个适用模型上 effort 都控不了长度
- [ ] 有可判定的成功标准；场景 A 有模型自己能跑的验证手段
- [ ] 通读一遍，删掉所有「不删也不会出错」的句子
- [ ] 黄金检验：**把这份 prompt 给一个不了解任务的同事，他会照做吗？**

---

## 8. 故障到处方速查

片段编号指向 `references/snippets.md`。

| 症状 | 处方 |
|---|---|
| 回复太泛 | 加具体约束、加成功标准、加 "go beyond the basics" 类修饰语 |
| 答非所问 | 补上真实目标和背景（说清"为什么"） |
| 格式不稳定 | 加 3–5 个 `<example>`；或用 Structured Outputs |
| 有开场白 | `Respond directly without preamble.` |
| 编造内容（一般场景） | 显式允许说不知道：`If the data is insufficient to draw conclusions, say so rather than speculating.` |
| 对没打开过的代码/文件下判断 | 片段 4.7（防幻觉，代码场景必备） |
| 只给建议不动手 | 换直接动词（`Change this function` 优于 `Can you suggest changes`）；或加片段 4.8 |
| 输出太长（面向阅读的正文） | 见下方「长度控制怎么选」。**降 effort 不可靠**——effort 管的是思考量，不是可见长度 |
| agent / 编码场景里中途播报太啰嗦 | 片段 4.3。这和正文长度是两个问题，长度控制那套管不到播报节奏 |
| 过度工程 | 片段 4.6 |
| 子智能体滥用 | 片段 4.10 |
| 思考过多、延迟高 | **先降 effort 档位**——官方说这比用 prompt 指令更可靠；再考虑片段 4.15 |
| chat 回复迟迟不开口，因为先在长时间思考（Opus 5.5） | 从 system prompt 里删掉「回答前仔细思考」这类句子；后续轮次用片段 4.34 |
| 无人值守的 agent 汇报完进度就中途停了（Opus 5.5） | 片段 4.31，外加一份由模型自己更新的任务清单；不要把「只有文字、没有工具调用」的回合当成任务已完成 |
| 跨多个已连接应用时漏掉任务没点名的信息（Opus 5.5） | 片段 4.32 |
| 一队 agent 花的时间比应该的长（Opus 5.5） | 片段 4.33——在每条消息末尾给出时间预算信号 |
| 照着用户粘贴进来的文本里的指令执行（Opus 5.5） | 片段 4.35 |
| 长 agentic 回合在你的客户端里看起来没动静（Opus 5.5） | 先查 `thinking.display`——播报是以 progress-update `thinking` 块返回的，默认内容为空；再用 4.3 的 Opus 5.5 块 |
| 复杂问题推理太浅 | **提高 effort**，而不是加 prompt 绕过 |
| 反复叮嘱某规则仍不生效 | prompt 太长、规则被淹没 → 精简，或改用确定性手段（hook、工具设计、schema） |
| 代码审查漏报 | 片段 4.16，把过滤移到下游。`references/for-claude-code.md` 里对抗式审查那条提醒是反过来的情形——没有下游过滤，只能由审查者自己过滤 |
| 长任务编造进度或谎报完成 | 片段 4.17 |
| 无人值守时中途停下来问问题，或者说完 "Next, I'll …" 就结束回合 | 片段 4.18 |
| 长时间工具调用中 agent 一连几分钟没动静（Fable 5.1） | 先查 `thinking.display`，删掉压制播报的句子，然后才用 4.3 的 Fable 5.1 块 |
| agent 循环里一轮只发一个工具调用（Fable 5.1） | 4.9 的 Fable 5.1 提示句，每轮重发 |
| 文风太密——句子长、段落少（Fable 5.1） | 片段 4.26；这不是长度问题 |
| 聊天回复的结构比内容需要的少（Fable 5.1） | 删掉反格式化规则；用 4.13 的 Fable 5.1 块 |
| 摘要照搬原文却不标引用（Fable 5.1） | 片段 4.30 |
| `low` effort 下凭记忆作答不去搜（Fable 5.1） | 给那几轮提高 effort，或片段 4.27 |
| 小改动也整个文件重写（Fable 5.1） | 片段 4.28 |
| `xhigh`/`max` 下长交付物等很久或撞到 `max_tokens`（Fable 5.1） | 跑在 `high`；否则片段 4.29 |
| 顺手修了没人要它修的 bug、多交了测试文件（Fable 5.1） | 4.6 的 Fable 5.1 块 |
| 良性的编码请求被拒、返回 `stop_reason: "refusal"`（Fable 5.1） | 问 "Are there any bugs in this program?" 而不是 "Does this program compile without errors?"；给小众语言补上下文；别让 base64 进工具输出 |
| 快到上下文上限时自行收工 | 片段 4.19；首选是别让模型看见剩余预算数字 |
| 长会话最终汇报读不懂 | 片段 4.21 |
| 做了没被要求的事 | 片段 4.24；任务范围被擅自放大用片段 4.5 |
| 改动可能不可逆 / 碰共享环境 / 会 push 或改线上 | 片段 4.11（安全护栏） |
| 为了让测试通过而写死值、绕路 | 片段 4.12 |
| 报告全是短 bullet、读不成文章 | 片段 4.13 |
| 需要并行读多个文件 / 工具调用可以并发 | 片段 4.9 |
| 数学公式渲染成 LaTeX 但环境不支持 | 片段 4.14 |
| 请求本身没说清"为了什么" | 片段 4.20 的模板 |
| agent 需要跨会话积累经验 | 片段 4.22（记忆写入规范） |
| 长时异步 agent 要把内容原样送到用户面前 | 片段 4.23（send_to_user 工具） |
| 前端一股 AI 味 | 片段 4.25；在 Opus 5.5 上点名具体要避开的样式——片段 4.36 |

### 长度控制怎么选

**主控手段只选一个**，叠加会和「最小信息集」原则冲突：

- **能给出具体数字就给数字**（"300–400 字"、"3–6 条，每条不超过 40 字"）。可判定、最有效，首选
- **给不出数字时**（输出形态本身不固定）才用片段 4.2 的通用简洁指令

**两个可以叠加的补充项**（它们管的是别的问题，不与主控手段冲突）：

- 产出物要**写成文件**时追加片段 4.4。理由：文件比对话回复更容易超长，而具体数字通常只约束了正文，管不住模型自行添加的填充章节。**给了数字又写文件时，两个都要**
- system prompt 很长、担心前面的长度要求被淹没时，在**末尾**追加 `<tone_preference>` 短提醒

一句话：**数字或 4.2 二选一；4.4 和尾部提醒按需叠加。**

---

## 9. 元原则（本文件没覆盖到的情况下靠这些判断）

1. **最小信息集**：能完整界定期望行为的最少 token。
2. **高度要合适**：既不写死脆弱的硬逻辑，也不给模糊到没有信号的空话。
3. **多给上下文，少给指令。**
4. **描述你想要的样子，而不是禁止你不想要的样子。**
5. **有工具就先改工具**：接口设计比 prompt 更能塑造行为。
6. **能确定性执行的，不要用 prompt**：该用 hook、schema、类型约束的地方别写"请务必"。
7. **模型变强，脚手架变负担**：定期回头删掉当初为弱模型加的东西。
8. **改完要测**：拿真实任务跑，不要凭感觉判断改好了。
