# 写 prompt 的规范（单文件版）


**适用模型**：Claude Opus 5 和 Claude Fable 5.1（含 Mythos 5.1）。更早模型的专属指引不在本文件范围内。有几条 Fable 5.1 的处方目前只发布在 Fable 5 专页上，保留它们的依据是 Fable 5.1 专页写明现有 Fable 5 prompt 无需修改即可沿用；每一处都在正文里标明了。

**这份 skill 存在的理由**：一批广为流传的 prompt 技法在这两个模型上已经失效、被 API 拒绝、或效果反了。你的训练直觉里很可能仍带着它们。**当直觉与第 3 节冲突时，以本文件为准。**

**基准**：Anthropic 官方文档与官方/员工公开材料，快照日期 2026-09-03。涉及具体 API 参数时回原文档确认——那是唯一会随模型更新的地方。

---


> **与 skill 的关系**：本文件是 `prompt-engineering` skill 的**单文件合并版**，由 skill 的 SKILL.md 与五个参考文件合并生成，两者保持一致。
> 装了 skill 就用 skill（按需加载、更省上下文）；这份文件用于**没装 skill 的场合**——贴给别的会话、别的模型、或离线查阅。


## 目录

| # | 章节 |
|---|---|
| **0** | 工作流 |
| **1** | 先判断场景 |
| **2** | 自行选定模型和 effort |
| **3** | 失效清单（硬规则） |
| **4** | 官方提示词片段库 |
| **5** | 通用骨架与结构 |
| **6** | 按模型调校：Opus 5 与 Fable 5.1 |
| **7** | 场景 A：写给 Claude Code 执行的任务 prompt |
| **8** | 场景 B：写给 desktop chat 或 Cowork session 执行的任务 prompt |
| **9** | Agent 与长任务：上下文工程 |
| **10** | 三种请求的处理差异 |
| **11** | 交付前自检 |
| **12** | 故障到处方速查 |
| **13** | 元原则（本文件没覆盖到的情况下靠这些判断） |

> 片段编号 4.1–4.30 全部在第 4 节，全文各处引用均指向那里。

---

## 0. 工作流

1. **判断目标场景**（第 1 节）——决定读第 7 节还是第 8 节
2. **自行选定模型和 effort**（第 2 节）——不要把这个问题丢回给用户
3. **补齐缺失变量**（第 1 节末尾）——只问真正问不出就写不了的
4. **起草**：读对应的场景章节（第 7 节或第 8 节）；需要现成模块时再读第 4 节
5. **过第 11 节自检**
6. **交付**：prompt 本体 + 一段交付说明。说明包含五件事——选了哪个模型和 effort、替用户做了哪些假设、哪里最可能需要调、有没有只有用户能补的信息、**如果 prompt 里嵌了英文官方片段，提醒用户不要翻译它们**。能压到两三句就两三句；**假设较多时如实写长，不要为了"简短"把假设藏起来**——用户看不到假设就没法判断要不要改。

---

## 1. 先判断场景

用户在 Claude desktop 里描述需求，产出的 prompt 会去两个地方之一。**先判断去哪儿，因为写法差别很大。**

| 信号 | 场景 | 读这一节 |
|---|---|---|
| 涉及代码库、文件、git、跑测试、重构、修 bug、建项目；用户说要拿去 Claude Code 或 VS Code 跑 | **A：给 Claude Code 的任务 prompt** | 第 7 节 |
| 研究、分析、写作、整理资料、做文档表格幻灯片、日程、信息汇总；在 desktop chat 或 Cowork session 里直接跑 | **B：给 chat 或 Cowork 的任务 prompt** | 第 8 节 |
| 用户在聊天途中丢来一段 prompt 问"这样写行不行" | **C：临时检查** | 见第 10 节，走轻量流程 |

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
| **高强度推理与长周期 agentic 工作**：连续跑数小时的 agent 会话、多步深度研究、从分析一路做到成品文档 / 表格 / 幻灯片；或者用户在 Opus 5 更高 effort 下的评测仍不达标 | **Fable 5.1** | 官方选型建议（模型总览页和选型页的矩阵）。官方称其为 Anthropic 目前最强的公开发布模型；相对 Fable 5 的提升在高 effort 下最大，集中在长会话 agentic 编码、文档 / 表格 / 幻灯片、多步研究、视觉、长上下文和 computer use |
| 触及安全分类器类别的工作——这些领域的良性工作也可能触发 | **做好被拒的准备**：`cyber` 两个模型都会拒；`bio`、`frontier_llm`、`general_harms`（以及 `reasoning_extraction`）只有 Fable 5.1 会拒 | Opus 5 只有网络安全一类的分类器；Fable 5.1 覆盖与 Fable 5 相同的五类。被拒时返回 `stop_reason: "refusal"`，要配置回落（Fable 5.1 的回落目标：Opus 4.8 或 Opus 5）。在源码里找漏洞是允许的 |

默认选 **Opus 5**——官方说法是多数负载从它起步（原文 `Most workloads start with Claude Opus 5.`）。需要高强度推理和长周期 agentic 工作、或者用户自己的评测显示 Opus 5 在更高 effort 下仍不达标时，才建议 Fable 5.1，并提醒用户：它是现役阵容里最慢的模型，长时间工具调用回合里可能一连几分钟没有动静，超时和使用方式要相应调整。

### 选 effort

**两个模型都从默认 `high` 起步。**你真正要判断的是什么时候往*上*调。

| 模型 | 起步值 | 什么时候上调 | 什么时候下调 |
|---|---|---|---|
| **Opus 5** | **`high`**（默认值） | 编码或 agentic 工作**足够重**时 → `xhigh`；任务值得不设上限地烧 token → `max` | 在意成本或延迟、且评测显示质量不掉 → `medium`/`low`，可以放开用，把它当成控制 token 成本和响应时间的主要手段。这两档质量强，而 token 与延迟只是高档位的一个零头 |
| **Fable 5.1** | **`high`**（默认值） | 最吃能力的 agentic 与编码工作 → `xhigh` 或 `max`。它相对 Fable 5 的提升在高档位最大 | 例行或延迟敏感的工作，评测显示质量不掉之后 → `medium`/`low`。官方说 `medium` 下大致追平 Fable 5 且成本更低；`low` 下在单任务成本上常能和 Opus、Sonnet 系列竞争而分数更高——凡是你原本会「拿小模型跑高 effort」的地方都把它纳入比较 |

如果你的 effort 设置是从更早的模型沿用过来的，官方要求是**在自己的评测上重新做一次 effort 扫描**，而不是直接照搬——在 Fable 5.1 上，即使在 Fable 5 上做过扫描也要重做，因为各档 effort 的名字在不同模型上对应的思考量并不相同。

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
- **在 Opus 5 上，effort 控制的是"想多少"不是"说多少"**。想让回复变短要写进 prompt（片段 4.2），降 effort 不可靠。（Fable 5.1 的页面没有这样的说法；它有记录的文风问题是「太密」，见片段 4.26）
- 在请求之间改**顶层** effort 会作废 prompt cache。Fable 5.1 和 Opus 5 支持按消息改 effort（beta），缓存不失效——仅 API，见第 6 节
- `xhigh`/`max` 下要给 `max_tokens` 留足空间（它是思考加正文的**总**上限，官方建议 64k 起）

模型的具体已知倾向与对应处方在第 6 节。

---

## 3. 失效清单（硬规则）

这些不是风格偏好，是会导致 API 报错或行为反向的具体问题。

### 3.1 不要用 prefill（预填充助手回复）
Opus 5 和 Fable 5.1 **均不支持**，在最后一个 assistant turn 预填充会返回 400（Fable 5.1 的 what's-new 页把它列在「与 Fable 5 相比未变」的行为里）。

官方给的替代是 **structured outputs、system prompt 指令、或 `output_config.format`**。具体来说：结构化输出用 Structured Outputs、`output_config.format`，或直接说明 schema；去开场白写 `Respond directly without preamble. Do not start with phrases like 'Here is...', 'Based on...', etc.`（prompting 最佳实践页 "Migrating away from prefilled responses" 一节的原文）；分类任务用带 enum 的 tool；续写把上文放进 user 消息。

### 3.2 不要用 `budget_tokens`
两个模型均不支持手动 extended thinking，设置返回 400。Opus 5 用 `thinking: {type: "adaptive"}` 加 `output_config: {effort: "..."}`；Fable 5.1 thinking 始终开启且无法配置——`{"type": "enabled"}` 带 `budget_tokens` 和 `{"type": "disabled"}` 都返回 400，不传 `thinking` 或传 `{"type": "adaptive"}`，只用 `effort` 控深度。需要硬上限用 `max_tokens`。

### 3.3 不要用堆砌指令代替上下文
核心原则：**给上下文，别给一长串指令。**

Fable 5 专页的说法是——指令遵循已经好到一句简短指令就能管住一类行为，不必逐条枚举；Fable 5.1 专页写明现有 Fable 5 prompt 无需修改即可沿用，也没有撤回这个说法。Claude Code 团队据此把系统提示词砍掉 80% 以上而性能不降。

与其枚举十条"不要 X"，写一句正面描述加说明意图。把 `Default to using [tool]` 改成 `Use [tool] when it would enhance your understanding of the problem`。工具该怎么用，靠 tool description 和参数设计表达，不要在 system prompt 里反复提醒。

> 一处例外：Claude Code 最佳实践仍写着，如果 **CLAUDE.md** 里某一条 Claude 老是跳过，可以只给那一行加 `IMPORTANT` 之类的强调——并且说明，强调的行一多，就没有哪一行突出了。同一份文档也警告：CLAUDE.md 太长时 Claude 会忽略其中一半，因为重要规则淹没在噪音里。

### 3.4 不要让 Opus 5「再检查一遍」
Opus 5 无需提示即自行验证。保留 `double-check your answer` / `re-verify before responding` / `include a final verification step` 会造成**过度验证**，浪费 token 且不提升质量。

官方措辞是**删除这些指令，而不是改写它们**。harness 里额外加的验证步骤同理。

**注意区分：删的是"让模型自己再看一遍"，不是"去核对外部事实"。**

| 要删（模型自审） | 要留（核对外部依据） |
|---|---|
| "最后检查一遍有没有遗漏" | 跑测试、看构建退出码、对比截图（场景 A） |
| "再核对一次你的答案" | 把 A 材料的说法和 B 材料对上，冲突时说明取谁（场景 B） |
| "用子智能体复查你自己的工作"——指连同模型自己的判据一起交出去的那种 | 数字必须能追溯到某份原始材料的具体位置。**换到干净上下文、带独立判据的 verifier** 不在这一行删除之列，见第 6 节 |

判据很简单：**检查的依据来自模型自己，就删；来自外部可查证的东西，就留。** 场景 B 的研究与报告类任务里，跨来源交叉核对是任务本身，不是自我验证，写 prompt 时不要误删。

Fable 5.1 的长自主运行方向相反，要显式加周期验证——见第 6 节（这条指令发布在 Fable 5 专页上，沿用）。

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
本文两个模型上，`temperature`、`top_p`、`top_k` 设非默认值都直接返回 400（Opus 5 迁移指南写了；Fable 5.1 的 what's-new 页把它列在「与 Fable 5 相比未变」的行为里）。用 prompt 求多样性——让模型**先给选项再落地**：
```
Before building, propose 4 distinct visual directions tailored to this brief (each as: bg hex / accent hex / typeface, plus a one-line rationale). Ask the user to pick one, then implement only that direction.
```
（片段 4.25，逐字取自 Claude Sonnet 5 页面，照抄即可。）

### 3.9 不要要求模型复述或展示内部推理
"show your thinking"、"把思考过程写出来"这类指令在 **Fable 5.1** 上**可能触发** `reasoning_extraction` 拒绝类别（thinking 页对 Fable 5.1 和 Fable 5 都写了这一条），请求被拒、回落备用模型的比例上升。需要推理可见性改读结构化 `thinking` 块；需要在长时运行中把进度呈现给用户，用 send-to-user 工具（片段 4.23）。

### 3.10 不要写"不要思考 / 不要推理"
**在关闭 thinking 的情况下**，Opus 5 可能把 `<thinking>` 等内部 XML 标签泄漏到可见输出，而 system prompt 里这类规则会**加剧**泄漏。（Opus 5 默认开启 thinking，`xhigh`/`max` 下还无法关闭，所以这条只对刻意关闭 thinking 的集成有效。）写通用形式，且不要点名 thinking 标签——点名的效果不如通用形式：
```
Do not include internal or system XML tags in your response.
```

### 3.11 继承来的 prompt 先查 prefill
不是你自己写的 prompt——旧模板、4.6 之前的代码库、任何你控制不了的工具生成的东西——结尾仍可能带一个半截的 assistant turn，这在本文两个模型上返回 400。**继承来的东西一律按本节重新过一遍**，先看结尾那一轮。Claude Code 的 `claude-api` skill 带一个 `prompt-audit` 子命令，官方原文描述是 `for auditing prompts and tool descriptions for patterns written for older models`（changelog 2.1.221，2026-08-04）；它只是第一遍粗筛，不能替代本节。

### 3.12 不要在 Fable 5.1 上强制工具调用
`tool_choice` 设为 `any` 或 `tool` 类型，在 Fable 5.1 和 Mythos 5.1 上返回 400 `invalid_request_error`（thinking 始终开启，强制调用会跳过它）；`auto` 和 `none` 不受影响，Opus 5 也不受影响。要 schema 合规的 JSON，保持 `tool_choice: {"type": "auto"}` 并用 strict tool use 设 `strict: true`，或者把 schema 交给 structured outputs。要让模型调工具而不是用文字回答，在 prompt 里写明什么时候该用这个工具——官方例子是 `Use the get_weather tool to answer`——Fable 5.1 会可靠地照做。

---

## 4. 官方提示词片段库

> 除 4.1 外，所有英文片段均为 Anthropic 官方文档**逐字原文**，已逐字符核对；4.1 是一张判断表，不是引文。凡是引文的条目都标了出处页面，方便你回原文核对——文档会更新，以线上为准。
>
> **有两条出自本文适用范围之外的页面。** 4.16 和 4.25 逐字取自 **Claude Sonnet 5** 页面。Sonnet 5 是现役模型而非被取代的旧模型，且这两段都是与模型无关的通用提示词——但出处已在对应条目上标注，你可以自行判断。
>
> **有七条正文和一个变体只发布在 Fable 5 专页上。** 4.17、4.19–4.24 以及 4.6 里的 Fable 5 变体，Fable 5.1 专页没有重新刊出。保留它们的依据是 Fable 5.1 专页开头那句原文：`Your existing Claude Fable 5 prompts should perform well on Claude Fable 5.1 without changes`；每条都在正文处写明了这一点。4.18 是 Fable 5.1 专页唯一改写过的条目，这里用的是新原文。4.26–4.30 是 Fable 5.1 新增的。

### 本节目录

- 怎么用这些片段
- 片段索引——即 4.1–4.30 的目录
- 片段正文 4.1–4.30

### 怎么用这些片段

**分两类，处理方式不同：**

| 类别 | 哪些 | 怎么用 |
|---|---|---|
| **行为调校型**——官方为纠正某个具体模型行为而调过措辞 | 4.2、4.3、4.4、4.5、4.6、4.7、4.15、4.16、4.17、4.18、4.19、4.21、4.24、4.25、4.26、4.27、4.28、4.29 | **原文英文照抄，一个词都不要改**。改了就失去了它被调校出来的效果 |
| **通用策略型**——内容是策略本身，措辞没有特殊性 | 4.8、4.9、4.10、4.11、4.12、4.13、4.14、4.20、4.22、4.23、4.30 | 优先照抄；但如果原文的语境和你的任务对不上（比如 4.10 讲的是多文件代码调查，而你的任务是读材料写报告），**可以改写成贴合语境的表述**，保留其判据即可 |

**一条里带两个模型各自的块**：4.3、4.6、4.9、4.13 各有一段 Opus 5 或通用的块，另有一段 Fable 5.1 的块，每段都标了是在哪个模型上测出来的。按目标模型选其一，不要两段都放。（4.18 不同：它的两段都是 Fable 5.1 的原文，官方要求两段一起用。）

**中英混排怎么处理**（最常见的情况：用户要中文 prompt，而片段是英文）：

- 英文片段**成段放置**，不要拆开插进中文句子里
- 放在它管辖的那一节内部，紧跟中文小标题之后
- 在交付说明里**明确告诉用户：这几段英文是官方原文，请不要翻译或"顺手改通顺"**——用户看到中文里夹英文，很自然会想去统一语言，一改就失效
- 如果整份 prompt 只需要一两个片段，且片段属于上表的"通用策略型"，可以直接用中文表达，避免为了两句话让整份 prompt 变成中英混杂

**中文部分是使用说明，不要放进 prompt。**

### 片段索引

| 编号 | 用途 | 出处 |
|---|---|---|
| 4.1 | 示例（few-shot）该不该给 | 决策表，非引用 |
| 4.2 | 控制冗长（Opus 5 必备） | [Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) |
| 4.3 | 控制 Agent 播报节奏（Opus 5 必备；另有反方向的 Fable 5.1 块） | [Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5)；Fable 5.1 块出自 [Prompting Claude Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) |
| 4.4 | 控制书面交付物长度 | [Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) |
| 4.5 | 约束任务范围 | [Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) |
| 4.6 | 减少过度工程（含 Fable 5 变体与 Fable 5.1 块） | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)；Fable 5 变体出自 [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5)（沿用，见正文）；Fable 5.1 块出自 [Prompting Claude Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) |
| 4.7 | 防幻觉（代码场景） | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| 4.8 | 默认动手 / 默认不动手 | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| 4.9 | 并行工具调用（含 Fable 5.1 的 agent 循环提示句） | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)；提示句出自 [Prompting Claude Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) |
| 4.10 | 限制子智能体滥用 | [Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) |
| 4.11 | 安全护栏（不可逆操作） | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| 4.12 | 防止只为通过测试而写死 | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| 4.13 | 抑制 Markdown、改为流畅散文（另有反方向的 Fable 5.1 块） | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)；Fable 5.1 块出自 [Prompting Claude Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) |
| 4.14 | 关闭 LaTeX | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| 4.15 | 抑制过度思考 | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| 4.16 | 代码审查（要覆盖率而非过滤） | 措辞出自 [Prompting Claude Sonnet 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-sonnet-5)；诊断出自 [Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) |
| 4.17 | 长任务进度真实性（在 Fable 5 上测得） | [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5)——Fable 5.1 专页未重刊；沿用，见正文 |
| 4.18 | 无人值守自主运行（Fable 5.1 新原文） | [Prompting Claude Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) |
| 4.19 | 上下文预算安抚 | [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5)——Fable 5.1 专页未重刊；沿用，见正文 |
| 4.20 | 「给理由，不只给请求」模板 | [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5)——Fable 5.1 专页未重刊；沿用，见正文 |
| 4.21 | 最终汇报的可读性 | [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5)——Fable 5.1 专页未重刊；沿用，见正文 |
| 4.22 | 记忆系统的写入规范 | [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5)——Fable 5.1 专页未重刊；沿用，见正文 |
| 4.23 | send_to_user 工具 | [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5)——Fable 5.1 专页未重刊；沿用，见正文 |
| 4.24 | 划定行为边界 | [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5)——在 Fable 5.1 上它的前后两半已并入 4.18，见正文 |
| 4.25 | 前端设计（避免 AI 味） | [Prompting Claude Sonnet 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-sonnet-5) —— **不是**最佳实践页上那段更长的 frontend_aesthetics |
| 4.26 | 文风密度：去掉「腔调化」的写法（Fable 5.1） | [Prompting Claude Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) |
| 4.27 | 低 effort 下的搜索触发（Fable 5.1） | [Prompting Claude Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) |
| 4.28 | 局部编辑而非整文件重写（Fable 5.1） | [Prompting Claude Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) |
| 4.29 | `xhigh`/`max` 下给长输出留空间（Fable 5.1，仅 API） | [Prompting Claude Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) |
| 4.30 | 摘要时标明引文（Fable 5.1） | [Prompting Claude Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) |

---

### 4.1 示例（few-shot）：先判断该不该用

这是最容易用错的一处。**分场景决策**：

| 场景 | 该不该给示例 | 依据 |
|---|---|---|
| 锁定输出**格式 / 语气 / 结构**（如固定 JSON 形状、固定文风） | **给**，3–5 个，diverse，用 `<example>` 包裹 | 官方文档："Examples are one of the most reliable ways to steer Claude's output format, tone, and structure" |
| Agent 系统提示词、希望模型**发挥判断和创造力** | **不给**，改为优化工具接口设计 | Claude Code 团队实践："removing examples was extremely helpful, because it was just more creative than the examples we gave it" |
| 展示**推理模式** | 在 few-shot 里用 `<thinking>` 标签演示，模型会泛化到自己的思考块 | 官方文档 |

给示例时要求：相关（贴近真实用例）、多样（覆盖边界，避免模型学到意外模式）、结构化（`<example>` / `<examples>` 包裹）。

### 4.2 控制冗长（Opus 5 必备）

```
Keep responses focused, brief, and concise. Keep disclaimers and caveats short, and spend most of the response on the main answer. When asked to explain something, give a high-level summary unless an in-depth explanation is specifically requested.
```

长 system prompt 时，在**末尾**再放一个简短提醒：

```
<tone_preference>
Keep outputs reasonably concise.
</tone_preference>
```

### 4.3 控制 Agent 播报节奏（Opus 5 必备）

```
Before your first tool call, say in one sentence what you're about to do. While working, give a brief update only when you find something important or change direction. When you finish, lead with the outcome: your first sentence should answer "what happened" or "what did you find," with supporting detail after it for readers who want it.
```

> 调整方向时同理：**用正面示例描述你想要的样子**，比说"不要怎样"有效得多。

**Fable 5.1 的问题正好反过来。** 它在长时间工具调用回合里默认给用户的播报*比 Fable 5 更少*，effort 越高、工具链越长越明显：agent 一连几分钟没有动静，或者最后一条消息只讲了最后一步。官方给的处理顺序，在加任何 prompt 之前：第一，先确认播报到底有没有送到你的客户端——它们以进度更新型 `thinking` 块的形式返回，而在默认的 `thinking.display: "omitted"` 下这些块是空的；设成 `"updates"`（beta，header `thinking-display-updates-2026-08-18`）或 `"summarized"`。第二，删掉为更啰嗦的旧模型写的压制播报的句子，例如 "hold all findings for the final response"。做完这两步还想要更多播报（结对编程等人在环场景）时，再加一句 system prompt：

```
Before you start, say in a line what you're about to do; brief updates while you work help the user follow along. Close with a short recap that stands on its own — what you found, what you did, and what's next — so a reader who only sees the last message has the full picture.
```

如果你的产品会折叠或隐藏工具输出，要告诉模型，否则它可能会跑命令去「给用户看」你界面根本不显示的输出。用 turn-scoped system message（`clear_at: "next_user_message"`，beta）传这句：

```
Only you see that command's output — the user's terminal shows at most a few lines of it. If the user needs to read any of it, put it in your reply.
```

### 4.4 控制书面交付物长度（Opus 5）

```
Match the length of written documents to what the task needs: cover the substance, but do not pad with filler sections, redundant summaries, or boilerplate.
```

### 4.5 约束任务范围（防止擅自扩大）

```
Deliver what was asked, at the scope intended. Make routine judgment calls yourself, and check in only when different readings of the request would lead to materially different work. If the request seems mistaken or a better approach exists, say so in a sentence and continue with the task as asked rather than quietly narrowing, widening, or transforming it. Finish the whole task, and stop short of actions that are clearly beyond what was asked.
```

### 4.6 减少过度工程

```
Avoid over-engineering. Only make changes that are directly requested or clearly necessary. Keep solutions simple and focused:

- Scope: Don't add features, refactor code, or make "improvements" beyond what was asked. A bug fix doesn't need surrounding code cleaned up. A simple feature doesn't need extra configurability.

- Documentation: Don't add docstrings, comments, or type annotations to code you didn't change. Only add comments where the logic isn't self-evident.

- Defensive coding: Don't add error handling, fallbacks, or validation for scenarios that can't happen. Trust internal code and framework guarantees. Only validate at system boundaries (user input, external APIs).

- Abstractions: Don't create helpers, utilities, or abstractions for one-time operations. Don't design for hypothetical future requirements. The right amount of complexity is the minimum needed for the current task.
```

**Fable 5 变体**（出自 Fable 5 专页，多了 feature flag / 向后兼容一句，专治高 effort 下的擅自整理。Fable 5.1 专页没有重刊这段，但写明现有 Fable 5 prompt 在 Fable 5.1 上无需修改即可正常工作）：

```
Don't add features, refactor, or introduce abstractions beyond what the task requires. A bug fix doesn't need surrounding cleanup and a one-shot operation usually doesn't need a helper. Don't design for hypothetical future requirements: do the simplest thing that works well. Avoid premature abstraction and half-finished implementations. Don't add error handling, fallbacks, or validation for scenarios that cannot happen. Trust internal code and framework guarantees. Only validate at system boundaries (user input, external APIs). Don't use feature flags or backwards-compatibility shims when you can just change the code.
```

**Fable 5.1 块**（Fable 5.1 专页 "Keep changes and tests to what the task asks for" 一节）。Fable 5.1 上相邻的倾向是「交付所要求的，*有时还多给*」：顺手修旁边的代码、扩展任务没提的行为、提交超出改动所需的测试文件。官方测得：加上这段后，未被要求的添加和提交的测试代码大幅减少，任务成功率没有可测的变化：

```
If, while working or testing, you find a pre-existing bug, a performance concern, or behavior the task doesn't mention, don't fix, optimize or extend it in this change unless the requested behavior cannot work without it; report it as a follow-up in your summary. Where the task is ambiguous, implement the reading its wording and the surrounding code most directly support, state that assumption in your summary, and don't build for the other readings as well. Verify your work however you like; scratch scripts and quick checks need not be kept. Commit tests only where the task asks for them or this repository already keeps tests for this kind of change, sized like the neighboring test files — roughly one focused test per stated behavior — and don't turn scratch checks into additional permanent test files. This is about extras only: implement every behavior the task asks for, completely.
```

### 4.7 防幻觉（代码场景）

```
<investigate_before_answering>
Never speculate about code you have not opened. If the user references a specific file, you MUST read the file before answering. Make sure to investigate and read relevant files BEFORE answering questions about the codebase. Never make any claims about code before investigating unless you are certain of the correct answer - give grounded and hallucination-free answers.
</investigate_before_answering>
```

### 4.8 默认动手 / 默认不动手

默认执行：
```
<default_to_action>
By default, implement changes rather than only suggesting them. If the user's intent is unclear, infer the most useful likely action and proceed, using tools to discover any missing details instead of guessing. Try to infer the user's intent about whether a tool call (e.g., file edit or read) is intended or not, and act accordingly.
</default_to_action>
```

默认只建议：
```
<do_not_act_before_instructions>
Do not jump into implementation or change files unless clearly instructed to make changes. When the user's intent is ambiguous, default to providing information, doing research, and providing recommendations rather than taking action. Only proceed with edits, modifications, or implementations when the user explicitly requests them.
</do_not_act_before_instructions>
```

> 另注：想让模型动手时，动词要直接。`Change this function to improve its performance.` 优于 `Can you suggest some changes to improve this function?` —— 后者模型往往只给建议。

### 4.9 并行工具调用

```
<use_parallel_tool_calls>
If you intend to call multiple tools and there are no dependencies between the tool calls, make all of the independent tool calls in parallel. Prioritize calling tools simultaneously whenever the actions can be done in parallel rather than sequentially. For example, when reading 3 files, run 3 tool calls in parallel to read all 3 files into context at the same time. Maximize use of parallel tool calls where possible to increase speed and efficiency. However, if some tool calls depend on previous calls to inform dependent values like the parameters, do NOT call these tools in parallel and instead call them sequentially. Never use placeholders or guess missing parameters in tool calls.
</use_parallel_tool_calls>
```

**Fable 5.1 的 agent 循环提示句。** 请求里点名要取几样东西时，Fable 5.1 会照常并行调用。例外是编码和 computer use 循环：下一批彼此独立的调用只是任务隐含的、没被明说，这时它可能一轮只发一个调用（Fable 5 会打包发）。不影响答案质量，但每多一轮就多花 token、一次往返和等待时间。官方的修法是在当前请求末尾加一句：

```
First privately list what you need next; then request every item that doesn't depend on another's result in this one response.
```

最佳实践页对 Fable 5.1 的长 agent 循环也是这么说的（每轮工具结果之后，把并行调用指令作为 turn-scoped system message 发一次）。每次回传工具结果时都重新附一份：放在那条 user 消息之后，作为 turn-scoped system message（`role: "system"`，`clear_at: "next_user_message"`；beta header `mid-conversation-system-clear-at-2026-08-21`），之前的副本原样留在数组里、一个字节都别动——有了更新的 user 消息后 API 会自动清掉它们，而删掉它们等于改了历史轮次。没开 beta 时，把这句放进同一条 user 消息里 `tool_result` 块之后的文本块。

### 4.10 限制子智能体滥用

```
Delegate to a subagent only for large tasks that are genuinely independent and parallelizable, such as a wide multi-file investigation. Do not delegate work you can finish yourself in a handful of tool calls, and do not use subagents to verify or double-check your own work. If one subagent can complete the task, use one rather than several, and keep spawn counts low.
```

### 4.11 安全护栏（不可逆操作）

```
Consider the reversibility and potential impact of your actions. You are encouraged to take local, reversible actions like editing files or running tests, but for actions that are hard to reverse, affect shared systems, or could be destructive, ask the user before proceeding.

Examples of actions that warrant confirmation:
- Destructive operations: deleting files or branches, dropping database tables, rm -rf
- Hard to reverse operations: git push --force, git reset --hard, amending published commits
- Operations visible to others: pushing code, commenting on PRs/issues, sending messages, modifying shared infrastructure

When encountering obstacles, do not use destructive actions as a shortcut. For example, don't bypass safety checks (e.g. --no-verify) or discard unfamiliar files that may be in-progress work.
```

### 4.12 防止只为通过测试而写死

```
Please write a high-quality, general-purpose solution using the standard tools available. Do not create helper scripts or workarounds to accomplish the task more efficiently. Implement a solution that works correctly for all valid inputs, not just the test cases. Do not hard-code values or create solutions that only work for specific test inputs. Instead, implement the actual logic that solves the problem generally.

Focus on understanding the problem requirements and implementing the correct algorithm. Tests are there to verify correctness, not to define the solution. Provide a principled implementation that follows best practices and software design principles.

If the task is unreasonable or infeasible, or if any of the tests are incorrect, please inform me rather than working around them. The solution should be robust, maintainable, and extendable.
```

### 4.13 抑制 Markdown、改为流畅散文

````
<avoid_excessive_markdown_and_bullet_points>
When writing reports, documents, technical explanations, analyses, or any long-form
content, write in clear, flowing prose using complete paragraphs and sentences. Use
standard paragraph breaks for organization and reserve markdown primarily for `inline
code`, code blocks (```...```), and simple headings (## and ###). Avoid using **bold**
and *italics*.

DO NOT use ordered lists (1. ...) or unordered lists (*) unless: a) you're presenting
truly discrete items where a list format is the best option, or b) the user explicitly
requests a list or ranking

Instead of listing items with bullets or numbers, incorporate them naturally into
sentences. This guidance applies especially to technical writing. Using prose instead of
excessive formatting will improve user satisfaction. NEVER output a series of overly
short bullet points.

Your goal is readable, flowing text that guides the reader naturally through ideas
rather than fragmenting information into isolated points.
</avoid_excessive_markdown_and_bullet_points>
````

**Fable 5.1 的方向相反。** 旧模型在聊天里滥用 bullet 和加粗，Fable 5.1 反过来：加粗用得更少，也更少用标题、列表和引号，所以为旧模型写的反格式化规则会压掉内容本来需要的结构。最佳实践页对上面这段本身就是这么说的：在 Fable 5.1 上 "a block like this can suppress structure the content needs. Remove it, or replace it with the shorter rule"——所谓 shorter rule 就是一条说明「什么时候该用什么格式」的规则，例如：

```
Use lists and bullet points when asked to, or when the content is multifaceted enough that they help with clarity. If the person explicitly requests minimal formatting, always format your responses without bullet points, headers, lists, or bold emphasis, as requested. In conversational, personal, or emotional exchanges, keep to plain prose.
```

### 4.14 关闭 LaTeX

```
Format your response in plain text only. Do not use LaTeX, MathJax, or any markup notation such as \( \), $, or \frac{}{}. Write all math expressions using standard text characters (e.g., "/" for division, "*" for multiplication, and "^" for exponents).
```

### 4.15 抑制过度思考

```
When you're deciding how to approach a problem, choose an approach and commit to it. Avoid revisiting decisions unless you encounter new information that directly contradicts your reasoning. If you're weighing two approaches, pick one and see it through. You can always course-correct later if the chosen approach fails.
```

抑制 adaptive thinking 触发频率：
```
Thinking adds latency and should only be used when it will meaningfully improve answer quality - typically for problems that require multistep reasoning. When in doubt, respond directly.
```

### 4.16 代码审查（要覆盖率而非过滤）

```
Report every issue you find, including ones you are uncertain about or consider low-severity. Do not filter for importance or confidence at this stage - a separate verification step will do that. Your goal here is coverage: it is better to surface a finding that later gets filtered out than to silently drop a real bug. For each finding, include your confidence level and an estimated severity so a downstream filter can rank them.
```

> **出处说明**：上面这段逐字取自 **Claude Sonnet 5** 页面（"Code review harnesses"），不是 Opus 5 页面。收进来是因为 Opus 5 页面自己也用散文给出了同一条处方——`ask it to report everything and filter in a separate pass instead`——但没有配套措辞。这段是与模型无关的通用提示词，用在 Opus 5 上时请知悉这一点。
>
> 依据：Opus 5 会**字面执行**"只报高危"、"保守一点"这类指令——它会同样深入地调查代码、同样找到 bug，然后把自己判定为低于你所设门槛的发现**不报出来**。表现为精确率上升、召回率下降。把过滤动作移到独立的下游步骤。

### 4.17 长任务进度真实性（在 Fable 5 上测得）

出自 Fable 5 专页（"Ground progress claims during long runs"）：Anthropic 在 Fable 5 上的测试中，这段几乎消除了编造的进度汇报，即便是刻意诱导它编造的任务。Fable 5.1 专页没有重刊这段，也没有重述这个倾向；它写明现有 Fable 5 prompt 在 Fable 5.1 上无需修改即可正常工作，本条据此沿用到 Fable 5.1：

```
Before reporting progress, audit each claim against a tool result from this session. Only report work you can point to evidence for; if something is not yet verified, say so explicitly. Report outcomes faithfully: if tests fail, say so with the output; if a step was skipped, say that; when something is done and verified, state it plainly without hedging.
```

### 4.18 无人值守自主运行（Fable 5.1 新原文）

Fable 5.1 专页改写了这一条，所以这里不再保留 Fable 5 的措辞。倾向：在复杂的异步任务上，模型有时只描述下一步要做什么而不去做（"Next, I'll …"），或者对原始请求已经覆盖的步骤停下来请示（"Shall I apply this?"）。官方给了两段 system prompt 补充，**两段一起用**；必须控制 prompt 长度时只用第一段，效果大部分保得住。第一段开头那句告诉模型「用户不在看」的话承担了大部分效果，**照原样保留**。如果你的产品需要模型在特定操作前停下确认，在它后面加一句列出来。这段也可能让模型对含糊请求更少发问，要在自己的任务上核一下这个取舍。

```
You are operating autonomously. The user is not watching in real time and cannot answer questions mid-task, so asking 'Want me to…?' or 'Shall I…?' will block the work. For reversible actions that follow from the original request, proceed without asking. Stop only for destructive actions or genuine scope changes the user must decide. Offering follow-ups after the task is done is fine; asking permission before doing the work is not.

Exception: when the user is describing a problem, asking a question, or thinking out loud rather than requesting a change, the deliverable is your assessment. Report your findings and stop. Don't apply a fix until they ask for one.

Before ending your turn, check your last paragraph. If it is a plan, an analysis, a question, a list of next steps, or a promise about work you have not done ('I'll…', 'let me know when…'), do that work now with tool calls. That includes retrying after errors and gathering missing information yourself. Do not stop because the context or session is long. End your turn only when the task is complete or you are blocked on input only the user can provide.

Before running a command that changes system state (such as restarts, deletes, or config edits), check that the evidence actually supports that specific action. A signal that pattern-matches to a known failure may have a different cause.
```

第二段把用户的请求定义为交付物的范围：

```
# Delivering work
The user's request — or the plan they approved — sets the scope, and the scope is the deliverable: don't quietly narrow, widen, or swap it. Read ambiguity the way a careful colleague would: make routine judgment calls yourself, and check in only when different readings would lead to materially different work. If you see a real problem with the task as specified, say so in a sentence or two and keep building under stated assumptions; if the user hears the concern and reaffirms, that is their decision, so deliver the full request.

If a question comes up partway, first do everything that doesn't depend on the answer; then state the assumption you made, or — when going ahead on a wrong guess would be unsafe or would make the work useless — put the question at the end of a turn that also delivers that progress. If one part turns out to be blocked, complete every other part in full and say exactly what you left out and why — the whole task is the deliverable, and scaling it down is the user's call, not yours. A step you have decided on is something to run, not to announce: describing the next step and ending the turn leaves it undone until the user replies.

Keep changes to what the request needs. Something else you notice worth doing — cleanup or documentation the task didn't call for, a change to a file the task didn't require — is a suggestion to make at the end, not a change to make; actions clearly beyond what the ask implies, and risky or destructive ones, still need the user's go-ahead.
```

第一段里的 "Exception:" 一段和最后一段就是 4.24 的前后两半，不要在这段之上再叠加 4.24。

### 4.19 上下文预算安抚（防止提前收工）

出自 Fable 5 专页（"Rare cases of context-budget concern"）：当 harness 会向模型显示剩余 token 倒计时，模型可能提议开新会话或自行缩减工作。**首选是不要向模型暴露剩余预算数字**；必须暴露时加下面这段。Fable 5.1 专页没有重刊这段，但写明现有 Fable 5 prompt 在 Fable 5.1 上无需修改即可正常工作；4.18 的 Fable 5.1 自主运行块里另有一句针对同一问题的话：`Do not stop because the context or session is long.`

```
You have ample context remaining. Do not stop, summarize, or suggest a new session on account of context limits. Continue the work.
```

### 4.20 「给理由，不只给请求」模板

出自 Fable 5 专页（"Give the reason, not only the request"）的通用请求模板，值得作为默认写法。Fable 5.1 专页没有重刊这段，但写明现有 Fable 5 prompt 在 Fable 5.1 上无需修改即可正常工作：

```
I'm working on [the larger task] for [who it's for]. They need [what the output enables]. With that in mind: [request].
```

### 4.21 最终汇报的可读性（长 agent 会话）

出自 Fable 5 专页（"Readability when communicating with the user"）。Fable 5.1 专页没有重刊这段，但写明现有 Fable 5 prompt 在 Fable 5.1 上无需修改即可正常工作；注意 Fable 5.1 专页描述的*默认*文风问题是「太密」而不是「满是缩写速记」，那是 4.26 管的事，不是这条：

```
Terse shorthand is fine between tool calls (that's you thinking out loud, and brevity there is good). Your final summary is different: it's for a reader who didn't see any of that.

If you've been working for a while without the user watching (overnight, across many tool calls, since they last spoke), your final message is their first look at any of it. Write it as a re-grounding, not a continuation of your working thread: the outcome first, then the one or two things you need from them, each explained as if new. The vocabulary you built up while working is yours, not theirs; leave it behind unless you re-introduce it.

When you write the summary at the end, drop the working shorthand. Write complete sentences. Spell out terms. Don't use arrow chains, hyphen-stacked compounds, or labels you made up earlier. When you mention files, commits, flags, or other identifiers, give each one its own plain-language clause. Open with the outcome: one sentence on what happened or what you found. Then the supporting detail. If you have to choose between short and clear, choose clear.
```

### 4.22 记忆系统的写入规范

出自 Fable 5 专页（"Construct a memory system"）。Fable 5.1 专页没有重刊这段，但写明现有 Fable 5 prompt 在 Fable 5.1 上无需修改即可正常工作：

```
Store one lesson per file with a one-line summary at the top. Record corrections and confirmed approaches alike, including why they mattered. Don't save what the repo or chat history already records; update an existing note rather than creating a duplicate; delete notes that turn out to be wrong.
```

从历史会话冷启动记忆库：
```
Reflect on the previous sessions we've had together. Use subagents to identify core themes and lessons, and store them in [X]. Make sure you know to reference [X] for future use.
```

### 4.23 send_to_user 工具（长时异步 agent）

长时间异步运行的 agent 需要一个「不结束回合就把内容原样送到用户面前」的通道。工具输入永远不会被摘要，所以内容能完整抵达。出自 Fable 5 专页（"Create a send-to-user tool"）；Fable 5.1 专页没有重刊这段，但写明现有 Fable 5 prompt 在 Fable 5.1 上无需修改即可正常工作。面向 Fable 5.1 时，先确认模型自己的进度播报有没有送到你的客户端（见 4.3）——这个工具是给必须原样送达的内容用的，不是给例行播报用的。

```json
{
  "name": "send_to_user",
  "description": "Display a message directly to the user. Use this for progress updates, partial results, or content the user must see exactly as written before the task finishes.",
  "input_schema": {
    "type": "object",
    "properties": {
      "message": {
        "type": "string",
        "description": "The content to display to the user."
      }
    },
    "required": ["message"]
  }
}
```

**只定义工具不够**——官方说法是，不在 system prompt 里加引导语，Fable 5 很少会调用它：
```
Between tool calls, when you have content the user must read verbatim (a partial deliverable, a direct answer to their question), call the send_to_user tool with that content. Use send_to_user only for user-facing content, not for narration or reasoning.
```

### 4.24 划定行为边界（防止未被要求的动作）

出自 Fable 5 专页（"State the boundaries"），那里描述的倾向是偶尔做未被要求的动作——没人要它写邮件却起草了一封、创建防御性的 git 分支备份。在 Fable 5.1 上，这段的前后两半已经并入 4.18 的自主运行块（即其中的 "Exception:" 段和最后一段，括号内措辞改为 `(such as restarts, deletes, or config edits)`）；想只要边界、不要整段自主运行框架时用这个独立版本，两者不要叠加：

```
When the user is describing a problem, asking a question, or thinking out loud rather than requesting a change, the deliverable is your assessment. Report your findings and stop. Don't apply a fix until they ask for one. Before running a command that changes system state (restarts, deletes, config edits), check that the evidence actually supports that specific action. A signal that pattern-matches to a known failure may have a different cause.
```

> 后半句常被忽略，但对自主运行很关键：**长得像某个已知故障的信号，成因可能完全不同**（原文 `A signal that pattern-matches to a known failure may have a different cause.`），动系统状态前要确认证据真的支持这个具体动作。

### 4.25 前端设计（避免 AI 味）

> **出处说明**：下面两段都逐字取自 **Claude Sonnet 5** 页面（"Design and frontend defaults"）。最佳实践页上另有一段**内容不同、长得多**的 `frontend_aesthetics`——想要那一段的话请直接去原页面取，不要用这里这段。

```
<frontend_aesthetics>
NEVER use generic AI-generated aesthetics like overused font families (Inter, Roboto, Arial, system fonts), cliched color schemes (particularly purple gradients on white or dark backgrounds), predictable layouts and component patterns, and cookie-cutter design that lacks context-specific character. Use unique fonts, cohesive colors and themes, and animations for effects and micro-interactions.
</frontend_aesthetics>
```

产生多样性（不要靠 temperature，用这招）：
```
Before building, propose 4 distinct visual directions tailored to this brief (each as: bg hex / accent hex / typeface, plus a one-line rationale). Ask the user to pick one, then implement only that direction.
```

### 4.26 文风密度：去掉「腔调化」的写法（Fable 5.1）

官方说 Fable 5.1 的写作整体上比此前的 Claude 模型更进一步——套话更少、没解释的行话更少——但有些情况下比 Fable 5 更密：句子更长、段落更少。这和「太长」是两个问题（4.2–4.4 管不到它）。官方的修法是一段定义反模式（mannered prose）的指令，加在 user 消息（首选）或 system prompt 里：

```
Mannered prose substitutes metaphor and flourish for direct statement. Instead of "a parameter worth varying," the mannered writer produces "a dial worth turning." Instead of "this point still matters," they write "this point earns its keep." The phrases exist to display the writer, not to convey the idea, and readers can tell. That is why mannered prose irritates: it makes the reader work harder so the writer can perform. It is also imprecise. Metaphors drag in connotations the writer did not choose and cannot control. The fix is to say what you mean. When a literal phrase is available, use it.
```

短版通常也有效：

```
Please remove all mannered prose.
```

### 4.27 低 effort 下的搜索触发（Fable 5.1）

在 `low` effort 下，Fable 5.1 比 Fable 5 更少去调用搜索或检索工具，更倾向于凭记忆作答。有些情况下最简单的修法是只给受影响的那几轮提高 effort，而不是整个会话（按消息设 effort，beta，见第 6 节）。否则在 system prompt 里说明：认得一个名字不等于知道它的现状，这类名字要按用户写的原样去搜：

```
When a query centers on a name you do not confidently recognize, or recognize from a fast-moving area like AI models and developer tools where the landscape shifts within months, the name itself is the thing to verify: search before answering, and include the name as the user wrote it in at least one query alongside any reformulations. This holds even when you have some background on it — partial background is exactly what makes an out-of-date answer sound authoritative, so familiarity is not a reason to skip the search.
```

### 4.28 局部编辑而非整文件重写（Fable 5.1）

Fable 5.1 比 Fable 5 更倾向于把整个文本文件重写一遍，而不是做定点编辑。结果文件通常一样，但除非文件很短或大部分内容都要改，重写会多花输出 token 和时间。追加到 system prompt 或第一条 user 消息；官方说这句能让 Fable 5.1 在中小改动上回到 Fable 5 的水平：

```
The number of tokens used to edit files is best minimized, all else being equal. Therefore, when it will not affect the end result, try to surgically edit a file rather than rewrite the entire thing.
```

### 4.29 `xhigh`/`max` 下给长输出留空间（Fable 5.1，仅 API）

在 `xhigh` 尤其是 `max` 下，Fable 5.1 开始写回复之前可能会思考更久。当单个请求要一份长交付物（比如把一篇长文档整个重写），它可能先在思考里把大半内容起草一遍，再作为回复写一遍：等更久、输出 token 更多。官方首选是把这类请求跑在 `high`，只有测出质量提升才上调。如果确实要跑在 `xhigh` 或 `max`：`max_tokens` 要给思考*和*回复都留出空间，并把下面这段追加到 user 消息末尾，`[max_tokens]` 换成请求实际的值（例如 64,000）。它能让散文和代码类请求的思考明显变短：

```
Everything produced in one reply, including any reasoning or drafting done before the reply, counts toward a single limit of about [max_tokens] tokens. If that limit is reached before the reply is finished, the person receives a cut-off response and has to start over. Composing an entire output or deliverable in full as reasoning and then again as a reply would double the length of the turn without improving the result, so don't do that.

Instead, when the person has asked for a long or effort-intensive deliverable such as a multi-section document, a large table or dataset, or a complete code file, spend extra effort on understanding the request, checking the inputs the answer depends on, settling the structure and other difficult decisions, and otherwise using the reasoning space to reason and the output space to write an output. Usually it is not needed to draft an output multiple times.
```

### 4.30 摘要时标明引文（Fable 5.1）

做文档摘要时，Fable 5.1 比 Fable 5 更容易把原文段落照搬进来却不标为引用。官方的修法是在 system prompt 里放一个完整的正确示例——用户请求、回复、以及一句说明为什么这个回复是对的：

```
<example>
<user>look up how the Riverton Ledger and the Coast Dispatch each covered the Harbor Bridge closure and compare their reporting</user>
<response>
[web_search: Harbor Bridge closure Riverton Ledger]
[web_search: Harbor Bridge closure Coast Dispatch]
Both outlets agree on the basics: the bridge closed on March 3 after inspectors found cracked welds, and the state expects repairs to take about eight months. Where they differ is emphasis. The Ledger treats it as a local-economy story. The Dispatch frames it as a funding failure; its editorial calls the closure "entirely foreseeable." Read together, the Ledger explains who is affected now and the Dispatch explains how it came to this — neither account alone gives the whole picture.
</response>
<rationale>CORRECT: The response is organized around where the two outlets agree and differ, not as a walk through either article. Each outlet's reporting is conveyed in one or two sentences of the assistant's own indirect speech. One short marked phrase from one source; every other claim is reworded. The response is still specific and complete.</rationale>
</example>
```

把示例里的两行 `[web_search: ...]` 换成你自己的工具名，让模型把它们读成模板化的工具输出，而不是要照写的字面文本。

---

## 5. 通用骨架与结构

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

### 用到英文片段时

产出中文 prompt 却嵌入英文官方片段是常态，处理方式固定：

- 英文**成段放置**，不要拆开插进中文句子里；放在它管辖的那一节内部，紧跟中文小标题之后
- **必须在交付说明里告诉用户：这几段英文是官方原文，请不要翻译或"顺手改通顺"**。用户看到中英混杂会很自然地想统一语言，一改就失效
- 只需要一两个片段、且属于第 4 节里标注的"通用策略型"时，可以直接用中文表达，避免为两句话让整份 prompt 变成混排

哪些片段必须逐字、哪些可以改写，见第 4 节开头的分级表。

---

## 6. 按模型调校：Opus 5 与 Fable 5.1

> 片段编号（如 4.2）指向第 4 节。
>
> 下表"处方"列里的英文块，**全部是官方文档原文**，照抄即可，不要翻译或改写。

### 本节目录

- Claude Opus 5
- Claude Fable 5.1 / Mythos 5.1——Fable 5.1 页面明确写了的；只在 Fable 5 页面写了、5.1 页面没有重述的
- 验证指令：Opus 5 与 Fable 5.1 方向相反
- effort 取值参考

### Claude Opus 5（默认 effort: high；thinking 默认开启）

| 已知倾向 | 处方 |
|---|---|
| 用户可见回复默认偏长；**降 effort 不能可靠缩短可见输出** | 显式写简洁指令（4.2） |
| Agent 场景播报频繁、单条消息长 | 显式描述播报节奏（4.3） |
| 写到磁盘的文档偏长 | 长度校准指令（4.4） |
| **自我验证充分** | **删除**所有验证类指令（见 3.4）。官方措辞是删除，不是改写 |
| 自行扩大任务范围 | 范围约束（4.5） |
| 更积极派生子智能体 | 显式设限（4.10）。在 Claude Code 和 Agent SDK 里，确定性上限是 `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH`、`CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS` 和 SDK 的 `max_budget_usd`（需要 Claude Code 2.1.217 及以上）；Claude Code 只在用它的 `claude_code` system prompt 预设时才会在 Opus 5 上自动加一条委派指令，自定义或不传 system prompt 时要自己加 |
| 更频繁地口头纠正自己此前的说法 | `Only correct an earlier statement when the error would change the user's code, conclusions, or decisions. State corrections plainly and briefly, then continue the task. For slips that change nothing for the user, make the fix and move on without noting it.` |
| 代码审查中"只报高危"会被字面执行 | 用 4.16；把过滤动作移到独立的下游步骤 |
| `low` / `medium` effort 质量强，而 token 与延迟只是高档位的一个零头 | **从默认 `high` 起步**，只有足够重的编码与 agent 工作才上 `xhigh`。评测显示质量不掉的地方放开用 `low`/`medium`，把它当成控成本和控延迟的主要手段。设置若从更早的模型沿用而来，重新做一次 effort 扫描 |
| 关闭 thinking 时可能把 tool call 写成文本（该调用不会执行，且泄漏文本会留在历史里污染后续轮次） | **首选保持 thinking 开启**，用低 effort 控成本。官方说法：多数任务上，开着 thinking 跑 `low` effort，效果好于在相近成本下关掉 thinking |
| 关闭 thinking 时可能把 `<thinking>` 等内部标签泄漏到可见输出 | 删掉 system prompt 里"不要思考/不要推理"的规则（它会加剧泄漏）；点名 thinking 标签的效果差于通用形式 |

必须关闭 thinking 的集成，官方给的是**一条同时兜住这两个问题的合并指令**：

```
When you use a tool, you may say a brief sentence first. If no tool can express what the user asked for, say so instead of guessing. Do not include internal or system XML tags in your response.
```

其他：1M token 上下文为默认且最大；`xhigh`/`max` 下不能关闭 thinking——在这两档设 `thinking: {"type": "disabled"}` 会返回 400。多数任务上，**开着 thinking 跑 `low` effort，效果好于在相近成本下关掉 thinking**。**在 `xhigh` 或 `max` effort 下**运行时要给 `max_tokens` 留足空间，官方建议以 64k 为合理起点。此项仅在直接调 API 时需要关心。

### Claude Fable 5.1 / Mythos 5.1（默认 effort: high；thinking 始终开启且不可配置）

thinking 始终开启，adaptive thinking 是唯一模式。Claude Mythos 5.1 是同一个模型，只面向 Project Glasswing 参与者提供，规格、价格和 prompting 指引相同；官方写明的差别里，唯一影响 prompt 或 harness 代码的是 Mythos 5.1 不执行 thinking 块的「历史只追加」检查（另有可用性上的差别）。2026 年 9 月 1 日发布；Fable 5 现已列为仍可用的旧模型（legacy）。在 Claude Code 里，`claude-fable-5-1` 从 2.1.257（2026-09-01）起是默认的 Fable 模型。

定位：官方称其为 **Anthropic 目前最强的公开发布模型**（原文 `Anthropic's most capable widely released model`）。官方选型建议是另一半：**多数负载从 Claude Opus 5 起步；需要高强度推理和长周期 agentic 工作、或者 Opus 5 在更高 effort 下的评测仍不达标时，用 Claude Fable 5.1。** 官方选型矩阵列出的用例是：连续跑数小时的 agent 会话、多步深度研究、从分析一路做到成品文档 / 表格 / 幻灯片。「最强」和「默认该选哪个」不是同一个问题。它相对 Fable 5 的提升在高 effort 下最大，集中在长会话 agentic 编码、文档 / 表格 / 幻灯片工作、多步研究、视觉、长上下文和 computer use。

**Fable 5 的指引怎么沿用。** Fable 5.1 的 prompting 专页开头原文：`Your existing Claude Fable 5 prompts should perform well on Claude Fable 5.1 without changes, but a handful of behavioral differences are worth knowing about.` 通用最佳实践页另加了一条适用于所有模型专属技巧的提醒：`Where a technique names a specific model, treat it as measured on that model and re-check it against your own evals before applying it to another.` 所以本文件把两者分开：第一张表是 Fable 5.1 页面明确写了的；第二张表是只在 Fable 5 页面写了、Fable 5.1 页面既没有重述也没有撤回的。

#### Fable 5.1 页面明确写了的

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
| 让它复述内部推理会触发 `reasoning_extraction` 拒绝——官方对 Fable 5.1 和 Fable 5 都写了这一条 | 见 3.9 |
| 安全分类器覆盖的类别与 Fable 5 相同（`cyber`、`bio`、`frontier_llm`、`reasoning_extraction`、`general_harms`），比 Opus 5 只有网络安全一类的分类器更宽。良性工作也可能触发；在源码里找漏洞是允许的；误报比 Fable 5 发布时少 | 处理 `stop_reason: "refusal"` 并配置回落（允许的回落目标：Opus 4.8 和 Opus 5）。减少误报：问 "Are there any bugs in this program?" 而不是 "Does this program compile without errors?"；给小众语言补上下文；别让 base64 数据进工具输出 |
| 强制工具调用（`tool_choice` 的 `any` / `tool`）返回 400 | 见 3.12 |
| **子智能体在跑时主 agent 闲着**（官方症状原句 "Lead agent idles while subagents run"） | 让它继续干活：启动子智能体的工具立即返回，结果之后以 `user` 消息回传，另给一个「想等的时候用来等」的工具。官方测得平均完成时间下降，质量、token 用量和成本相近；模型仍然经常选择等 |
| 密集图表和图像 | 给它裁剪放大工具（或一个装有原图和 PIL / OpenCV 的容器），让它可以反复分析、裁剪、核对——单独一个裁剪工具就能拿到大部分提升 |

Fable 5.1 页面写明的 harness 层规则（仅 API；见第 9 节）：对话历史只追加不修改，因为改动历史轮次会让它的 thinking 块失效（Mythos 5.1 不执行这项检查）；会话中途改 effort 用按消息设置而不是改顶层值；即使在 Fable 5 上做过 effort 扫描，也要重新做一遍。

#### 只在 Fable 5 页面写了、Fable 5.1 页面没有重述的

下表每条处方都是 Fable 5 页面的官方原文，Fable 5.1 页面写明这些 prompt 可以沿用；但*倾向本身*在 Fable 5.1 上是否仍然存在，官方两边都没有说。把它们当作在 Fable 5 上测得的，在自己的评测上复核。

| 已知倾向（Fable 5） | 处方 |
|---|---|
| **单次请求可能跑很多分钟，自主运行可达数小时** | 迁移前先调整客户端超时、流式和进度提示；考虑改成异步轮询而非阻塞等待。Fable 5.1 页面从侧面印证了这个量级（"quiet for minutes at a time"；相对延迟 "Slower"），但没有重述这句话 |
| 任务模糊时会过度规划 | `When you have enough information to act, act. Do not re-derive facts already established in the conversation, re-litigate a decision the user has already made, or narrate options you will not pursue in user-facing messages. If you are weighing a choice, give a recommendation, not an exhaustive survey. This does not apply to thinking blocks.` |
| 高 effort 下会做未被要求的整理 / 重构 | 用 4.6 的 **Fable 5 变体**（同一条目里的 Fable 5.1 块是相邻倾向——擅自修 bug、多交测试文件——在 5.1 上测得的处方） |
| **指令遵循能力强到一句话就能管住一类行为**，不必逐条枚举 | 用简短指令代替长清单（3.3） |
| 长自主运行中可能编造进度 | 4.17——「几乎消除了编造的进度汇报」是在 Fable 5 上测得的 |
| 会做未被要求的动作（自动起草邮件、建防御性 git 分支） | 4.24 独立版；在 Fable 5.1 上它的前后两半已并入 4.18 |
| 更主动派并行子智能体 | **鼓励使用**，但给明确的委派判据；优先异步通信而非阻塞等待；长生命周期子智能体能靠缓存读取省时省钱（Fable 5.1 页面关于这件事的 harness 建议在上一张表） |
| 看到剩余 token 倒计时会提议开新会话 | 4.19；更好的做法是别把倒计时暴露给模型（4.18 的 Fable 5.1 自主运行块另加了一句 `Do not stop because the context or session is long.`） |
| **为旧模型写的 skills 往往过于规定化，可能降低输出质量** | 审查并删减；Fable 5 也能根据任务自行更新 skill |

**脚手架建议（Fable 5 页面，Fable 5.1 页面未重述）**：从你难度范围的**上限**开始试（挑一个比你会派给旧模型更难的任务），让它先界定范围、提澄清问题、再执行。

### ⚠️ 验证指令：Opus 5 与 Fable 5.1 方向相反

这是最容易搞错的一处，两条官方指引看似矛盾，实则场景不同：

| 场景 | 做法 | 依据 |
|---|---|---|
| **Opus 5，常规任务** | **删除**所有验证/复查指令 | 它自己会验证；保留会造成过度验证 |
| **Fable 5.1，长时自主运行** | **显式写入**周期性自我检查 | 官方原文发布在 Fable 5 页面，Fable 5.1 页面既没有重述也没有撤回（它写明 Fable 5 prompt 可以原样沿用）：`Establish a method for checking your own work at an interval of [X] as you build. Run this every [X interval], verifying your work with subagents against the specification.` |

关键区别：官方（Fable 5 页面同一段）同时指出，**独立的、干净上下文的 verifier 子智能体，效果优于模型自我批评**。所以"删验证"删的是**让模型自己再看一遍**，"加验证"加的是**换一个上下文来查**。两者不冲突。

（注意这与 4.10「不要用子智能体验证自己的工作」也不冲突。4.10 要按同一条「上下文是否干净」的轴来读：它禁的是**把自我复查外包出去**——把模型自己的产出连同它自己的判断标准一起交给子智能体，等于为 Opus 5 本来就会做的判断付两遍钱。**换到干净上下文、带独立判据的 verifier**，在两个模型上都是允许的。Fable 系列的长跑加的不是许可，而是**周期性**：把这个检查按间隔反复跑，而不是只在最后跑一次。）

### effort 取值参考

`max` > `xhigh` > `high`（默认）> `medium` > `low`

**两个模型都从默认 `high` 起步，区别在什么时候上调：**

- **Opus 5**：**足够重**的编码与 agent 工作上 `xhigh`；值得不设上限烧 token 的任务上 `max`。评测显示质量不掉的地方，放开用 `low`/`medium`，把它当成控制 token 成本和响应时间的主要手段
- **Fable 5.1**：最吃能力的 agentic 与编码工作上 `xhigh` 或 `max`；例行或延迟敏感的工作，在评测显示质量不掉之后降到 `medium`/`low`。官方说它相对 Fable 5 的提升在各档 effort 上都有、高档位最大；`medium` 下大致追平 Fable 5 且成本更低，`low` 下在单任务成本上常能和 Opus、Sonnet 系列竞争而分数更高——凡是你原本会「拿小模型跑高 effort」的地方，都把它纳入比较

官方把 `xhigh` 定位为面向长周期工作的扩展能力，适用范围写的是**超过 30 分钟、token 预算以百万计**的长周期 agentic 与编码任务——这就是「足够重」的具体标准。`low` 被定位为最省的一档，用于需要最快速度与最低成本的简单任务，官方点名的例子是子智能体。

**如果 effort 设置是从更早的模型沿用过来的，要在自己的评测上重新做一次 effort 扫描，而不是直接照搬。** effort 页对 Opus 5 写了这一条；Fable 5.1 专页写得更狠：即使在 Fable 5 上做过扫描也要重做，因为各档 effort 的名字在不同模型上对应的思考量并不相同。

官方给出的两条先后顺序：

- **调 effort 往往比换模型更有效**（官方原文 `Tuning effort is often a better lever than switching models`）：先在手上这个模型上把档位跑一遍，再考虑换模型
- 如果模型思考的频率超出任务需要，**先降 effort 档位，再考虑用 prompt 去引导**。prompt 措辞是第二顺位的杠杆，不是第一顺位，而且效果对具体措辞很敏感

要点：
- `effort: "high"` 与完全不传该参数**行为完全相同**
- effort 是**行为信号而非硬预算**，影响正文、工具调用和思考的全部 token
- **低 effort 会直接减少工具调用次数**、合并操作、省去前言；高 effort 则更多工具调用、先讲计划、给详细总结
- 在请求之间改**顶层** effort 会让 prompt cache 失效。Fable 5.1、Mythos 5.1 和 Opus 5 支持**按消息改 effort**（beta，header `mid-conversation-output-config-2026-07-01`：一条 `content` 为空、带 `output_config.effort` 的 `role: "system"` 消息），从下一条 user 消息起生效且缓存不失效。在 Fable 5.1 上官方还明确更推荐这种形式，因为改顶层值对模型的引导不够可靠——它之前的回复是在旧档位下写的，会倾向于保持一致。Fable 5 不支持（返回 400）
- 要给 `max_tokens` 留足空间——它是思考加正文的**总**上限。Opus 5 在 `xhigh`/`max` 下要注意（官方建议 64k 起）；Fable 5.1 的页面则从 `high` 起就点了这件事。此项仅在直接调 API 时需要关心

---

## 7. 场景 A：写给 Claude Code 执行的任务 prompt

用户在 Claude desktop 里说需求，你产出一段 prompt，他复制到 Claude Code（含 VS Code 插件的图形窗口）里执行。

### 本节目录

- 第一优先：给一个它自己能跑的检查
- 这个检查「卡」得有多硬
- 如果这个检查还不存在怎么办
- 一次说清楚，不要指望多轮补充
- 具体到什么程度
- 只有用户能提供的值怎么处理
- 探索、规划、实现、提交
- 大特性：让它先面试用户生成 SPEC
- 上下文是第一约束
- 这次的 prompt vs 该沉淀进项目配置
- 项目配置放哪：七种手段取舍表
- 把手工复查固化成验证回路
- 四种循环
- 五种常见失败模式

---

### 第一优先：给一个它自己能跑的检查

这是官方最佳实践里排第一的一条，也是这个场景和场景 B 最大的区别。

官方原文的第二人称说的是读者，在这里指的是**你的用户**而不是你，所以下面这段是改写而非原文：

> Claude 在"看起来做完了"的时候就会停。如果没有它能运行的检查，"看起来做完了"就是唯一的信号，而**用户自己就变成了验证回路**——每个错误都要等他去发现。给它一个能产出 pass/fail 的东西，这个回路就自己闭合了。

检查可以是：测试套件、构建的退出码、linter、把输出和 fixture 做 diff 的脚本、和设计稿对比的浏览器截图。

**写 prompt 时的对照**：

| 差 | 好 |
|---|---|
| `implement a function that validates email addresses` | `write a validateEmail function. example test cases: user@example.com is true, invalid is false, user@.com is false. run the tests after implementing` |
| `make the dashboard look better` | `[paste screenshot] implement this design. take a screenshot of the result and compare it to the original. list differences and fix them` |
| `the build is failing` | `the build fails with this error: [paste error]. fix it and verify the build succeeds. address the root cause, don't suppress the error` |

> **注意与 3.4 的区别**：3.4 说的是删掉"让模型自己再看一遍"这类主观复查指令。这里给的是**外部确定性检查**——跑测试、看退出码、对比截图。前者要删，后者必须给。

还要求它**给出证据**而不是断言成功：测试输出、跑了什么命令返回了什么、结果截图。审阅证据比自己重跑一遍快，而且对用户没盯着的会话同样有效。

### 这个检查「卡」得有多硬

检查存在之后还有第二个决定，而这个决定光靠 prompt 做不到。官方给了四档，用配置成本换「这趟运行需要多少人盯着」：

| 档位 | 是什么 | 什么时候值得配 |
|---|---|---|
| **写在一条 prompt 里** | 在同一条消息里要求它跑检查并迭代 | 任何任务，立刻可用。你正在写的这条 prompt 就是这一档 |
| **贯穿整个会话** | 把检查设成 `/goal` 条件——每轮结束后由独立评估器复查 | 用户希望它一直跑到标准达成为止 |
| **确定性闸门** | `Stop` hook 把检查当脚本跑，不通过就不许结束这一轮（连续挡 8 次后 Claude Code 会强制放行） | 这个检查不能被跳过 |
| **换个人评** | 验证子智能体——或者一个会复核自己发现的 dynamic workflow——让干活的和打分的不是同一个。只要查正确性，Claude Code 自带的 `/code-review` skill 会在干净的子智能体里审查当前 diff | 正确性比速度重要 |

**这一档位在交付说明里值得点一句。** 一条 prompt 天然只能做到第一档；如果用户说的是无人值守的长跑，就告诉他真正让它「没人看着也能正确收工」的是 `/goal` 或 `Stop` hook。

### 如果这个检查还不存在怎么办

上面那条规则的前提是"有一个可跑的检查"。但**最常见的 Claude Code 请求恰恰是没有的**——老代码、覆盖率低、出过事故、不敢动。这时候不要跳过验证直接改，而是**把建立检查本身作为第一阶段**：

1. **先写钉住现状的测试（characterization test）**：不管当前行为对不对，先把它**如实断言下来**，包括你认为是 bug 的行为。目的是建立一张"改动前后行为是否一致"的网，不是判断对错
2. **可疑行为照实钉住，另外标记**：发现疑似 bug 时，测试里仍按当前实际行为断言，同时加注释标明"这里可能是 bug，见 X"。**这一条必须在 prompt 里写明**——一个善意的读者被告知"写测试钉住当前行为"时，很自然会顺手把 bug 修掉，那样网就失效了
3. **网建好之后冻结**：后续重构阶段**不允许修改这些测试**。测试变了就说明行为变了，而重构的定义就是行为不变。确实需要改测试时，必须单独提出来说明理由
4. **然后才开始重构**，每一步都跑这张网

对应的 prompt 结构是分阶段并在阶段之间设检查点，而不是一句"重构这个模块"。行为变更（修 bug、改逻辑）应该是**重构完成之后的独立一轮**，不要和重构混在一起——混在一起时测试报红你分不清是重构错了还是行为改了。

---

### 一次说清楚，不要指望多轮补充

官方明确指出：**把任务、意图和相关约束在第一轮就说全，能最大化自主性和智能，同时把用户轮次之后的额外 token 消耗降到最低。** 反过来，模糊或欠说明的需求分多轮渐进给出，反而会降低 token 效率、有时还降低性能。

这条对场景 A 尤其重要，因为用户是**复制粘贴一次就走**，中途不一定在旁边。

---

### 具体到什么程度

| 策略 | 差 | 好 |
|---|---|---|
| **划定范围**：指明哪个文件、什么场景、测试偏好 | `add tests for foo.py` | `write a test for foo.py covering the edge case where the user is logged out. avoid mocks.` |
| **指向来源**：告诉它去哪儿找答案 | `why does ExecutionFactory have such a weird api?` | `look through ExecutionFactory's git history and summarize how its api came to be` |
| **引用现有模式**：指出代码库里的样板 | `add a calendar widget` | `look at how existing widgets are implemented on the home page to understand the patterns. HotDogWidget.php is a good example. follow the pattern to implement a new calendar widget...` |
| **描述症状**：症状加可能位置加"修好"长什么样 | `fix the login bug` | `users report that login fails after session timeout. check the auth flow in src/auth/, especially token refresh. write a failing test that reproduces the issue, then fix it` |

模糊 prompt 也有用武之地——探索阶段、可以承受走偏时。`what would you improve in this file?` 能问出你想不到要问的东西。但那是探索，不是执行。

**提供丰富输入的几种方式**（写进 prompt 时告诉用户可以这么用）：`@` 引用文件、直接粘贴截图、给文档 URL、`cat error.log | claude` 管道输入、或者让 Claude 自己去取（bash、MCP 工具、读文件）。

### 只有用户能提供的值怎么处理

场景 A 里总有一些东西**既推断不出、也没法提前问**：仓库路径、测试命令、那两次线上事故到底是什么、内部系统的约定。这些往往还是整份 prompt 里价值最高的输入。

做法是在 prompt 正文里留**显式占位**，并给出留空时的兜底：

```
两次线上事故的具体表现：【填：如果记得就写在这里】
如果上面留空，先从 git log 和 issue 里检索相关修复记录，把你推断出的事故场景列出来让我确认，再继续。
```

两个要点：占位要**长得像占位**（用【】或类似标记，让用户一眼扫到），以及**必须给兜底指令**——只留空不给兜底，模型会自己编一个或者直接忽略。填了更好，没填也能跑。

---

### 探索、规划、实现、提交

任务范围不明或改动跨多文件时，prompt 应当引导走这四步，而不是直接开写：

1. **探索**（plan mode）：只读不改，先理解现状
2. **规划**：产出详细实施计划
3. **实现**：按计划写，并对照计划验证
4. **提交**：描述性 commit message，开 PR

**什么时候跳过**：范围清楚且改动很小时（改错字、加一行日志、重命名变量）直接做。官方判据很实用——**如果你能用一句话描述这个 diff，就跳过规划。**

**"对照计划验证"算不算 3.4 要删的自我验证？** 取决于这份计划有没有经过外部确认：

- 模型自己写完计划、没人看过就自己对照着检查 → **算自我验证**，价值有限
- 计划**经用户看过并确认**（或写进了 SPEC.md 这类文件、成为约定） → **算外部依据**，对照它验证是有效的

所以四阶段流程里的"规划"之后应当有一个确认动作。写 prompt 时把这个检查点显式写出来：让它出完计划**停下来等确认**，再进入实现。用户不在场、无法确认时，就不要写"对照计划验证"，改为依赖测试等真正的外部检查。

规划最有价值的时候：你不确定该怎么做、改动跨多个文件、或者你对要改的代码不熟。

---

### 大特性：让它先面试用户生成 SPEC

需求较大且用户自己也没完全想清楚时，产出的 prompt 可以是这个：

```text
I want to build [brief description]. Interview me in detail using the AskUserQuestion tool.

Ask about technical implementation, UI/UX, edge cases, concerns, and tradeoffs. Don't ask obvious questions, dig into the hard parts I might not have considered.

Keep interviewing until we've covered everything, then write a complete spec to SPEC.md.
```

然后**开一个全新会话**执行这份 spec——新会话上下文干净、完全聚焦实现。

官方的判断值得转告用户：**把 spec 打磨精确所花的时间，回报高于盯着实现过程看的时间。** 最有用的 spec 是自包含的：点名涉及的文件和接口、写明什么不在范围内、并以一个端到端验证步骤收尾。

---

### 上下文是第一约束

Claude Code 的大部分最佳实践都源自同一个约束：**上下文窗口填得快，而且填满后性能下降。** 一次调试或代码库探索可能就消耗几万 token。

写 prompt 时能做的：

- **把探索隔离出去**：`Use subagents to investigate how our authentication system handles token refresh, and whether we have any existing OAuth utilities I should reuse.` 子智能体在独立上下文里读一堆文件，只回报摘要

> **子智能体的统一判据**（这条散落在几处，这里说全）：**大范围只读探索要鼓励用，让模型在自己的上下文里重看一遍自己的产出不要用。** 换到**干净上下文**的独立 verifier 子智能体是例外，官方指出它优于自我批评——上文「换个人评」那一档、以及本节末尾对抗式审查那段说的都是这一种。
>
> 所以：前者是它最大的价值——把读几十个文件的开销挡在主上下文外面；后者是 Opus 5 已经会做的事，派子智能体去做只是加倍成本。介于两者之间的判断标准是片段 4.10：几次工具调用就能自己做完的，不要派。

- **不要写成开放式调研**：`investigate X` 不划范围会让它读几百个文件把上下文填满
- 提醒用户：不相关的任务之间用 `/clear`；同一个问题纠正两次还不对，就 `/clear` 重来并把学到的写进新 prompt

---

### 这次的 prompt vs 该沉淀进项目配置

判断很简单：**这次任务特有的信息写进 prompt；每次都成立的约定沉淀进项目配置。**

如果你发现自己在 prompt 里写"我们项目用 pnpm 不用 npm"、"提交前要跑 lint"这类**每次都要说一遍**的话，就该建议用户把它放进 CLAUDE.md 或 hook，而不是每次复制粘贴。

---

### 项目配置放哪：七种手段取舍表

| 手段 | 加载时机 | 上下文成本 | 适合放什么 |
|---|---|---|---|
| **CLAUDE.md（根）** | 会话开始，全程驻留，compaction 后重新读入 | **高**（始终加载） | 构建命令、目录结构、monorepo 布局、团队约定。**建议 <200 行**，且要有人负责审查变更 |
| **CLAUDE.md（子目录）** | 触及该目录时按需加载 | 低 | 目录级约定 |
| **Rules** | 会话开始（无 scope）或访问文件时（有 scope） | 中 | 具体约束。**加 `paths:` frontmatter 做路径限定**以降低成本 |
| **Skills** | 名称加描述在开头，正文在调用时 | 低 | 流程性工作流：部署清单、发版流程、审查流程 |
| **Subagents** | 名称加描述在开头，正文调用时加载，只返回最终消息 | 低（隔离上下文） | 只需要结论的侧任务：日志分析、依赖审计、深度搜索 |
| **Hooks** | 生命周期事件触发，绕过 compaction | 低（在主上下文之外） | **确定性**自动化：lint、发通知、阻断特定命令 |
| **Output styles** | 会话开始，永不压缩 | **高** | 只在需要大幅改变 Claude 角色时用 |

**反模式**：

- ✗「每次 X 都要 Y」写进 CLAUDE.md → 改用 hook。**advisory（建议性）的指令永远不如 deterministic（确定性）的执行**
- ✗「绝不允许…」的约束 → 改用 `PreToolUse` hook 或权限配置
- ✗ 30 行的操作流程写进 CLAUDE.md → 移到 skill
- ✗ 只跟 API 相关的规则却不限定路径 → 加 `paths:` frontmatter
- ✗ 个人偏好写进项目共享文件 → 用用户级本地配置

**CLAUDE.md 写什么**：模型猜不到的 bash 命令、与默认不同的代码风格、测试方式、仓库礼仪（分支命名、PR 约定）、项目特有的架构决策、环境怪癖、非显而易见的坑。

**不写什么**：读代码就知道的、语言通用惯例、详细 API 文档（给链接）、频繁变动的信息、逐文件说明、"写干净代码"这类空话。

**自检**：对每一行问「删掉它会不会让 Claude 犯错？」不会就删。
**故障信号**：反复叮嘱某条规则仍不生效，通常是**文件太长导致规则被淹没**，而不是措辞不够强。
**工具**：`/doctor` 命令可以帮你精简 skills 和 CLAUDE.md。

**Skills 怎么写**（三层渐进式披露）：

| 层 | 内容 | 何时加载 |
|---|---|---|
| 元数据 | YAML frontmatter，只有 `name` 和 `description` | 启动时预载入系统提示词 |
| 主指令 | `SKILL.md` 正文 | Claude 判断相关时 |
| 附加文件 | 从 SKILL.md 引用的其他文件 | 只在真正需要那个场景时 |

`name` 和 `description` 是激活的关键，没触发要先改这两句。正文臃肿就拆文件。有帮助时可以打包可执行代码：用 token 生成来排序列表，远比直接跑一个排序算法昂贵。**Fable 5 专页写明，为旧模型写的 skill 对它往往过于规定化，可能降低输出质量** —— 面向 Fable 系列时要定期删减。（Fable 5.1 专页没有重述这条；它写明现有 Fable 5 prompt 无需修改即可沿用，而通用最佳实践页说模型专属的结论要当作在那个模型上测得的。）

---

### 把手工复查固化成验证回路

判据：**任何用户不得不一遍遍手动执行的检查，都值得固化。** 尤其是通用 linter 抓不到的确定性规则，官方举的例子是 `reject any migration that drops a column without a backfill step`。

| 模式 | 触发 | 适合 |
|---|---|---|
| **Standalone 独立** | 手动调用 | 不普遍适用的横切检查：安全扫描、无障碍审计、许可证核查 |
| **Embedded 内嵌** | 在产出 skill 的流程里自动跑 | 只属于某个特定任务的检查 |
| **Chained 串联** | 一个 skill 完成后调下一个 | 端到端交接；也是给**不可修改的 skill** 加验证的唯一办法 |
| **On every PR** | CI 上跑 | 把个人习惯变成团队基础设施 |

**坑**：内嵌只对可编辑的 skill 有效。内置或插件管理的 skill 会自动更新，改动会被覆盖，那种情况必须用串联。

---

### 四种循环

| 类型 | 命令 | 适合 |
|---|---|---|
| **Turn-based** | 手动逐轮 | 短任务 |
| **Goal-based** | `/goal get the homepage Lighthouse score to 90 or above, stop after 5 tries` | 有**可验证退出标准**、单轮不够的任务 |
| **Time-based** | `/loop 5m check my PR, address review comments, and fix failing CI` | 周期性工作、对接外部系统 |
| **Proactive** | `/schedule every hour: check #project-feedback for bug reports` | 定义明确的持续工作流：bug 报告、issue 分诊、迁移 |

控成本：**确定性的工作用脚本做**（跑脚本比逐 token 推理便宜），周期匹配实际变化频率，大规模跑之前先试点。官方结论——**模型和 effort 档位属于循环成本上最大的几个杠杆。**

---

### 五种常见失败模式

写 prompt 时可以顺带提醒用户避开：

| 失败模式 | 症状 | 解法 |
|---|---|---|
| **大杂烩会话** | 一个任务问到一半岔到别的再岔回来 | 不相关任务之间 `/clear` |
| **反复纠正** | 纠正一次不对，再纠正还不对 | **两次失败就 `/clear`**，带着学到的写一个更好的初始 prompt |
| **过度规定的 CLAUDE.md** | 文件太长，重要规则被淹没 | 狠心修剪 |
| **信任但没验证** | 看起来合理但边界情况没处理 | 永远提供验证手段；**验证不了的东西不要发布** |
| **无边界探索** | "调研一下"结果读了几百个文件 | 收窄范围，或用 subagent 隔离 |

**对抗式审查的一个重要提醒**：让一个子智能体在干净上下文里审查 diff 很有效，但官方同时警告——**被要求找问题的审查者通常总能报出一些问题，即使工作本身没问题**，因为那是它被要求做的事。追着每条发现改会导致过度工程。所以审查 prompt 要写明：**只报影响正确性或既定需求的问题，其余视为可选。**这是一条过滤指令，只适用于**由你来做过滤**的场合。如果发现要交给下游的排序或分诊步骤，改用片段 4.16、让审查者全报——第 6 节提醒过：Opus 5 会字面执行你设的严重度门槛，低于门槛的发现会被直接不报。

---

## 8. 场景 B：写给 desktop chat 或 Cowork session 执行的任务 prompt

用户在 Claude desktop 里说需求，产出的 prompt 在 **desktop chat** 或 **Cowork session** 里直接跑。非代码任务：研究、分析、写作、整理资料、做文档表格幻灯片、日程、信息汇总。

### 本节目录

- 先分清 chat 还是 Cowork
- 输出长度必须写死
- 材料怎么摆
- 要格式稳定就给示例
- 研究类任务
- Cowork 特有：交付物与文件
- Cowork 特有：会自动接管的 skill
- 不要写进去的东西
- 一个可直接套用的骨架
- 可复用 / 周期性 prompt 的变量块
- 本节信源说明

---

### 先分清 chat 还是 Cowork

两者定位不同，prompt 写法跟着变。以下依据官方对桌面端各模式的说明（第三种 Claude Code 属于场景 A，见第 7 节）。

| | desktop chat | Cowork session |
|---|---|---|
| **定位** | 快速来回、迭代式工作：提问、头脑风暴、起草、把问题聊清楚 | 需要真功夫的活：从多个来源取材、消化、产出成品 |
| **本机文件** | 靠桌面连接器接本地工具；**不能**系统性地访问多个来源 | **直接读写已连接文件夹里的本地文件**，无需手动上传下载 |
| **产出** | 对话里的内容 | **成品交付物**：带可用公式的 Excel、PowerPoint、格式化文档 |
| **持续时长** | 不适合长时间持续任务 | 可长时间自主运行，用户可以离开；支持**定时重复任务** |
| **并行** | 无 | **子智能体协作**，复杂工作拆成并行工作流 |
| **扩展** | 连接器 | 连接器、skills、plugins、Claude in Chrome |

**一条重要边界**：Cowork **不能**像 Claude Code 那样访问完整文件系统或运行终端命令——它在受保护的环境里、只对已连接的文件夹工作。所以涉及仓库级操作、git、构建部署的任务不要走场景 B，那是场景 A。

**判断方法**：交付物是"一段可以直接读的内容"→ chat 就够；交付物是"一个要保存下来的文件"、或需要跨多份材料系统性取材 → Cowork。

如果用户没说，而任务明显需要产出文件或处理批量材料，**默认按 Cowork 写，并在交付说明里点明这一点**。

**两个容易忽略的 Cowork 能力**，值得在 prompt 里主动用上：

- **定时重复任务**：如果用户的需求本质是"每周一给我一份 X"，prompt 应该写成可复用的任务描述，并提示他设成定时任务，而不是每次手动跑。
  **定时任务意味着无人值守**：跑的时候用户不在，任何"要不要继续？"都会把整个任务卡死到下一周。所以周期性 prompt 必须写明**遇到问题怎么自己往下走**——素材缺失、文件打不开、来源冲突无法判定，一律记录下来继续做，最后在产出里单列"需要你确认"一节，而不是停下来问。（完整的无人值守片段见 4.18，但对周报这类轻量任务不必整段搬，写清这一条即可。）
  **写周期性交付物的 prompt 时还有一条必须加**：让它先读上一期的产出，然后**只写增量**——发生了什么变化、上期说要做的做了没有、哪些还挂着。不加这一条，每期都会变成从零开始的平铺罗列，读的人自己去比对差异。这对周报、站会纪要、状态更新、简报、变更日志都成立
- **子智能体并行**：材料很多时，prompt 里可以明确说"用子智能体并行处理各份材料，每个只回报结构化摘要"——这既快又省主上下文

---

### 输出长度必须写死

这是场景 B 最高频的问题。Opus 5 的用户可见回复**默认就比前代长**，而且降 effort 不能可靠缩短（effort 控的是想多少，不是说多少）。

所以任何面向阅读的产出，prompt 里都要有明确的长度约束。**具体怎么选、哪些能叠加，按第 12 节「长度控制怎么选」的规则**，简言之：

- 主控手段二选一——**能给数字就给数字**（"300–400 字"、"3–6 条，每条不超过 40 字"，不要写"简洁一些"），给不出数字才用片段 4.2
- 产出物写成文件时**追加**片段 4.4（数字通常只约束正文，管不住模型自行添加的填充章节）
- system prompt 很长时，末尾**追加** `<tone_preference>` 短提醒

---

### 材料怎么摆

用户要 Claude 处理一批资料（文档、邮件、聊天记录、网页内容）时：

**长材料放最前面，问题和指令放最后面。** 官方测试中这一条最多可提升 30% 的回答质量，在复杂多文档输入上尤其明显。

```xml
<documents>
  <document index="1">
    <source>2025年报.pdf</source>
    <document_content>
      {{材料内容}}
    </document_content>
  </document>
  <document index="2">
    <source>竞品分析_Q2.xlsx</source>
    <document_content>
      {{材料内容}}
    </document_content>
  </document>
</documents>

（问题和指令写在这里，最后）
```

**长文档还有一招 grounding in quotes**：先让它把相关引文摘进 `<quotes>` 标签，再基于引文回答。这能让它聚焦相关内容、忽略其余部分，对"从一堆材料里找依据"类任务效果明显。

---

### 要格式稳定就给示例

场景 B 里大量任务是"每次都按这个格式输出"——周报、会议纪要、复盘、简报、卡片。这正是第 5 节里**该给示例**的那一支。

- 3–5 个，用 `<example>` 包裹
- 要**多样**，覆盖边界情况，避免模型学到意外模式
- 最有价值的示例是**成对的**：原始素材 → 成品。这样教的是压缩比和取舍标准，不只是长什么样
- 如果用户手上有过去写好的真实样本，让他放两三个进去——这是单点回报最高的一步

**用户还没有样本时怎么办**（一次性交付、问不了用户的情况很常见）：**给一个你自己写的成对示例就够，不要硬凑 3–5 个**。理由是——凭空编出来的多个示例会把一套你臆想的"标准格式"当成规范教给模型，用户之后很难察觉哪里不对。正确做法是给一个示例说明**压缩比和取舍标准**，然后在 prompt 里留出明确位置，并在交付说明里告诉用户：**把你真实的两三期换进去，这是提升最大的一步**。

反过来，如果任务是开放式创作、头脑风暴、或者你希望它自己判断切入角度，**不要给示例**，示例会把它的探索空间收窄到示例附近。

---

### 研究类任务

需要跨多个来源查证、综合的任务，官方给了一段现成的结构化指令。**照抄，包括里面那句自我批评在内**——理由见下方的例外说明：

```
Search for this information in a structured way. As you gather data, develop several competing hypotheses. Track your confidence levels in your progress notes to improve calibration. Regularly self-critique your approach and plan. Update a hypothesis tree or research notes file to persist information and provide transparency. Break down this complex research task systematically.
```

> **这是 3.4 的一处例外。** 这段里有 `Regularly self-critique your approach and plan`，而 3.4 说 Opus 5 上要删掉自我验证类指令。这和片段 4.7 里那句 `you MUST read the file` 属于同一类例外：**它是调校过的官方原文，照抄即可**。两者说的不是一回事——3.4 删的是*答案写完后再自审一遍*；这句是*跑的过程中随时纠偏*，证据还在陆续进来的时候调整方向。如果任务是一次性写一篇东西、而不是多步检索，那这句没有可纠的对象，可以只删掉这一行。

另外三条：**给清晰的成功标准**（什么样的答案算回答了这个问题）、**要求跨来源交叉验证**、**显式允许说不知道**：

```
If the data is insufficient to draw conclusions, say so rather than speculating.
```

最后一条对研究任务特别重要——不给退出通道，模型会倾向于把不确定的东西说得很肯定。

---

### Cowork 特有：交付物与文件

Cowork 能产出真实文件。写 prompt 时**明确说出要什么文件类型**，因为这直接决定它走哪条路径：

| 交付物 | 在 prompt 里怎么说 |
|---|---|
| 可编辑文档 | "输出为 .docx" |
| 数据表 | "输出为 .xlsx，第一行表头，每列……" |
| 演示 | "做成 .pptx，X 页，每页……" |
| 可分享的页面 / 看板 / 追踪表 | "做成单文件 HTML" |
| 纯文字给我直接看 | 明说"直接在对话里给我，不要生成文件" |

**要不要落到本机**：如果用户要文件保存到自己电脑某个文件夹，prompt 里要写清楚**目标文件夹**，否则默认只在对话里交付。

**批量处理**：涉及多个文件时，写明处理顺序和命名规则，以及**处理不了的怎么办**（跳过并记录，还是停下来问）。不写这条，遇到坏文件时行为不可预期。

---

### Cowork 特有：会自动接管的 skill

Cowork 里装了一批输出格式 skill（docx、xlsx、pptx、pdf 等）。它们会在涉及对应文件类型时自动介入。这意味着：

- **不需要在 prompt 里教它怎么生成 docx**，那是 skill 的事，写了反而是冗余（3.3）
- prompt 该说的是**内容要求**：结构、口径、数据来源、长度
- 但**要说清楚样式偏好**，比如"用公司模板"、"不要用默认配色"——这些 skill 猜不到

有一个顺序值得写进 prompt：**先把事实查全、再动手做文件**。官方对输出格式类 skill 的指引是——研究完成之前不要开始搞文档机制，否则会过早锚定在排版上而手里还没有正确的内容。

---

### 不要写进去的东西

场景 B 里最容易多写的三类：

1. **自我验证指令**（"最后检查一遍"、"再核对一次"）——Opus 5 自己会做，写了造成过度验证。见 3.4。注意 3.4 划的那条界线：**跨来源交叉核对是任务本身，不是自我验证**——研究类 prompt 里不要连它一起删掉
2. **人设铺垫**（"你是一位有二十年经验的资深分析师"）——对现行模型没有增益，纯占 token。要影响视角就直接说关注什么——官方例子是 `Analyze this investment portfolio, focusing on risk tolerance and long-term growth potential`，而不是给它安一个角色
3. **一长串"不要"**——改成正面描述加原因。见 3.5

---

### 一个可直接套用的骨架

```
（如果有长材料，放最前面）
<documents>
  <document index="1">
    <source>……</source>
    <document_content>……</document_content>
  </document>
</documents>

<variables_this_cycle>   （只有周期性模板才需要，见最后一节）
时间范围：……
素材位置：……
</variables_this_cycle>

<task>
一句话说清要做什么，以及这是为谁做的、他拿到之后要用来干什么。
</task>

<requirements>
1. ……
2. ……
（正面表述，每条带上为什么）
</requirements>

<output>
形态：（对话内容 / .docx / .xlsx / .pptx / 单文件 HTML）
长度：（具体数字）
结构：（章节或字段）
</output>

<examples>
  <example>……</example>
</examples>
（仅在需要锁定格式时加）
```

最后加一句**成功标准**——什么样的产出算做对了。没有这一条，你没法自检，用户也没法判断要不要返工。

### 可复用 / 周期性 prompt 的变量块

用户说"我想固定下来每周用"时，prompt 要写成模板而不是一次性指令。加一个变量块，把每期会变的东西集中在一处，用户只改这一块：

```
<variables_this_cycle>
时间范围：2026-07-20 ~ 2026-07-26
素材位置：（文件夹路径）
上期产出：（上一期文件路径，用于对比增量）
</variables_this_cycle>
```

**放在 `<task>` 正前面**——也就是材料之后，就是上面骨架里的那个位置。这样"长材料在前、指令在后"的摆位规则不受影响，同时用户每期要改的那几行就紧挨着它所参数化的那条指令。

---

### 本节信源说明

本节中关于 chat 与 Cowork 能力边界的描述，依据 Anthropic 官方的 Cowork 概览与桌面端模式对比说明；prompt 技法部分依据官方 prompting best practices。涉及产品能力的具体细节可能随版本变化，判断不准时以用户实际看到的界面为准。

---

## 9. Agent 与长任务：上下文工程

> 片段编号（如 4.17）指向第 4 节。
>
> 本节中的英文块均为 Anthropic 官方长任务指引**原文**（见仓库里的 SOURCES.md），照抄即可，不要翻译或改写。

### 本节目录

- 上下文是第一约束
- 跨上下文窗口的工作流
- 状态管理
- 上下文压缩感知
- 复杂研究任务
- 让 Claude 自己编排，而不是把一切塞进上下文
- 设计上下文以最大化缓存命中
- 什么时候该把 bash 动作提升为专用工具
- 工具设计优先于提示词

### 上下文是第一约束
性能随上下文填充而下降（context rot）。目标是**能最大化达成目标概率的最小高信噪比 token 集**。

### 跨上下文窗口的工作流
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

### 状态管理
- 结构化数据（测试结果、任务状态）→ JSON
- 进度笔记 → 自由文本
- 版本与检查点 → git（现行模型用得很好）
- 明确要求增量推进

### 上下文压缩感知
```
Your context window will be automatically compacted as it approaches its limit, allowing you to continue working indefinitely from where you left off. Therefore, do not stop tasks early due to token budget concerns. As you approach your token budget limit, save your current progress and state to memory before the context window refreshes. Always be as persistent and autonomous as possible and complete tasks fully, even if the end of your budget is approaching. Never artificially stop any task early regardless of the context remaining.
```

**如果压缩是在客户端做的**（Fable 5.1 专页）：服务端压缩已经会告诉模型该保留什么；自己做摘要那一步，官方给的指令在下面。在 Fable 5.1 上最简单的形式是：用一条摘要消息加新的 user 消息替换整个历史，其他什么都不回放——没有 thinking 块被带过去，就不会撞上「历史只追加不修改」的检查。另外 Fable 5.1 的缓存读取更便宜，为省钱而提早压缩可能已经不划算，试试更晚的压缩点。

```
Summarize the transcript inside <summary></summary> tags. Include relevant information in the summary such that this conversation will be continued by a new context window without needing to redo work or be reprovided with relevant constraints or context. Be sure to preserve: (1) any difficulties or problems that came up, and how they were handled or resolved; (2) any possibilities, options, or approaches that were raised, tried, or set aside, and why; (3) anything that was asked for, decided, agreed, ruled out, or established as a preference, constraint, or boundary — stated exactly; (4) exactly where things stand now — what has been covered, settled, or completed so far; (5) anything still open, unresolved, promised, or expected to happen next; (6) specific details that would be hard to reconstruct — names, numbers, dates, exact wording, links or references — kept exactly. Be complete on these even at the cost of length; keep everything else concise. Weight the two voices differently: keep what the user said, asked for, shared, or established carefully and close to their own words; your own explanations and reasoning can be condensed much further, to what they concluded or produced — as long as nothing in the six items above is dropped.
```

### 复杂研究任务
```
Search for this information in a structured way. As you gather data, develop several competing hypotheses. Track your confidence levels in your progress notes to improve calibration. Regularly self-critique your approach and plan. Update a hypothesis tree or research notes file to persist information and provide transparency. Break down this complex research task systematically.
```

### 让 Claude 自己编排，而不是把一切塞进上下文

这是 harness 层面收益最大的一条。**不要把每个工具结果都当 token 塞回上下文窗口**，也不要把过滤逻辑硬编码进工具设计。

给 Claude 代码执行能力（bash、语言 REPL），让它自己写逻辑来表达工具调用、过滤输出——**只有执行结果进入它的注意力**。官方数据：在 BrowseComp 网页浏览任务上，给模型输出过滤能力使准确率从 **45.3% 提升到 61.6%**。

同一思路的另外两条：
- **让 Claude 自己管上下文**：用 skills 做渐进式披露（frontmatter 常驻、正文按需），用 context editing 清掉过期的工具结果，用子智能体隔离上下文（BrowseComp +2.8%）
- **让 Claude 自己持久化上下文**：compaction + memory 文件夹（agentic search 任务 60.4% → 67.2%）

> 出处说明：以上三组数字来自 Anthropic 在**更早一代模型**上的实测。文章本身的论点是这个趋势会随模型变强而延续——模型越强，越该把决策交还给它。

### 设计上下文以最大化缓存命中

**缓存 token 的成本只有基础输入 token 的 10%**（Fable 5.1 和 Mythos 5.1 上是 2.5%），所以这是纯粹的成本杠杆。五条原则：

| 做法 | 要点 |
|---|---|
| **静态在前** | 稳定内容（系统提示词、工具定义）放在动态内容之前 |
| **用消息追加而非改提示词** | 需要注入提醒时，追加到 messages 里，不要去改 system prompt |
| **不要中途换模型** | 缓存是按模型隔离的 |
| **工具增删会作废缓存** | 需要动态发现工具时用 tool search，而不是增删工具定义 |
| **移动断点** | 多轮应用里把 cache breakpoint 移到最新消息 |

> 直接后果：**在请求之间改顶层 effort 会作废缓存**，同理（见第 2 节）。Fable 5.1、Mythos 5.1 和 Opus 5 支持按消息改 effort（beta），缓存不失效——见第 6 节。

**第二条的机制。** prompt caching 按 `tools` → `system` → `messages` 的顺序对请求前缀做哈希，命中要求前缀逐字节一致。所以往 system prompt 末尾追加一句话，作废的不只是 system prompt，还包括它之后所有已缓存的消息。如果某条指令是跑到中途才变得相关，就在 `messages` 里那个位置追加一条 `{"role": "system"}` 消息，而不是去改顶层 `system` 字段：缓存前缀保持不变，指令仍然以系统指令的身份生效，并且对其后的轮次优先于顶层字段。Fable 5.1、Mythos 5.1、Fable 5、Mythos 5、Opus 4.8、Opus 5 支持且不需要 beta header；**Sonnet 5 不支持**，那边只能用顶层 `system` 字段。`role: "system"` 消息还可以用 `clear_at: "next_user_message"` 限定只对本轮生效（beta，header `mid-conversation-system-clear-at-2026-08-21`）：当前轮里它有 system prompt 的权威，出现更新的 user 消息后就不再渲染，本身仍留在数组里所以前面什么都没变，清掉后也不计输入 token——这是官方给「工具循环里每轮一条提醒」安排的位置。

**历史只追加不修改（Fable 5.1）。** 每个 assistant 回合原样追加（含 thinking 块），请求之间不要改动历史轮次。在 Fable 5.1 上，thinking 块只在产生它的那个对话里有效：2026 年 8 月 31 日及之后创建的账号，一旦它前面的内容（system prompt、工具列表或任何更早的消息）变了再回放它，就返回 400；设了 `thinking.block_binding.prefix_mismatch_behavior: "drop_block"`（beta，`thinking-binding-controls-2026-08-01`）则改为丢弃该块；官方预告后续模型会对所有账号执行这项检查。会踩到它的改动和会重置缓存的改动是同一批——注入又删除每轮提醒、就地摘要旧轮次、会话中途改 system prompt——修法也一样：提醒用 turn-scoped system message，改指令和工具用会话中 system 消息和工具变更而不是重写 `system` 或 `tools`，裁剪上下文交给服务端压缩或 context editing。从最老的一端连续移除若干 thinking 块、移动 `cache_control` 标记、在请求之间改 `effort`，都不会让后面的块失效。

### 什么时候该把 bash 动作提升为专用工具

bash 杠杆大但缺乏可观测性，harness 无法拦截、把关或审计。以下四种情况应当做成专用的声明式工具：

1. **安全边界**：难以撤销的动作，需要用户确认（如调用外部 API）
2. **写操作需要陈旧性检查**：防止覆盖已被改动的文件
3. **需要面向用户呈现**：弹窗、多选项展示
4. **需要结构化日志、追踪和回放**

（低风险工作可以走另一条路：用第二个 Claude 实例判断 bash 命令是否安全，这是 Claude Code auto 模式的做法，可以减少专用工具的数量。）

### 工具设计优先于提示词
有工具的场景下，**改工具比改提示词更有效**：
- 每个工具功能互不重叠，模型能一眼分清何时调用哪个。判断法很直接——**如果工程师自己都说不清某个场景该用哪个工具，agent 更不可能分得清**
- 参数名语义明确（`user_id` 而非 `user`）——把无语义的 UUID 解析成可读名称能显著提升精度
- 按服务和资源类型做命名空间（`asana_projects_search`）
- 返回值 token 高效：分页、范围选择、过滤、截断，并给合理默认上限（Claude Code 的默认是 25000 token）
- 提供 `ResponseFormat` 枚举（`concise` / `detailed`）——官方给的例子里详细版 206 token、精简版 72 token，省 65%
- 错误信息要可执行：说清怎么改，附正确格式示例，而不是抛栈
- 工具描述当作给新同事的入职说明来写，把隐含前提写明

---

## 10. 三种请求的处理差异

**写或改 prompt（场景 A/B）**：按第 0 节工作流走完，交付说明按第 0 节第 6 步的要求写。不要塞入用户没要的解释、免责声明或替代方案清单。

**审查已有 prompt**：按第 3 节逐条比对，指出**具体问题和对应修法**，不要直接给一份全新的 prompt 替代掉——除非用户要求重写。按严重度排序：会导致 API 报错的（3.1、3.2、3.8、3.12）排最前，行为反向的（3.4、3.9、3.10）次之，风格问题最后。

**聊天途中临时检查（场景 C）**：走轻量流程。**不要小题大做**——不用完整跑一遍工作流，不用通读全文。扫一眼第 3 节的硬规则，有问题就直接说"这里有个问题：X，改成 Y"，没问题就说没问题然后继续原来的话题。用户此刻要的是不被打断，不是一份审查报告。

---

## 11. 交付前自检

- [ ] 已经自己选定模型和 effort，并在交付说明里写清楚了
- [ ] 如果用了英文官方片段：交付说明里写了「这几段是官方原文，请不要翻译或改写」的提醒
- [ ] 没有 prefill、没有 `budget_tokens`
- [ ] 没有为防欠触发而堆的 `CRITICAL: You MUST` / `If in doubt, use [tool]`（**官方片段里的原文措辞不算**，例如 4.7 内含的 `you MUST read the file` 是调校过的原文，照抄即可）
- [ ] 目标是 Opus 5：**没有**任何自我验证或双重检查类指令（外部确定性检查不算；官方研究片段里那句 `Regularly self-critique your approach and plan` 也不算，见第 8 节）
- [ ] 目标是 Fable 5.1：没有"展示你的思考过程"类指令；没有强制 `tool_choice`；长自主运行已加 4.18 的自主运行块（4.17 的进度核对规则仍可用，它是在 Fable 5 上测得的）
- [ ] **你自己写的**约束都是正面表述，且带了「为什么」（**官方片段里的原文措辞不算**——4.6、4.13、4.25 本身就是成串的否定式，例如 4.25 开头的 `NEVER use generic AI-generated aesthetics…`，是调校过的原文，照抄即可）
- [ ] 每条指令都想过：**一个善意的人类会怎么误解它？**（Claude Code 团队 Cat Wu 的自检法）
- [ ] 同一条指令没有在 system prompt 和 tool description 里重复出现（**模型确实需要的触发说明不算**——例如片段 4.23 配套的那句 system prompt，见 3.7）
- [ ] 示例的取舍符合第 5 节的场景判断，不是无脑加或无脑删
- [ ] 长上下文场景：材料在前、问题在后
- [ ] 输出长度有明确要求（Opus 5 尤其必要）
- [ ] 有可判定的成功标准；场景 A 有模型自己能跑的验证手段
- [ ] 通读一遍，删掉所有「不删也不会出错」的句子
- [ ] 黄金检验：**把这份 prompt 给一个不了解任务的同事，他会照做吗？**

---

## 12. 故障到处方速查

片段编号指向第 4 节。

| 症状 | 处方 |
|---|---|
| 回复太泛 | 加具体约束、加成功标准、加 "go beyond the basics" 类修饰语 |
| 答非所问 | 补上真实目标和背景（说清"为什么"） |
| 格式不稳定 | 加 3–5 个 `<example>`；或用 Structured Outputs |
| 有开场白 | `Respond directly without preamble.` |
| 编造内容（一般场景） | 显式允许说不知道：`If the data is insufficient to draw conclusions, say so rather than speculating.` |
| 对没打开过的代码/文件下判断 | 片段 4.7（防幻觉，代码场景必备） |
| 只给建议不动手 | 换直接动词（`Change this function` 优于 `Can you suggest changes`）；或加片段 4.8 |
| 输出太长（面向阅读的正文） | 见下方「长度控制怎么选」。**在 Opus 5 上降 effort 不可靠** |
| agent / 编码场景里中途播报太啰嗦 | 片段 4.3。这和正文长度是两个问题，长度控制那套管不到播报节奏 |
| 过度工程 | 片段 4.6 |
| 子智能体滥用 | 片段 4.10 |
| 思考过多、延迟高 | 片段 4.15；或降 effort |
| 复杂问题推理太浅 | **提高 effort**，而不是加 prompt 绕过 |
| 反复叮嘱某规则仍不生效 | prompt 太长、规则被淹没 → 精简，或改用确定性手段（hook、工具设计、schema） |
| 代码审查漏报 | 片段 4.16，把过滤移到下游。第 7 节里对抗式审查那条提醒是反过来的情形——没有下游过滤，只能由审查者自己过滤 |
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

## 13. 元原则（本文件没覆盖到的情况下靠这些判断）

1. **最小信息集**：能完整界定期望行为的最少 token。
2. **高度要合适**：既不写死脆弱的硬逻辑，也不给模糊到没有信号的空话。
3. **多给上下文，少给指令。**
4. **描述你想要的样子，而不是禁止你不想要的样子。**
5. **有工具就先改工具**：接口设计比 prompt 更能塑造行为。
6. **能确定性执行的，不要用 prompt**：该用 hook、schema、类型约束的地方别写"请务必"。
7. **模型变强，脚手架变负担**：定期回头删掉当初为弱模型加的东西。
8. **改完要测**：拿真实任务跑，不要凭感觉判断改好了。
