# 2026 年的 Prompt 工程：规则从哪来

> 这是 `prompt-engineering` skill 的背景读物。skill 讲的是**怎么做**，这份文档讲的是**为什么是这样**——每条规则背后的官方依据、它推翻了什么旧习惯、以及几处各页面说法不一样的地方该怎么看。
>
> 快照日期 **2026-11-09**。其间有几类修正：官方页面本身改过（effort 建议）；一轮逐字复核纠正了两条片段的出处标注（那是本文这边的错，不是官方改的）；一个页面被整个撤下，带走了某条规则唯一的依据；三次新模型发布（Claude Fable 5.1，2026 年 9 月 1 日；Claude Opus 5.5，2026 年 9 月 22 日；Claude Fable 5.2 和 Mythos 5.2，2026 年 11 月 9 日）让所有模型专属的说法重新逐条裁定；2026-10-23 那一轮，一个月里官方页面本身被改了不少，大多没有公告；以及 2026-11-09 这一轮，按仓库所有者的要求删掉了所有找不到官方出处的内容，同时 Claude Code 最佳实践页已经整页重写。附录有记录。全部内容来自 Anthropic 官方文档、官方文章和支持站；只保留了一条员工公开发言，用到的地方都标明了；skill 里另有三条标明「用户规则，非官方」的做法，本文提到时同样标明。Claude 5 世代之前的专属指引已剔除，而不是留着当"应该还能用"。信源逐条列在 [SOURCES.md](../SOURCES.md)。
>
> **阅读方式**：第 1、2、3 章是理解框架；第 4 章是最重要的「排毒」章节（如果你是在 2026 年之前学的 prompt 工程，那一章会推翻你的一部分认知）；第 5 和 5b 章各讲一个单点：示例该不该给，验证指令按模型怎么写；第 6 章讲四个现行模型，以及怎么选模型、选 effort；第 7 章往后按需查阅。
>
> **适用范围**：官方现行的四个模型——**Claude Fable 5.2**（含 Mythos 5.2）、**Claude Opus 5.5**、**Claude Sonnet 5.5**、**Claude Haiku 5.5**，以及已验证与现行标准无冲突的通用技法。Claude Fable 5.1、Fable 5、Opus 5 和 Sonnet 5 都在官方的「Legacy models (still available)」列表里，本文只在它们是某条措辞或某个测量的出处、迁移的起点、或现行页面点名的对照（价格或基准对比）时提到。每条针对具体模型的说法分三类：当前页面写了的直接用；只在前代页面写过、当前页面既没重述也没撤回的，标明「在该模型上测得」；已撤回或与当前页面冲突的不收。第 6 章开头解释为什么这样分。更早模型的专属指引已剔除。
>
> **一句免责**：涉及具体 API 参数时，以官方文档的实时版本为准。文末附录记录了一次真实的教训——本文引用的某段官方措辞，在核查期间就被官方改掉了。

---

## 目录

1. 一句话说清发生了什么
2. 三条主线：prompt → context → harness
3. 地基：至今有效的六件事
4. **排毒：八条已经作废的旧习惯**
5. 示例到底还要不要给
5b. 验证指令：各模型的官方说法不一样
6. 四个现行模型，以及怎么选模型和 effort
7. Agent 与长任务：上下文工程
8. Claude Code 场景：该把话写在哪
9. 怎么练
10. 信源与推荐阅读顺序

附：核查记录

---

## 1. 一句话说清发生了什么

**过去两年 prompt 工程教你的很多东西，正在被 Anthropic 自己推翻。**

最硬的一个证据：2026 年 7 月 24 日 Claude Code 团队的一篇文章说，他们为 Opus 5、Fable 5 这类模型把 Claude Code 的系统提示词**删掉了 80% 以上，在编码评测上没有可测的损失**。

这不是个孤立事件。另外两条线索指向同一个方向：

- 随着 Claude 变强，harness 里当初必要的脚手架变成死重，可能拖累模型的表现（Lance Martin，Claude Platform 团队，2026-04 的文章）
- Claude Fable 5.2 的 prompting 专页（2026-11）写明，为早先模型写的一些指令在它身上 `raised cost, caused side effects, or no longer had an effect`

方向可以概括成一句话：

> **从「精心构造指令」转向「精心构造上下文与工具」。**

为什么？因为模型的判断力上来了。过去你要写 `Never write multi-paragraph docstrings or multi-line comment blocks — one short line max`，是因为模型自己判断不好；现在你写 `Write code that reads like the surrounding code: match its comment density, naming, and idiom`，它能自己看出来（这两句分别是那篇文章里旧、新两版 Claude Code 系统提示词的原文）。

前一种写法的代价，文章自己说了：这类规则对一部分请求本来就是错的——文档写多少注释，用户可能有自己的偏好；特别复杂的代码可能真需要多行注释块。他们读自己的使用记录时，看到同一个请求里 system prompt、skill 和用户请求互相打架，比如 "leave documentation as appropriate" 和 "DO NOT add comments" 同时出现，Claude 得先在这些重叠、冲突的信息之间多想一层，才能决定怎么做。

Fable 5.2 的专页给了更具体的一面：规则会被照字面执行，原话是 `the model applies them as written`。官方在没人审批的模拟任务里测过：一条把 `sending messages to people outside this conversation` 列为需要审批的规则，让模型压下了大约一半发给同事和其他团队的例行请求。规则写多宽，它就执行多宽。

---

## 2. 三条主线：prompt → context → harness

理解 2026 年的图景，需要区分三个层次。

### 2.1 Prompt Engineering（提示词工程）

为最优结果而编写和组织给 LLM 的指令。**一次性的写作行为**。适用于单轮、任务明确的场景。

### 2.2 Context Engineering（上下文工程）

在 LLM 推理过程中策划并维护最优 token 集合的一整套策略，包括所有落在提示词之外的信息。

关键区别在于：**提示词工程是一次性写作，上下文工程是迭代的——每一轮你都要重新决定往模型里塞什么。**

系统指令、工具、外部数据、消息历史、记忆——全都算。多轮 agent 场景下这个层次比提示词层次重要得多。

核心概念是 **context rot（上下文腐坏）**：模型有有限的注意力预算（attention budget），每加一个 token 就消耗一点。Transformer 的注意力是平方复杂度，上下文越长，注意力越被稀释。所以目标不是「把知道的都告诉它」，而是：

> **the smallest possible set of high-signal tokens that maximize the likelihood of some desired outcome.**

也就是：找到能最大化达成目标概率的、最小的高信噪比 token 集。

### 2.3 Harness / Loop Engineering（框架与循环工程）

围绕模型的软件基础设施：循环、工具、上下文管理、护栏。Lance Martin 的核心洞察是：

随着 Claude 变强，harness 里的假设会**过期**，当初必要的脚手架变成死重。模型能力每上一个台阶，都要回头问：**我能停止做什么？**

**实际启示**：如果你发现自己在 prompt 里写了一大堆规则，先停下来问——这个问题是不是应该在更外层解决？换个工具设计？加个确定性的检查？还是压根不需要管？

---

## 3. 地基：至今有效的六件事

### 3.0 先说前置条件

官方 prompt engineering 入门页开宗明义地讲了一件很多人跳过的事——**调 prompt 之前你应该已经有**：

1. 清晰的成功标准
2. 能对照标准做实证测试的方法
3. 一版初稿

没有前两项就去改 prompt，等于闭着眼睛调参。（初稿不必自己从零写，官方提供了 [metaprompt recipe](https://colab.research.google.com/github/anthropics/claude-cookbooks/blob/main/misc/metaprompt.ipynb) 帮你生成一版。）

同一页还有一句同样重要的话：**不是每个成功标准、每个失败的 eval 都最适合用 prompt 工程来解。** 官方举的例子是延迟和成本：有时换个模型更容易改善。

这条心态很关键：**prompt 是工具箱里的一把工具，不是唯一的锤子。** 后面第 6、8 章会反复出现同一个模式：有些问题该在模型和 effort 的选择、hook 或权限配置这些层面解决，而不是在 prompt 里再加一句。Claude Code 最佳实践说到必须发生或绝不能发生的事时，原话是 `A prompt alone can't guarantee either of those`。

---

新规则推翻的是「过度约束」，不是「说清楚」。以下这些官方文档里仍然在教，而且是现行版本。

### 3.1 清晰直接

把 Claude 当成一个**聪明但刚入职的新同事**——他很强，但不了解你们的规范和工作流。

**关键检验法**：把你的 prompt 给一个对这个任务所知甚少的同事看，让他照着做。如果他会困惑，Claude 也会。

想要超出基本要求的表现，就明说，不要指望模型从模糊的提示里推断。

- ✗ `Create an analytics dashboard`
- ✓ `Create an analytics dashboard. Include as many relevant features and interactions as possible. Go beyond the basics to create a fully-featured implementation.`

### 3.2 给上下文，而且要说「为什么」

解释你为什么这么要求，模型能从解释中泛化。Fable 5.2 专页对积攒下来的规则给的也是这个方向：`rewrite absolute rules so that they state their scope and reason`。

- ✗ `NEVER use ellipses`
- ✓ `Your response will be read aloud by a text-to-speech engine, so never use ellipses since the text-to-speech engine will not know how to pronounce them.`

注意第二种写法里其实还是有 "never"，但它挂在一个**原因**上。模型知道了原因，遇到没被明确覆盖的边界情况时也能自己判断。这就是差别。

### 3.3 用 XML 标签做结构

> ⚠️ 有一篇 2025 年 11 月的官方博客说 XML 标签 "less necessary with modern models"。**现行文档与此相矛盾**，裁定见第 10 章阅读顺序第 6 条和 [SOURCES.md](../SOURCES.md) 的「Known conflicts」。

XML 标签能减少误读，尤其是 prompt 混合了指令、背景、示例和变量输入时。官方原文：

> "XML tags help Claude parse complex prompts unambiguously, especially when your prompt mixes instructions, context, examples, and variable inputs. Wrapping each type of content in its own tag (for example, `<instructions>`, `<context>`, `<input>`) reduces misinterpretation."

用一致且有描述性的标签名，有层级时嵌套。

Markdown 标题（`## Tool guidance`、`## Output description`）同样是官方认可的分区方式，与 XML 可以混用——这一点出自 Anthropic 的上下文工程文章。

### 3.4 给角色

在 system prompt 里设定角色能聚焦行为和语气。**一句话就有效果**：

```
You are a helpful coding assistant specializing in Python.
```

> ⚠️ 同一篇 2025-11 博客还说 `heavy-handed role prompting is often unnecessary`——现行文档仍在推荐 role prompting，裁定同见第 10 章。不过那篇博客有个有价值的补充：**比起贴一个身份标签，说清要关注什么视角往往更有效**。博客自己的例子是 `Analyze this investment portfolio, focusing on risk tolerance and long-term growth potential`，而不是给它安一个角色。两者不冲突，可以叠加。

### 3.5 长上下文的摆位

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

### 3.6 说要什么，而不是不要什么

这一条贯穿所有官方材料。

- ✗ `Do not use markdown in your response`
- ✓ `Your response should be composed of smoothly flowing prose paragraphs.`

还有个巧妙的补充手段：**让你的 prompt 风格贴近你想要的输出风格**。比如 prompt 里去掉 markdown，可以减少输出里的 markdown。

Opus 5 的页面在播报风格这个具体场景下也说了同样的话：`Positive examples of the communication style you want tend to be more effective than instructions about what not to do.`

---

## 4. 排毒：八条已经作废的旧习惯

这一章是这份文档存在的主要理由。以下每条都注明了「旧说法出处」和「现行依据」，可以自己去核。

### 4.1 ❌ Prefill（预填充助手回复）——已被 API 禁止

**旧说法**：在 assistant 消息里预填 `{` 来强制 JSON 输出，或预填 `Here is the requested summary:` 来去掉开场白。这在 2025 年 11 月的官方博客里还被列在 "Advanced prompt engineering techniques" 一节。

**现状**：**四个现行模型均不支持。** 官方原文：

> "Starting with Claude 4.6 models and Claude Mythos Preview, prefilled responses (providing a partial assistant message for Claude to continue from) on the last assistant turn are no longer supported. Requests with prefilled assistant messages to these models return a 400 error."

Haiku 5.5 的迁移指南另外写明：Haiku 4.5 在关掉 thinking 时还接受 prefill，Haiku 5.5 即使关掉 thinking 也返回 400。

**为什么取消**：官方给的理由是，模型的智能和指令遵循能力已经进步到大多数 prefill 用例不再需要它。

**替代**：

| 原用途 | 现在怎么做 |
|---|---|
| 强制 JSON / 固定结构 | **Structured Outputs** 功能；或直接把 schema 说清楚（你提出要求时，新模型能可靠匹配复杂 schema，配合重试尤其如此） |
| 分类任务 | 带 enum 字段的 tool，或 structured outputs |
| 去开场白 | `Respond directly without preamble. Do not start with phrases like 'Here is...', 'Based on...', etc.` |
| 绕过误拒绝 | 官方的说法是 Claude 现在的拒绝判断准多了，正常在 user 消息里说清楚即可 |
| 续写被截断的回复 | 把上文放进 user 消息里让它接着写；或者在错误处理里、没有体验损失时直接重试 |

### 4.2 ❌ `budget_tokens` 控制思考预算——已废弃

**旧写法**：`thinking: {type: "enabled", budget_tokens: 10000}`

**现状**：四个现行模型都不接受手动 extended thinking，设置 `budget_tokens` 返回 400。

**新写法**：

```python
thinking={"type": "adaptive"},       # or omit it: the two are equivalent
output_config={"effort": "medium"},  # API default on Opus 5.5 and Haiku 5.5; "high" on Fable 5.2 and Sonnet 5.5
```

**能不能关掉 thinking，四个模型不一样**：Fable 5.2 和 Opus 5.5 的 thinking 始终开启，`{"type": "disabled"}` 和 `{"type": "between_tools"}` 都返回 400；Sonnet 5.5 的 `disabled` 在每个档位都返回 400，想关掉前置思考要传 `between_tools`，只在 `high` 及以下可用；Haiku 5.5 在 `high` 及以下可以 `disabled`，`xhigh`/`max` 下返回 400。

**adaptive thinking 是什么**：模型自己决定何时思考、思考多少，依据两个因素——`effort` 参数和查询复杂度。简单问题直接答，复杂问题多想；Fable 5.2 和 Mythos 5.2 是例外，在 `high` 及以上几乎每次回复前都思考。官方内部评测显示 **adaptive thinking 可靠地优于手动 extended thinking**。

**主要的深度控制手段是 `effort`，不是 prompt 里的话。** 这一条在四个模型上都写明了：Fable 5.2 页面说 `answer directly without deliberating` 这类句子在 `medium` 及以上不能让它停止思考，官方给的修法是 `Remove these lines and lower the effort level`；Sonnet 5.5 页面说在 system prompt 里叫它少思考 `doesn't reliably reduce its thinking`；Haiku 5.5 页面说叫它直接回答并没有让它停止思考；Opus 5.5 页面说降 effort 比 prompt 指令更可靠。这就是 skill 3.10 的依据。

需要硬上限时用 `max_tokens`（注意它是**思考 + 正文的总上限**）。

### 4.3 ❌ 用强硬措辞防止「欠触发」——已经反向

**旧写法**：`CRITICAL: You MUST use this tool when...`

**为什么当年这么写**：早期模型对工具的触发不够积极，需要加压。

**现状**：prompting 最佳实践页说，Claude Opus 4.5 和 Opus 4.6 对 system prompt 更敏感，为防欠触发写的 prompt 在它们身上可能过度触发，修法是把激进的措辞改软：`Use this tool when...` 就够了。针对 Opus 4.6，同一页还把 `Default to using [tool]` 改成 `Use [tool] when it would enhance your understanding of the problem`，并说 `If in doubt, use [tool]` 会导致过度触发。这几句点名的是 Opus 4.5 和 4.6，现行四个模型的页面没有重述，所以先看看你的 prompt 是否还需要旧写法。

Claude Code 团队自己的系统提示词也是这样：为 Opus 5、Fable 5 这类模型删掉了 80% 以上，在编码评测上没有可测的损失。

**连 CLAUDE.md 里的强调也不再推荐了。** 本文早先的版本在这里留了一个例外：Claude Code 最佳实践曾允许只给 Claude 老是跳过的那一行加 "IMPORTANT"。重写后的页面把它反了过来：某一行 Claude 老是忽略时，`adding IMPORTANT probably won't fix it`——要么文件长到那一行被淹没了，该缩短文件；要么它是不能有例外的规则，该做成 hook。同一页的「What you can stop doing」表把给老被漏掉的指令加 IMPORTANT 或全大写这一条，列为可以停掉的旧做法，理由是：

> `Current ones follow a plainly worded line, and piling on emphasis mostly makes Claude over-apply the rule.`

**新写法**：平实的一句陈述。真观察到工具欠触发，再针对性地写一句什么时候该用它。

### 4.4 ⚠️ 一条验证规则套所有模型——各页面的说法其实不一样

**旧建议**：通用文档至今仍写着 `Before you finish, verify your answer against [test criteria]`。本文早先的版本反过来，把 Opus 5 页面要你删掉验证指令的那句当成招牌规则，又在第 5b 节把它和 Fable 5 页面的周期自查调和成一条通用规律。

**现状**：最佳实践页仍然把自检当作通用建议，Opus 5 是它唯一点名的例外，原话 `remove these instructions rather than rewriting them`。这一句点名的是现在已经是 legacy 的 Opus 5；Opus 5.5 的页面对过度验证既没重述也没撤回；Claude Code 文档说在 Fable 模型上，提醒它测试或检查通常没有必要；Sonnet 5.5 和 Haiku 5.5 的页面则反过来，各自给了一段验证段落，见到它在低档位没跑检查就报告完成时加进 system prompt。四个现行模型四种说法，所以 skill 3.4 按模型分别列出原话，不再合成一条规则。逐条对照见第 5b 节。

这是全文最清楚的一个例子，说明**改个模型名不等于完成了一次模型升级**。

### 4.5 ❌ 堆砌「绝不允许」式约束

**旧习惯**：用 `NEVER`、`always` 把每一条边界写死，怕模型越界。

**现状**：最佳实践页的原则是 `Tell Claude what to do instead of what not to do`，并且要给原因（第 3.2、3.6 节）。第 1 章那篇文章讲了为什么可以这么做：旧的绝对规则对一部分请求本来就是错的，而新模型 `have better judgement and can handle these decisions well without explicit rules`。Claude Code 最佳实践从另一个方向说了同一件事：`Almost every guideline has exceptions`，眼前有实际代码时，Claude 通常比你事先写的规则判断得更好。

Fable 5.2 专页给了这条最具体的测量：它照规则的字面执行，另起一行补的例外压不过写死的规则。官方的模拟任务里，负责人已经批准了某一步：在一条 "never" 规则下，模型大约十次里只做一次；另起一行写明已批准的步骤可以做，几乎没有改变这个比例；把例外写进规则本身（规则里写明先拿到批准）之后，十次里大约做九次。所以它给的修法是把绝对规则改写成带范围和原因的，需要例外时写进规则本身。

真正不能有例外的事，交给 hook 和 deny 规则，见第 8.2 节。

### 4.6 ❌ 在多处重复同一条指令

**旧习惯**：怕模型看不见，system prompt 里说一遍，tool description 里再说一遍，文档里又说一遍。

**现状**：第 1 章那篇文章说，早先的模型有时需要重复的指令，或者更听上下文末尾而不是开头的指令；现在他们把重复的删掉了——工具的使用说明**只保留在 tool description 一处**，不再在 system prompt 里再写一遍。

**不算重复的**：什么时候该用这个工具。强制 `tool_choice` 在三个现行模型上被拒之后（本章末尾「补充」），官方给的替代就是在 prompt 里写明什么时候用这个工具，所以 skill 3.7 把触发时机单列出来，不算重复。

### 4.7 ❌ 手工维护 CLAUDE.md 作为记忆

**旧做法**：用 `#` 热键把要点写进 CLAUDE.md。

**现状**：Claude Code 现在会自动保存与工作和你有关的记忆，不再需要手工维护那部分（同一篇文章）。

（注意：CLAUDE.md 本身没有作废——它仍然是放 Claude 读代码得不到的东西的地方：确认工作用的命令、代码里有误导性的地方、不在仓库里的决定。作废的是「拿它当记事本手工记录」这个用法。见第 8 章。）

### 4.8 ❌ 用简化的 markdown 说明书当规格

**旧做法**：写一份简化的 markdown 说明，**用它代替**真实材料交给模型。

**新做法**：给**丰富的引用**——实际代码、详细的测试套件、另一个代码库里要移植的那个函数、HTML artifact。文章的理由是代码文件给 Claude 的是清楚、高保真的指令，用的是它非常熟悉的语言；一个 HTML 的设计 mockup 通常比对设计的文字描述或截图效果好。Claude Code 最佳实践的说法一样：手上已经有它需要的东西，就直接交给它，不要描述。

要说清楚这条**没有**否定什么：用户读过并确认的 spec 是另一回事。第 8.5 节讲的正是产出这样一份 spec，它之所以有效，是因为它的身份是一份**约定**，而不是代码的有损替代品。

### 补充：几个 API 层面的变化

- **采样参数（`temperature` / `top_p` / `top_k`）设非默认值，四个现行模型都返回 400**——thinking 页列出的模型里包括这四个，原文是这类值 `return a 400 error on every request, regardless of whether thinking is used`；Fable 5.2 的 overview 写的是 `Omit temperature, top_p, and top_k`。旁边还有两个坑：`top_p` 默认是 `0.99`，所以 `top_p: 1` 会被拒，同时传 `temperature` 和 `top_p` 也会被拒；从 API 版本 `2026-10-21` 起，这三个参数取任何值都会在所有模型上返回 400。别再靠采样参数求多样性，改用提示词——做设计时，Sonnet 5 页面推荐的做法是让模型先提几个不同方向，**由用户挑一个**，再落地它（Sonnet 5.5 页面写明 Sonnet 5 的做法仍是合理起点）。
- **强制工具调用在三个现行模型上没有了，Haiku 5.5 是例外。** `tool_choice` 设为 `any` 或 `tool` 类型在 Fable 5.2、Mythos 5.2、Opus 5.5 和 Sonnet 5.5 上返回 400；Haiku 5.5 接受，但这时回复直接以工具调用开头。（本文早先引过 Fable 5.1 what's-new 页给的原因；那一页现在重定向到 Fable 5.1 的 overview，现行页面只写报错、不给原因，所以不再引。）保持 `auto`，要 schema 合规的 JSON 就用 strict tool use 或 structured outputs（用了客户管理加密密钥（CMEK）的组织里，Fable 5.2 和 Mythos 5.2 这两样都不可用，要在自己代码里校验），并在 prompt 里写明什么时候该用这个工具。对从 Opus 5、Fable 5 或 Sonnet 5 迁过来的代码，这是破坏性变更。
- **从 Claude 4.7 之前的模型迁过来，同样的文本 token 数会变多。** Haiku 5.5 的迁移指南说比 Haiku 4.5 多约 30%，Opus 5.5 的迁移指南说比 Opus 4.7 之前的模型最多多约 35%。旧的 `max_tokens` 和成本估算要重看。

### 一张按模型的硬规则表

2026-11-09 这一轮之前，本文只覆盖 Opus 5.5 和 Fable 5.1，两者在这些点上完全一样，一句「两个模型都返回 400」就够了。Sonnet 5.5 和 Haiku 5.5 进入范围之后，这句话在三处不再成立，所以 skill 第 3 节改成了按模型的表：

| | Fable 5.2 / Mythos 5.2 | Opus 5.5 | Sonnet 5.5 | Haiku 5.5 |
|---|---|---|---|---|
| prefill | 400 | 400 | 400 | 400 |
| `budget_tokens` | 400 | 400 | 400 | 400 |
| 关掉 thinking | 400 | 400 | 400；改用 `between_tools`，限 `high` 及以下 | `high` 及以下可以；`xhigh`/`max` 返回 400 |
| 非默认采样参数 | 400 | 400 | 400 | 400 |
| `reasoning_extraction` 拒绝 | 有（官方点名的是 Fable 5.2） | 有 | 有 | 没有这个类别 |
| 强制 `tool_choice` | 400 | 400 | 400 | 接受 |

### 一张排毒总表

| 旧习惯 | 现状 | 替代 |
|---|---|---|
| prefill 预填充 | 4.6+ 返回 400，四个现行模型都是 | Structured Outputs / 明确指令 |
| `budget_tokens` | 返回 400 | 只用 `effort`，`thinking` 不传或传 `adaptive` |
| prompt 里叫它少想、别想 | 官方测试里不能让 Fable 5.2 在 `medium` 及以上停止思考；Sonnet 5.5、Haiku 5.5 上也不可靠 | 降 effort |
| `CRITICAL: You MUST`（防工具欠触发） | 已不需要；CLAUDE.md 里加 IMPORTANT 也列进了可以停掉的旧做法 | `Use this tool when...`；不能有例外的规则做成 hook |
| 一条验证规则套所有模型 | 各模型页面说法不同 | 按模型查（第 5b 节） |
| 堆砌 `Never do X` | 规则被照字面执行，另起一行的例外几乎不起作用 | 正面描述 + 写明范围和原因 |
| 多处重复指令 | 冗余 | 单一权威来源（触发时机除外） |
| 手工 CLAUDE.md 记忆 | 已有自动记忆 | 自动记忆 |
| 用 markdown 说明**代替**真实材料 | 不如代码、测试、mockup 这类引用 | 代码 / 测试 / artifact |
| 用 temperature 求多样性 | 四个现行模型都返回 400 | 让模型先提几个方向，你挑一个再落地 |
| 强制 `tool_choice` | Fable 5.2、Opus 5.5、Sonnet 5.5 返回 400；Haiku 5.5 接受 | `auto` 加 strict tool use 或 structured outputs；写明什么时候该用工具 |

---

## 5. 示例到底还要不要给

本文早先的版本在这里把文档的说法和 Claude Code 团队一位员工的公开发言并列，解读成「看起来打架、其实各管一块」，并推出过一条「agent 系统提示词不给示例」的规则。那位员工的话不是官方页面，那条规则和那个解读都在 2026-11-09 这一轮按仓库所有者的要求删掉了（附录有记录）。现在这一节只讲官方信源怎么说。

prompting 最佳实践页的说法很直接：`Examples are one of the most reliable ways to steer Claude's output format, tone, and structure.` 它建议 `Include 3–5 examples for best results.`，并给了三条要求：**相关**（贴近真实用例）、**多样**（覆盖边界，别让模型学到意外的模式）、**结构化**（`<example>` 包裹，多个时外面再套 `<examples>`）。拿不准自己的示例好不好，还可以让 Claude 帮你看：`You can also ask Claude to evaluate your examples for relevance and diversity, or to generate additional ones based on your initial set`。

| 你的目标 | 怎么给 | 依据 |
|---|---|---|
| 让输出**落进一个固定的框**——固定 JSON 形状、固定文风、固定报告结构 | 3–5 个，多样化，`<example>` 包裹。格式能用 schema 管时，Structured Outputs 是另一条路 | 最佳实践页 |
| 想演示**推理方式** | 给完整的例题，原话 `Present each example as a problem, the method to apply, and the expected answer.`；它们会影响模型在自己的思考块里怎么处理同类问题。**不要**预留一个让模型填写的 `<thinking>` 区段 | 最佳实践页；refusals 页把让模型在回答前填写的 `<thinking>`、`<reasoning>` 或草稿区段列为 `reasoning_extraction` 拒绝的常见来源（Fable 5.2、Opus 5.5、Sonnet 5.5 上有这个类别，Haiku 5.5 没有） |

教它怎么用工具时，skill 先改接口：第 1 章那篇文章（写的是 Opus 5、Fable 5 这一代）说 `With our newest models, we've found that giving examples actually constrains them to a certain exploration space`，并建议改为把工具、脚本和文件的参数设计得更有表达力。

---

## 5b. 验证指令：各模型的官方说法不一样

这是早先版本里另一处被本文「调和」过的地方。当时的说法是：删掉「让模型自己再看一遍」，保留「换一个干净上下文来查」；在 Opus 5.5 上先实测再删；Fable 长跑要按周期自查。这套调和是本文自己的读法，没有哪个页面这样写，在 2026-11-09 这一轮删掉了。现在 skill 3.4 只按模型列各页面的说法：

| 模型 | 官方怎么说 | 出处 |
|---|---|---|
| 通用建议 | 让模型自检，`This catches errors reliably, especially for coding and math` | 最佳实践页 |
| Opus 5（legacy） | 上面那条建议唯一点名的例外：它不用提示就会验证自己的工作，沿用来的验证指令可能造成过度验证，`remove these instructions rather than rewriting them` | 最佳实践页；Opus 5 专页 |
| Opus 5.5 | 对过度验证既没重述也没撤回。无人值守长跑另有一条**完成度**检查：事先写明完成条件，然后 `have a separate, smaller model check the conversation against it at each end of turn`——它查的是有没有做完，不是质量 | Opus 5.5 专页 |
| Fable 模型（在 Claude Code 里） | `Skip the verification reminders: it verifies its own work with less prompting, so reminders to test or check are usually unnecessary` | Claude Code model-config「Work with Fable」 |
| Fable 5.2 | 在 `xhigh`、`max` 这类高档位下观察到它用子智能体复查自己的工作（Fable 5.1 在同样任务上不会）；同一段提醒子智能体可能占会话成本的很大一部分 | Fable 5.2 专页 |
| Sonnet 5.5 | 编码时一般会在报告完成前检查；`low` 下有时没跑能检验改动的检查就报告完成 → 片段 4.46。`xhigh`、`max` 下反过来，做完后可能自己开几轮复查和验证 → 片段 4.42 | Sonnet 5.5 专页 |
| Haiku 5.5 | `low` 和 `medium` 下有时没跑检查就报告代码改动完成 → 片段 4.46 | Haiku 5.5 专页 |

**为什么不合成一条规则。** 这些是对不同模型的陈述，而通用 prompting 页对点名了模型的技巧要求 `treat it as measured on that model and re-check it against your own evals before applying it to another`。把 Opus 5 的结论套到 Opus 5.5 上，或者把 Sonnet 5.5 的验证段落套到 Fable 5.2 上，都是把一个模型上的测量当成另一个模型的事实。

Fable 5 页面建议长自主运行时按间隔自查，并说独立的、干净上下文的 verifier 子智能体往往优于自我批评。Fable 5.2 的页面没有重述，而 Claude Code 文档对 Fable 模型现在写的是上表那句 `Skip the verification reminders`，所以 skill 不再把周期自查当成对 Fable 5.2 的建议。

**和模型无关的一条：给它一个能跑的检查。** Claude Code 最佳实践要的不是「提醒它再看一遍」，而是一个结果是 pass 或 fail 的东西，并要它给出证据（第 8.4 节）。该页的「What you can stop doing」表也把检查和叫它继续做下去的那类话分开了：`Giving Claude a check to pass is different, and still worth doing.`

---

## 6. 四个现行模型，以及怎么选模型和 effort

官方现行阵容，成本页原文：`From lowest to highest cost and capability, the current models are Claude Haiku 5.5, Claude Sonnet 5.5, Claude Opus 5.5, and Claude Fable 5.2 (the frontier model)`。但能力最高的那个和默认该选的那个不是一回事（6.5 节），每个模型也各有页面写明的倾向和对应的 prompt 修法。

### 6.0 先说每条说法怎么归档

通用 prompting 页对所有点名模型的技巧有一条提醒：

> "Where a technique names a specific model, treat it as measured on that model and re-check it against your own evals before applying it to another."

每个后继模型的页面又各自说了一句前代的做法还能不能用：

- **Opus 5.5**：`Existing Claude Opus 5 prompts should perform well without changes, and the patterns in Prompting Claude Opus 5 remain a reasonable starting point.` 迁移指南另有一句 `Instructions tuned for Claude Opus 5's behavior might no longer be needed`。
- **Sonnet 5.5**：`Existing Claude Sonnet 5 prompts should perform well without changes`，后半句和 Opus 5.5 的一样。
- **Haiku 5.5**：`Existing Claude Haiku 4.5 prompts should perform well without changes.`
- **Fable 5.2**：**没有这样的话**。它写的是：

> "When you migrate to Claude Fable 5.2, review your prompts. In Anthropic's testing, some instructions written for earlier models raised cost, caused side effects, or no longer had an effect."

迁移指南那句 `Migration is mostly drop-in` 讲的是 API 层面。

两条合起来就是 skill 的三类归档：当前页面写了的直接用；只在前代页面写过、当前页面既没重述也没撤回的，放进「在该模型上测得」的表，用之前在自己的评测上复核；已撤回或与当前页面冲突的不收。沿用句只说前代的*prompt 做法*仍是合理起点，没说前代的*行为判断*在新模型上仍然成立，所以就连 Opus 5 的倾向表也标的是「在 Opus 5 上测得」。而 Fable 5.2 连沿用句都没有，Fable 5.1 和 Fable 5 页面上的一切就都成了「在该模型上测得」的材料。

### 6.1 Claude Opus 5.5（以及在 Opus 5 上测得的部分）

2026-09-22 发布，定位是长时间运行的 agentic 编码与知识工作。**默认 effort `medium`**（API 和 Claude Code 都是），Opus 5 是 `high`；adaptive thinking 始终开启且无法关闭；1M token 上下文；最大输出 128K；$4 / $20 每 MTok，对比 Opus 5 的 $5 / $25。官方说它生成输出 token 比 Opus 5 快 30% 以上，而且完成同一个任务倾向于用更少的 token。选型矩阵把它定为多数工作负载的起点；Claude Code 新会话在多数计划和 provider 上默认跑的也是它，档位 `medium`。

**要记住的那个数字**：官方测试中，**Opus 5.5 在 `medium` 达到或超过 Opus 5 在 `high`** 的编码与知识工作评测成绩。所以旧的 `high` 设置对应的是新的默认档，页面也要你 `test several levels against your own evals rather than carrying over the setting you used on Claude Opus 5`。

| Opus 5.5 页面写明的倾向 | 怎么处理 |
|---|---|
| 同一档位下每回合往往思考得比 Opus 5 多，`xhigh` 和 `max` 尤其明显 | 重跑扫描；只把 `xhigh`、`max` 留给**实测**到收益的工作。想少思考，先降 effort，再考虑 prompt 指令 |
| 工具调用之间的文字以 **progress-update `thinking` 块**返回，不是 `text` 块，默认 `display` 下内容为空 | 设 `display: "updates"`。只渲染 `text` 块的客户端可能在整个长 agentic 回合里显得安静，而且不报任何错 |
| 有些进度汇报是**以文字而不是工具调用结束回合**的 | 无人值守的循环不能把它当成任务完成：保持一份清单，并发一条简短的消息点名还有哪些没做完；自动续跑两三次就停 |
| 往往上手很快；多应用任务描述松散时会漏掉请求没点名的上下文 | 一句话让它在动手前先广泛探索 |
| 对已过去多少时间这类信息非常敏感 | 在多智能体 harness 里给出任务的时间预算，并在每条消息末尾报告已用时间，它会为了在预算内完成而调整节奏 |
| 在 chat 里回答前会长时间思考，后续追问时有时还会回头重看早先的答案 | 考虑删掉让它回答前仔细思考的指令；希望它把已答过的当定论时，加两句（希望它持续复查早先工作的场景别加，比如长篇分析、或后面一步可能暴露前面错误的 agentic 任务；如果这一点重要，测一下它是否不再主动指出自己早先的错误）。这两句用在 Fable 5.2 上有明确记载的副作用，见 6.2 |
| 对粘贴内容里的指令很稳健——**前提是你告诉它哪段是粘贴来的** | 给粘贴块套上带同一个随机 id 的标签，并在 system prompt 里说明这个约定 |
| 不借助工具读图表、流程图和截图就比 Opus 5 准得多 | 重新检验为更早模型搭的视觉脚手架，可能已经不需要了 |
| 跑的分类器包括生物、网络安全、推理抽取——这三类里只有生物对**从 Opus 5 过来的人是新增的** | 处理 `stop_reason: "refusal"`；注意服务端 fallback **不会**重试 `reasoning_extraction`。要它简短解释答案、或概括做了哪些操作，仍然可以 |

**四项破坏性变更**会打到已经跑在 Opus 5 上的代码：thinking 无法关闭、强制 `tool_choice` 返回 400、thinking 块同时和模型与对话绑定、以及在 Claude API 上旧的 `computer_20251124` 返回 400（Amazon Bedrock 上仍接受），要改用 `computer_toolset_20260801` 工具集。

#### 在 Opus 5 上测得、Opus 5.5 没有重述的部分

下面这些都出自 Opus 5 页面。凭 Opus 5.5 页面那句沿用句，这些做法仍是合理起点；但倾向本身在 Opus 5.5 上是否仍然存在，官方两边都没说，所以它们是**在 Opus 5 上测得的**，其中两条还与 Opus 5.5 有明确记载的行为方向相反。

| 倾向（Opus 5） | 你要做的 |
|---|---|
| 用户可见回复默认偏长 | 显式要求简洁。**注意：降 effort 不能可靠缩短可见输出**——effort 控制的是想多少，不是说多少。另一面是 Opus 5.5 `tends to complete the same task with fewer tokens`，所以先实测，别默认这个问题还在 |
| Agent 场景播报频繁 | 明确描述你要的播报节奏。在 Opus 5.5 上有记载的问题**正好相反**——播报以默认为空的 `thinking` 块返回，回合显得安静。加任何东西之前先查 `thinking.display` |
| 写到磁盘的文档往往也比前代长 | 单独加长度校准指令 |
| 不用提示就会自我验证 | 最佳实践页只对 Opus 5 说 `remove these instructions rather than rewriting them`，见第 5b 节 |
| 可能自行扩大任务范围 | 窄任务要显式约束范围 |
| 更积极派生子智能体 | 显式设上限；用在小任务上会成倍增加成本和时间 |
| 更爱口头纠正自己之前的说法 | 限定为只在会影响用户决策时才说 |
| 代码审查时可能会**字面执行** `only report high-severity issues` | 让它全报，在单独的一步里再过滤（Opus 5 页原话：`ask it to report everything and filter in a separate pass instead`） |

**一个坑，只在 Opus 5 关闭 thinking 时有记载**：关闭 thinking 时，Opus 5 偶尔会把 tool call 写成普通文本（调用不会执行，而且在 agentic 循环里，这段文本会留在对话历史里影响后续轮次），或者把 `<thinking>` 标签泄漏到可见输出。官方建议是**保持 thinking 开启，用低 effort 控成本**，而不是关掉它——多数任务上，开着 thinking 跑 `low` effort，效果好于在相近成本下关掉 thinking。（在 `xhigh` / `max` 档位下也确实关不掉，请求会返回 400。Opus 5.5 根本关不掉 thinking；Opus 5.5 页面说这两种现象只在 Opus 5 关闭 thinking 时出现，让你检查是否还需要那条缓解指令，并且不管怎样都**删掉继承来的禁止思考的规则**。）

如果某个集成必须关闭 thinking，官方给的是一条同时兜住这两个问题的合并指令：

```
When you use a tool, you may say a brief sentence first. If no tool can express what the user asked for, say so instead of guessing. Do not include internal or system XML tags in your response.
```

点名 thinking 标签的效果差于上面这种通用写法。

### 6.2 Claude Fable 5.2 / Mythos 5.2

2026 年 11 月 9 日发布，Fable 5.1 随之进入「Legacy models (still available)」列表。$8 / $40 每 MTok（Fable 5.1 是 $10 / $50）；1M token 上下文；最大输出 128K；adaptive thinking 始终开启且无法关闭；延迟标注 Moderate，和 Opus 5.5 相同。**effort 默认值按界面不同**：Claude API 上是 `high`，Claude Code 和 Claude 应用里是 `medium`。选型页称它是 `Anthropic's most capable model open to all customers`，`built for demanding reasoning and long-horizon agentic work`。Mythos 5.2 能力相同，只提供给通过 Anthropic 验证计划（例如 Cyber Verification Program）认证的组织。Fable 5.2 要求 30 天数据保留，zero data retention 下不可用（除非 Anthropic 明确授权）。

能力最强和默认该选哪个不是同一件事：选型矩阵仍然是 `Most workloads start with Claude Opus 5.5`。什么时候从 Opus 5.5 换到 Fable 5.2，官方几处的说法不完全一样，skill 按 2026-11-09 那篇选型文章的步骤走（6.5 节）。

**为什么 Fable 5.1 退出了范围。** 没有哪个官方页面在任何一点上推荐它而不是 Fable 5.2：它在 legacy 列表里；它自己的 overview 写着 `you should consider migrating to Claude Fable 5.2`；它更贵；Fable 5.2 专页说 Fable 5.2 在 `low` 的分数 `sat close to Claude Fable 5.1's scores at higher effort levels`。方向相反的分数和成本测量只在成本页上：DeepResearch Bench II 上，Fable 5.2 在各档位比 Fable 5.1 便宜 20% 到 61%，分数低 4.7 到 7.0 分；SWE-bench Pro 子集上，Fable 5.2 在 `high` 解出的任务更多（96.3% 对 92.4%），每个解出的任务却更贵（$1.39 对 $1.17）。成本页紧挨着研究基准那组数字的建议不是「留在 Fable 5.1」，而是 `Measure what a more expensive model buys on your own workload`。所以 skill 把这组测量记在 models.md，Fable 5.1 和 Opus 5、Fable 5 一样，只作为具体措辞的出处、迁移的起点、或现行页面点名的对照出现。

**旧 prompt 怎么办——这是和前两次升级最不一样的地方。** Fable 5.1 的专页写着现有 Fable 5 prompt 应当无需修改就能表现良好，本文和 skill 据此把 Fable 5 的 prompt 块沿用到 Fable 5.1。Fable 5.2 的页面没有这句话，只有 6.0 节引的那句 `When you migrate to Claude Fable 5.2, review your prompts.`它的 prompting 页没有逐字重刊任何一段带 Fable 标注的旧块。所以 Fable 5.1 和 Fable 5 的条目在 skill 里一律标成「在 Fable 5.1 上测得」或「在 Fable 5 上测得」；同一个问题 Fable 5.2 页面给了新措辞的，条目会指过去。

**Fable 5.2 页面写明的倾向。** skill 的 models.md 有完整的表，这里只列理解 skill 规则需要的几条：

| 倾向（Fable 5.2 页面的描述） | 修法 | skill 里对应 |
|---|---|---|
| 长的多步任务上有时没做完就结束回合：结尾列出还剩什么、提出可以继续、或停在超时这类普通障碍上；`low` 和 `medium` 下更常见 | 片段 4.37。它区分两类障碍：普通障碍先诊断再绕过去，有人特意设的阻断不碰、如实报告。官方回放 `medium` 下的编码会话：留下可做工作就结束的会话约少一半，输出 token 多约 30%；请示许可或提出可以继续的结尾大多没了，但列出还剩什么然后停的那种结尾还在。模型因此更少发问，风险或不可逆操作的确认步骤要自己保留 | 第 8 节速查；第 7 节自检 |
| system prompt 里的审批规则会被照字面执行 | 规则点名覆盖哪些具体动作；让它继续时说清要它做什么（官方例子 `Yes, run the migration on the staging database`），而不是一句 "continue" | 第 7 节自检；models.md；场景 A「一次说清楚」 |
| 谨慎类指令在记忆、笔记、指令文件和 agent 之间的消息里越积越多，它照得很紧，于是可能要不必要的审批、压着例行工作不做（少数早期测试者见过，多在几个 agent 协作的场合） | 删掉不再需要的笔记；点名哪些步骤不需要审批（官方测试里，笼统地叫它放手去做没有带来变化）；检查 reviewer agent 附带了什么；把例外写进规则本身。风险或不可逆操作的确认步骤自己保留 | 3.3、3.5、3.13 |
| 对 prompt 里代码归谁的说法照得很紧：没说归属时倾向于往现有结构上加东西（哪怕重复已有代码），旁边的问题只报告不修 | 想让它边做边改善结构用 4.38；代价是输出 token 约 1.5 倍、会做没人要求的改动、收尾总结常漏掉小改动，所以要看 diff。用户希望改动尽量小时不要加 | 第 8 节速查 |
| 叫它少思考的句子在 `medium` 及以上不管用；`high` 及以上它几乎每次回复前都思考，包括回一句问候 | `Remove these lines and lower the effort level` | 3.10 |
| 照得很紧的还有文风规则：硬性上限（官方例子 `one instruction per sentence, in 20 words or fewer`）可能让文字一顿一顿的 | 先删掉不再需要的文风规则，再加 4.40 | 3.13；第 8 节「长度控制」 |
| effort 几乎不改变回复长度 | `Effort barely changes how long the model's replies are. To change the length and tone of a reply, use your prompt.` | 第 1 节变量表；第 8 节「长度控制」 |
| 长串工具调用期间有时很少写给用户看的文字 | 先确认客户端收得到：播报以 progress-update `thinking` 块返回，默认 `display: "omitted"` 下为空，设 `"updates"` 或 `"summarized"`；还不够再用 4.3 的 Fable 5.2 块或 4.39 | 第 8 节速查 |
| harness 写进 prompt 的 token 预算，预算大小会改变它花多少 | 去掉预算试一下：官方在 `high` 和 `xhigh` 的编码任务上测得，去掉一个大预算后成本下降、表现相当 | 3.13 |
| Opus 5.5 给 chat 的那两句（4.34，让它把已答过的当定论）在它身上有副作用：追问轮次的思考少了，但它远不如原来那样会指出自己早先答案里的错 | 希望它持续复查的地方不要加；别处先测 | 3.13 |
| 委派多少子智能体取决于 harness 和任务；高档位下观察到它用子智能体复查自己的工作 | 先查 system prompt、skills、agent 文件里叫它委派的句子；原话 `subagents can account for a large part of what a session costs, so check which model your harness runs subagents on by default` | 第 2 节子智能体；3.13 |

**在前代 Fable 页面上测得的。** Fable 5.1 专页（仍在线；它的 what's-new 页和迁移指南已重定向）上的几个倾向——一轮只发一个工具调用、文风更密、聊天里格式更少、摘要照搬原文不标引用、`low` 下凭记忆作答、小改动整文件重写、`xhigh`/`max` 下长交付物先在思考里起草一遍——Fable 5.2 的页面既没重述也没撤回，skill 把它们放在「在 Fable 5.1 上测得」的表里。Fable 5 专页上的也一样：长回合、任务模糊时过度规划、长跑中编造进度、未被要求的动作、提议开新会话、为旧模型写的 skill 过于规定化。最后这条的原话是：

> "Skills developed for prior models are often too prescriptive for Claude Fable 5 and can degrade output quality."

**旧 skill 不只是无用，它还可能主动拖低质量**——这是在 Fable 5 上测得的。在 Fable 5.2 上是否成立，官方两边都没说；Fable 5.2 的页面给的是它自己的一张复查清单（"Review instructions written for earlier models"，skill 3.13），skills 和 agent 文件都在复查范围里。

**不再收的。** 前代 Fable 页面还有四条说法，Fable 5.2 的页面讲的是另一回事，所以不再作为对 Fable 5.2 的建议：一条是它更主动派并行子智能体、应当鼓励（Fable 5 页面；5.2 页面：先查委派指令，注意子智能体成本）；一条是一句简短指令就能管住一类行为（Fable 5 页面；5.2 页面：规则照字面执行，要点名具体动作，官方测试里笼统的一句没有带来变化）；一条是做开放式功能时多交、高 effort 下做没人要求的整理和重构（Fable 5.1 和 Fable 5 页面，片段 4.6；5.2 页面：没说代码归谁时它倾向于往现有结构上加、旁边的问题只报告不修，说代码归它之后才写到没人要求的改动）；一条是复杂的异步任务上只描述下一步而不去做（Fable 5.1 页面，片段 4.18；5.2 页面对同一类问题给了新措辞 4.37）。4.6 和 4.18 仍在 skill 的 snippets.md 里，标着各自是在哪个模型上测得的。本文早先还写过 Fable 是延迟最慢的模型、长回合里会安静好几分钟、档位越高提升越大；这些都出自前代页面，而 Fable 5.2 标注的延迟是 Moderate，它的页面说 `Most of the gain in capability is already present at high`，所以一并删掉了。

**API 层要知道的（仅自己拼请求时）。** prefix check 对 Fable 5.2 的所有账号生效：`system`、`tools` 或更早的消息变了之后再回传 thinking 块，可能返回 400，所以历史只追加不修改，每轮的提醒用 turn-scoped system message，prompt 上的改动只在新会话里做。只有 Fable 5.2 和 Mythos 5.2 读得了 Fable 5.2 的 thinking 块，路由或 fallback 把对话换到别的模型之后，之后的轮次就没有它的推理了。会话中途改 effort 用按消息设置（beta），不改顶层值。详见第 7.6 节和 skill 的 agents.md。

### 6.3 Claude Sonnet 5.5 与 Claude Haiku 5.5

2026-10-23 那一轮它们已经发布，但默认推荐没变，skill 让它们留在范围之外。这一轮进入范围，是因为 skill 现在要在整个现行阵容里选模型——包括子智能体，而选型矩阵把子智能体任务列在 Haiku 5.5 名下。它们的 API 行为在三处和另外两个模型不一样，正是第 4 章那张按模型的表存在的原因。

**Sonnet 5.5**（2026-09-28 发布，$2 / $10 每 MTok，延迟标注 Fast）。官方定位 `The best combination of speed and intelligence`；专页同时写明 `For the hardest long-horizon work, an Opus model is the better choice.` 片段 4.16 和 4.25 出自 Sonnet 5 页面，凭的是 Sonnet 5.5 页面那句 `Existing Claude Sonnet 5 prompts should perform well without changes`。理解 skill 规则需要知道的几处：

- `disabled` 在每个档都返回 400，关前置思考用 `between_tools`，只在 `high` 及以下可用
- effort 档位重新校准过，同一档位的思考量和 Sonnet 5 不一样；API 默认 `high`，Claude Code 里默认 `medium`
- `low` 和 `medium` 下做 agentic 编码时有时没做完就停下来确认（4.41）；`low` 下有时没跑检查就报告完成（4.46）；`xhigh`、`max` 下做完可能自己开几轮复查（4.42，官方在 `max` 的编码任务上测得：不再派 reviewer 子智能体，会话成本约降三分之一，质量不变）
- 叫它少思考 `doesn't reliably reduce its thinking`
- 用户中途发来的真消息有时被当成注入，尤其是放在 `tool_result` 块里的时候

**Haiku 5.5**（2026-10-07 发布；prompt 不超过 100,000 token 时 $0.10 / $0.50 每 MTok，超过时 $0.50 / $2.50；延迟标注 Fastest；默认 `medium`）。官方定位 `For high-volume, latency-sensitive tasks such as classification, extraction, and routing`；专页写明 `Existing Claude Haiku 4.5 prompts should perform well without changes.` 它是第一个有 effort 档位的 Haiku。三处 API 行为和另外三个模型都不一样：`high` 及以下可以关掉 thinking；接受强制 `tool_choice`；没有 `reasoning_extraction` 拒绝类别。prompt 层面：给了搜索工具就要给当天日期（4.47）；`low` 和 `medium` 下有时没跑检查就报告代码改动完成（4.46，与 Sonnet 5.5 页面的原文相同）；部署成 chatbot 时可能被用户磨得不守 system prompt（4.50）。

### 6.4 effort 参数

`max` > `xhigh` > `high` > `medium` > `low`。四个现行模型五档都支持。

effort 页给每一档的通用定位（各模型自己的建议与它不同时，以各模型的为准）：

| 档位 | 官方定位 | 典型用途 |
|---|---|---|
| `max` | 绝对最大能力，不限制 token 花费 | 需要最深推理、最彻底分析的任务 |
| `xhigh` | 面向长周期工作的扩展能力 | **超过 30 分钟**、token 预算以百万计的长周期 agentic 与编码任务 |
| `high` | 该花多少 token 就花多少以拿到优秀结果。**除 Claude Opus 5.5 和 Claude Haiku 5.5 外，所有支持 effort 的模型的默认档** | 复杂推理、困难的编码问题、agentic 任务 |
| `medium` | 平衡，适度省 token。**Claude Opus 5.5 和 Claude Haiku 5.5 的默认档** | 需要在速度、成本、性能之间平衡的 agentic 任务 |
| `low` | 最省。显著省 token，能力有一定下降 | 需要最快速度和最低成本的简单任务，比如子智能体 |

**默认值按模型、也按界面不同**：

| | Fable 5.2 | Opus 5.5 | Sonnet 5.5 | Haiku 5.5 |
|---|---|---|---|---|
| Claude API | `high` | `medium` | `high` | `medium` |
| Claude Code | `medium` | `medium` | `medium` | `medium` |

Claude 应用里每个模型的推荐档在菜单里标着 "Default"；发布公告说 Fable 5.2 在 Claude 应用和 Claude Code 里默认 `medium`。讲应用设置的那篇支持站文章另有一句 `For complex coding and agentic tasks on Opus 4.7 or newer, try Extra high (xhigh) first`；Opus 5.5 和 Fable 5.2 的页面都把 `xhigh` 留给实测到提升的工作，Sonnet 5.5 和 Haiku 5.5 的页面也只在评测显示质量提升时才用 `xhigh`，所以 skill 按模型页选档；场景 B 的任务是复杂编码或 agentic 工作时，在交付说明里把两种说法都告诉用户。同一个模型、同一个不传 `effort` 的请求，在 API 和 Claude Code 里跑的档位可能不一样，这是 skill 第 2 节把两种界面分开写的原因。

> **起步建议改过不止一次。** Opus 5 的起点从 `xhigh` 改成过 `high`，Opus 5.5 又把默认降到 `medium`，Fable 5.2 的默认则按界面分成两个。经过见文末附录。

几个要点：

- **把 `effort` 设成该模型的默认值，与完全不传这个参数行为完全相同**（官方原文：`Setting effort to the model's default ("medium" on Claude Opus 5.5 and Claude Haiku 5.5, "high" on other models) produces exactly the same behavior as omitting the effort parameter entirely.`）——这说的是 API 上的默认值
- effort 是**行为信号，不是硬性 token 预算**——官方原文：`Effort is a behavioral signal, not a strict token budget. At lower effort levels, Claude still thinks on sufficiently difficult problems, but thinks less than it would at higher effort levels for the same problem.`
- 它影响**全部输出** token：正文、工具调用、思考
- **低 effort 意味着工具调用更少、更简短**；低档位往往会合并操作、跳过前言，完成后给简短的确认，高档位则可能更多工具调用、先讲计划、给详细总结。这一点常被忽略——**effort 不只是「想多久」，也是「做多少」**
- **在 Fable 5.2 上，effort 几乎不改变回复长度**（它的页面写明；Opus 5 页面也说降 effort 不能可靠缩短可见输出）。长度写在 prompt 里，这是 skill 第 8 节「长度控制」的依据
- **按工作负载显式设定 effort**，而不是拿一个档位跑遍所有场景
- **依赖缓存的会话里，顶层 effort 保持不变**——官方原文：`Changing the top-level effort value between requests invalidates prompt caching, so vary it across workloads rather than within a conversation that relies on cache hits. On models that support it, use a per-message effort change instead, which preserves the cache.` 四个现行模型都支持按消息改 effort（beta）：一条 `content` 为空、`output_config.effort` 设为新档位的 `role: "system"` 消息。紧跟在带新输入的 user 消息之后时，它决定对那条消息的回复用什么档位；放在别处则从下一条带新输入的 user 消息起生效（只装着 `tool_result` 块的不算）。Sonnet 5.5 用 `between_tools`、或 Haiku 5.5 关掉 thinking 时，按消息改 effort 返回 400
- 高 effort 时要给 `max_tokens` 留余量（它是思考 + 正文的**总**上限），而且即使思考文本不返回给你，思考也计入这个上限。Opus 5.5 面向 agentic 编码的长回合，页面写的是 `a max_tokens of 128,000, the model's maximum, has worked well in Anthropic's testing`；Sonnet 5.5 做 agentic 编码同样设 128,000 并用流式；Fable 5.2 从 `high` 起就要设大。Claude Code 和 Claude 应用里这个值由客户端管

> 顺带一提：Claude Code 里的 `ultracode` **不是**一个额外的 API effort 档位。[dynamic workflows 页面](https://code.claude.com/docs/en/workflows)的定义是 `a Claude Code setting that turns on automatic workflow orchestration for the session, at whichever effort level the session runs at`——它是会话级设置，Messages API 里并不存在这个档位。作为关键词写进 prompt 时，它只让那一个任务以 workflow 方式运行：`To run a single task as a workflow without changing the session's effort level, include the keyword ultracode in your prompt`。

### 6.5 选模型和 effort：按任务选

本文早先给过一个判断什么算足够重的 agentic 工作的两条件测试，该上 `xhigh` 就看它。那个测试没有任何官方出处，这一轮删掉了。官方给出的唯一数字门槛是 effort 页对 `xhigh` 的描述（超过 30 分钟、token 预算以百万计），而且各模型自己的建议优先于那张通用表。

现在 skill 第 2 节分六个小节：一张按任务选起步模型和 effort 的表；一张每个模型什么时候上调、下调的表；质量不够或成本太高时按什么顺序处理，并点出官方说法不一致的几处；一张子智能体的表（6.6 节）；大批量处理，所有者标明的用户规则和官方数字并列；用量与成本，带一张各参数在哪儿设的表。第一张表逐行对应 Anthropic 2026-11-09 的文章「Which Claude model should you build with?」里的 "Pick a model by task"。那篇文章的起点是 `you should start most work on Opus 5.5 at its default effort level`，也就是 `medium`。平台文档的选型矩阵同样以 Opus 5.5 为起点（`Most workloads start with Claude Opus 5.5`），但有两类工作分法不同：矩阵把大规模重构放在 Opus 5.5 名下，把要一直做到成品文档、表格或幻灯片的分析放在 Fable 5.2 名下。

| 任务 | 起步模型 | 起步 effort | 文章给的理由 |
|---|---|---|---|
| 日常工程和调试；跨多个 PR 的功能开发 | Opus 5.5 | `medium`（默认），不够再 `high` | 日常任务上成本和能力的平衡 |
| 分类、路由、打标、高吞吐抽取 | Haiku 5.5 | `medium`（默认） | 全系列里成本和延迟最低 |
| 做文件搜索、读日志、做摘要的子智能体 | Haiku 5.5 或 Sonnet 5.5 | 该模型的默认档 | 错了容易发现，而且判断本来就留在主模型 |
| chat 和交互式会话 | Sonnet 5.5（大量的简单 chat 用 Haiku 5.5） | `low` 或 `medium` | Sonnet 5.5 快，`low` 和 `medium` 下尤其快 |
| 范围清楚的编码：修 bug、功能迭代、照 spec 做 | Sonnet 5.5 | `medium`，更难的任务 `high` | 范围清楚的任务它做得好，token 价格是 Opus 5.5 的一半 |
| 日常的文档、幻灯片、表格 | Sonnet 5.5 | `high`（API 默认；Claude Code 里是 `medium`） | 迭代快，设计眼光好。复杂的财务模型、或要从一大堆材料里做出来的文档，从 Opus 5.5 起 |
| 反复跑的 agent 任务：调查、审查、起草 | Sonnet 5.5 | `medium` | 定义清楚、反复做的任务很少需要更大的模型 |
| computer use 和重视觉的流程 | Opus 5.5 | `medium` | 这类运行是跨很多截图和图像的长 agentic 循环，正是 Opus 5.5 为之设计的那种工作 |
| 大型重构和迁移、架构规划、带很多子智能体 | Fable 5.2 | `medium`，不够再 `high`（API 默认是 `high`） | Anthropic 面向技术领域、普遍可用的最强模型 |
| 深度研究、科学或定量分析 | Fable 5.2 | `medium`，不够再 `high`（API 默认是 `high`） | 为编码、金融和科学里最难的技术项目而设计 |
| 还拿不准 | Opus 5.5 | `medium`（默认） | Claude Code 在各订阅计划和 Anthropic API 上默认用它，平台文档也把它作为起点 |

同一篇文章还补了两点。往下走时，`Just because you have a clear spec doesn't mean you should use Sonnet`：迁移或性能优化可以 spec 很清楚，难度仍然够得上 Opus 5.5，大的够得上 Fable 5.2。往上到 Fable 5.2 有三种情况：编码、金融和科学里最难的技术项目；Opus 5.5 在 `high` 下仍然不够；长时间无人值守的运行。

**每个模型怎么调档**（出处是 effort 页各模型一节、四个模型的 prompting 专页、Claude Code 的 model-config 页）：

| 模型 | 默认档 | 往上 | 往下 |
|---|---|---|---|
| Haiku 5.5 | `medium` | 知识工作、较长的 agent 任务、严格遵循指令 → `high`。`xhigh`/`max` 只在评测显示质量提升时用，并同时拿 Sonnet 5.5 比一比 | chat、短工具任务、简单的高吞吐请求 → `low` |
| Sonnet 5.5 | API `high`；Claude Code `medium` | 说清了的 agentic 编码和多步工具使用从 `medium` 起，更难或更长的 → `high`。`xhigh`/`max` 只在评测显示质量提升时用 | chat 和延迟敏感的工作 → `medium` 或 `low` |
| Opus 5.5 | `medium` | 需要验证、或很可能有边界情况的工作 → `high`（model-config 页的例子是在现有代码库里修 bug）。`Reserve xhigh and max for work where you've measured a quality gain`；选型文章另说，难任务上把 Opus 5.5 提到 `high` 以上之前，先试 Fable 5.2 | 评测显示质量不掉 → `low` |
| Fable 5.2 | API `high`；Claude Code 和 Claude 应用 `medium` | 困难的编码、研究、agentic 工作和长时间无人值守的运行 → `high`。`Most of the gain in capability is already present at high`，`Above high, gains are uneven and depend on the task`；`xhigh` 留给最难的长时间工作且测到了提升，`max` 用来测上限 | 质量不掉的地方 → `medium`；快速来回、说清了的任务、高吞吐 → `low` |

**模型不够好时的顺序。** 选型文章给了三步，依次做：

1. 确认模型有办法验证自己的工作：一个测试、一次构建、或一个调用端点的脚本。文章说，给它自我验证的手段往往比换档位更划算、加的延迟更少
2. effort 升一档
3. 换到上一档模型，并且从比小模型上用的更低的 effort 起

配套的几条官方说法：

- **先调 effort，再换模型**：`Tuning effort is often a better lever than switching models`（choosing-a-model 页）
- **过了 `high` 就两条路都比一比**：选型文章说，再升一档 effort 的花费可能和换一个更大的模型一样多；已经过了 `high`，就试上一档模型配更低的 effort
- **升 effort 补不了模型没有的知识或能力**，那种情况要换模型（选型文章）
- **成本太高、质量没问题**：在现在这个模型上试更低的 effort（成本页）
- **不要把别的模型上调好的档位照搬过来**：Fable 5.2 页面原话 `rerun your effort sweep instead of carrying over a setting you tuned on Claude Fable 5.1 or Claude Opus 5.5`
- **按每个完成的任务比成本**：`Compare models on cost per completed task, not per token`

**下面几处官方说法不完全一致，skill 第 2 节列出了它们，交付说明里用到哪一处就点到哪一处（场景 B 还有一处，支持站的 `try Extra high (xhigh) first`，见 6.4 节）：**

- **什么时候从 Opus 5.5 换到 Fable 5.2。** 选型文章说 Opus 5.5 在 `high` 下仍然不够时；choosing-a-model 页说评测在 `xhigh` 或 `max` 下仍然不够时；Fable 5.2 的 what's-new 页、models overview 和成本页说 `at higher effort` 仍然不够时。成本页的 "Start here" 表又是一种写法：质量不够时，降过 effort 就先恢复，否则 `try the next tier up at its default effort`。skill 按选型文章那三步走，因为它是几处里唯一把自我验证放在第一步的
- **研究类任务。** 任务表和选型矩阵都把深度研究放在 Fable 5.2 名下；成本页在 DeepResearch Bench II 上测得 Fable 5.2 的成本是 Opus 5.5 的 2.9 倍、分数低 1.9 分，紧挨着这组数字的建议是 `Measure what a more expensive model buys on your own workload`。（成本页另一句 `On research tasks, measure the model you ship` 在讲 effort 的那一节，比的是同一个模型在不同档位上的得失，不是这组跨模型的数字。）两边都是官方的，skill 不替用户挑一边
- **Fable 5.2 从哪一档起。** 任务表（选型文章）从 `medium` 起，Claude Code 和 Claude 应用里它的默认档也是 `medium`。Fable 5.2 专页、what's-new 页、effort 页和 choosing-a-model 页写的都是从 `high` 起，也就是 Claude API 的默认档；专页原话是 `Start at high, the default on the Claude API`，并说 `high` 适合困难的编码、研究和 agentic 工作。场景 A、B 里 skill 按任务表从 `medium` 起、不够再 `high`；写给 API 的 prompt 按平台页从 `high` 起
- **从哪个模型起步。** 选型文章和选型矩阵都让多数工作从 Opus 5.5 起。choosing-a-model 页另给了一条效率优先的路：先用 Haiku 5.5，只在有具体能力缺口时才升级。它列的适用情况是初期原型和开发、延迟要求紧的应用、对成本敏感的实现、大量而直接的任务。skill 按任务表走；用户的任务属于这四种情况时，把这条路也告诉用户
- **Claude Code 里日常编码用哪个模型。** 选型文章让日常工程从默认的 Opus 5.5 起。Claude Code 最佳实践说 `the default Opus 5.5 handles most coding well and Sonnet costs less on routine work`。Claude Code 成本页说 `Sonnet handles most coding tasks well and costs less than Opus. Reserve Opus for complex architectural decisions or multi-step reasoning.` skill 按任务表走：范围清楚的编码用 Sonnet 5.5，其余日常工程从 Opus 5.5 起

**在 Claude Code 里**，四个现行模型都从 `medium` 起；Fable 模型在任何计划上都不是默认，要用 `/model fable` 显式选。Claude Code 最佳实践说 `medium` 是 `plenty for everyday feature work with a clear scope`；对错取决于容易漏掉的情况时（不是你写的代码里的 bug、flaky test、要找出旧结构所有读取方的 schema 变更）升到 `high` 或 `xhigh`；来回很快、每个结果都会看时降到 `low`；`max` 留给想让它自己啃下来的难题，普通任务上容易想过头；只有下一条消息需要更深推理时，在那条 prompt 里写 `ultrathink`。

**skill 里的三条用户规则（用户规则，非官方；2026-11-09 确认）。** skill 第 2 节另有三条仓库所有者自己的做法，它们只管在官方给的选项之间怎么取，不是官方指引：模型和 effort 默认由写 prompt 的 Claude 自己定，取舍推断不出来时才问用户一次；不刻意省用量，也不无依据地往上加；大批量任务里主会话和批量子智能体分开选模型。最后一条 2026-11-10 改定过：原来写的是批量子智能体一律用 `opus`、effort 用 `high`、复核用 `xhigh`，官方没有测过「Fable 5.2 主会话配 Opus 5.5 子智能体」这个组合，这两个档位也没有官方依据，所有者把它换成了一张取自成本页缺陷搜索基准的表：默认 `sonnet` 配 `high`（Fable 5.2 编排、Sonnet 5.5 子智能体测得 0.833、$165，测量里 Sonnet 5.5 跑的就是 `high`）；试跑发现漏项或判错多，换 `opus` 配 `high`（最接近的测量是 Opus 5.5 每部分一个会话，0.850、$236）；每一份都难时不传模型、让它跟主会话的 Fable 5.2，effort `medium`（0.877、$378，花的是 Fable 的每周上限）；只读文件做摘要、产出能用脚本校验的用 `haiku`。最后的独立复核跟主会话的模型、用 `high`，`xhigh` 只在测到质量提升时用。什么时候启用这条规则也是用户定的，不是官方给的触发条件。`model` 和 `effort` 这两个参数只对非 fork 的子智能体起作用：fork 跑在主会话的模型上，而 fork 模式默认开着，所以 skill 要求 prompt 里写明这批工作派给非 fork 的子智能体。skill 还要求和用户的任务对得上时在交付说明里提官方测得的两点：一个上下文装得下、又没有长成本尾巴的工作，编排者模型自己用更低的 effort 做更划算（在更早的模型上测得）；Fable 5.2 单独做的分数比带 Sonnet 5.5 子智能体的配置高，拆出去换来的是成本（6.6 节）。

### 6.6 子智能体与用量

**为什么子智能体要单独选。** Claude Code 里没指定模型的子智能体跟主会话用同一个模型。所以主会话一旦选了 Fable 5.2，它派出的子智能体默认也跑在 Fable 5.2 上（有几个例外：内置的 Explore 在用订阅登录时跑在 `opus` 别名指向的模型上，statusline-setup 和 claude-code-guide 分别固定用 Sonnet 和 Haiku；设了 `CLAUDE_CODE_SUBAGENT_MODEL` 时默认值也会变）。Fable 5.2 页面那句 `subagents can account for a large part of what a session costs, so check which model your harness runs subagents on by default` 就是冲着这个来的。

| 子智能体做什么 | 模型 | 官方依据 |
|---|---|---|
| 搜索文件、读日志、做摘要、检查文档链接 | `haiku` 或 `sonnet`，effort 用该模型的默认档 | 选型文章的任务表；Claude Code 最佳实践：`Set model: haiku in a subagent's definition if its work is simple` |
| 高吞吐、产出能校验的批量工作 | 先试 `haiku` | 成本页 "Start here" 表 |
| 超出一个上下文窗口的工作，拆成每部分一个子智能体 | 比主会话便宜的模型；Opus 5.5 做编排者时，官方测得保住分数的是 `sonnet`，`haiku` 也测了，分数明显更低 | 成本页：Opus 5.5 配 Sonnet 5.5 子智能体，比每部分一个 Opus 5.5 会话便宜约三分之一，分数相当；配 Haiku 5.5 子智能体分数明显更低；编排者换成 Fable 5.2，成本高 19%，分数相当 |
| agent team 的队友 | `sonnet` | Claude Code 成本页：`Use Sonnet for teammates` |

选型文章给的分工原则：`Keep small models on work where a mistake is cheap to spot, like finding files, running tests and reading logs. Keep code edits and judgment calls on the main model.`

**什么时候值得派。** 成本页测得委派只在两种情况下划算：有很长成本尾巴的例行工作（在更早的模型上测得；这种情况官方要求先测准确率和成本），和超出一个上下文窗口的工作；工作是一条前后依赖的链、或一个上下文装得下又没有长成本尾巴时，编排者模型自己用更低的 effort 更划算（同样是在更早的模型上测的）。同一个缺陷搜索基准上，Fable 5.2 自己单独做（`low` 到 `high`）比它带 Sonnet 5.5 子智能体时高 3.2 到 7.5 分，成本是 2.2 到 3.4 倍，所以拆出去换来的是成本，不是分数。Claude Code 最佳实践说子智能体带来的是并行、把长输出挡在主会话外、一个没看过代码怎么写出来的 reviewer、以及更少的工具，原话是 `You get parallelism or a fresh pair of eyes out of it, but not a smarter answer.` 成本页对要不要搭配第二个模型的判据：先在现在的模型上扫 effort，`Build a pairing only if it beats the single model's whole curve of score against spend`。

**子智能体的 effort**：选型文章给搜索、读日志、做摘要这类子智能体的起步档是该模型的默认档。effort 页把子智能体列为 `low` 的例子，Fable 5.2 页面说 `low` 适合又快又聪明的子智能体。降档前先测：缺陷搜索基准上，Opus 5.5 单独做时 `low` 比 `medium` 低 34 分，Fable 5.2 单独做时 `low` 比 `high` 低 4.3 分、成本少 37%。所以 skill 让子智能体从该模型的默认档起。

**用量和价格**里写 prompt 时最常用到的几条（完整数字在 skill 的 models.md）：

- 价格每百万 token（输入 / 输出）：Fable 5.2 $8 / $40，Opus 5.5 $4 / $20，Sonnet 5.5 $2 / $10，Haiku 5.5 $0.10 / $0.50 起
- 订阅上，子智能体的请求和主会话计入同一个用量上限；Max 和 Ultra 上 Fable 模型最多用掉每周上限的一半，而且用得比其他模型快；effort 越高，越快到上限
- 输出 token 比输入贵五倍，而且 agent 循环里模型写的每个 token 之后每轮都会作为输入再算一次——所以成本页的建议是 `Ask for the answer you will read, not the one that looks thorough`
- 产出能校验时（API）：`low` 跑一遍、失败的用 `high` 重跑，官方在 Opus 5.5 上测得比全部 `high` 便宜约 40%，通过率相同

---

## 7. Agent 与长任务：上下文工程

### 7.1 上下文是有限资源：context rot

模型有有限的注意力预算（attention budget）。上下文越长，注意力越被稀释，性能下降越明显。这不是工程 bug：transformer 注意力在 n 个 token 之间有 n² 对关系，模型在长序列上的训练数据也更少，所以性能是渐进下降，而不是在某处断崖式跌落。

所以上下文工程的目标不是「装得下」，而是：**最小的高信噪比 token 集**。

### 7.2 合适的高度（right altitude）

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
4. **可以考虑用「全新窗口」代替 compaction**——这条反直觉但有依据：现行模型极擅长从文件系统恢复状态，某些场景下可以利用这一点来代替压缩（官方措辞是 `consider` 和 `in some cases`，不是一律如此）。这时开场要具体，官方给的三句原文是：`Call pwd; you can only read and write files in this directory.`、`Review progress.txt, tests.json, and the git logs.`、`Manually run through a fundamental integration test before moving on to implementing new features.`
5. **提供验证工具**（浏览器自动化、computer use 等）——任务越长越自主，模型越需要在没有持续人类反馈的情况下验证正确性
6. **鼓励用满上下文**：让它先高效地做完一个部分再往下走，并告诉它把整个输出上下文都花在这个任务上是被鼓励的，只要别在还有大量未提交的工作时把上下文用完

状态管理的分工：结构化数据用 JSON，进度笔记用自由文本，版本和检查点用 **git**（现行模型用 git 追踪跨会话状态的表现特别好）。

### 7.5 让 Claude 自己编排

这一条常被忽略，因为它不在提示词层，而在 harness 层。

常见做法是：每次工具调用的结果，都作为 token 塞回上下文窗口，过滤逻辑硬编码在工具里。

**更好的做法**：给 Claude 代码执行能力（bash、语言 REPL），让它自己写逻辑来表达工具调用、自己过滤输出——**只有执行结果进入它的注意力**。

官方给了数字：在 BrowseComp 网页浏览任务上，给 Opus 4.6 输出过滤能力，准确率从 **45.3% 升到 61.6%**。

同一思路的另外两条，也都有数字：

| 做法 | 效果 |
|---|---|
| 让 Claude 自己管上下文（skills 渐进披露、context editing、子智能体隔离） | 子智能体：BrowseComp 比最好的单智能体结果高 2.8%（Opus 4.6） |
| 让 Claude 自己持久化上下文（compaction + memory 文件夹） | compaction：BrowseComp 上 Opus 4.6 84%、Sonnet 4.5 43%；memory 文件夹：BrowseComp-Plus 60.4% → 67.2%（Sonnet 4.5） |

> **出处说明**：这些数字来自 Anthropic 在**更早一代模型**上的实测——就是上面点名的 Opus 4.6 和 Sonnet 4.5——不是本文适用范围内的模型。文章的论点是模型越强，越可以把决策交还给它，同时 `Assumptions about what Claude can't do need to be re-tested with each step change in its capability`——所以这些数字是在现行模型上要重新验证的方向，不是可以直接沿用的数字。

官方还给了个很生动的对比：较新的模型玩宝可梦时会写出有组织的战术笔记（10 个分目录组织的文件，拿到三枚道馆徽章）；而早一代模型把记忆当成流水账（31 个文件，其中两个近乎重复，还停在第二个城镇）。**同样的记忆机制，效果天差地别，文章把它当作 Claude 使用记忆文件夹的能力提升的例子**——这正是「脚手架会过期」的另一面：模型强了之后，你该把更多决策**交还**给它。

### 7.6 缓存命中：一个纯粹的成本杠杆

**缓存读取的成本是基础输入 token 的 10%**（Opus 5.5 和 Sonnet 5.5 上是 5%，Fable 5.2 和 Mythos 5.2 上是 2.5%）。所以上下文的排列顺序直接决定你花多少钱。五条原则：

| 做法 | 要点 |
|---|---|
| **静态在前** | 稳定内容（系统提示词、工具定义）放在动态内容之前 |
| **用消息追加而非改提示词** | 要注入提醒时追加到 messages 里，不要回头改 system prompt |
| **不要中途换模型** | 缓存按模型隔离 |
| **工具增删会作废缓存** | 需要动态发现工具时用 tool search，或用会话中工具变更，而不是增删工具定义 |
| **移动断点** | 多轮应用把 cache breakpoint 移到最新消息 |

会话中工具变更是第四条的新官方做法：在 `tools` 里一开始就声明全套工具，之后用 `role: "system"` 消息里的 `tool_addition` / `tool_removal` 块从对话的某一点起提供或撤回某个工具（beta）。从 2026 年 9 月 22 日起，`tool_addition` 还可以直接带上一个首个请求时谁都不知道的工具的完整定义。`tools` 数组始终不变，所以缓存前缀——以及四个现行模型上它后面的 thinking 块——都保持有效。

后面那半句现在比缓存还要紧：四个现行模型的 thinking 块都只对产生它时前面的内容有效，改了 `system`、`tools` 或更早的消息再回传，就通不过 prefix check。在 Messages API 上，Fable 5.2 对所有账号都直接返回 400；Opus 5.5、Sonnet 5.5、Haiku 5.5 对 2026-08-31 及之后创建的账号返回 400（Message Batches API 里，这些情况改为丢掉通不过的块）。会踩到 prefix check 的改动大多也会重置缓存，修法也一样：提醒用 turn-scoped system message，改指令和工具用会话中 system 消息和工具变更，而不是重写 `system` 或 `tools`。

这条也解释了第 6.4 节那个看起来很随意的提醒——**依赖缓存的会话里顶层 effort 保持不变**：同样是会破坏缓存（按消息改 effort 正是为了不动前缀就换档位）。

### 7.7 什么时候该把 bash 动作提升为专用工具

bash 杠杆大，但 harness 只能看到一串命令字符串，每种动作都是同一个形状；做成专用工具，harness 才有针对这个动作、带类型化参数的钩子，可以拦截、把关、渲染或审计。以下四种情况值得做成专用的声明式工具：

1. **安全边界**：难以撤销的动作（比如调用外部 API）可以用用户确认来把关
2. **写操作需要陈旧性检查**：防止覆盖自上次读取后已经变动的文件
3. **需要面向用户呈现**：弹窗、多选项展示
4. **需要结构化日志、追踪、回放**

还有另一条路，但只应该用在用户信任大方向的任务上：**用第二个 Claude 实例判断 bash 命令是否安全**——这就是 Claude Code auto 模式的做法，能减少对专用工具的需要；某些高风险动作仍值得用专用工具。

Claude Code 自己就是这样取舍的。按 tools reference 页，在 macOS、Linux 和 WSL 上，Glob 和 Grep 不在默认工具集里，Claude 通过 Bash 跑 `find` 和 `grep` 搜索；而改文件仍走专门的 Edit 工具，编辑生效前要先通过它的几项检查，页面列出的是 read-before-edit、match 和 uniqueness 三项。

> 这一节和 7.5 看起来矛盾（一个说「少造工具、用通用 bash」，一个说「该造专用工具」），实际是同一篇文章里的两条：`ask what you can stop doing`，以及 `carefully set boundaries with the agent harness`。模型能自己搞定的，别用工具限制它；产品必须能拦截、审计、渲染的，才做成工具。而且文章说，要不要把一个动作提升成工具，应当随模型能力不断重新评估。

### 7.8 工具设计

出自 Anthropic 的「Writing effective tools for agents」和「Effective context engineering for AI agents」两篇文章。好工具的标准：

- **每个工具有清楚、彼此不同的用途**：`Make sure each tool you build has a clear, distinct purpose`。判断法很简单：`If a human engineer can't definitively say which tool should be used in a given situation, an AI agent can't be expected to do better`
- **参数名不要有歧义**：`user_id` 而非 `user`
- **工具返回有意义的标识符**：把无意义的 UUID 解析成有语义的名称（甚至用从 0 开始的编号），能减少幻觉、显著提升 Claude 在检索任务上的精度
- **命名空间**：按服务和资源类型分组（`asana_projects_search`、`asana_users_search`）
- **返回值 token 高效**：分页、范围、过滤、截断，给合理默认上限（Claude Code 的默认是 25000 token）
- **提供 `ResponseFormat` 枚举**（`concise` / `detailed`）——官方给的例子里详细版 206 token，精简版 72 token，大约三分之一
- **错误信息要可执行**：说清怎么改、附正确格式示例，而不是抛栈
- **工具描述当作给新同事的入职说明写**：把隐含的前提、专有查询格式、术语定义、资源关系写明

---

## 8. Claude Code 场景：该把话写在哪

这是很多人的实际痛点：同一句话，写进 CLAUDE.md、skill、hook、subagent，效果和成本完全不同。

先说一个变化：Claude Code 最佳实践页在 2026-10-23 和 2026-11-09 两轮之间整页重写了。explore / plan / implement / commit 那一节、本文早先第 8.5 节依据的那五种常见失败模式，以及本文早先引的大部分示例 prompt 都不在了；新增了确定性与非确定性指令、子智能体的代价、无人值守运行，以及一张「What you can stop doing」表。skill 的 for-claude-code.md 按新页面重写，这一章跟着它走。

### 8.1 取舍表

Anthropic 的「Steering Claude Code」一文列了七种方法（这里 CLAUDE.md 分成根目录和子目录两行），内容按现行文档校过：

| 手段 | 加载时机 | 上下文成本 | 适合放什么 |
|---|---|---|---|
| **CLAUDE.md（根）** | 会话开始，全程驻留，compaction 后重新读入 | **高** | 确认工作用的命令、代码里有误导性的地方、不在仓库里的决定、已经纠正过两次的事。文章原话：`Keep CLAUDE.md under 200 lines, give it an owner, and review changes to it like code.` |
| **CLAUDE.md（子目录）** | 触及该目录时 | 低 | 目录级约定 |
| **Rules** | 会话开始或访问文件时 | 中 | 只在代码库某一部分才要紧的规则。**加 `paths:` 限定路径**，只在 Claude 真在那里干活时加载 |
| **Skills** | 名称 + 描述常驻，正文按需 | 低 | 团队的流程和知识：发版流程、事故手册。有副作用的流程加 `disable-model-invocation: true`，只在你自己输入命令时才跑 |
| **Subagents** | 隔离上下文，只回最终结论 | 对主上下文低；它自己的请求照样计用量 | 只要结论的侧任务：日志分析、依赖审计 |
| **Hooks** | 生命周期事件触发，绕过压缩 | 低 | 零例外的规则和**确定性**自动化：格式化、阻断命令。要真正强制，用 `command` 或 `http` 类型的 hook；多数事件上，hook 自己失败或超时时仍会放行，想在这种时候也拦住就设 `onFailure: "block"` |
| **Output styles** | 会话开始，永不压缩 | **高** | 每次回复都要的语气、长度或格式，或者让 Claude 以软件工程师之外的角色工作 |
| **追加 system prompt** | 以 CLI 参数在启动时传入 | 中 | 脚本和自动化里需要放在 system prompt 层的指令（`--append-system-prompt`） |

### 8.2 确定性指令和非确定性指令

重写后的最佳实践页把每条指令分成两种：非确定性的由模型读了之后自己决定怎么办（prompt、CLAUDE.md、Claude 自己会用的 skill），通常会照做，但没有保证；确定性的由 Claude Code 这个 harness 执行（hook、deny 规则、managed setting），不管模型怎么决定都会生效。页面把同一件事的两种形态并排列了出来。比如判断做没做完：非确定性的是要证据、`/goal`、`/code-review`，确定性的是测试、类型检查、`Stop` hook、CI。又比如控制花多少：非确定性的是 effort 档位，确定性的是 `--max-turns`、`--max-budget-usd` 或你设的 spend limit。

官方的取舍是**大多数时候让 Claude 决定**——这正是第 4.5 节那句 `Almost every guideline has exceptions` 的上下文。确定性那一种留给零例外的事：`a hook is for a rule with zero exceptions`，有例外的偏好（官方例子 `Prefer named exports`）留在 CLAUDE.md。反过来也成立：给只是偏好的东西配了 hook，删掉，让 Claude 决定。页面还提醒强制执行的那一套要保持小：`If you ever find yourself at forty of them, most are probably preferences that Claude could have weighed on its own.`

Steering 一文列的几个反模式，放在这个框架里都说得通：

- ✗ 把 "Every time X, always do Y" 写进 CLAUDE.md → 用 hook；模型自己选择去跑格式化工具，和格式化工具自动运行，不是一回事
- ✗ "Never do this" 这类约束 → 用 `PreToolUse` hook 或权限配置。文章原话：`A real guardrail needs to be deterministic`
- ✗ 30 行的操作流程写进 CLAUDE.md → 移到 skill
- ✗ 只跟 API 相关的规则却不限定路径 → 加 `paths:` frontmatter
- ✗ 个人偏好写进项目共享文件 → 放 `CLAUDE.local.md`，并加进 `.gitignore`

### 8.3 CLAUDE.md：只写它看不见的

最佳实践页 "Tell Claude what it can't see" 一节开头的提示是：`Only write down what Claude can't find by reading the repo, or what it would otherwise spend time rediscovering every session`。Claude 开始任务时先读代码，文件布局、约定、测试框架它自己找得到；它找不到的是从来不在仓库里的东西——团队在会上做的决定、只在某个人脑子里的部署流程、集成测试要的环境变量。写下任何东西之前，官方的检验是问两句：它读代码能找到吗？它真的会去找吗？两个都是「是」，就不写。

**该写**：怎么确认这个仓库里的工作（确切的测试、类型检查、lint 命令，以及怎么只跑一个测试文件）；代码有误导性的地方（到处都是但新代码不许照抄的旧模式、看着没用其实在用的目录）；不在仓库里的决定；已经不得不纠正过两次的事。仓库里已经有能跑的检查时，CLAUDE.md 里最有用的一行就是跑它的命令。

**不该写**：Claude 读代码就能得出的东西。在 CLAUDE.md 里描述架构和文件布局，被列进了可以停掉的旧做法，理由是 Claude 自己会读仓库。

**某一行老被忽略时**：本文早先在这里写的是只给那一行加 IMPORTANT。现在的页面说加强调多半没用，要么缩短文件，要么做成 hook，并且给了一个判断信号：`If you keep adding emphasis to a CLAUDE.md line and Claude still misses it, that's a sign the line belongs in a hook.` 同一页举的例子很能说明两种指令的区别：CLAUDE.md 里那一行类型检查命令是为了让 Claude 自己能检查；你要的若是「不通过就不许收工」，那是 `Stop` hook 的事，`Always typecheck before you finish` 这样一行字只是大多数时候管用。

**工具**：`/init` 从代码库生成一个起始版本，之后主要是删行；`/doctor` 建议删掉 Claude 能从代码里得出的内容；`/doctor prompt-audit` 标出为旧模型写的、指向已不存在的文件的、或互相矛盾的指令；会话里 `/context all` 确认文件真的加载了。

### 8.3b Skills 怎么写：三层渐进式披露

skill 的整个架构就是一件事——**每层信息只在需要时才进上下文**：

| 层 | 内容 | 何时加载 |
|---|---|---|
| **1. 元数据** | YAML frontmatter，只有 `name` 和 `description` | 启动时预载入系统提示词 |
| **2. 主指令** | `SKILL.md` 正文 | Claude 判断相关时 |
| **3. 附加文件** | 从 SKILL.md 引用的其他文件 | 只在真正需要那个场景时 |

官方的类比很贴切：**像一本组织良好的手册——先目录，再具体章节，最后详细附录。**

几条要点：

- **CLAUDE.md 那条规则对 skill 同样适用**：`only write down what Claude can't find on its own`（Claude Code 最佳实践）
- **`name` 和 `description` 是最关键的两个字段**，Claude 就靠它们决定要不要激活这个 skill。要写清这个 skill 做什么、什么时候该用
- 定位是**让 Claude 需要时能找到信息的轻量指引**，不是详尽规程。第 1 章那篇文章的原话：`Avoid making them overconstrained, except in highly important areas`
- 正文变臃肿就**拆文件**。互斥的场景分开放能省不少 token。官方例子：PDF skill 把填表说明放在单独的 `forms.md`，正文保持精简，Claude 只在真要填表时才去读它
- **可以打包可执行代码**，理由很实在——用 token 生成来排序一个列表，比直接跑排序算法昂贵得多。确定性操作交给代码
- **从评测出发**：先测出能力缺口再补 skill，别预先臆想需求
- **让 Claude 帮你写**：它用某个 skill 做任务时跑偏了，就让它反思哪里出了问题，这能帮你发现它**实际需要的是什么上下文**，而不用事先去猜
- **安全**：只装可信来源的 skill；来源存疑要审查打包文件、依赖，以及任何让它连外网的指令

结合第 6.2 节那条——Fable 5 专页写明，为旧模型写的过度规定化 skill **可能主动降低**它的输出质量（在 Fable 5 上测得）；迁移到 Fable 5.2 时，skills 和 agent 文件都在它那张复查清单的范围里。所以 skill 不是写得越细越好。

### 8.3c 把手工复查固化成验证回路

判据很简单：**任何你不得不一遍遍手动执行的检查，都是固化的候选。** 尤其是通用 linter 抓不到的确定性规则——官方举的例子是 `Reject any migration that drops a column without a backfill step`。

四种部署模式：

| 模式 | 触发 | 适合 |
|---|---|---|
| **Standalone 独立** | 手动调用 | 不普遍适用的横切检查：安全扫描、无障碍审计、许可证核查 |
| **Embedded 内嵌** | 在产出 skill 的流程里自动跑 | 只属于某个特定任务的检查 |
| **Chained 串联** | 一个 skill 完成后调下一个 | 端到端交接；也是给**不可修改的 skill** 加验证的办法 |
| **On every PR** | CI 上跑 | 把个人习惯变成团队基础设施，不再依赖作者自觉 |

一个容易踩的坑：**内嵌只对可编辑的 skill 有效**。内置或插件管理的 skill 更新时会被覆盖，你改的也会跟着没了，那种情况必须用串联。

创建流程：挑出你最常做的手工复查 → 先试内置的 `/verify` → 像交给入职第一天的新同事那样用平实的话写清步骤 → 交给 skill-creator，或自己把 markdown 文件放进 `.claude/skills/` → 在新任务上确认检查真的跑了 → 再尝试串联。

### 8.3d 四种循环（loop）

| 类型 | 命令 | 适合 |
|---|---|---|
| **Turn-based** | 手动逐轮 | 短任务 |
| **Goal-based** | `/goal get the homepage Lighthouse score to 90 or above, stop after 5 tries` | 有**可验证退出标准**、单轮不够的任务 |
| **Time-based** | `/loop 5m check my PR, address review comments, and fix failing CI` | 周期性工作、对接外部系统 |
| **Proactive** | `/schedule every hour: check #project-feedback for bug reports` | 定义明确的持续工作流：bug 报告、issue 分诊、迁移 |

控成本的关键：**确定性的工作用脚本做**（跑脚本比逐 token 推理便宜），周期匹配实际变化频率，大规模跑之前先试点。文章的原话是 `Your model and effort level choices are among the biggest levers on what a loop costs.` Claude Code 最佳实践另补了一句：`/loop` 的每次运行都是同一个会话里的又一轮——`Each run is another turn in the same session, so don't leave one going in a long conversation.`

### 8.4 给 Claude 一个它自己能跑的检查

重写后的最佳实践页开头把三件事并列：`tell Claude what it can't see, give it a check to run, and enforce the rules that can't have exceptions`。第一件是 8.3，第三件是 8.2，这一节讲第二件：

> Claude stops when the work looks done. If it has nothing to run that tells it whether the work actually *is* done, "looks done" is all it has to go on. In practice that makes you the test suite, and every mistake sits there until you happen to notice it.

也就是说：没有它能运行的检查，"looks done" 就是它唯一的依据，你自己就成了测试套件，每个错误都要等你碰巧发现；给它一个结果是 pass 或 fail 的东西，它就能自己闭合回路：改、跑检查、读输出、修、再跑。

检查可以是：测试文件、构建的退出码、linter、把输出和 fixture 做 diff 的脚本、摆在设计稿旁边的浏览器截图。

**要它给出证据**：跑了什么命令、输出是什么，或者截图。官方原话：`"All tests pass" on its own tells you very little`；读输出比自己重跑一遍快，对你没盯着的会话来说，这也是唯一的证据。Claude 说做完之后，还可以在会话里跑 `/verify`：它会构建并运行应用、看结果，能抓到测试和类型检查漏掉的那类问题。

检查存在之后还有一个 prompt 本身答不了的问题：**它「卡」得有多硬？** 官方给了四档，从 Claude 自己判断到 Claude Code 强制执行：

| 档位 | 是什么 | 官方的说法 |
|---|---|---|
| 写在 prompt 里 | `run the auth tests after, and keep going until they pass` | Claude 自己判断什么时候满意；任何任务都能用，不需要配置 |
| 设成 goal | `/goal all tests in tests/auth pass and lint is clean`：每轮之后由另一个模型对照会话检查条件 | 自己盯着时很少需要；用处在长时间的后台运行——Claude 有时在还有未完成项时以进度总结结束回合，goal 会把它送回去 |
| `Stop` hook | Claude 想结束回合时跑你的脚本，脚本以 exit code 2 退出就把它送回去干活 | 由 Claude Code 强制执行。连续拦截八次且中间没有工具调用时，Claude Code 会放行下一次；用 `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP` 调这个上限 |
| 换个没写这段代码的来评 | `/code-review` 在一个没看过这场对话的全新子智能体里审查 diff | 它不受产生这段代码的那套推理影响 |

用户说的是没人盯着的长时间运行时，skill 会在交付说明里点一句官方的说法：这种运行靠 `/goal` 或 `Stop` hook 来强制。

想让审查对照计划而不是泛泛地找 bug，就自己写审查 prompt。官方例子：

```text
use a subagent to review the rate limiter diff against SPEC.md, check every requirement is implemented and the listed edge cases have tests, report gaps that affect correctness and skip style
```

官方特别提醒不要去掉 `skip style`：`A reviewer that's asked to find problems will always find something.` 每条发现都改，最后会多出一堆抽象层和给不可能发生的情况写的测试。官方这个例子里，子智能体审查完直接向你汇报。片段 4.16 出自 Sonnet 5 页面的 "Code review harnesses" 一节，做法相反：让审查者全报，把按置信度和严重度的过滤移出找问题这一步；该页说后面没有真正的第二步也能用（另见第 6.1 节代码审查那一行）。Opus 5 页也提醒，设了严重度门槛时模型 `might follow that instruction literally and report less`；这一倾向 Opus 5.5 页面既没有重述也没有撤回。

### 8.5 写 prompt 本身：一次说清楚

最佳实践页说具体的 prompt 有用，理由和 CLAUDE.md 一样：它给了 Claude 原本没有的信息。说出文件名、描述症状、事先讲清修好了是什么样，通常能省掉之后一轮纠正。在两个合理做法之间有偏好时也一样——Claude 没法从代码里读出你的偏好。

| 不这样写 | 这样写 |
|---|---|
| `fix the login bug` | `users report login fails after the session times out. look at token refresh in src/auth/, write a failing test that reproduces it, then fix it` |
| `add a calendar widget` | `follow the pattern in HotDogWidget.php and build a calendar widget that lets you pick a month and page between years. no new libraries` |

手上已经有它需要的东西，就直接给，不要描述：`@` 加路径带进文件内容；截图直接粘进 prompt；命令输出从 shell 管道进来，例如 `cat error.log | claude -p "what's going on here"`。

措辞上有三条官方说明。`think step by step`、`be thorough` 这类话是普通的 prompt 文字，不是关键词，控制它想多深的主要是 effort（第 6.4 节），这类措辞只能在当前档位之内推一把。仍然管用的一句是 `be ambitious`：Claude 倾向于保守地读请求，做出满足它的最小东西，想要更完整的版本就说出来——这是在告诉它**范围**，而范围只有你知道：

```text
build the settings page from the mockup. be ambitious: empty and error states, the save flow wired up, keyboard navigation, not just the layout
```

在探索而不是修某个具体问题时，`what would you improve in this file?` 这种模糊 prompt 完全可以，代价是途中多纠正几次。

**需求没想清楚时，让 Claude 反过来面试你**：

```text
I want to add rate limiting to the public API but haven't thought it through yet. Interview me until you could write the spec, then write it to SPEC.md.
```

Claude 会就没定下来的部分提多选题，一直问到答案够写成 spec。然后**开一个全新会话**照 `SPEC.md` 去做：上下文干净，而且有一份写下来的完成定义；之后让审查者对照的也是这个文件（8.4 节那条审查 prompt）。

### 8.6 可以停掉的旧做法，以及放手让它跑之前

**「What you can stop doing」**。那一节开头一句是 `Every one of these was good advice when it was written`——它们多是为模型当时还做不到的事打的补丁。审查用户给的 Claude Code prompt 或 CLAUDE.md 时，skill 照这张表查：

| 旧做法 | 官方现在的说法 |
|---|---|
| 把上下文窗口当主要约束、激进地管理 | 多数现行模型有 1M token 窗口，compaction 自动发生，很少会用完。任务之间 `/clear` 仍然值得做 |
| 给老被漏掉的指令加 IMPORTANT 或全大写 | 现行模型照平实的一句话就会做，堆强调多半让它把规则用过头 |
| 用子智能体保护上下文窗口 | 子智能体仍适合并行、挡住嘈杂输出、独立审查，但每一样都多花 token |
| 派给子智能体以求更好的答案 | 把同一个任务交给子智能体不会让答案更好；全新的 reviewer 有帮助，是因为它没看过你的推理 |
| prompt 里写 `think hard`、`think harder`、`be thorough` | 那是 effort 管的事；`ultrathink` 仍然有效，只对那一轮 |
| 叫它继续、循环到做完、别偷懒 | 现行模型通常不用说就会把说定的任务做下去，这类话更可能造成做过头。给它一个要通过的检查是另一回事 |
| 在 CLAUDE.md 里描述架构和文件布局 | Claude 自己会读仓库 |

**放手让它跑之前的五个问题**（"Let Claude run without you"）：它知道做完是什么样吗（goal 条件、spec 文件、要跑绿的失败测试）？它能检查自己的工作吗？它被允许碰什么？什么能让它停（`--max-turns`、`--max-budget-usd`、spend limit）？结果去哪儿、谁签字？官方的理由很直白：你盯着的时候，Claude 缺一点上下文不要紧，一两条消息内你就会发现；睡前启动的运行里没人告诉它，第二天早上才发现它一整夜都在错误的假设上往下做。

**子智能体的代价**（"Know what subagents cost"）。值得派的四种情况：同时跑彼此独立的工作；把长输出挡在主会话外；要一个没看过代码怎么写出来的 reviewer；给某个任务比主会话更少的工具。代价主要是 token，有时还有时间——全新的子智能体要先收集主会话已经有的上下文。省钱的办法见第 6.6 节。

**会话没按预期走时**，页面开头的诊断：做出来的和你想的不一样，多半是缺了读仓库得不到的信息（8.3）；说做完了其实没完全做完，多半是没有办法检查自己的工作（8.4）；做了绝不能发生的事或跳过了必须做的一步，那是光靠 prompt 保证不了的（8.2）；做对了但花得太多，看 `/usage`。会话里的三个控制：`Esc` 在动作中途停下并保留到那一刻为止的一切；`/rewind` 把代码、对话或两者回滚到之前的某条 prompt；同一件事纠正了两次还不对，就 `/clear`，把学到的并进一条新 prompt——第三次纠正多半只是往一段已经含有两次失败尝试的对话里再加东西。

---

## 9. 怎么练

### 9.1 学习路径

**第 1 周 —— 排毒**

把第 4 章那张表打印出来贴着。检查你手头现有的 prompt，逐条删掉已作废的东西。要迁到 Fable 5.2 的，再过一遍它专页上 "Review instructions written for earlier models" 那张清单（skill 3.13）。

**第 2 周 —— 练「说清楚」**

每写一个 prompt，用两个检验：

1. 给一个对这个任务所知甚少的人看，他能照着做下来吗？（最佳实践页的检验）
2. **一个善意的人会怎么误解它？** 这一条出自 Claude Code 团队 Cat Wu 2026-07 的公开对谈，是员工发言，不是官方文档：`you should think about the ways in which it could be misinterpreted by a well-intentioned human, in order to better understand how the model might interpret it`。skill 保留它，是因为 Fable 5.2 的页面写明规则会被照字面执行（第 1 章）

**第 3 周 —— 练「减法」**

拿你写得最长的一个 prompt，尝试砍掉一半，跑同样的任务对比结果。这一步是在建立对「脚手架变死重」的直觉；结论以对比结果为准，不以感觉为准。

**第 4 周 —— 练「往外层解决」**

每次想加一条规则时，先问：这个问题能不能不靠提示词解决？改工具设计？加确定性检查？调整信息呈现方式？

### 9.2 判断一份 prompt 好不好的六个问题

1. 有可判定的成功标准吗？
2. 每条约束都说了「为什么」吗？
3. 有没有句子删掉后什么都不会变？（有就删）
4. 一个善意的人会怎么误解它？（见第 2 周）
5. 这些内容真的需要**每次**都加载吗？（不需要就下沉到 skill / 按需加载）
6. 这条规则该由提示词管，还是该由工具 / schema / hook 确定性地保证？

### 9.3 一个持续有效的心法

> **模型每升一级，就回头删一遍你的提示词。**

Lance Martin 的表述是：随着 Claude 变强，harness 里的假设会过期，当初必要的脚手架变成死重，可能成为瓶颈。所以要**持续重新评估**。

这大概是这份文档里唯一一条不会过时的建议——因为它讲的正是「如何应对过时」。

---

## 10. 信源与推荐阅读顺序

完整信源清单、每条的读取深度、以及已知冲突的裁定，见 [SOURCES.md](../SOURCES.md)。下面只给阅读顺序。

### 建议的阅读顺序

1. **先读**：[The new rules of context engineering for Claude 5 generation models](https://claude.dev/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models/)（2026-07-24）—— 建立方向感，7 分钟读完
2. **再读**：[Best practices for Claude Code](https://code.claude.com/docs/en/best-practices)（官方文档，持续更新，在本仓库 2026-10-23 和 2026-11-09 两轮核查之间整页重写过）—— 实操内容最密集的一篇
3. **查阅**：[Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) + 你在用的模型专页 + [Choosing the right model](https://platform.claude.com/docs/en/about-claude/models/choosing-a-model) —— 当工具书查，不必通读
4. **理解原理**：[Effective context engineering for AI agents](https://www.anthropic.com/research/effective-context-engineering-for-ai-agents)（2025-09）
5. **听人怎么说**：[Fireside Chat: Cat Wu & Thariq](https://www.youtube.com/watch?v=uU5Gv2h8-9g)（2026-07）—— 有 [文字稿](https://simonwillison.net/2026/Jul/21/cat-and-thariq/)。这是员工的公开对谈，不是官方文档；本文和 skill 只保留了其中 Cat Wu 的一句检验（第 9 章），其余内容在 2026-11-09 这一轮删掉了
6. **要带着保留读**：[Best practices for prompt engineering for 2026](https://claude.com/resources/articles/best-practices-for-prompt-engineering)（2025-11）。总纲部分和 `Common mistakes to avoid` 那份清单仍然值得看，第 3.4 节也从它那儿取了一条有价值的补充，skill 里研究类 prompt 允许它说不知道的那句，也是它那个例子的第二句。但它**有三处与现行文档矛盾**：推荐 prefill（现在 API 会拒绝）；称 XML 标签 `less necessary with modern models`（文档仍在推荐）；建议在回复里用 `<thinking>` 标签做结构化的思维链，有 extended thinking 时也这样做（最佳实践页说这样的 prompt 在 Fable 5.2、Opus 5.5 和 Sonnet 5.5 上 `might be declined`）。这三处以文档为准。还有一处只是看起来矛盾：它说过度的角色设定往往没必要，文档则推荐给一个简短的角色；它反对的是长篇人设，不是角色本身，所以两者可以叠加（第 3.4 节）。它还建议先给一个示例、输出仍不合要求时再加，而最佳实践页说给 3–5 个效果最好；skill 按最佳实践页（第 5 节）

### ⚠️ 刻意排除的材料

| 材料 | 排除理由 |
|---|---|
| [prompt-eng-interactive-tutorial](https://github.com/anthropics/prompt-eng-interactive-tutorial) | 仍以 **Claude 3 Haiku** 为目标模型，主推的技法已被推翻。**需要说公道话**：官方 overview 页至今仍把它列为交互式学习者的入口，所以它并未被官方废弃；不过该页现在注明它 `was written for Claude 3 models, so follow Prompting best practices where the two differ`，而且同一页明确写着 Prompting best practices 才是 `the living reference; start there`。当概念入门可以，具体技法一律以 living reference 为准 |
| [AI prompt engineering: a deep dive](https://www.youtube.com/watch?v=T9aRN5JkmL8)（2024-12） | 网上转载最多。心智模型仍然成立，但战术层面已被取代 |
| 第三方「Claude prompt 终极指南」类文章 | 二手摘要 |

---

## 附：核查记录

保留这一节，是因为它本身就是一条方法论：**关于快速变动的文档，"核查过"不是一个状态，是一个持续动作。**

### 内容核查

初稿完成后做过一轮独立的对抗式事实核查——把每一条英文片段、每一个数字、每一处模型归属都拿回原文比对。改掉的地方包括：

- 修正了「强硬措辞会导致过触发」的适用范围，并补上 CLAUDE.md 的例外
- 修正「官方片段几乎全用 XML」的过度概括
- 修正 grep / glob 工具那段的发言人归属（Thariq，不是 Cat）
- 把 Simon Willison 的引语还原到原话强度
- 给「全新窗口优于 compaction」加回官方原有的限定词
- 把 `max_tokens` 64k 的建议限定到 `xhigh`/`max` effort 场景

### 完整性核查

后续几轮针对「是否真的全文读过每一页」，补上了这些缺口：

- **Fable 5 专页此前从未抓取**，却在模型对照表里被引用——现已读全文并补齐 Fable 5 章节，同时带出 `reasoning_extraction` 拒绝、为旧模型写的 skill 往往过度规定化并可能降低输出质量这一结论、进度真实性审计等一批此前完全缺失的内容
- **effort 页此前只看了摘要**——读全文后补上了三点：低 effort 会减少工具调用次数，`high` 等于不传参数，以及各模型起步值的差异。（第 6 节里 `ultracode` 那一条**不出自这一页**，后来查明来自 Claude Code 的 dynamic workflows 页，现已改标到那里。）
- **overview 页此前没读**——补上第 3.0 节的前置条件（成功标准 / 可测试 / 初稿），以及那句 `Not every success criterion or failing eval is best solved by prompt engineering`
- **新增第 5b 节**——发现 Opus 5 让你删验证指令、Fable 5 让你显式加自我验证，看似矛盾，实则不矛盾，这是原稿完全没处理的一处冲突
- **四篇信源此前只有薄摘要**：Agent Skills 工程文（其实从未抓取）、Agent Harness Design、验证回路、Loop engineering。补读后带出 skill 的三层渐进披露架构、缓存命中五原则、「让 Claude 自己编排」的三组实测数字、验证回路的四种部署模式、四类循环的具体命令
- **Console prompt improver 的链接已失效**（跳走），且当时跳到的那个页面仍在推荐**已被 API 禁用的 prefill**——这是个会让人直接踩坑的问题。那个替代页面后来也被撤掉了：现在跳转落在 Prompting best practices 上，而这一页明确要求迁移掉 prefill。本文原先从那个已撤页面取的内容，必须改挂在仍然存在的依据上，见 [SOURCES.md 的《The 2026-08-08 re-check》](../SOURCES.md#the-2026-08-08-re-check)
- 按适用范围做了裁剪：移除了更早模型的专属指引，而不是留着当「应该还能用」

### 2026-11-09 这一轮：第三次模型发布，四个现行模型全部进入范围，非官方内容删掉

Claude Fable 5.2 和 Claude Mythos 5.2 于 2026-11-09 发布，Fable 5.1 随之进入「Legacy models (still available)」列表。这一轮沿用前两次的做法，不做改名，每一条 Fable 专属的说法都拿 Fable 5.2 的页面重新裁定。不同的是，这一次没有沿用句可依：Fable 5.1 的页面写着现有 Fable 5 prompt 无需修改就能表现良好，Fable 5.2 的页面只说迁移时要复查 prompt，而且它的 prompting 页没有逐字重刊任何一段带 Fable 标注的旧块。所以 Fable 5.1 和 Fable 5 的材料全部改标「在该模型上测得」；Fable 5.2 页面说法不同的旧说法不再收（鼓励派并行子智能体、一句简短指令管住一类行为、做开放式功能时多交和高 effort 下擅自整理、复杂异步任务上只描述下一步而不去做、长跑周期自查、延迟最慢、长回合里安静好几分钟、档位越高提升越大）。Fable 5.1 本身退出了范围：没有哪个页面在任何一点上推荐它而不是 Fable 5.2，成本页上方向相反的几项测量，连同紧挨着研究基准那组数字的建议 `Measure what a more expensive model buys on your own workload`，记在 skill 的 models.md（第 6.2 节）。每个页面都重新抓取了一遍，旧 skill 拆成 869 条说法，逐条拿保存下来的页面审过。skill 的片段库现在有 52 个条目，其中 51 个是逐字原文，共 75 个代码块；4.37 到 4.52 是新加的。

**Sonnet 5.5 和 Haiku 5.5 进入范围。** skill 现在要在整个现行阵容里选模型，包括子智能体，所以这两个模型有了自己的表和片段，第 3 节的硬规则改成按模型的表——三处不一样：Sonnet 5.5 关前置思考用 `between_tools`；Haiku 5.5 在 `high` 及以下可以关 thinking、接受强制 `tool_choice`、没有 `reasoning_extraction` 类别。本文第 4 章里所有「两个模型都返回 400」的句子因此改写。选模型和 effort 也重写了，主会话和子智能体分开，并加了官方数字的价格、用量上限和省用量做法（第 6.5、6.6 节）。skill 起初按任务分了四档、每档一个模型；同一天的复查把它换成跟着 Anthropic 当天发布的选型文章「Which Claude model should you build with?」走：一张逐行取自它 "Pick a model by task" 的任务表、一张每个模型的调档表、模型不够好时的先后步骤和官方说法不一致的几处、一张子智能体的表。旧的那个判断什么算足够重的 agentic 工作的两条件测试没有出处，删掉了。

**非官方内容删掉了。** 仓库所有者要求删掉一切找不到官方出处的内容。落在本文上的有：第 1 章和第 5 章引的两段 Thariq Shihipar 对谈原话、第 3.2 节借用的他的一句话，以及由此推出的「agent 系统提示词不给示例」和「两种说法其实不矛盾」；第 5b 节对验证指令的整套调和（删自我复查、留外部检查，Opus 5.5 上先测再删，干净上下文 verifier 的读法，Fable 长跑周期自查）；第 4.5 节 Cat Wu 讲九成正确的规则和 always verify 的那段话；第 7.8 节 Claude Code 团队讲 grep / glob 工具、文件编辑工具、以及工具设计更像生物学的几段话，以及「先改工具」「低基数」两种说法；第 3.3 节「包了标签的都是常驻方针」的观察；第 8 章的特征测试流程；还有几句没有依据的判断（「投入产出比最高的一条」「这一步的收益通常最大」「你多半会发现质量没降」）。员工发言只保留一条：Cat Wu 那句检验，问一个善意的人会怎么误解它，标明是员工发言，保留的理由是 Fable 5.2 的页面写明规则会被照字面执行。skill 第 2 节有三条仓库所有者自己的规则，标着「用户规则，非官方；2026-11-09 确认」，本文提到时用同样的标注（第 6.5 节）。从 skill 本身删掉了哪些，完整清单见 [SOURCES.md](../SOURCES.md#the-2026-11-09-re-check)。

**Claude Code 最佳实践页整页重写了。** explore / plan / implement / commit 那一节、五种常见失败模式那一节、本文早先引的 CLAUDE.md 诊断句和大部分示例 prompt（包括 validateEmail 那张对照表和那段面试 prompt）都不在了。新增的有确定性与非确定性指令、子智能体的代价、无人值守运行和「What you can stop doing」表；表里有两样是本文和 skill 以前推荐过的：给 Claude 老是漏掉的那一行加 IMPORTANT，以及在 CLAUDE.md 里描述架构和文件布局。第 4.3 节的 CLAUDE.md 例外因此反了过来，第 8 章按新页面重写。完整记录见 [SOURCES.md](../SOURCES.md#the-2026-11-09-re-check)。

**所有者的子智能体草稿，以及信源范围。** 仓库所有者有一份没提交的草稿，讲批量子智能体和主会话用不同的模型，其中每一句都拿 Claude Code 文档核过。照文档写进来的：子智能体模型的解析顺序；每次调用传的 `model`（只接受别名）和 `effort`（Claude Code 2.1.292 及以上）；主会话是 Fable、用订阅登录时，Explore 跑在 `opus` 别名指向的模型上；同时运行的上限 20 及其改法，取代了草稿里写的大约 8 个；子智能体因用量上限这类 API 错误结束时回报什么。改正的：草稿说 `.claude/agents/` 下的定义在会话开始时缓存，文档写的是改动几秒内就会被发现，下一次委派就用新定义。文档没有提到、所以删掉的：作为批量检查点的进度文件、每批的模型记录和分层抽查、BLOCKED 约定。（Anthropic 有一篇文章确实建议 Opus 5.5 的长任务把任务清单记在文件里，这样 compaction 之后清单还在；skill 的 for-claude-code.md 照文章的写法收了这一条。）按所有者的决定、标成用户规则保留的：大批量任务里主会话和批量子智能体分开选模型；具体的模型和档位 2026-11-10 改成了取自官方基准的一张表（第 6.5 节）。信源范围也变了：claude.dev/blog 算作官方信源，claude.com/docs 下的页面和 academy 教程不再算，所以 skill 的 Chat 与 Cowork 文件改按 support.claude.com 重写，Chat 和 Cowork 的对照表删掉了。哪些事没有任何官方页面确认（例如主会话撞到用量上限时正在跑的子智能体会怎样，这一点只有两条 changelog 记录提到；又如该同时跑几个子智能体），以及 Fable 5.2 各页面逐节的覆盖表，见 [SOURCES.md](../SOURCES.md#the-2026-11-09-re-check)。

**同一天的复查。** 独立复查前后一共三轮，分别有 92、100 和 75 条发现；本文在第二、三轮的复查范围内，按第三轮发现做的修改之后逐行对着页面核对过一遍，没有再开第四轮。skill 在本文写完之后又复查了一遍，本文跟着改了这几处。第 6.2、6.5 节和本节原来把成本页的 `On research tasks, measure the model you ship` 挂在 Fable 5.2 对 Opus 5.5 那组研究基准数字上；那组数字旁边的原话其实是 `Measure what a more expensive model buys on your own workload`，前一句讲的是 effort 档位。第 7.1 节的标题原来叫「核心约束」，而重写后的 Claude Code 最佳实践把 `The context window is your main constraint, manage it aggressively` 列为可以停掉的旧做法。第 8.4 节原来说一条 prompt 只能做到四档里的第一档，第 7.7 节原来说 Edit 工具的检查正是陈旧性检查那一种，这两句都比页面说的多。还有几处「会让」「会造成」「没有效果」改回了页面的语气（`can make`、`can cause`、`changed nothing`）。

### 2026-10-23 这一轮：没有新模型进入适用范围，但官方页面一个月里悄悄改了不少

这一轮没有模型进入适用范围。Claude Sonnet 5.5（2026-09-28）和 Claude Haiku 5.5（2026-10-07）带着各自的 prompting 专页发布，Sonnet 5 进了 legacy 列表，但默认推荐没有变——选型矩阵开头仍是 "Most workloads start with Claude Opus 5.5"——所以 skill 让这两个新模型继续留在范围之外，只加了一段说明：它的硬规则在哪些地方套不到它们身上。

变的是已经在跟踪的页面，而且大多没有发布说明。仓库引用的官方句子里有十三句被改了措辞，小到一个词（"finish" 改成 "complete"，"may" 改成 "might"），大到模型列表变长；另有一句随它所在的页面一起消失了。一段被引用的代码块——Fable 5 那段最终汇报可读性片段——被删掉了一个括号里的说明。其中两处改动改的是事实而不只是措辞。`medium` 默认档不再是 Opus 5.5 独有——Haiku 5.5 也是——所以所有「其他模型都是 `high`」的句子都得改掉。`reasoning_extraction` 在 Opus 5.5 上也不再被说成是新增的：what's-new 页现在介绍它时用的是 "As on Claude Opus 5"，所以在它点名的分类器里，对从 Opus 5 过来的人来说新增的只有生物分类器。

本文有一条建议不是改措辞，而是撤回。第 5 章的表格原来说：演示推理方式最好在 few-shot 示例里用 `<thinking>` 标签。官方文档已经不这么说了——现在的写法是把每个例题写成问题、要用的方法、预期答案——而且 refusals 页把让模型填写的 `<thinking>` 区段列为 `reasoning_extraction` 拒绝的常见来源。

这一轮还查出好几处本仓库自己的错，这里记其中三处。第 6.2 节仍然给的是 Opus 5.5 之前的选型建议（「从 Claude Opus 5 起步」），第 4 章也仍然把 Opus 系列说成不受强制工具调用变化影响：2026-09-22 那一轮扫过 skill 和单文件版里残留的 Opus 5 说法，但没有扫本文。2026-09-22 那一轮改标为「仓库自己写的」那句 `If the data is insufficient to draw conclusions, say so rather than speculating.`，其实是 Anthropic 2025-11 那篇 prompt engineering 博客里那个例子的第二句，现在已注明出处。还有一句「官方对输出格式类 skill 的指引是先研究」，查下来根本没有出处；建议本身保留，改标为 skill 自己的建议。完整记录见 [SOURCES.md](../SOURCES.md#the-2026-10-23-re-check)。

### 2026-09-22 这一轮：第二次模型发布，以及一条没能活下来的规则

Claude Opus 5.5 于 2026-09-22 发布，Claude Opus 5 随之和 Fable 5 一起进入「Legacy models (still available)」列表。这一轮沿用了 Fable 那一轮的做法：不做改名，每一条 Opus 专属说法都拿 Opus 5.5 的页面重新裁定；凡是拿不到 Opus 5.5 明确依据的，要么进「在 Opus 5 上测得」的表，要么标成未确认。

**值得一读的发现是第 4.4 节。**「删掉自我验证指令」本来是本文的招牌规则之一，而它在 Opus 5.5 上并不成立为规则。Opus 5.5 的 prompting 页和 what's-new 页对过度验证这件事两边都没说；最佳实践页仍然把 Claude Opus 5 列为它"Ask Claude to self-check"这条通用建议的唯一例外；Opus 5.5 的迁移指南只给了一句 `Instructions tuned for Claude Opus 5's behavior might no longer be needed`。如果这一轮做的是全局把「Opus 5」替换成「Opus 5.5」，就会把一条有记录的、模型专属的实测结论，变成对一个没有人测过的模型的确定断言。为什么要逐条裁定而不是改名——这一条规则就是全部理由。

另有三处发生了形态变化。effort 的默认值变成按模型定（Opus 5.5 是 `medium`），这让旧的「两个模型都从 `high` 起步」这个框架和 `effort: "high"` ≡ 不传参数的简写都失效了。三条硬规则扩大到覆盖 Opus 5.5 的破坏性变更——thinking 无法关闭、强制 `tool_choice` 返回 400、以及 thinking 页把 `reasoning_extraction` 拒绝类别也点名到了 Opus 5.5。还有进度播报机制变了：工具调用之间的文字现在以默认为空的 `thinking` 块返回，所以一个为 Opus 5 写的客户端会在整个长 agentic 回合里安静下来，而且没有任何报错可以指认。

这一轮还标出一件属于隔壁的事：skill 的 chat-and-Cowork 参考文件里有一个代码块——`If the data is insufficient to draw conclusions, say so rather than speculating.`——就放在一段被介绍为「照抄即可」的官方块下面，却没有任何标记说明它不一样。那一轮的结论是它在任何 Anthropic 页面上都不存在，于是把它标成了仓库自己的句子。这个结论本身是错的——这句话是 Anthropic 2025-11 那篇 prompt engineering 博客里那个例子的第二句，那一轮没有搜那篇博客——2026-10-23 这一轮已经改正了标注。完整记录见 [SOURCES.md](../SOURCES.md#the-2026-09-22-re-check)。

### 2026-09-03 这一轮：一次模型发布让所有 Fable 的说法重新裁定

Claude Fable 5.1 于 2026 年 9 月 1 日发布，带着自己的 prompting 专页，Fable 5 进入旧模型列表。这一轮没有做全局改名，而是把仓库里每一条 Fable 专属的说法拿 Fable 5.1 的页面重新裁定：Fable 5.1 页面重刊或写明沿用的保留；Fable 5.1 页面说法不同的改写；被 Fable 5.1 页面取代的退役（片段 4.18 的自主运行块）；Fable 5.1 页面既没重述也没撤回的行为判断，则明确标上「在 Fable 5 上测得」而不是悄悄带过去。查了什么、哪些页面搬了家、哪些规则因此改写，记录在 [SOURCES.md](../SOURCES.md#the-2026-09-03-re-check)。顺带查出两处出处标错：去开场白那句原文在 prompting 最佳实践页而不是迁移指南；"most capable widely released model" 这句当时在选型页而不是模型总览页（选型页此后已改了措辞）。

### 一次真实的教训：文档在核查期间被改了

这一条值得完整记录，因为它比上面所有条目都更能说明问题。

第 6.4 节说起的 Opus 5 effort 起步值，本文最初写的是「编码 / agent 任务起步用 `xhigh`」，依据是官方 Opus 5 页当时的措辞：`for coding and agentic work, xhigh remains the recommended starting point`。

审查过程中有两次独立复核报告说该页写的是从 `high` 起步。当时对同一页做了三次直接抓取，三次都返回上面那句 `xhigh`；同时 effort 页也没有 Opus 5 专属章节。于是这两份报告被记为「无法复现」。

结果是**报告没错，页面改了**。当时改后的那一页写的是：

> "**Efficiency at lower effort:** `low` and `medium` effort produce strong quality at a fraction of the tokens and latency of higher settings. Start with the default (`high`) and adjust based on your evals: use `low` and `medium` liberally as your primary control for token cost and response time wherever quality holds, and step up to `xhigh` for demanding coding and agentic work."

`remains the recommended starting point` 这句话已经不在页面上任何位置，effort 页也新增了 Opus 5、Fable 5 以及（2026 年 9 月起）Fable 5.1 和 Opus 5.5 各自的推荐档位章节。本文和 skill 都已按新版改正。

**教训有两条**：

1. **实际差别是真的。** 不是措辞润色——一个普通编码任务的起步值从 `xhigh` 变成了 `high`，直接影响成本。
2. **对快速变动的文档，快照出错是常态，不是例外。** 凡是用来规划成本或延迟的，都别只看二手整理（包括本文），去原页面确认一遍——那是唯一会随模型更新的地方。

---

*除标明的那一条员工发言和三条用户规则外，本文的所有技术主张均可追溯到 [SOURCES.md](../SOURCES.md) 列出的官方信源。若与你在别处看到的说法冲突，请优先核对官方文档的发布 / 更新日期。发现分歧欢迎提 issue——以官方文档为准，不以本仓库为准。*
