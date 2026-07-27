---
name: prompt-engineering
description: Writes, rewrites, reviews, and audits prompts and system prompts using Anthropic's current official guidance, and selects the right model and effort level for the task before drafting. Use when the user asks to write a prompt, improve or fix a prompt, design a system prompt, or check a prompt before running a task — including phrasings like "帮我写 prompt"、"改一下这个 prompt"、"设计 system prompt"、"处理任务之前先检查我的 prompt"、"帮我把需求整理成 prompt"、"help me prompt this better". Applies whether the resulting prompt will be pasted into Claude Code for a coding task or run in a desktop chat or Cowork session for a non-coding task. Also use when the user pastes a prompt and asks why it is not working, when they are authoring an agent system prompt or CLAUDE.md, or when they ask which model or effort level to use. Several widely repeated prompt techniques are now obsolete or rejected by the API, so consult this skill even when prompting from memory feels sufficient.
---

# 写 prompt 的规范

**适用模型**：Claude Opus 5 和 Claude Fable 5（含 Mythos 5）。更早模型的专属指引不在本文件范围内。

**这份 skill 存在的理由**：一批广为流传的 prompt 技法在这两个模型上已经失效、被 API 拒绝、或效果反了。你的训练直觉里很可能仍带着它们。**当直觉与第 3 节冲突时，以本文件为准。**

**基准**：Anthropic 官方文档与官方/员工公开材料，快照日期 2026-07-26。涉及具体 API 参数时回原文档确认——那是唯一会随模型更新的地方。

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
| **输出形态与长度** | 交付物是文档或报告时**必须确定**。Opus 5 默认输出偏长，不明说就会超长 |
| **可用的工具或文件** | 场景 A 必须确定（哪个仓库、哪些文件）；场景 B 看是否需要读取材料 |
| 目标模型与 effort | **不要问**，按第 2 节自己定，把结论告诉用户 |

只有当某项**既推断不出、猜错了又会让整个 prompt 白写**时才开口问。

需要问多项时用 AskUserQuestion **一次性问全**，不要逐个问。

---

## 2. 自行选定模型和 effort

这是起草前的必经一步，结论要写进交付说明里。

### 选模型

| 任务特征 | 选 | 依据 |
|---|---|---|
| 复杂 agentic 编码、多文件特性、较大重构、端到端功能开发 | **Opus 5** | 官方定位：最擅长困难编码任务，会完整做完而不留 stub 或占位 |
| 代码审查、找 bug | **Opus 5** | 高精确率与召回率，且低 effort 下准确率也保持 |
| 企业知识工作、文档、表格、幻灯片、长上下文分析 | **Opus 5** | 1M token 上下文为默认且最大，全窗口内指令遵循稳定 |
| 此前"太复杂、太长、太模糊"而做不了的问题；人要花**数小时到数周**的端到端工作；多日目标导向的自主长跑 | **Fable 5** | 官方定位就是接这类活；效果最好的团队把它用在自己最难的未解问题上 |
| 需要长时间无人值守自主运行、大量并行子智能体协作 | **Fable 5** | 长周期自主性和子智能体调度显著更强 |
| 攻击性网络安全、生物与生命科学 | **不要用 Fable 5** | 会触发安全分类器返回 `refusal`，需配置回落 |

默认选 **Opus 5**。只有当任务确实是"人要干几小时到几周"这个量级时才建议 Fable 5，并提醒用户：单次请求可能跑很多分钟，需要相应调整超时和使用方式。

### 选 effort

**两个模型都从默认 `high` 起步。**你真正要判断的是什么时候往*上*调。

| 模型 | 起步值 | 什么时候上调 | 什么时候下调 |
|---|---|---|---|
| **Opus 5** | **`high`**（默认值） | 编码或 agentic 工作**足够重**时 → `xhigh`；任务值得不设上限地烧 token → `max` | 在意成本或延迟、且评测显示质量不掉 → `medium`/`low`，可以放开用，把它当成控制 token 成本和响应时间的主要手段。这两档质量强，而 token 与延迟只是高档位的一个零头 |
| **Fable 5** | **`high`**（默认值） | 属于你手上最吃能力的那一档任务 → `xhigh` | 日常例行工作 → `medium`/`low`，它的低 effort 常优于旧模型的 `xhigh`；任务能完成但耗时超出需要，也该降 |

如果你的 effort 设置是从更早的模型沿用过来的，官方要求是**在自己的评测上重新做一次 effort 扫描**，而不是直接照搬。

**"足够重的 agentic 工作"指什么**（这是上调决策的分水岭，先判断清楚）。两个条件，必须同时成立：

1. **agentic**：任务需要**模型自己决定下一步做什么、并反复调用工具推进**，典型特征是多轮工具调用、中途要根据结果改变计划、单次运行时间长。写代码、跑测试改 bug、跨多份材料反复检索求证都算。反之，材料已经给全、一次性产出一份东西（写报告、做表、翻译、改写），即使产出物很大也**不算** agentic。
2. **足够重**：这一趟跑得久。官方对 `xhigh` 的定位是长周期 agentic 与编码任务——**超过 30 分钟、token 预算以百万计**。三次工具调用就能修完的 bug，是 agentic，但不够重。

**拿不准算不算 agentic 时看这一条**：**谁决定去哪找材料？** 用户把材料攒好放在那儿、模型只是逐份读完 → 不算；模型自己决定该去查什么、查完再根据结果决定下一步查什么 → 算。

**拿不准就留在 `high`。** 一个后来发现确实很重的任务少调了一档，代价小于把每个日常任务都跑在 `xhigh` 上。

**这些参数在哪儿设**（场景不同，能碰的东西不同）：

| | 场景 A：Claude Code | 场景 B：chat / Cowork | 直接调 API |
|---|---|---|---|
| effort | 用界面或斜杠命令设 | 用界面设 | `output_config: {effort}` |
| thinking | 由客户端管理 | 由客户端管理 | `thinking: {type: "adaptive"}` |
| `max_tokens` | **管不到** | **管不到** | 自己设；官方建议 `xhigh`/`max` 下从 64k 起步再按需调 |

所以给场景 A、B 写交付说明时，**只说"用哪个模型、设成哪档 effort"就够了**，不要把 `max_tokens`、`thinking` 这些 API 字段写进去——用户在那个界面里根本没有这些开关，写了只会让人困惑。下面几条里带 API 字段的部分，只在直接调 API 时才需要交代。

必须知道的几点：

- `effort: "high"` 与**完全不传该参数行为完全相同**
- effort 是**行为信号而非硬预算**，影响正文、工具调用和思考的全部 token
- **低 effort 会直接减少工具调用次数**、合并操作、省去前言；高 effort 则更多工具调用、先讲计划、给详细总结
- **effort 控制的是"想多少"不是"说多少"**。想让回复变短要写进 prompt（片段 4.2），降 effort 无效
- 启用 prompt caching 的会话中不要中途改 effort，会作废缓存
- `xhigh`/`max` 下要给 `max_tokens` 留足空间（它是思考加正文的**总**上限，官方建议 64k 起）

模型的具体已知倾向与对应处方在 `references/models.md`。

---

## 3. 失效清单（硬规则）

这些不是风格偏好，是会导致 API 报错或行为反向的具体问题。

### 3.1 不要用 prefill（预填充助手回复）
Opus 5 和 Fable 5 **均不支持**，在最后一个 assistant turn 预填充会返回 400。

官方给的替代是 **structured outputs、system prompt 指令、或 `output_config.format`**。具体来说：结构化输出用 Structured Outputs、`output_config.format`，或直接说明 schema；去开场白写 `Respond directly without preamble. Do not start with phrases like 'Here is...', 'Based on...', etc.`（迁移指南原文）；分类任务用带 enum 的 tool；续写把上文放进 user 消息。

### 3.2 不要用 `budget_tokens`
两个模型均不支持手动 extended thinking，设置返回 400。Opus 5 用 `thinking: {type: "adaptive"}` 加 `output_config: {effort: "..."}`；Fable 5 thinking 始终开启且无法配置，只用 `effort` 控深度。需要硬上限用 `max_tokens`。

### 3.3 不要用堆砌指令代替上下文
核心原则：**给上下文，别给一长串指令。**

Fable 5 官方文档的说法是——指令遵循已经好到一句简短指令就能管住一类行为，不必逐条枚举。Claude Code 团队据此把系统提示词砍掉 80% 以上而性能不降。

与其枚举十条"不要 X"，写一句正面描述加说明意图。把 `Default to using [tool]` 改成 `Use [tool] when it would enhance your understanding of the problem`。工具该怎么用，靠 tool description 和参数设计表达，不要在 system prompt 里反复提醒。

> 一处例外：Claude Code 最佳实践仍写着，可以在 **CLAUDE.md** 里加 `IMPORTANT` 或 `YOU MUST` 提高某条规则的遵守度。但只用在少数几行上——同一份文档也警告：CLAUDE.md 太长时 Claude 会忽略其中一半，因为重要规则淹没在噪音里。

### 3.4 不要让 Opus 5「再检查一遍」
Opus 5 无需提示即自行验证。保留 `double-check your answer` / `re-verify before responding` / `include a final verification step` 会造成**过度验证**，浪费 token 且不提升质量。

官方措辞是**删除这些指令，而不是改写它们**。harness 里额外加的验证步骤同理。

**注意区分：删的是"让模型自己再看一遍"，不是"去核对外部事实"。**

| 要删（模型自审） | 要留（核对外部依据） |
|---|---|
| "最后检查一遍有没有遗漏" | 跑测试、看构建退出码、对比截图（场景 A） |
| "再核对一次你的答案" | 把 A 材料的说法和 B 材料对上，冲突时说明取谁（场景 B） |
| "用子智能体复查你自己的工作"——指连同模型自己的判据一起交出去的那种 | 数字必须能追溯到某份原始材料的具体位置。**换到干净上下文、带独立判据的 verifier** 不在这一行删除之列，见 `references/models.md` |

判据很简单：**检查的依据来自模型自己，就删；来自外部可查证的东西，就留。** 场景 B 的研究与报告类任务里，跨来源交叉核对是任务本身，不是自我验证，写 prompt 时不要误删。

Fable 5 的长自主运行方向相反，要显式加周期验证——见 `references/models.md`。

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

> **一处例外：触发时机。** 片段 4.23 就是官方给的例子——Fable 5 光靠 tool description 很少会去调 `send_to_user`，所以官方在工具之外还配了一句写进 system prompt 的话，告诉它**什么时候**该用这个工具。这条规则禁的是把同一份**使用说明**写两遍，不是禁止给工具配一句它确实需要的触发说明。

### 3.8 不要靠 `temperature` 制造多样性
新一代模型对采样参数的支持在收紧，部分模型设非默认值直接返回 400。用 prompt 求多样性——让模型**先给选项再落地**：
```
Before building, propose 4 distinct visual directions tailored to this brief (each as: bg hex / accent hex / typeface, plus a one-line rationale). Ask the user to pick one, then implement only that direction.
```
（片段 4.25，逐字取自 Claude Sonnet 5 页面，照抄即可。）

### 3.9 不要要求模型复述或展示内部推理
"show your thinking"、"把思考过程写出来"这类指令在 **Fable 5** 上**可能触发** `reasoning_extraction` 拒绝类别，请求被拒、回落备用模型的比例上升。需要推理可见性改读结构化 `thinking` 块；需要在长时运行中把进度呈现给用户，用 send-to-user 工具（片段 4.23）。

### 3.10 不要写"不要思考 / 不要推理"
**在关闭 thinking 的情况下**，Opus 5 可能把 `<thinking>` 等内部 XML 标签泄漏到可见输出，而 system prompt 里这类规则会**加剧**泄漏。（Opus 5 默认开启 thinking，`xhigh`/`max` 下还无法关闭，所以这条只对刻意关闭 thinking 的集成有效。）写通用形式，且不要点名 thinking 标签：
```
Do not include internal or system XML tags in your response.
```

### 3.11 Console prompt improver 的输出必须人工过一遍
它的官方文档明写产出物包含 "Strategic prefills" —— 而 prefill 在这两个模型上返回 400。可以用它拿初稿，产出后必须按本节重新过一遍。

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
| **`references/models.md`** | 选完模型后读。Opus 5 与 Fable 5 的已知倾向与对应处方、两者相反的验证策略 |
| **`references/snippets.md`** | 需要现成模块时读。25 条编号条目，其中 24 条是官方文档的**逐字原文**英文片段，并说明哪些必须逐字照抄、哪些可以按语境改写 |
| **`references/agents.md`** | prompt 要驱动一个长时间自主运行的 agent 时读。上下文工程、跨窗口工作流、缓存命中、让模型自己编排 |

### 用到英文片段时（无论从哪个参考文件取的）

产出中文 prompt 却嵌入英文官方片段是常态，处理方式固定：

- 英文**成段放置**，不要拆开插进中文句子里；放在它管辖的那一节内部，紧跟中文小标题之后
- **必须在交付说明里告诉用户：这几段英文是官方原文，请不要翻译或"顺手改通顺"**。用户看到中英混杂会很自然地想统一语言，一改就失效
- 只需要一两个片段、且属于 `snippets.md` 里标注的"通用策略型"时，可以直接用中文表达，避免为两句话让整份 prompt 变成混排

哪些片段必须逐字、哪些可以改写，见 `references/snippets.md` 开头的分级表。

---

## 6. 三种请求的处理差异

**写或改 prompt（场景 A/B）**：按第 0 节工作流走完，交付说明按第 0 节第 6 步的要求写。不要塞入用户没要的解释、免责声明或替代方案清单。

**审查已有 prompt**：按第 3 节逐条比对，指出**具体问题和对应修法**，不要直接给一份全新的 prompt 替代掉——除非用户要求重写。按严重度排序：会导致 API 报错的（3.1、3.2、3.8）排最前，行为反向的（3.4、3.9、3.10）次之，风格问题最后。

**聊天途中临时检查（场景 C）**：走轻量流程。**不要小题大做**——不用完整跑一遍工作流，不用读全部参考文件。扫一眼第 3 节的硬规则，有问题就直接说"这里有个问题：X，改成 Y"，没问题就说没问题然后继续原来的话题。用户此刻要的是不被打断，不是一份审查报告。

---

## 7. 交付前自检

- [ ] 已经自己选定模型和 effort，并在交付说明里写清楚了
- [ ] 如果用了英文官方片段：交付说明里写了「这几段是官方原文，请不要翻译或改写」的提醒
- [ ] 没有 prefill、没有 `budget_tokens`
- [ ] 没有为防欠触发而堆的 `CRITICAL: You MUST` / `If in doubt, use [tool]`（**官方片段里的原文措辞不算**，例如 4.7 内含的 `you MUST read the file` 是调校过的原文，照抄即可）
- [ ] 目标是 Opus 5：**没有**任何自我验证或双重检查类指令（外部确定性检查不算；官方研究片段里那句 `Regularly self-critique your approach and plan` 也不算，见 `references/for-chat-and-cowork.md`）
- [ ] 目标是 Fable 5：没有"展示你的思考过程"类指令；长自主运行已加进度真实性约束
- [ ] **你自己写的**约束都是正面表述，且带了「为什么」（**官方片段里的原文措辞不算**——4.6、4.13、4.25 本身就是成串的否定式，例如 4.25 开头的 `NEVER use generic AI-generated aesthetics…`，是调校过的原文，照抄即可）
- [ ] 每条指令都想过：**一个善意的人类会怎么误解它？**（Claude Code 团队 Cat Wu 的自检法）
- [ ] 同一条指令没有在 system prompt 和 tool description 里重复出现（**模型确实需要的触发说明不算**——例如片段 4.23 配套的那句 system prompt，见 3.7）
- [ ] 示例的取舍符合第 4 节的场景判断，不是无脑加或无脑删
- [ ] 长上下文场景：材料在前、问题在后
- [ ] 输出长度有明确要求（Opus 5 尤其必要）
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
| 输出太长（面向阅读的正文） | 见下方「长度控制怎么选」。**注意降 effort 无效** |
| agent / 编码场景里中途播报太啰嗦 | 片段 4.3。这和正文长度是两个问题，长度控制那套管不到播报节奏 |
| 过度工程 | 片段 4.6 |
| 子智能体滥用 | 片段 4.10 |
| 思考过多、延迟高 | 片段 4.15；或降 effort |
| 复杂问题推理太浅 | **提高 effort**，而不是加 prompt 绕过 |
| 反复叮嘱某规则仍不生效 | prompt 太长、规则被淹没 → 精简，或改用确定性手段（hook、工具设计、schema） |
| 代码审查漏报 | 片段 4.16，把过滤移到下游。`references/for-claude-code.md` 里对抗式审查那条提醒是反过来的情形——没有下游过滤，只能由审查者自己过滤 |
| 长任务编造进度或谎报完成 | 片段 4.17 |
| 无人值守时中途停下来问问题 | 片段 4.18 |
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
| 前端一股 AI 味 | 片段 4.25 |

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
