# 2026 年的 Prompt 工程：规则从哪来

> 这是 `prompt-engineering` skill 的背景读物。skill 讲的是**怎么做**，这份文档讲的是**为什么是这样**——每条规则背后的官方依据、它推翻了什么旧习惯、以及两处看起来矛盾的官方指引该怎么解释。
>
> 快照日期 **2026-07-25**（effort 相关内容于 2026-07-26 依据文档更新做过修订）。全部内容来自 Anthropic 官方文档与官方 / 员工公开分享，Claude 5 世代之前的专属指引已剔除，而不是留着当"应该还能用"。信源逐条列在 [SOURCES.md](../SOURCES.md)。
>
> **阅读方式**：第 1、2、3 章是理解框架；第 4 章是最重要的「排毒」章节（如果你以前学过 prompt 工程，那一章会推翻你的一部分认知）；第 5 和 5b 章是两个最容易搞错的单点，都源于两条官方指引看起来打架、实则场景不同；第 6 章往后按需查阅。
>
> **适用范围**：**Claude Opus 5 和 Claude Fable 5**（含 Mythos 5），以及已验证与现行标准无冲突的通用技法。所有更早模型（Opus 4.x、Sonnet 系列）的专属指引已剔除。
>
> **一句免责**：涉及具体 API 参数时，以官方文档的实时版本为准。文末附录记录了一次真实的教训——本文引用的某段官方措辞，在核查期间就被官方改掉了。

---

## 目录

1. 一句话说清发生了什么
2. 三条主线：prompt → context → harness
3. 地基：至今有效的六件事
4. **排毒：八条已经作废的旧习惯**
5. 最容易搞错的一点：示例到底还要不要给
5b. 第二个容易搞错的点：验证指令到底删不删
6. 模型差异：Opus 5 与 Fable 5 的性格
7. Agent 与长任务：上下文工程
8. Claude Code 场景：该把话写在哪
9. 怎么练
10. 信源与推荐阅读顺序

附：核查记录

---

## 1. 一句话说清发生了什么

**过去两年 prompt 工程教你的很多东西，正在被 Anthropic 自己推翻。**

最硬的一个证据：2026 年 7 月 24 日 Claude Code 团队的一篇文章说，他们把 Claude Code 的系统提示词**砍掉了 80% 以上，性能没有下降**。

这不是个孤立事件。另外两条独立线索指向同一个方向：

- `removing examples was extremely helpful, because it was just more creative than the examples we gave it`（Thariq Shihipar，AI Engineer World's Fair，2026-07）
- 随着 Claude 变强，harness 里当初必要的脚手架变成死重，反而成为瓶颈（Lance Martin，Claude Platform 团队，2026-04）

方向可以概括成一句话：

> **从「精心构造指令」转向「精心构造上下文与工具」。**

为什么？因为模型的判断力上来了。过去你要写 `Never write multi-line docstrings`，是因为模型自己判断不好；现在你写 `Write code that reads like the surrounding code`，它能自己看出来。而且前一种写法有个隐藏代价：**硬约束一旦和用户后来的真实需求冲突，模型会陷入混乱**。

Thariq Shihipar 在同一场对谈里的原话：

> "The other thing we did is try to give it more context and fewer 'do not do this' instructions, because that's a very strong impulse for Claude, and especially if it conflicts with user instructions later on, that can be extremely confusing to Claude — 'I've got this skill that says this and the system prompt says this.'"

最后那半句是关键：混乱的具体形态不是「模型不听话」，而是**模型手里拿着两份互相打架的指令，不知道该听谁的**。

---

## 2. 三条主线：prompt → context → harness

理解 2026 年的图景，需要区分三个层次。很多人的困惑来自把它们混为一谈。

### 2.1 Prompt Engineering（提示词工程）

「为最优结果而编写和组织给 LLM 的指令」。**一次性的写作行为**。适用于单轮、任务明确的场景。

### 2.2 Context Engineering（上下文工程）

「在 LLM 推理过程中策划并维护最优 token 集合的一整套策略，包括所有落在提示词之外的信息」。

关键区别在于：**提示词工程是一次性写作，上下文工程是迭代的——每一轮你都要重新决定往模型里塞什么。**

系统指令、工具、外部数据、消息历史、记忆——全都算。多轮 agent 场景下这个层次比提示词层次重要得多。

核心概念是 **context rot（上下文腐坏）**：模型有有限的「注意力预算」，每加一个 token 就消耗一点。Transformer 的注意力是平方复杂度，上下文越长，注意力越被稀释。所以目标不是「把知道的都告诉它」，而是：

> **找到能最大化达成目标概率的、最小的高信噪比 token 集。**

### 2.3 Harness / Loop Engineering（框架与循环工程）

围绕模型的软件基础设施：循环、工具、上下文管理、护栏。Lance Martin 的核心洞察是：

> 随着 Claude 变强，harness 里当初必要的假设会**过期**，脚手架变成死重。要定期回头问：**我能停止做什么？**

**实际启示**：如果你发现自己在 prompt 里写了一大堆规则，先停下来问——这个问题是不是应该在更外层解决？换个工具设计？加个确定性的检查？还是压根不需要管？

---

## 3. 地基：至今有效的六件事

### 3.0 先说前置条件

官方 prompt engineering 入门页开宗明义地讲了一件很多人跳过的事——**调 prompt 之前你应该已经有**：

1. 清晰的成功标准
2. 能对照标准做实证测试的方法
3. 一版初稿

没有前两项就去改 prompt，等于闭着眼睛调参。（没有初稿倒不要紧，官方提供了 [metaprompt recipe](https://colab.research.google.com/github/anthropics/claude-cookbooks/blob/main/misc/metaprompt.ipynb) 帮你生成一版。）

还有一句同样重要的话：**不是每个失败的 eval 都该用 prompt 工程来解。** 官方举的例子是——延迟和成本，有时换个模型比调 prompt 容易得多。

这条心态很关键：**prompt 是工具箱里的一把工具，不是唯一的锤子。** 后面第 7、8 章会反复出现同一个模式——很多看起来像「提示词没写好」的问题，实际上该在工具设计、schema、hook 或模型选择层面解决。

---

新规则推翻的是「过度约束」，不是「说清楚」。以下这些官方文档里仍然在教，而且是现行版本。

### 3.1 清晰直接

把 Claude 当成一个**聪明但刚入职的新同事**——他很强，但不了解你们的规范和工作流。

**黄金检验法**：把你的 prompt 给一个不了解这个任务的同事看，让他照着做。如果他会困惑，Claude 也会。

想要「超出基本要求」的表现，就明说，不要指望模型从模糊的提示里推断。

- ✗ `Create an analytics dashboard`
- ✓ `Create an analytics dashboard. Include as many relevant features and interactions as possible. Go beyond the basics to create a fully-featured implementation.`

### 3.2 给上下文，而且要说「为什么」

解释你为什么这么要求，模型能从解释中泛化。这一条现在比以前更重要——它正是「多给上下文、少给指令」的具体做法。

- ✗ `NEVER use ellipses`
- ✓ `Your response will be read aloud by a text-to-speech engine, so never use ellipses since the text-to-speech engine will not know how to pronounce them.`

注意第二种写法里其实还是有 "never"，但它挂在一个**原因**上。模型知道了原因，遇到没被明确覆盖的边界情况时也能自己判断。这就是差别。

### 3.3 用 XML 标签做结构

> ⚠️ 有一篇 2025 年 11 月的官方博客说 XML「现在不太必要了」。**这条已被现行文档否定**，裁定见第 10 章的排除清单和 [SOURCES.md](../SOURCES.md) 的「Known conflicts」。

当你的 prompt 混合了指令、背景、示例和变量输入时，XML 标签能显著减少误读。官方原文：

> "XML tags help Claude parse complex prompts unambiguously, especially when your prompt mixes instructions, context, examples, and variable inputs. Wrapping each type of content in its own tag (for example, `<instructions>`, `<context>`, `<input>`) reduces misinterpretation."

用一致且有描述性的标签名，有层级时嵌套。

官方片段库里有一个规律，但它**只朝一个方向成立**：**凡是包了标签的，都是「常驻方针」**——默认要不要动手、工具怎么调、输出怎么排版、语气保持什么样：`<default_to_action>`、`<do_not_act_before_instructions>`、`<use_parallel_tool_calls>`、`<investigate_before_answering>`、`<avoid_excessive_markdown_and_bullet_points>`、`<frontend_aesthetics>`、`<tone_preference>`。

反过来不成立。相当多的常驻方针是纯文本——不可逆动作的安全护栏、无人值守自主运行的指令、记忆书写约定，都没包。最能说明问题的是同一条方针的两种形态：控制冗长的那段正文是纯文本，而放在 prompt 末尾的那句同内容的简短提醒，包了 `<tone_preference>`。

所以判据既不是长度（库里最短的带标签块只有一句话），也不是主题。**标签的价值在于让这一块能被当作一个有名字的单元来对待**——与周围内容隔开，或者被别处按名字引用。这是观察到的规律，不是官方明文规定；包一层标签几乎没有成本，拿不准就包上。

Markdown 标题（`## Tool guidance`、`## Output description`）同样是官方认可的分区方式，与 XML 可以混用——这一点出自 Anthropic 的上下文工程文章。

### 3.4 给角色

在 system prompt 里设定角色能聚焦行为和语气。**一句话就有效果**：

```
You are a helpful coding assistant specializing in Python.
```

> ⚠️ 同样有 2025-11 博客说 role prompting「往往没必要」——现行文档仍在推荐它，裁定同见第 10 章。不过那篇博客有个有价值的补充：**比起贴一个身份标签，说清「关注什么视角」通常更有用**。与其 `You are a financial advisor`，不如 `Analyze this portfolio, focusing on risk tolerance and long-term growth`。两者不冲突，可以叠加。

### 3.5 长上下文的摆位（性价比最高的一条）

处理 20k+ token 的长文档时：

- **长文档放最前面，指令和问题放最后面。**
- 官方原文：`Queries at the end can improve response quality by up to 30 percent in tests, especially with complex, multidocument inputs.`

多文档时这样包：

```xml
<documents>
  <document index="1">
    <source>annual_report_2023.pdf</source>
    <document_content>
      {{ANNUAL_REPORT}}
    </document_content>
  </document>
</documents>

Analyze the annual report...   <!-- the question goes here, last -->
```

还有一招叫 **grounding in quotes**：先让模型从长文档里摘出相关引文放进 `<quotes>` 标签，再基于引文回答。这能让它聚焦在相关内容上，忽略其余部分。

### 3.6 说「要什么」而不是「不要什么」

这一条贯穿所有官方材料，是最普适的原则。

- ✗ `Do not use markdown in your response`
- ✓ `Your response should be composed of smoothly flowing prose paragraphs.`

还有个巧妙的补充手段：**让你的 prompt 风格贴近你想要的输出风格**。比如你的 prompt 里如果去掉 markdown，输出里的 markdown 也会减少。

Opus 5 的官方建议在「沟通风格」这个具体场景下也说了同样的话：**展示你想要的风格的正面示例，往往比「别怎样」这类指令更有效**。

---

## 4. 排毒：八条已经作废的旧习惯

这一章是这份文档存在的主要理由。以下每条都注明了「旧说法出处」和「现行依据」，可以自己去核。

### 4.1 ❌ Prefill（预填充助手回复）——已被 API 禁止

**旧说法**：在 assistant 消息里预填 `{` 来强制 JSON 输出，或预填 `Here is the summary:` 来去掉开场白。这在 2025 年 11 月的官方博客里还被列为「高级技巧」。

**现状**：**Opus 5 和 Fable 5 均不支持。** 官方原文：

> "Starting with Claude 4.6 models and Claude Mythos Preview, prefilled responses (providing a partial assistant message for Claude to continue from) on the last assistant turn are no longer supported. Requests with prefilled assistant messages to these models return a 400 error."

**为什么取消**：官方给的理由是，模型的智能和指令遵循能力已经进步到大多数 prefill 用例不再需要它。

**替代**：

| 原用途 | 现在怎么做 |
|---|---|
| 强制 JSON / 固定结构 | **Structured Outputs** 功能；或直接把 schema 说清楚（新模型能可靠匹配复杂 schema） |
| 分类任务 | 带 enum 字段的 tool，或 structured outputs |
| 去开场白 | `Respond directly without preamble. Do not start with phrases like 'Here is...', 'Based on...', etc.` |
| 绕过误拒绝 | 现在模型的拒绝判断准多了，正常在 user 消息里说清楚即可 |
| 续写被截断的回复 | 把上文放进 user 消息里让它接着写；或者直接重试 |

### 4.2 ❌ `budget_tokens` 控制思考预算——已废弃

**旧写法**：`thinking: {type: "enabled", budget_tokens: 10000}`

**现状**：Opus 5 和 Fable 5 均不支持手动 extended thinking，设置 `budget_tokens` 返回 400。

**新写法**（Opus 5）：

```python
thinking={"type": "adaptive"},
output_config={"effort": "high"},
```

Fable 5 的 thinking 始终开启且完全无法配置，能调的只有 `effort`。

**adaptive thinking 是什么**：模型自己决定何时思考、思考多少，依据两个因素——`effort` 参数和查询复杂度。简单问题直接答，复杂问题多想。官方内部评测显示 **adaptive thinking 可靠地优于手动 extended thinking**。

需要硬上限时用 `max_tokens`（注意它是**思考 + 正文的总上限**）。

### 4.3 ⚠️ 用强硬措辞防止「欠触发」——已经反向（但有适用范围）

**旧写法**：`CRITICAL: You MUST use this tool when...`

**为什么当年这么写**：早期模型对工具的触发不够积极，需要加压。

**现状**：这套加压写法已经不需要了。Fable 5 的官方原话是：

> "Instruction-following is improved enough that you can steer most behaviors with a brief instruction rather than enumerating each behavior by name."

Claude Code 团队据此把系统提示词砍掉 80% 以上而性能不降。堆砌加压措辞现在只会挤占注意力预算，并在与用户真实需求冲突时制造混乱。

**新写法**：`Use this tool when...` 就够了。同类的 `If in doubt, use [tool]` 和 `Default to using [tool]` 也一样，改成 `Use [tool] when it would enhance your understanding of the problem`。

**注意范围**（这一条容易被一刀切，需要区分）：

- 默认按正常陈述句写。真观察到工具欠触发再针对性地加一句说明
- Claude Code 官方最佳实践**至今仍建议**：在 **CLAUDE.md** 里可以用 `IMPORTANT` 或 `YOU MUST` 来加强某条规则的遵守度

这两处不矛盾——前者说的是 API system prompt 里防工具欠触发的历史包袱，后者说的是项目配置文件里强调关键约定。**判断标准是「你在解决什么问题」，不是「这个词能不能用」。**

### 4.4 ❌ 让 Opus 5「再检查一遍」——反而有害

**旧建议**：通用文档至今仍写着 `Before you finish, verify your answer against [test criteria]`。

**Opus 5 是例外**，而且官方措辞很硬：

> "If your prompt contains explicit verification instructions ('include a final verification step for any non-trivial task,' 'use a subagent to verify'), remove them: instructions like these cause over-verification on Claude Opus 5, and removing them reduces wasted tokens with no loss in quality."

注意官方用的词是「**remove**（删除）」，不是改写。这一点很反直觉，值得单独记住。

同理，Opus 5 也不需要 `double-check your answer` / `re-verify before responding`。

### 4.5 ❌ 堆砌「绝不允许」式约束

已在第 1 章讲过原理。补充一个 Claude Code 团队的实操细节，Cat Wu 描述他们的审查流程：

> "we found a few cases where yes, this statement is 90% true, but there's a real 10% of cases where it's not true"

他们的做法是把这类「90% 正确」的硬规则**删掉**，因为那 10% 会和用户真实意图冲突，代价大于收益。

如果这条规则确实值得留，改写的方式是把命令变成情境描述。比如从 `always run the app locally before finishing` 改成：

> "most of the time when you're doing front-end work you can't fully understand the experience by hitting the backend endpoints, so when you make larger changes to the user experience, please run the app locally."

看出区别了吗？后者交代了**何时、为什么**，把判断权交还给模型。

### 4.6 ❌ 在多处重复同一条指令

**旧习惯**：怕模型看不见，system prompt 里说一遍，tool description 里再说一遍，文档里又说一遍。

**现状**：现行模型不需要反复强调。Claude Code 团队的做法是——工具的使用说明**只保留在 tool description 一处**，把 system prompt 里那份重复的删掉。

### 4.7 ❌ 手工维护 CLAUDE.md 作为记忆

**旧做法**：用 `#` 热键把要点写进 CLAUDE.md。

**现状**：自动记忆已经能捕捉相关上下文，不再需要手工维护那部分。

（注意：CLAUDE.md 本身没有作废——它仍然是放构建命令、目录结构、团队约定的地方。作废的是「拿它当记事本手工记录」这个用法。见第 8 章。）

### 4.8 ❌ 用简化的 markdown 说明书当规格

**旧做法**：写一份简化的 markdown 说明，**用它代替**真实材料交给模型。

**新做法**：给**丰富的引用**——实际代码、测试套件、另一个代码库里的对应函数、可交互的 HTML artifact。现行模型能处理复杂的引用形式，信息密度远高于手写摘要。

要说清楚这条**没有**否定什么：用户读过并确认的 spec 是另一回事。第 8.4 节推荐的正是产出这样一份 spec，它之所以有效，是因为它的身份是一份**约定**，而不是代码的有损替代品。

### 补充：两个 API 层面的变化

- **采样参数（`temperature` / `top_p` / `top_k`）的支持在收紧**，部分新模型设非默认值直接返回 400。别再靠采样参数求风格多样性，改用提示词——最有效的一招是让模型先提几个不同方向，**由你挑一个**，再落地它。
- **新模型可能换 tokenizer**，同样文本的 token 数会变，旧的 `max_tokens` 设置可能导致截断。升级时去 migration guide 确认这两点。

### 一张排毒总表

| 旧习惯 | 现状 | 替代 |
|---|---|---|
| prefill 预填充 | 4.6+ 返回 400 | Structured Outputs / 明确指令 |
| `budget_tokens` | 返回 400 | Opus 5 用 `adaptive` + `effort`；Fable 5 只能用 `effort` |
| `CRITICAL: You MUST`（防工具欠触发） | 已不需要，只挤占注意力；CLAUDE.md 里标记少数关键约定除外 | `Use this tool when...` |
| 让 Opus 5 自检 | 导致过度验证 | **删除** |
| 堆砌 `Never do X` | 与用户意图冲突时致乱 | 正面描述 + 说明原因 |
| 多处重复指令 | 冗余 | 单一权威来源 |
| 手工 CLAUDE.md 记忆 | 已有自动记忆 | 自动记忆 |
| 用 markdown 说明**代替**真实材料 | 信息密度低 | 代码 / 测试 / artifact |
| 用 temperature 求多样性 | 支持在收紧，部分模型返回 400 | 让模型先提几个方向，你挑一个再落地 |

---

## 5. 最容易搞错的一点：示例到底还要不要给

这是目前信息最混乱的地方，因为你会同时看到两种说法，而且**都来自 Anthropic 官方**：

- **官方文档**：`Examples are one of the most reliable ways to steer Claude's output format, tone, and structure.` 建议 `Include 3–5 examples for best results.`
- **Claude Code 团队**：`removing examples was extremely helpful, because it was just more creative than the examples we gave it.`

Simon Willison 在那场对谈里也被这个问题绊了一下——他说给示例是自己最常给人的 prompt 建议之一，如果这不再成立，他的 prompt 心智模型就有点站不住了。

**答案是：两者都对，因为适用场景根本不同。**

| 你的目标 | 该不该给示例 | 原因 |
|---|---|---|
| 让输出**落进一个固定的框**——固定 JSON 形状、固定文风、固定报告结构 | **给**。3–5 个，多样化，`<example>` 包裹 | 示例是「约束器」，这时候你**就是**要约束 |
| 让模型**自由发挥判断力**——agent 系统提示词、开放式创作 | **不给**。改为优化工具接口设计 | 示例会把模型的探索空间收缩到示例附近，而它本可以做得更好 |
| 想演示**推理方式** | 在 few-shot 里用 `<thinking>` 标签，模型会泛化到自己的思考块 | 官方文档明确支持 |

一句话记忆：

> **示例是约束器。你要约束时用它，你要创造力时删掉它。**

Claude Code 团队之所以删示例，是因为他们写的是一个通用编码 agent 的系统提示词——那里需要的恰恰是模型自己判断，不是照着模板做。而如果你要的是「每次都输出这个格式的周报」，示例仍然是最有效的工具。

给示例时的三条要求（官方）：**相关**（贴近真实用例）、**多样**（覆盖边界，别让模型学到意外模式）、**结构化**（`<example>` 包裹）。

---

## 5b. 第二个容易搞错的点：验证指令到底删不删

这是另一处「看起来矛盾」的地方，比示例那条更隐蔽。

你会同时读到两条官方指引：

- **Opus 5 页**：如果你的 prompt 里有显式验证指令，**删掉它们**——这类指令会在 Opus 5 上造成过度验证、白白烧 token，而删掉它们不会损失质量。
- **Fable 5 页**：在长运行任务的 prompt 里，**明确写入自我检查**：`Establish a method for checking your own work at an interval of [X] as you build. Run this every [X interval], verifying your work with subagents against the specification.`

**怎么解释？** 关键在那句话里的「用子智能体」，以及 Fable 5 页紧挨着它的前一句：**独立的、干净上下文的 verifier 子智能体，效果优于模型自我批评。**

所以真正的规律是：

| | 该删 | 该加 |
|---|---|---|
| **形式** | 让模型**自己再看一遍**（`double-check`、`re-verify`、「最后加一步验证」） | 换**一个干净上下文**来查（fresh-context verifier subagent） |
| **场景** | 常规任务，模型一次就能自我纠错 | 多小时的自主长跑，误差会累积 |

一句话记忆：

> **删掉「自己检查自己」，保留「换个人检查」。**

这同时解释了另一处看起来打架的地方：Opus 5 的官方建议里有「不要用子智能体来复查自己的工作」，而 Fable 5 建议「用子智能体按周期对照规格验证」。前者说的是**几次工具调用就能完成的常规任务**，后者说的是**跑几小时的自主流程**。任务时长不同，成本收益就反过来了。

还有一个更容易踩的坑：**「验证」这个词在两个意思之间滑动。** 该删的是「让模型主观地再想一遍对不对」；**不该删**的是「跑一遍测试、看构建退出码、把截图和设计稿比对」这类**外部确定性检查**——那不是自我验证，那是给模型一个它自己能读的信号。第 8 章的头条规则讲的正是后者。

---

## 6. 模型差异：Opus 5 与 Fable 5 的性格

现行模型不再是「越大越好」的线性关系，它们**性格不同**，需要不同的调校方向。

### 6.1 Claude Opus 5

定位：复杂 agentic 编码与企业任务，长周期任务见长。默认 effort `high`，thinking 默认开启，1M token 上下文（默认即最大）。

| 倾向 | 你要做的 |
|---|---|
| 用户可见回复默认偏长 | 显式要求简洁。**注意：降 effort 不能可靠缩短可见输出**——effort 控制的是「想多少」不是「说多少」 |
| Agent 场景播报频繁 | 明确描述你要的播报节奏 |
| 写到磁盘的文档也偏长 | 单独加长度校准指令 |
| 自我验证很充分 | **删掉**所有验证类指令 |
| 会自行扩大任务范围 | 窄任务要显式约束范围 |
| 更积极派生子智能体 | 显式设上限，否则小任务成本翻倍 |
| 更爱口头纠正自己之前的说法 | 限定为「只在会影响用户决策时才说」 |
| 代码审查时会**字面执行**「只报高危」 | 改成「全报，下游再过滤」 |
| `low`/`medium` 质量强，而 token 与延迟只是高档位的一个零头 | **从默认 `high` 起步**，足够重的编码 / agent 工作才上 `xhigh`。设置若从更早的模型沿用而来，重新做一次 effort 扫描 |

**一个坑**：关闭 thinking 时，Opus 5 偶尔会把 tool call 写成普通文本（调用不会执行，而且这段文本会留在对话历史里污染后续轮次），或者把 `<thinking>` 标签泄漏到可见输出。官方建议是**保持 thinking 开启，用低 effort 控成本**，而不是关掉它。（在 `xhigh` / `max` 档位下也确实关不掉，请求会返回 400。）

### 6.2 Claude Fable 5 / Mythos 5

定位很不一样：**专门用来接此前「太复杂、太长、太模糊」而做不了的问题**，擅长人类要花数小时到数周的端到端工作。thinking 始终开启且不能关闭。

官方有句话值得注意：**效果最好的团队是把它用在自己最难的未解问题上；只拿简单任务测试，往往会低估它的能力范围。**

| 倾向 | 你要做的 |
|---|---|
| **单次请求可能跑很多分钟，自主运行可达数小时** | 迁移前先改客户端超时、流式、进度提示；考虑改成异步轮询而不是阻塞等待 |
| 任务模糊时会过度规划 | 加一句「信息够了就动手，不要把不打算走的路都陈述一遍」 |
| 指令遵循强到**一句话就能管住一类行为** | 不用逐条枚举，写一条简短指令即可 |
| 长自主运行中可能编造进度 | 这是最值钱的一条处方：**要求它每条进度声明都对照本次会话的工具结果做审计**。官方测试中这几乎消除了编造 |
| 会做未被要求的事（自动起草邮件、建防御性 git 分支） | 明确划边界：「用户在描述问题、提问或出声思考，而不是要求改动时，交付物是你的评估——报告发现然后停下」。官方原文还有关键的后半句：**动系统状态（重启、删除、改配置）之前，先确认证据真的支持这个具体动作；一个「长得像某个已知故障」的信号，成因可能完全不同** |
| 更主动派并行子智能体 | **鼓励用**，但给明确判据；优先异步通信；长生命周期子智能体靠缓存读取省钱 |
| 看到剩余 token 倒计时会提议开新会话 | 告诉它上下文还很充裕、不要因此收工；更好的做法是**别让模型看到剩余预算数字** |
| **为旧模型写的 skills 往往太规定化，可能降低 Fable 5 的输出质量** | 审查并删减 |

最后一条值得单独强调——它是「模型变强、脚手架变负担」这个主题最直白的官方表述。官方原文：

> "Skills developed for prior models are often too prescriptive for Claude Fable 5 and can degrade output quality."

**旧 skill 不只是无用，它还可能主动拖低质量。**

还有一个 Fable 5 专有的坑：**不要指示它复述 / 展示自己的内部推理**。官方原文：

> "Prompts, skills, or harness instructions that tell the model to echo, transcribe, or explain its internal reasoning as response text can trigger the `reasoning_extraction` refusal category on Claude Fable 5, causing elevated fallbacks to Claude Opus 4.8."

需要推理可见性就去读 API 返回的结构化 `thinking` 块；需要在长跑中间给用户看东西，就做一个 send-to-user 工具。

### 6.3 effort 参数

`max` > `xhigh` > `high`（默认）> `medium` > `low`

官方给每一档的定位：

| 档位 | 官方定位 | 典型用途 |
|---|---|---|
| `max` | 绝对最大能力，不限制 token 花费 | 需要最深推理、最彻底分析的任务 |
| `xhigh` | 面向长周期工作的扩展能力 | **超过 30 分钟**、token 预算以百万计的长周期 agentic 与编码任务 |
| `high` | 高能力。**等同于不设这个参数** | 复杂推理、困难的编码问题、agentic 任务 |
| `medium` | 平衡，适度省 token | 需要在速度、成本、性能之间平衡的 agentic 任务 |
| `low` | 最省。显著省 token，能力有一定下降 | 需要最快速度和最低成本的简单任务，比如子智能体 |

**两个模型都从默认 `high` 起步**，区别在什么时候往上调：

| 模型 | 什么时候上调 |
|---|---|
| **Opus 5** | 编码或 agentic 工作**足够重**时上 `xhigh`；任务值得不设上限烧 token 时上 `max`。评测显示质量不掉的地方，放开用 `low`/`medium` 控成本 |
| **Fable 5** | 只有最吃能力的任务才上 `xhigh`，日常降到 `medium`/`low`——它的低 effort 常常还优于旧模型的 `xhigh` |

> **这一条改过。** 本文早期版本写的是「Opus 5 编码 / agent 任务起步用 `xhigh`」，依据是官方页当时的措辞 `for coding and agentic work, xhigh remains the recommended starting point`。该页现在改成了 `Start with the default (high) ... step up to xhigh for demanding coding and agentic work`。实际差别是真的：一个普通的编码任务现在从 `high` 起步，不是 `xhigh`。经过见文末附录。

几个要点：

- `effort: "high"` 与**完全不传这个参数行为完全相同**（官方原文：`Setting effort to "high" produces exactly the same behavior as omitting the effort parameter entirely.`）
- effort 是**行为信号，不是硬性 token 预算**——官方原文：`Effort is a behavioral signal, not a strict token budget. At lower effort levels, Claude will still think on sufficiently difficult problems, but it will think less than it would at higher effort levels for the same problem.`
- 它影响**全部输出** token：正文、工具调用、思考
- **低 effort 会直接减少工具调用次数**、合并操作、跳过前言，完成后只给一句简短确认；高 effort 则更多工具调用、先讲计划、给详细总结。这一点常被忽略——**effort 不只是「想多久」，也是「做多少」**
- **按工作负载显式设定 effort**，而不是拿一个档位跑遍所有场景
- **启用 prompt caching 时不要中途改 effort**——官方原文：`Changing the effort value between requests invalidates prompt caching, so vary effort across workloads rather than within a conversation that relies on cache hits.`
- 高 effort 时要给 `max_tokens` 留余量（它是思考 + 正文的**总**上限），否则会出现「大部分预算被思考吃掉、正文被截断」。Opus 5 在 `xhigh`/`max` 下官方建议 64k 起

> 顺带一提：Claude Code 里的 `ultracode` **不是**一个额外的 API effort 档位。它是 `xhigh` 加上「允许启动多智能体工作流」的常驻授权。

---

## 7. Agent 与长任务：上下文工程

### 7.1 核心约束：context rot

模型有有限的「注意力预算」。上下文越长，注意力越被稀释，性能下降越明显。这不是工程 bug，是 transformer 注意力机制的平方复杂度带来的固有性质。

所以上下文工程的目标不是「装得下」，而是：**最小的高信噪比 token 集**。

### 7.2 「合适的高度」（right altitude）

系统提示词有两种失败方式：

- **太具体**：把复杂脆弱的逻辑硬编码进提示词，造成脆弱性和维护负担
- **太模糊**：给高层空话，没有具体信号，或者假设了模型并不共享的背景

正确的区间是：**足够具体到能有效引导行为，又足够灵活到给模型好的启发式**。

### 7.3 三种长周期策略

**① Compaction（压缩）**

对话接近上下文上限时，总结内容并用摘要重启一个新窗口。保留架构决策、未解决的 bug、实现细节；丢弃冗余的工具输出和消息。

调优方法（官方给的顺序很实用）：**先最大化召回**，确保压缩提示词能抓到轨迹里所有相关信息；**再迭代提升精度**，去掉多余内容。

**② 结构化笔记**

让 agent 定期把笔记写到上下文窗口之外的持久存储，之后再拉回来。开销极小，却能跨复杂任务维持进度。

**③ 子智能体架构**

主智能体管高层计划，子智能体用干净的上下文做深度技术工作，只返回**浓缩后的摘要（通常 1000–2000 token）**。这实现了关注点分离——子智能体可以做大量探索，但探索过程不污染主上下文。

### 7.4 跨上下文窗口的实操

官方给的一套具体做法：

1. **第一个窗口用不同的 prompt**：先搭框架（写测试、建 setup 脚本），后续窗口才迭代 todo list
2. **让模型用结构化格式写测试**（`tests.json`），并明确告诉它：`It is unacceptable to remove or edit tests because this could lead to missing or buggy functionality.`
3. **建 `init.sh` 之类的脚本**：起服务、跑测试、跑 lint，避免每个新窗口重复劳动
4. **可以考虑用「全新窗口」代替 compaction**——这条反直觉但有依据：现行模型极擅长从文件系统恢复状态，某些场景下这比压缩更划算（官方措辞是「考虑」和「在某些情况下」，不是一律如此）。这时开场要具体：`Call pwd`、`Review progress.txt, tests.json, and the git logs`、`Manually run through a fundamental integration test before moving on`
5. **提供验证工具**（浏览器自动化、computer use 等）——任务越长越自主，模型越需要在没有人类反馈的情况下自我验证
6. **鼓励用满上下文**，明确告诉它不要因为怕超预算而提前收工

状态管理的分工：结构化数据用 JSON，进度笔记用自由文本，版本和检查点用 **git**（现行模型用 git 追踪跨会话状态的表现特别好）。

### 7.5 让 Claude 自己编排：收益最大的一个改动

这一条常被忽略，因为它不在提示词层，而在 harness 层——但它可能是整份文档里**投入产出比最高的一条**。

常见做法是：每次工具调用的结果，都作为 token 塞回上下文窗口，过滤逻辑硬编码在工具里。

**更好的做法**：给 Claude 代码执行能力（bash、语言 REPL），让它自己写逻辑来表达工具调用、自己过滤输出——**只有执行结果进入它的注意力**。

官方给了数字：在 BrowseComp 网页浏览任务上，仅仅是给模型输出过滤能力，准确率从 **45.3% 升到 61.6%**。

同一思路的另外两条，也都有数字：

| 做法 | 效果 |
|---|---|
| 让 Claude 自己管上下文（skills 渐进披露、context editing、子智能体隔离） | BrowseComp +2.8% |
| 让 Claude 自己持久化上下文（compaction + memory 文件夹） | agentic search 60.4% → 67.2% |

> **出处说明**：这三组数字来自 Anthropic 在**更早一代模型**上的实测（原文点名了具体型号，此处按本文的适用范围略去）。文章本身的论点是这个趋势会随模型变强而延续——**模型越强，越该把决策交还给它**，所以对 Opus 5 / Fable 5 只会更成立，不会更弱。

官方还给了个很生动的对比：较新的模型玩宝可梦时会写出有组织的战术笔记（10 个文件、追踪道馆徽章）；而早一代模型把记忆当成流水账（31 个重复文件，卡在第二个城镇出不去）。**同样的记忆机制，模型能力不同，用出来的效果天差地别**——这正是「脚手架会过期」的另一面：模型强了之后，你该把更多决策**交还**给它。

### 7.6 缓存命中：一个纯粹的成本杠杆

**缓存 token 的成本只有基础输入 token 的 10%。** 所以上下文的排列顺序直接决定你花多少钱。五条原则：

| 做法 | 要点 |
|---|---|
| **静态在前** | 稳定内容（系统提示词、工具定义）放在动态内容之前 |
| **用消息追加而非改提示词** | 要注入提醒时追加到 messages 里，不要回头改 system prompt |
| **不要中途换模型** | 缓存按模型隔离 |
| **工具增删会作废缓存** | 需要动态发现工具时用 tool search，而不是增删工具定义 |
| **移动断点** | 多轮应用把 cache breakpoint 移到最新消息 |

这条也解释了第 6.3 节那个看起来很随意的提醒——**effort 不要中途改**：同样是会破坏缓存。

### 7.7 什么时候该把 bash 动作提升为专用工具

bash 杠杆大，但 harness 无法拦截、把关或审计它。以下四种情况值得做成专用的声明式工具：

1. **安全边界**：难以撤销的动作需要用户确认（比如调用外部 API）
2. **写操作需要陈旧性检查**：防止覆盖已被别人改过的文件
3. **需要面向用户呈现**：弹窗、多选项展示
4. **需要结构化日志、追踪、回放**

低风险的工作还有另一条路：**用第二个 Claude 实例判断 bash 命令是否安全**——这就是 Claude Code auto 模式的做法，能减少专用工具的数量。

> 这一节和 7.5 看起来矛盾（一个说「少造工具、用通用 bash」，一个说「该造专用工具」），实际是同一个判断的两面：**为模型的能力让路，为产品的确定性设界。** 模型能自己搞定的，别用工具限制它；产品必须能拦截 / 审计 / 渲染的，才做成工具。

### 7.8 有工具时，先改工具

这是新规则里最有价值的一条：**工具接口的表达力比提示词文本更能塑造行为。**

好工具的标准：

- **低基数**：每个工具功能互不重叠，模型能一眼分清何时调用哪个。判断法很简单——如果**工程师自己都说不清某个场景该用哪个工具，agent 更不可能分得清**
- **参数名语义明确**：`user_id` 而非 `user`；把无意义的 UUID 解析成有语义的名称能显著提升精度
- **命名空间**：按服务和资源类型分组（`asana_projects_search`、`asana_users_search`）
- **返回值 token 高效**：分页、范围、过滤、截断，给合理默认上限（Claude Code 的默认是 25000 token）
- **提供 `ResponseFormat` 枚举**（`concise` / `detailed`）——官方给的例子里详细版 206 token，精简版 72 token，省 65%
- **错误信息要可执行**：说清怎么改、附正确格式示例，而不是抛栈
- **工具描述当作给新同事的入职说明写**：把隐含的前提、专有查询格式、术语定义、资源关系全部写明

Claude Code 团队提到一个有意思的取舍。Thariq 说他们删掉了自家的搜索工具：

> "we removed our grep and other search tools — glob tools — in favor of native bash."

因为模型本来就会用标准命令行工具。但 Cat 解释他们**保留了专门的文件编辑工具**：

> "The reason we had a dedicated file edit tool was so that we could deterministically know that Claude was making a file change"

——因为需要确定性地知道「Claude 正在改文件」，才能渲染出「你是否批准这个改动」的审批 UI。

**这个取舍值得记住**：工具存在的理由不一定是「模型需要」，也可能是「产品需要确定性」。

> 工具设计的哲学，Thariq 说得挺妙：`the models are kind of more of a biology than a physics, and tool design especially is quite hard...tool design is more of an art, maybe — or a biology.`

---

## 8. Claude Code 场景：该把话写在哪

这是很多人的实际痛点：同一句话，写进 CLAUDE.md、skill、hook、subagent，效果和成本完全不同。

### 8.1 取舍表

| 手段 | 加载时机 | 上下文成本 | 适合放什么 |
|---|---|---|---|
| **CLAUDE.md（根）** | 会话开始，全程驻留 | **高** | 构建命令、目录结构、团队约定。官方原话：`Keep CLAUDE.md under 200 lines, give it an owner, and review changes to it like code.` |
| **CLAUDE.md（子目录）** | 触及该目录时 | 低 | 目录级约定 |
| **Rules** | 会话开始或访问文件时 | 中 | 具体约束。**加 `paths:` 限定路径**可大幅降成本 |
| **Skills** | 名称 + 描述常驻，正文按需 | 低 | 流程性工作流：部署清单、发版流程、审查流程 |
| **Subagents** | 隔离上下文，只回最终结论 | 低 | 只要结论的侧任务：日志分析、依赖审计 |
| **Hooks** | 生命周期事件触发，绕过压缩 | 低 | **确定性**自动化：lint、发通知、阻断命令 |
| **Output styles** | 会话开始，永不压缩 | **高** | 只在需要大改 Claude 角色时用 |

### 8.2 反模式

- ✗「每次 X 都要 Y」写进 CLAUDE.md → 用 hook。官方原话：**真正的护栏必须是确定性的**（a real guardrail needs to be deterministic）
- ✗「绝不允许…」→ 用 `PreToolUse` hook 或权限配置
- ✗ 30 行的操作流程写进 CLAUDE.md → 移到 skill
- ✗ 只跟 API 相关的规则却不限定路径 → 加 `paths:` frontmatter
- ✗ 个人偏好写进项目共享文件 → 用用户级本地配置

### 8.3 CLAUDE.md 的自检法

对每一行问：**「删掉它会不会让 Claude 犯错？」** 不会就删。

**该写**：模型猜不到的 bash 命令、与默认不同的代码风格、测试方式、仓库礼仪（分支命名、PR 约定）、项目特有的架构决策、环境怪癖、非显而易见的坑。

**不该写**：读代码就知道的、语言通用惯例、详细 API 文档（给链接就行）、频繁变动的信息、逐文件说明、「写干净代码」这类空话。

**一个重要的诊断信号**：

> 如果你反复叮嘱某条规则，Claude 还是不照做，通常**不是措辞不够强，而是文件太长导致这条规则被淹没了**。

官方原话：臃肿的 CLAUDE.md 会导致 Claude 忽略你真正的指令。

同理，如果 Claude 反复问你 CLAUDE.md 里已经写了的东西，说明那句话写得有歧义。

**把 CLAUDE.md 当代码对待**：出问题时审查它、定期修剪、改动后观察 Claude 行为是否真的变了。

**有个现成工具**：Claude Code 里运行 `/doctor`，它能帮你精简 skills 和 CLAUDE.md。这个命令是随 2026-07 那篇新规则一起推出的，正是为了帮人做「删减历史包袱」这件事。

### 8.3b Skills 怎么写：三层渐进式披露

skill 的整个架构就是一件事——**每层信息只在需要时才进上下文**：

| 层 | 内容 | 何时加载 |
|---|---|---|
| **1. 元数据** | YAML frontmatter，只有 `name` 和 `description` | 启动时预载入系统提示词 |
| **2. 主指令** | `SKILL.md` 正文 | Claude 判断相关时 |
| **3. 附加文件** | 从 SKILL.md 引用的其他文件 | 只在真正需要那个场景时 |

官方的类比很贴切：**像一本组织良好的手册——先目录，再具体章节，最后详细附录。**

几条要点：

- **`name` 和 `description` 是最关键的两个字段**，Claude 就靠它们决定要不要激活这个 skill。该触发时没触发，先改这两句，不是去改正文
- 定位是「让 Claude 需要时能找到信息的**轻量指引**」，不是详尽规程。**除非是极重要的领域，否则不要过度约束**
- 正文变臃肿就**拆文件**。互斥的场景分开放能省不少 token。官方例子：PDF skill 把填表说明放在单独的 `forms.md`，正文保持精简，**相信 Claude 只在真要填表时才去读它**
- 代码要写清楚是让 Claude **直接执行**的还是当**参考文档读**的
- **可以打包可执行代码**，理由很实在——用 token 生成来排序一个列表，比直接跑排序算法昂贵得多。确定性操作交给代码
- **从评测出发**：先测出能力缺口再补 skill，别预先臆想需求
- **让 Claude 帮你写**：skill 没触发时，让它反思**实际需要的是什么上下文**，比你去猜准得多
- **安全**：只装可信来源的 skill；来源存疑要审查打包文件、依赖，以及任何让它连外网的指令

结合第 6.2 节那条——为旧模型写的过度规定化 skill **可能主动降低** Fable 5 的输出质量。所以 skill 不是写得越细越好，**恰恰相反**。

### 8.3c 把手工复查固化成验证回路

判据很简单：**任何你不得不一遍遍手动执行的检查，都值得固化。** 尤其是通用 linter 抓不到的确定性规则——官方举的例子是「拒绝任何删除列却没有 backfill 步骤的 migration」。

四种部署模式：

| 模式 | 触发 | 适合 |
|---|---|---|
| **Standalone 独立** | 手动调用 | 不普遍适用的横切检查：安全扫描、无障碍审计、许可证核查 |
| **Embedded 内嵌** | 在产出 skill 的流程里自动跑 | 只属于某个特定任务的检查 |
| **Chained 串联** | 一个 skill 完成后调下一个 | 端到端交接；也是给**不可修改的 skill** 加验证的唯一办法 |
| **On every PR** | CI 上跑 | 把个人习惯变成团队基础设施，不再依赖作者自觉 |

一个容易踩的坑：**内嵌只对可编辑的 skill 有效**。内置或插件管理的 skill 会自动更新，你改的会被覆盖，那种情况必须用串联。

创建流程：记录你最常做的手工复查 → 像交给入职第一天的新同事那样写清步骤 → 先试内置的 `/verify` → 不够用再用 `/skill-creator` 或手写 → 在真实任务上验证它确实跑了 → 再尝试串联。

### 8.3d 四种循环（loop）

| 类型 | 命令 | 适合 |
|---|---|---|
| **Turn-based** | 手动逐轮 | 短任务 |
| **Goal-based** | `/goal get the homepage Lighthouse score to 90 or above, stop after 5 tries` | 有**可验证退出标准**、单轮不够的任务 |
| **Time-based** | `/loop 5m check my PR, address review comments, and fix failing CI` | 周期性工作、对接外部系统 |
| **Proactive** | `/schedule every hour: check #project-feedback for bug reports` | 定义明确的持续工作流：bug 报告、issue 分诊、迁移 |

控成本的关键：**确定性的工作用脚本做**（跑脚本比逐 token 推理便宜），周期匹配实际变化频率，大规模跑之前先试点。官方的结论是——**模型和 effort 档位属于循环成本上最大的几个杠杆。**

### 8.4 Claude Code 里最值钱的一条实践

官方最佳实践文档把一件事放在了最前面：**给 Claude 一个它自己能跑的检查。**

> Claude 在「看起来做完了」的时候就会停。如果没有它能运行的检查，「看起来做完了」就是唯一的信号，而**你自己就变成了验证回路**——每个错误都要等你去发现。给它一个能产出 pass/fail 的东西，这个回路就自己闭合了。

检查可以是：测试套件、构建的退出码、linter、把输出和 fixture 做 diff 的脚本、和设计稿对比的浏览器截图。

对照示例：

| 差 | 好 |
|---|---|
| `implement a function that validates email addresses` | `write a validateEmail function. example test cases: user@example.com is true, invalid is false, user@.com is false. run the tests after implementing` |
| `make the dashboard look better` | `[paste screenshot] implement this design. take a screenshot of the result and compare it to the original. list differences and fix them` |
| `the build is failing` | `the build fails with this error: [paste error]. fix it and verify the build succeeds. address the root cause, don't suppress the error` |

**如果检查还不存在呢？** 这是最常见的情况——遗留代码、覆盖率低、没人愿意碰。别跳过验证直接改，**把「造出这个检查」当成第一阶段**：先写**特征测试（characterization tests）**把当前行为钉住，**包括你怀疑是 bug 的行为也照原样断言**（这一点必须在 prompt 里明说，否则一个善意的执行者会顺手把 bug 修掉，那张网就废了）；网建好之后冻结它，重构阶段不许改测试；然后才开始重构。修 bug 属于**之后另一轮**的事——两件事混在一起，测试变红时你分不清是重构写坏了还是行为本来就该变。

另外一个很实用的技巧——**让 Claude 反过来面试你**：

```
I want to build [brief description]. Interview me in detail using the AskUserQuestion tool.

Ask about technical implementation, UI/UX, edge cases, concerns, and tradeoffs. Don't ask obvious questions, dig into the hard parts I might not have considered.

Keep interviewing until we've covered everything, then write a complete spec to SPEC.md.
```

然后**开一个全新会话**去执行这份 spec——新会话上下文干净，完全聚焦在实现上。官方说：**把 spec 打磨精确所花的时间，回报高于盯着实现过程看的时间。**

### 8.5 五种常见失败模式

| 失败模式 | 症状 | 解法 |
|---|---|---|
| **大杂烩会话** | 一个任务问到一半岔到别的，再岔回来 | 不相关任务之间 `/clear` |
| **反复纠正** | 纠正一次不对，再纠正还不对 | **纠正两次仍失败就 `/clear`**，带着学到的东西重写一个更好的初始 prompt |
| **过度规定的 CLAUDE.md** | 文件太长，重要规则被淹没 | 狠心修剪；模型本来就做对的事，指令直接删掉 |
| **信任但没验证** | 看起来合理但边界情况没处理 | 永远提供验证手段；**验证不了的东西不要发布** |
| **无边界探索** | 让它「调研一下」，结果读了几百个文件 | 收窄范围，或用 subagent 隔离探索 |

**关于对抗式审查的一条重要提醒**：让子智能体用干净上下文来审 diff 确实有效，但官方同时警告——**一个被要求「找缺口」的审查者，即使工作本身没问题，通常也会报出一些缺口，因为那正是它被要求做的事**。逐条去追每个发现会导致过度工程。所以审查 prompt 必须写清楚：**只标出影响正确性或既定需求的缺口，其余视为可选。**

---

## 9. 怎么练

### 9.1 学习路径

**第 1 周 —— 排毒**

把第 4 章那张表打印出来贴着。检查你手头现有的 prompt，逐条删掉已作废的东西。这一步的收益通常最大，因为大多数人的 prompt 里堆着一堆对现行模型有害的历史包袱。

**第 2 周 —— 练「说清楚」**

每写一个 prompt，用两个检验：

1. 给一个不懂这个任务的人看，他会照做吗？
2. **一个善意的人会怎么误解它？** Cat Wu 的原话：`you should think about the ways in which it could be misinterpreted by a well-intentioned human, in order to better understand how the model might interpret it`

**第 3 周 —— 练「减法」**

拿你写得最长的一个 prompt，尝试砍掉一半，跑同样的任务对比结果。你多半会发现质量没降。这一步是在建立对「脚手架变死重」的直觉。

**第 4 周 —— 练「往外层解决」**

每次想加一条规则时，先问：这个问题能不能不靠提示词解决？改工具设计？加确定性检查？调整信息呈现方式？

### 9.2 判断一份 prompt 好不好的六个问题

1. 有可判定的成功标准吗？
2. 每条约束都说了「为什么」吗？
3. 有没有句子删掉也不会出错？（有就删）
4. 一个善意的人会怎么误解它？
5. 这些内容真的需要**每次**都加载吗？（不需要就下沉到 skill / 按需加载）
6. 这条规则该由提示词管，还是该由工具 / schema / hook 确定性地保证？

### 9.3 一个持续有效的心法

> **模型每升一级，就回头删一遍你的提示词。**

Lance Martin 的表述是：随着 Claude 变强，harness 里的假设会过期，当初必要的脚手架变成死重、成为瓶颈。所以要**持续重新评估**。

这大概是这份文档里唯一一条不会过时的建议——因为它讲的正是「如何应对过时」。

---

## 10. 信源与推荐阅读顺序

完整信源清单、每条的读取深度、以及已知冲突的裁定，见 [SOURCES.md](../SOURCES.md)。下面只给阅读顺序。

### 建议的阅读顺序

1. **先读**：[The new rules of context engineering for Claude 5 generation models](https://claude.com/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models)（2026-07-24）—— 建立方向感，10 分钟
2. **再读**：[Best practices for Claude Code](https://code.claude.com/docs/en/best-practices)（官方文档，持续更新）—— 最大量的实操，如果你用 Claude Code，这是回报最高的一篇
3. **查阅**：[Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) + 你在用的模型专页 —— 当工具书查，不必通读
4. **理解原理**：[Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)（2025-09）
5. **听人怎么说**：[Fireside Chat: Cat Wu & Thariq](https://www.youtube.com/watch?v=uU5Gv2h8-9g)（2026-07）—— 有 [文字稿](https://simonwillison.net/2026/Jul/21/cat-and-thariq/)

### ⚠️ 刻意排除的材料

| 材料 | 排除理由 |
|---|---|
| [prompt-eng-interactive-tutorial](https://github.com/anthropics/prompt-eng-interactive-tutorial) | 仍以 **Claude 3 Haiku** 为目标模型，主推的技法已被推翻。**需要说公道话**：官方 overview 页至今仍把它列为「交互式学习者」的入口，所以它并未被官方废弃；但同一页明确写着 Prompting best practices 才是 `the living reference; start there`。当概念入门可以，具体技法一律以 living reference 为准 |
| ["AI prompt engineering: a deep dive"](https://www.youtube.com/watch?v=T9aRN5JkmL8)（2024-12） | 网上转载最多，但战术层面已被取代 |
| [Prompt engineering best practices for 2026](https://claude.com/blog/best-practices-for-prompt-engineering)（2025-11） | **部分内容与现行文档矛盾**：它推荐 prefill（已禁用）、称 XML 和 role prompting「不再必要」（文档仍在推荐）。总纲部分可看，这三处以文档为准 |
| 各类第三方「2026 终极指南」 | SEO 二手内容 |

---

## 附：核查记录

保留这一节，是因为它本身就是一条方法论：**关于快速变动的文档，"核查过"不是一个状态，是一个持续动作。**

### 内容核查

初稿完成后做过一轮独立的对抗式事实核查——把每一条英文片段、每一个数字、每一处模型归属都拿回原文逐字比对。改掉的地方包括：

- 修正了「强硬措辞会导致过触发」的适用范围，并补上 CLAUDE.md 的例外
- 修正「官方片段几乎全用 XML」的过度概括
- 修正 grep / glob 工具那段的发言人归属（Thariq，不是 Cat）
- 把 Simon Willison 的引语还原到原话强度
- 给「全新窗口优于 compaction」加回官方原有的限定词
- 把 `max_tokens` 64k 的建议限定到 `xhigh`/`max` effort 场景

### 完整性核查

后续几轮针对「是否真的全文读过每一页」，补上了这些缺口：

- **Fable 5 专页此前从未抓取**，却在模型对照表里被引用——现已读全文并补齐 Fable 5 章节，同时带出 `reasoning_extraction` 拒绝、「为旧模型写的 skill 往往过度规定化、可能降低输出质量」、进度真实性审计等一批此前完全缺失的内容
- **effort 页此前只看了摘要**——读全文后补上「低 effort 会减少工具调用次数」、「high 等于不传参数」、各模型起步值差异、ultracode 说明
- **overview 页此前没读**——补上第 3.0 节的前置条件（成功标准 / 可测试 / 初稿），以及「不是所有问题都该用 prompt 解决」
- **新增第 5b 节**——发现 Opus 5「删验证」与 Fable 5「加验证」看似矛盾，实为场景不同，这是原稿完全没处理的一处冲突
- **四篇信源此前只有薄摘要**：Agent Skills 工程文（其实从未抓取）、Agent Harness Design、验证回路、Loop engineering。补读后带出 skill 的三层渐进披露架构、缓存命中五原则、「让 Claude 自己编排」的三组实测数字、验证回路的四种部署模式、四类循环的具体命令
- **Console prompt improver 的链接已失效**（302 跳走），且该页仍在推荐**已被 API 禁用的 prefill**——这是个会让人直接踩坑的问题
- 按适用范围做了裁剪：移除所有更早模型（Opus 4.x、Sonnet 系列）的专属指引，只保留 Opus 5、Fable 5 和已验证的通用内容

### 一次真实的教训：文档在核查期间被改了

这一条值得完整记录，因为它比上面所有条目都更能说明问题。

第 6.3 节的 Opus 5 effort 起步值，原本写的是「编码 / agent 任务起步用 `xhigh`」，依据是官方 Opus 5 页当时的措辞：`for coding and agentic work, xhigh remains the recommended starting point`。

审查过程中有两次独立复核报告说该页写的是「从 `high` 起步」。当时对同一页做了三次直接抓取，三次都返回上面那句 `xhigh`；同时 effort 页也没有 Opus 5 专属章节。于是这两份报告被记为「无法复现」。

结果是**报告没错，页面改了**。现在这一页写的是：

> "**Efficiency at lower effort:** `low` and `medium` effort produce strong quality at a fraction of the tokens and latency of higher settings. Start with the default (`high`) and adjust based on your evals: use `low` and `medium` liberally as your primary control for token cost and response time wherever quality holds, and step up to `xhigh` for demanding coding and agentic work."

`remains the recommended starting point` 这句话已经不在页面上任何位置，effort 页也新增了 Opus 5 与 Fable 5 各自的推荐档位章节。本文和 skill 都已按新版改正。

**教训有两条**：

1. **实际差别是真的。** 不是措辞润色——一个普通编码任务的起步值从 `xhigh` 变成了 `high`，直接影响成本。
2. **对快速变动的文档，快照出错是常态，不是例外。** 涉及具体参数配置时，别只看二手整理（包括本文）。真要上生产，去原页面确认一遍——那是唯一会随模型更新的地方。

---

*本文的所有技术主张均可追溯到 [SOURCES.md](../SOURCES.md) 列出的官方信源。若与你在别处看到的说法冲突，请优先核对官方文档的发布 / 更新日期。发现分歧欢迎提 issue——以官方文档为准，不以本仓库为准。*
