---
name: prompt-engineering
description: Writes, rewrites, reviews, and audits prompts and system prompts using Anthropic's current official guidance, and selects the right model and effort level for the task before drafting. Use when the user asks to write a prompt, improve or fix a prompt, design a system prompt, or check a prompt before running a task — including phrasings like "帮我写 prompt"、"改一下这个 prompt"、"设计 system prompt"、"处理任务之前先检查我的 prompt"、"帮我把需求整理成 prompt"、"help me prompt this better". Applies whether the resulting prompt will be pasted into Claude Code for a coding task or run in a desktop chat or Cowork session for a non-coding task. Also use when the user pastes a prompt and asks why it is not working, when they are authoring an agent system prompt or CLAUDE.md, or when they ask which model or effort level to use. Several widely repeated prompt techniques are now obsolete or rejected by the API, so consult this skill even when prompting from memory feels sufficient.
---

# 写 prompt 的规范

**适用模型**：官方现行的四个模型——Claude Fable 5.2（含 Mythos 5.2）、Claude Opus 5.5、Claude Sonnet 5.5、Claude Haiku 5.5。Claude Fable 5.1、Fable 5、Opus 5、Sonnet 5 已在官方 legacy 列表里（仍可用），它们在这里只以三种身份出现：某条措辞的出处、迁移的起点、现行页面点名的对照（fallback 目标、价格或基准对比）。每条针对具体模型的说法分三类处理：当前页面明确写了的直接用；只在前代页面写过、当前页面既没重述也没撤回的，放进 `references/models.md` 里标明「在该模型上测得」的表；已撤回或与当前页面冲突的不收。

**这份 skill 存在的理由**：一批广为流传的 prompt 技法在现行模型上已经失效、被 API 拒绝、或效果反了。你的训练直觉里很可能仍带着它们。**当直觉与第 3 节冲突时，以本文件为准。**

**基准**：Anthropic 官方文档、官方博客与支持站，快照日期 2026-11-09。标着「用户规则，非官方」的几条是本 skill 所有者定的做法，不是官方指引。涉及具体 API 参数时回原文档确认——那是唯一会随模型更新的地方。

---

## 0. 工作流

1. **判断目标场景**（第 1 节）——决定读哪个 target 参考文件
2. **选定模型和 effort**（第 2 节）——默认由你自己定，什么时候才问用户见第 2 节的用户规则
3. **补齐缺失变量**（第 1 节末尾）——只问真正问不出就写不了的
4. **起草**：读对应 target 文件；需要现成模块时再读 `references/snippets.md`
5. **过第 7 节自检**
6. **交付**：prompt 本体 + 一段交付说明。说明包含四件事——选了哪个模型和 effort 以及依据、替用户做了哪些假设、哪里最可能需要调、有没有只有用户能补的信息。能压到两三句就两三句；**假设较多时如实写长，不要为了"简短"把假设藏起来**——用户看不到假设就没法判断要不要改。

---

## 1. 先判断场景

用户在 Claude desktop 里描述需求，产出的 prompt 会去两个地方之一。**先判断去哪儿。**

| 信号 | 场景 | 读这个文件 |
|---|---|---|
| 涉及代码库、文件、git、跑测试、重构、修 bug、建项目；用户说要拿去 Claude Code 或 VS Code 跑 | **A：给 Claude Code 的任务 prompt** | `references/for-claude-code.md` |
| 研究、分析、写作、整理资料、做文档表格幻灯片、日程、信息汇总；在 Claude 应用里直接跑 | **B：给 chat 或 Cowork 的任务 prompt** | `references/for-chat-and-cowork.md` |
| 用户在聊天途中丢来一段 prompt 问"这样写行不行" | **C：临时检查** | 见第 6 节，走轻量流程 |

用户说了在哪里跑，就按他说的：在 Claude Code 里跑的非代码任务（批量读文件、提取、汇总）也走 A。没说、又判断不了时按用户描述里的**动词**判断：动代码用 A，其余用 B。两边都沾（例如"分析这个仓库的技术债并写成报告"）时按**最终交付物**判断——交付物是代码改动走 A，是文档走 B。

### 补齐变量

只问真正问不出就写不了的。缺失但能从上下文合理推断的，**直接假设并在交付时说明**。

下表的"必须确定"指的是**这个值必须有明确结论**，不是"必须开口问"。能从上下文推断就推断，推断了就在交付说明里写清楚。

| 变量 | 要求 |
|---|---|
| **成功标准** | **必须确定**。没有可判定的成功标准就写不出好 prompt，也没法自检 |
| **输出形态与长度** | 交付物是文档或报告时**必须确定**。长度要写在 prompt 里，不要指望 effort：Fable 5.2 页面写明 effort 几乎不改变回复长度，Opus 5 页面写明降 effort 不能可靠缩短可见输出（见第 8 节「长度控制」） |
| **可用的工具或文件** | 场景 A 必须确定（哪个仓库、哪些文件）；场景 B 看是否需要读取材料 |
| 目标模型与 effort | 按第 2 节自己选定，把结论和依据告诉用户；只在第 2 节用户规则 1 说的那种情况下才问 |

需要问多项时用 AskUserQuestion **一次性问全**，不要逐个问。

---

## 2. 选定模型和 effort

这是起草前的必经一步，结论要写进交付说明里。本节只写官方给了依据的东西；官方没给阈值的地方不设阈值。数字和更细的说明在 `references/models.md`。

**三条用户规则**（用户规则，非官方；2026-11-09 确认。它们管的是在官方给的选项之间怎么取，不改变下面各档的官方依据）：

1. **模型和 effort 默认由你自己定。** 只有取舍确实存在、从上下文又推断不出来时，才在输出 prompt 之前问用户一次，并带上选项和推荐项。
2. **不刻意省用量，也不无依据地往上加。** 用户是 Ultra 订阅：不要仅仅为了省用量，把模型或档位选得比任务对应的那一档低；也不要在没有本节所列依据的情况下，把模型或档位往上提。
3. **大批量任务里，主会话和批量子智能体分开选模型**，见本节「大批量处理」。那里给批量子智能体定的模型和档位是用户对这类任务的明确选择，不受规则 2 约束。

官方的起点（Anthropic 2026-11-09 的选型文章「Which Claude model should you build with?」）：`you should start most work on Opus 5.5 at its default effort level`，也就是 `medium`。四个现行模型按价格和能力从低到高是 Haiku 5.5、Sonnet 5.5、Opus 5.5、Fable 5.2。

### 主会话：按任务选

这张表是那篇选型文章的 "Pick a model by task"，逐行对应。平台文档的选型矩阵同样以 Opus 5.5 为起点（`Most workloads start with Claude Opus 5.5`），但有两行分法不同：矩阵把大规模重构放在 Opus 5.5 名下，把要一直做到成品文档、表格或幻灯片的分析放在 Fable 5.2 名下。

| 任务 | 起步模型 | 起步 effort | 官方给的理由 |
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

同一篇文章对往下、往上走的补充：范围清楚的日常工作往下到 Sonnet 5.5，但 `Just because you have a clear spec doesn't mean you should use Sonnet`——迁移或性能优化可以 spec 很清楚，难度仍然够得上 Opus 5.5，大的够得上 Fable 5.2。往上到 Fable 5.2 的三种情况：编码、金融和科学里最难的技术项目；Opus 5.5 在 `high` 下仍然不够；长时间无人值守的运行。Claude Code 的 model-config 页：Fable 模型在任何计划上都不是默认，要用 `/model fable` 显式选。

### 每个模型怎么调档

任务表给的是起步档；任务一开始就符合下表某一格的上调或下调条件时，直接用那一格的档位。

| 模型 | 默认档 | 什么时候上调 | 什么时候下调 |
|---|---|---|---|
| **Haiku 5.5** | `medium` | 知识工作、较长的 agent 任务、要求严格遵循指令 → `high`。`xhigh`/`max` 只在评测显示质量提升时用，并且同时拿 Sonnet 5.5 比一比 | chat、短工具任务、简单的高吞吐请求 → `low`（长 agent prompt 里它在这一档更容易跳过搜索、提前停、跳过检查） |
| **Sonnet 5.5** | API `high`；Claude Code `medium` | agentic 编码和多步工具使用：说清了的任务从 `medium` 起，更难或更长的 → `high`。`xhigh`/`max` 只在评测显示质量提升时用；Anthropic 的 Sonnet 5.5 文章说，这两档下有些任务会失去 Sonnet 在质量、速度和成本之间的平衡，那种情况考虑 Opus 5.5 | chat 和其他延迟敏感的工作 → `medium` 或 `low` |
| **Opus 5.5** | `medium` | → `high`：需要验证、或很可能有边界情况的工作，比如在现有代码库里修 bug（Claude Code 文档）。→ `xhigh`/`max`：官方原文 `Reserve xhigh and max for work where you've measured a quality gain`；选型文章另说，难任务上把 Opus 5.5 提到 `high` 以上之前，先试 Fable 5.2 | 在意成本或延迟且评测显示质量不掉 → `low`；来回很快、每个结果都会看（头脑风暴、初稿、重命名这类小改动） |
| **Fable 5.2** | API `high`；Claude Code 和 Claude 应用 `medium` | `medium` 适合交互式编码、chat 和日常知识工作；→ `high`：困难的编码、研究、agentic 工作和长时间无人值守的运行（官方：从 `medium` 到 `high` 这一步在它跟踪的多数编码和知识工作评测上提高了分数，也多花 token 和时间）。→ `xhigh`：最难的长时间编码和 agentic 工作，且测到了提升。`max`：用来测模型上限，或时间和 token 都无所谓的少数任务 | 评测显示质量不掉的地方 → `medium`；快速来回、说清了的任务、高吞吐（比如子智能体）→ `low` |

出处：effort 页各模型一节、四个模型的 prompting 专页、Claude Code 的 model-config 页（"Choose an effort level"）；Fable 5.2 在 Claude Code 和 Claude 应用里默认 `medium` 出自 model-config 页和发布公告。Claude Code 里只有下一条消息需要更深推理时，在那条 prompt 里写 `ultrathink`；`max` 在普通任务上容易想过头。

### 质量不够、或成本太高时的顺序

选型文章给的步骤，模型在某个任务上不够好时依次做：

1. 确认模型有办法验证自己的工作：一个测试、一次构建、或一个调用端点的脚本。文章的话：给它自我验证的手段，往往比换档位更划算、加的延迟更少
2. effort 升一档
3. 换到上一档模型，并且**从比小模型上用的更低的 effort 起**

配套的几条官方说法：

- **先调 effort，再换模型**：`Tuning effort is often a better lever than switching models`（choosing-a-model 页）
- **过了 `high` 就比较两条路**：再升一档 effort 的花费可能和换一个更大的模型一样多；已经过了 `high`，就试上一档模型配更低的 effort
- **升 effort 补不了模型没有的知识或能力**，那种情况要换模型
- **成本太高、质量没问题** → 在现在这个模型上试更低的 effort（成本页）
- **不要把别的模型上调好的档位照搬过来**；**按每个完成的任务比成本，不按每 token**

**下面几处官方说法不完全一致，交付说明里用到哪一处就点到哪一处**（场景 B 还有一处，支持站的 `try Extra high (xhigh) first`，见 `references/for-chat-and-cowork.md`）**：**

- **什么时候从 Opus 5.5 换到 Fable 5.2。** 选型文章：Opus 5.5 在 `high` 下仍然不够。choosing-a-model 页：评测在 `xhigh` 或 `max` 下仍然不够。成本页：质量不够时，降过 effort 就先恢复，否则换到上一档模型的默认 effort。what's-new 页和模型总览：在更高的 effort 下仍然不够。本 skill 按选型文章那三步走，它是其中唯一把自我验证放在第一步的；各处的原话见 `references/models.md`
- **研究类任务。** 任务表和选型矩阵都把深度研究放在 Fable 5.2 名下；成本页在 DeepResearch Bench II 上测得 Fable 5.2 的成本是 Opus 5.5 的 2.9 倍、分数低 1.9 分，它的话是 `Measure what a more expensive model buys on your own workload`
- **Fable 5.2 从哪一档起。** 任务表（选型文章）从 `medium` 起，Claude Code 和 Claude 应用的默认档也是 `medium`；Fable 5.2 专页、what's-new 页、effort 页和 choosing-a-model 页写的都是从 `high` 起，那是 Claude API 的默认档，专页还说 `high` 适合困难的编码、研究和 agentic 工作。场景 A、B 按任务表从 `medium` 起、不够再 `high`；写给 API 的按平台页从 `high` 起
- **从哪个模型起步。** 选型文章和选型矩阵：多数工作从 Opus 5.5 起。choosing-a-model 页另给了一条效率优先的路：先用 Haiku 5.5 实现，只在有具体能力缺口时才升级，它列的适用情况是初期原型和开发、延迟要求紧的应用、对成本敏感的实现、大量而直接的任务。本 skill 按任务表；用户的任务落在这四种情况里时，把这条路也告诉他
- **Claude Code 里日常编码用哪个模型。** 选型文章：日常工程从默认的 Opus 5.5 起。Claude Code 最佳实践：`the default Opus 5.5 handles most coding well and Sonnet costs less on routine work`。Claude Code 成本页：`Sonnet handles most coding tasks well and costs less than Opus. Reserve Opus for complex architectural decisions or multi-step reasoning.` 本 skill 按任务表：范围清楚的编码用 Sonnet 5.5，其余日常工程从 Opus 5.5 起

### 子智能体：按任务选（场景 A）

Claude Code 里没指定模型的子智能体跟主会话用同一个模型，所以主会话是 Fable 5.2 时，子智能体默认也在 Fable 5.2 上跑（例外见 `references/for-claude-code.md`：几个内置子智能体有自己的模型，设了 `CLAUDE_CODE_SUBAGENT_MODEL` 时默认值也会变）。Fable 5.2 专页的原话：`subagents can account for a large part of what a session costs, so check which model your harness runs subagents on by default`。解析顺序、每次调用能传的参数、并发和出错时的行为，见 `references/for-claude-code.md`「子智能体」一节。

| 子智能体做什么 | 模型 | 官方依据 |
|---|---|---|
| 搜索文件、读日志、做摘要、检查文档链接 | `haiku` 或 `sonnet`，effort 用该模型的默认档 | 选型文章的任务表；Claude Code 最佳实践：`Set model: haiku in a subagent's definition if its work is simple` |
| 高吞吐、产出能校验的批量工作 | 先试 `haiku` | 成本页：`High-volume work with outputs you can check` → `Try Claude Haiku 5.5 first` |
| 超出一个上下文窗口的工作，拆成每部分一个子智能体 | 比主会话便宜的模型；官方测得 Opus 5.5 编排者带 `sonnet` 时分数和它自己在 `medium` 单独做相当；`haiku` 也测了，分数明显更低；Fable 5.2 编排者的数字见下 | 成本页：`Split the work and hand the parts to lower-cost models`。测量见下 |
| agent team 的队友 | `sonnet` | Claude Code 成本页：`Use Sonnet for teammates` |

**官方的分工原则**（选型文章）：`Keep small models on work where a mistake is cheap to spot, like finding files, running tests and reading logs. Keep code edits and judgment calls on the main model.` 主模型越大，精简的主上下文越要紧：`Keep the expensive model's context for the decisions, and let cheaper subagents do the verbose reading.`

**官方测过什么**（成本页，内部缺陷搜索基准，工作量超出一个上下文窗口）：Opus 5.5 带 Sonnet 5.5 子智能体，比每部分一个 Opus 5.5 会话便宜约三分之一，分数相当；带 Haiku 5.5 子智能体时分数明显更低；把编排者换成 Fable 5.2，成本高 19%，分数相当。Fable 5.2 自己单独做（`low` 到 `high`），比它自己带 Sonnet 5.5 子智能体时高 3.2 到 7.5 分，成本是 2.2 到 3.4 倍。编码任务上的 Haiku 5.5 子智能体，成本页的结论是 `an alternative to lowering effort, not better`。

**什么时候值得派**：官方测得委派只在两种情况下划算——有很长成本尾巴的例行工作（这种情况官方要求先测准确率和成本），和超出一个上下文窗口的工作。工作是一条前后依赖的链、或一个上下文装得下又没有长成本尾巴时，编排者模型自己用更低的 effort 更划算（在更早的模型上测过的每个案例里都是如此）；选型文章的说法是那种情况下规划和合并的花费比省下的多。Claude Code 文档：子智能体带来并行、把长输出挡在主会话外、和一双没看过过程的眼睛，`but not a smarter answer`。成本页对要不要搭配第二个模型的判据：先在现在的模型上扫 effort，`Build a pairing only if it beats the single model's whole curve of score against spend`。

**子智能体的 effort**：选型文章给搜索、读日志、做摘要这类子智能体的起步档是该模型的默认档。effort 页把子智能体列为 `low` 的例子，Fable 5.2 专页说 `low` 适合高吞吐工作里又快又聪明的子智能体。降档前先测：缺陷搜索基准上，Opus 5.5 单独做时 `low` 比 `medium` 低 34 分，Fable 5.2 单独做时 `low` 比 `high` 低 4.3 分、成本少 37%。

### 大批量处理：主会话和批量子智能体分开选模型

**什么时候用**（用户规则 3 的适用范围，是用户定的，不是官方给的触发条件）：场景 A，主会话选的是 Fable 5.2，而起草前就能预判任务要并行派很多子智能体、按同一份口径做批量工作——读几十个文件或阅读包、给成百上千条内容提取 / 标注 / 核对。遇到这种任务直接写进 prompt，并在交付说明里点一句。

**官方对这类工作怎么说**（成本页 "Orchestrator strategy"）：适合编排的是 `Work that fans out across genuinely independent files, documents, or cases, especially more than one context window of it`；做法是拆开、把各部分交给更便宜的模型，编排的那个模型负责 `Plans, dispatches, and synthesizes`。选型文章的说法是把代码编辑和判断留在主模型。官方同时测得两点，和用户的任务对得上时在交付说明里提：一个上下文装得下、又没有长成本尾巴的工作，编排者模型自己用更低的 effort 做更划算（在更早的模型上测得）；Fable 5.2 自己单独做的分数比带 Sonnet 5.5 子智能体的配置高（见上一小节的数字），拆出去换来的是成本。

**用哪个模型和档位**（用户规则，非官方；2026-11-09 确认）：

- 批量子智能体：派发时 Agent 工具传 `model: "opus"`。官方没有测过「Fable 5.2 主会话配 Opus 5.5 子智能体」这个组合；官方写明并测过的更便宜的选项是 `sonnet`（缺陷搜索基准上配 Opus 5.5 编排者分数相当），以及读文件做摘要、产出能校验的高吞吐工作用 `haiku`
- effort：提炼、标注、抽样核对、找证据传 `effort: "high"`，最后的独立复核传 `"xhigh"`。这比官方给的起点高：最接近的官方说法是选型文章给搜索、读日志、做摘要这类子智能体的起步档，即模型的默认档（Opus 5.5 是 `medium`），`xhigh` 留给实测到质量提升的工作

`model` 和 `effort` 这两个参数只对非 fork 的子智能体起作用：fork 跑在主会话的模型上，而 fork 模式默认开着，所以 prompt 里要写明这批工作派给非 fork 的子智能体（见 `references/for-claude-code.md` 里 fork 那一条）。这批工作改用 dynamic workflow 跑时，让脚本给批量阶段写 `opus`：workflows 页说，脚本给某个阶段写的模型算作每次调用传的那一项；workflow agent 的 effort 官方没有写。每次调用传 `effort` 需要 Claude Code 2.1.292 及以上；并发上限、子智能体出错或撞到用量上限时结果怎么回来，见 `references/for-claude-code.md`「子智能体」一节。

### 用量与成本

只收官方说了的，完整数字在 `references/models.md`「价格、用量上限与省用量做法」。写 prompt 时最常用到的几条：

- 价格每百万 token（输入 / 输出）：Fable 5.2 $8 / $40，Opus 5.5 $4 / $20，Sonnet 5.5 $2 / $10，Haiku 5.5 $0.10 / $0.50 起
- 订阅上，子智能体的请求和主会话计入同一个用量上限；Max 和 Ultra 上 Fable 模型最多用掉每周上限的一半，而且用得比其他模型快；effort 越高，越快到上限
- 输出 token 比输入贵五倍，agent 循环里模型写的每个 token 之后每轮都会作为输入再算一次，所以要它给你会读的那个答案
- 产出能校验时（API）：`low` 跑一遍、失败的用 `high` 重跑，官方在 Opus 5.5 上测得比全部 `high` 便宜约 40%、通过率相同

**这些参数分别在哪儿设**：

| | 场景 A：Claude Code | 场景 B：Claude 应用 | 裸 API |
|---|---|---|---|
| 模型 | `/model`、`--model`；子智能体见上 | 发送按钮旁的模型菜单 | `model` |
| effort | `/effort` 或 `/model` 里调，按模型分别保存（`max` 只对当前会话有效）；`ultrathink` 只管一轮 | 模型菜单里的 "Effort"，推荐档标着 "Default" | `output_config: {effort}` |
| thinking | 由客户端管理 | 现行四个模型上关不掉 | 见 3.2 |
| `max_tokens` | prompt 里设不了（只有 `CLAUDE_CODE_MAX_OUTPUT_TOKENS` 环境变量） | 没有这个设置 | 你自己设，见 `references/models.md` |

所以场景 A 和 B 的交付说明里写清模型和 effort 档位就够了，不要把 `max_tokens`、`thinking` 写进去。

---

## 3. 失效清单（硬规则）

3.1、3.2、3.8、3.12 是 API 层面的拒绝；其余是官方写明的行为问题。各模型不一样的地方：

| | Fable 5.2 / Mythos 5.2 | Opus 5.5 | Sonnet 5.5 | Haiku 5.5 |
|---|---|---|---|---|
| prefill（3.1） | 400 | 400 | 400 | 400 |
| `budget_tokens`（3.2） | 400 | 400 | 400 | 400 |
| 关掉 thinking（3.2） | 400 | 400 | 400；改用 `between_tools`，限 `high` 及以下 | `high` 及以下可以；`xhigh`/`max` 返回 400 |
| 非默认采样参数（3.8） | 400 | 400 | 400 | 400 |
| `reasoning_extraction` 拒绝（3.9） | 有（官方点名的是 Fable 5.2） | 有 | 有 | 没有这个类别 |
| 强制 `tool_choice`（3.12） | 400 | 400 | 400 | 接受 |

### 3.1 不要用 prefill（预填充助手回复）
四个现行模型**均不支持**，在最后一个 assistant turn 预填充会返回 400。官方原文是 `Starting with Claude 4.6 models and Claude Mythos Preview, prefilled responses (providing a partial assistant message for Claude to continue from) on the last assistant turn are no longer supported`；Haiku 5.5 的迁移指南写明它即使关掉 thinking 也拒绝。

官方给的替代是 **structured outputs 或 system prompt 指令**。具体来说：结构化输出用 Structured Outputs、`output_config.format`，或直接说明 schema；去开场白写 `Respond directly without preamble. Do not start with phrases like 'Here is...', 'Based on...', etc.`（prompting 最佳实践页 "Migrating away from prefilled responses" 一节的原文）；分类任务用带 enum 的 tool；续写把上文放进 user 消息。

### 3.2 不要用 `budget_tokens`——关 thinking 的办法按模型不同
四个现行模型都不接受手动 extended thinking，`{"type": "enabled"}` 带 `budget_tokens` 返回 400。不传 `thinking` 或传 `{"type": "adaptive"}`（两者等价），用 `effort` 控深度。需要硬上限用 `max_tokens`。

- **Fable 5.2 和 Opus 5.5**：thinking 始终开启、无法关闭，`{"type": "disabled"}` 和 `{"type": "between_tools"}` 都返回 400。官方报错文案：

```
"thinking.type.disabled" is not supported for this model. Use "thinking.type.adaptive" and "output_config.effort" to control thinking behavior.
```

- **Sonnet 5.5**：`disabled` 在每个 effort 档都返回 400；想关掉前置思考，传 `thinking: {"type": "between_tools"}`，只在 `high` 及以下可用
- **Haiku 5.5**：`disabled` 在 `low`、`medium`、`high` 下可用，`xhigh` 和 `max` 下返回 400

因为回复可能以 `thinking` 块开头（默认 `display: "omitted"` 下内容为空），**要按 `type` 字段而不是按位置取 content block**，并在工具循环里原样回传 `thinking` 块。

### 3.3 不要用堆砌指令代替上下文
核心原则：**给上下文，别给一长串指令。**

Claude Code 团队为 Opus 5、Fable 5 这类模型把系统提示词删掉了 80% 以上，在他们的编码评测上没有可测的损失。Claude Code 最佳实践现在的说法是：只写下 Claude 读仓库找不到的东西（`Only write down what Claude can't find by reading the repo, or what it would otherwise spend time rediscovering every session`）。Fable 5.2 专页讲的是反面：记忆、笔记和指令文件里的谨慎类指令会越积越多，而它照得很紧，于是可能要不必要的审批、或压着例行工作不做（少数早期测试者见过，多在几个 agent 协作的场合）。

与其枚举十条"不要 X"，写一句正面描述加说明意图。最佳实践页给过一个例子（针对 Opus 4.6 写的）：把 `Default to using [tool]` 改成 `Use [tool] when it would enhance your understanding of the problem`。

> **强调词不是办法。** Claude Code 最佳实践写明：CLAUDE.md 里某一行 Claude 老是忽略时，`adding IMPORTANT probably won't fix it`——要么文件长到那一行被淹没了，该缩短文件；要么它是不能有例外的规则，该做成 hook。同一页把给老被漏掉的指令加 IMPORTANT 或全大写列进了可以停掉的旧做法：现行模型照平实的一句话就会做，堆强调多半只会让它把规则用过头。

### 3.4 自我验证指令：按模型看官方怎么说
各页面的说法不一样，不能一条规则套所有模型。原话和出处的完整表在 `references/models.md`「验证指令」一节，写 prompt 时按下面处理：

- **通用建议**（最佳实践页）是让模型自检，`This catches errors reliably, especially for coding and math`
- **Opus 5（legacy）是这条建议唯一点名的例外**：它自己会验证，沿用的验证指令可能造成过度验证，官方原话 `remove these instructions rather than rewriting them`。Opus 5.5 的页面对此既没重述也没撤回。Anthropic 的文章「What a task costs on Opus 5.5」（2026-09）报告过一次测量：一个内部客服基准从 Opus 4.8 迁到 Opus 5.5（`low` 档），审计删掉了让模型多写、重复调用工具的例行指令（`a mandatory six-step procedure, a scratchpad rule, a verify-twice rule, and instructions that contradicted each other`），成本又降了 9%；文章自己说这只是一个基准上的例子
- **Fable 模型在 Claude Code 里**：`Skip the verification reminders: it verifies its own work with less prompting, so reminders to test or check are usually unnecessary`
- **Sonnet 5.5 在 `low`、Haiku 5.5 在 `low` 和 `medium` 做编码时**有时不跑检查就报告完成：看到这种情况就加片段 4.46
- **Sonnet 5.5 在 `xhigh` 和 `max`** 反过来，可能自己开几轮复查：想收住就加片段 4.42

和模型无关的一条（Claude Code 最佳实践）：场景 A 要给它一个能跑、结果是 pass 或 fail 的检查，并要它给出证据。这不是「提醒它再看一遍」，而是给它一个外部信号，见 `references/for-claude-code.md`。

### 3.5 不要用否定式堆砌约束
最佳实践页的原则是 `Tell Claude what to do instead of what not to do`，并且要给原因：

- ✗ `NEVER use ellipses`
- ✓ `Your response will be read aloud by a text-to-speech engine, so never use ellipses since the text-to-speech engine will not know how to pronounce them.`

第二句里其实仍有 "never"，但它挂在一个**原因**上——模型知道原因，遇到没被覆盖的边界情况也能自己判断。Fable 5.2 专页对积攒下来的规则给的也是这个方向：`rewrite absolute rules so that they state their scope and reason`；需要例外时把例外写进规则本身；官方测得另起一行说明，结果基本没变。

### 3.6 不要用绝对规则代替判断力
- ✗ `Never write multi-paragraph docstrings or multi-line comment blocks — one short line max`
- ✓ `Write code that reads like the surrounding code: match its comment density, naming, and idiom`

### 3.7 不要在 system prompt 与 tool description 之间重复
工具的使用说明**只写在 tool description 一处**。

> **不算重复的：触发时机。** 强制工具调用被拒之后，官方给的替代就是在 prompt 里说明什么时候用这个工具（见 3.12）。片段 4.23 也是这样：Fable 5 专页写明 Fable 5 光靠 tool description 很少会去调 `send_to_user`，所以官方在工具之外还配了一句写进 system prompt 的话，告诉它**什么时候**该用。

### 3.8 不要靠 `temperature` 制造多样性
四个现行模型上，`temperature`、`top_p`、`top_k` 设非默认值都直接返回 400。thinking 页的原文：`non-default temperature, top_p, or top_k values return a 400 error on every request, regardless of whether thinking is used`，它列出的模型包括这四个；Fable 5.2 的 overview 写的是 `Omit temperature, top_p, and top_k`。旁边还有两个坑：`top_p` 的默认值是 `0.99`，所以传 `top_p: 1` 会返回 400，同时传 `temperature` 和 `top_p`（哪怕都是默认值）也返回 400；从 API 版本 `2026-10-21` 起，这三个参数无论取什么值，只要出现就在所有模型上返回 400。用 prompt 求多样性——做设计时，Sonnet 5 页面的做法是让模型**先给选项再落地**：
```
Before building, propose 4 distinct visual directions tailored to this brief (each as: bg hex / accent hex / typeface, plus a one-line rationale). Ask the user to pick one, then implement only that direction.
```
（片段 4.25，逐字取自 Claude Sonnet 5 页面。）

### 3.9 不要要求模型复述或展示内部推理
"show your thinking"、"把思考过程写出来"这类指令在 **Fable 5.2、Opus 5.5、Sonnet 5.5** 上可能触发 `reasoning_extraction` 拒绝（Haiku 5.5 没有这个类别）。thinking 页原文：`On Claude Fable 5.2, Claude Fable 5.1, Claude Opus 5.5, Claude Opus 5, Claude Sonnet 5.5, and Claude Fable 5, a request that attempts to elicit the model's internal reasoning as part of the response text might be refused with stop_details.category: "reasoning_extraction"`。refusals 页列出了常见来源：让模型在回答前填写的 `<thinking>`、`<reasoning>` 或草稿区段；JSON 输出或工具入参里的 `reasoning`、`thinking`、`trace` 字段；模型的私人笔记或推理流水账；要求逐字或完整给出推理。这类措辞可能藏在 system prompt、skill 或 tool description 里；Claude Code 的错误参考页还列了 CLAUDE.md、子智能体 prompt、output style 和 MCP 工具描述，并给了排查办法：用 `claude --safe-mode` 开一个不加载这些定制的会话，发同一条 prompt。

**仍然允许的：** 简短解释答案、或概括做了哪些操作，以及给出结论背后的依据（refusals 页）。这个类别没有推荐的 fallback 模型，服务端 fallback 不会重试，拒绝会直接返回给你，所以要改 prompt，而不是重发。需要推理可见性就设 `display: "summarized"` 去读 `thinking` 块；需要在长时运行中把进度呈现给用户，设 `display: "updates"`；回合中途有必须原样交给用户的内容，用 send-to-user 工具（片段 4.23）。

### 3.10 不要写"不要思考"；想让它少思考，先降 effort
- **Fable 5.2**：`answer directly without deliberating` 这类句子在 `medium` 及以上不能让它停止思考。官方原话：`Remove these lines and lower the effort level`
- **Sonnet 5.5**：在 system prompt 里叫它少思考 `doesn't reliably reduce its thinking`；用 `between_tools` 时，叫它不要思考的指令还会让它更容易把内部 XML 标签写进可见输出，要删
- **Haiku 5.5**：叫它直接回答并没有让它停止思考；要少思考就降 effort，或在 `high` 及以下关掉 thinking
- **Opus 5.5**：迁移一个原本关闭 thinking 的 prompt 时，官方指令是 `check whether you still need the instruction, and remove the no-thinking rule either way`。降 effort 比 prompt 指令更可靠；首 token 时间仍然要紧时，官方给过一句 `Answer directly without deliberating.`，加了要测质量

四个模型上，想少思考的第一手段都是降 effort。最佳实践页另留了一段用 prompt 调低思考触发频率的示例（片段 4.15 的第二段），并写明它不适用于 Fable 5.2。Claude Code 的 model-config 页另有一句：想让 Claude 比当前档位想得更多或更少，可以直接在 prompt 或 CLAUDE.md 里说，模型会在 effort 设置之内响应。这和上面 Fable 5.2、Sonnet 5.5、Haiku 5.5 专页的说法不一致，本 skill 按各模型的专页。

### 3.11 继承来的 prompt 先查 prefill
不是你自己写的 prompt——旧模板、4.6 之前的代码库、任何你控制不了的工具生成的东西——结尾仍可能带一个半截的 assistant turn，这在现行模型上返回 400。**继承来的东西一律按本节重新过一遍**，先看结尾那一轮。Claude Code 的 `claude-api` skill 带一个 `prompt-audit` 子命令，官方原文描述是 `for auditing prompts and tool descriptions for patterns written for older models`（changelog 2.1.221，2026-08-04）；另有 `/doctor prompt-audit`（也可写 `/checkup prompt-audit`），用来审查 CLAUDE.md、skill、agent 和命令，官方原文是 `for prompting patterns written for older models`（changelog 2.1.283）。两者都只是第一遍粗筛，不能替代本节。

### 3.12 不要强制工具调用——Haiku 5.5 除外
`tool_choice` 设为 `any` 或 `tool` 类型，在 **Fable 5.2、Mythos 5.2、Opus 5.5、Sonnet 5.5** 上都返回 400 `invalid_request_error`；`auto` 和 `none` 不受影响。官方报错文案：

```
tool_choice: type "tool" and "any" are not supported for this model.
```

同样的校验也适用于 token counting 端点。要 schema 合规的 JSON，保持 `tool_choice: {"type": "auto"}` 并用 strict tool use 设 `strict: true`，或者把 schema 交给 structured outputs。用了客户管理加密密钥（CMEK）的组织里，Fable 5.2 和 Mythos 5.2 这两样都不可用：去掉 `strict` 和 `output_config.format`，在自己代码里校验。要让模型调工具而不是用文字回答，在 prompt 里写明什么时候该用这个工具。

**Haiku 5.5 接受强制 `tool_choice`**，但这时回复直接以工具调用开头、没有 `thinking` 块；想让它调用前先思考，用 `auto` 并在 prompt 里说明什么时候用。

### 3.13 迁移到 Fable 5.2 时要复查的旧指令
Fable 5.2 专页 "Review instructions written for earlier models" 列的，从 Fable 5.1 或 Opus 5.5 带过来的 prompt、skills、harness 里逐项查：

- **常驻的委派指令**：把活推给子智能体的句子。派得比任务需要的多，就删
- **harness 写进 prompt 的 token 预算**：预算大小会改变它花多少；去掉预算试一下
- **会改动历史轮次的 harness 代码**（仅 API）：见 `references/agents.md`
- **The "treat that answer as done" lines**（片段 4.34，把已答过的当定论的那两句）：会让它远不如原来那样指出自己早先的错
- **叫它少思考的句子**：删，改为降 effort（3.10）
- **硬性的文风规则**：可能让文字一顿一顿的；先删掉不再需要的，再加片段 4.40
- **积攒下来的规则、笔记和记忆**：删掉理由已经不成立的谨慎类条目，把绝对规则改写成带范围和原因的

场景 A 另有一张表：Claude Code 最佳实践的 "What you can stop doing"，见 `references/for-claude-code.md`。

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

**结构化手段**：XML 标签或 Markdown 标题分区。标签名一致且有描述性，内容有天然层级时就嵌套。

**长上下文（20k+ token）摆位**：长材料放**最前面**，问题和指令放**最后面**。官方测试中这一条最多可提升 30% 的回答质量，在复杂、多文档输入上尤其明显。多文档用 `<document>` 包裹，内含 `<source>` 和 `<document_content>`。

### 示例（few-shot）该不该给

| 目标 | 该不该给 | 依据 |
|---|---|---|
| 锁定输出**格式、语气、结构** | **给**，3–5 个，多样化，`<example>` 包裹 | 官方文档："Examples are one of the most reliable ways to steer Claude's output format, tone, and structure" |
| 展示**推理模式** | 给完整的例题——每个都写出问题、要用的方法、预期答案；模型会把这种思路带进自己的思考块。不要预留 `<thinking>` 区段让它填写（见 3.9） | 官方文档 |
| 教它**怎么用工具** | 先改接口，不靠示例：把工具、脚本和文件的参数设计得更有表达力 | Anthropic 2026-07 的 context engineering 文章（写的是 Opus 5、Fable 5 这一代）：`With our newest models, we've found that giving examples actually constrains them to a certain exploration space` |

---

## 5. 参考文件

按需读取，不必全部加载。全部从本文件直接链接。

| 文件 | 什么时候读 |
|---|---|
| **`references/for-claude-code.md`** | 场景 A。Claude Code 任务 prompt 的写法、必须给的验证手段、子智能体的模型与 effort 怎么定、该写进 CLAUDE.md 还是写进这次 prompt |
| **`references/for-chat-and-cowork.md`** | 场景 B。Claude 应用里 chat 与 Cowork 的 prompt 写法、材料摆位、交付物形态 |
| **`references/models.md`** | 选完模型后只读用到的模型那一节（主会话和子智能体用不同模型时各读各的）；要 effort 细节、价格或用量规则时再读文件后半对应的小节。内容：四个现行模型的已知倾向与对应处方——当前页面明确写了的、和只在前代页面写了的分开——以及 effort 取值、价格、用量上限和省用量做法 |
| **`references/snippets.md`** | 需要现成模块时读。52 条编号条目，其中 51 条是官方文档的**逐字原文**英文片段，每条标了出自哪个模型的页面 |
| **`references/agents.md`** | prompt 要驱动一个长时间自主运行的 agent 时读。上下文工程、跨窗口工作流、缓存命中、让模型自己编排 |

### 用到英文片段时

- 逐字片段（4.1 之外的全部）是官方页面上的英文原文；放在哪里按片段所在页面说的来——多数是 system prompt，个别写明放在末尾或紧跟某一段。产出是一段要粘贴进 Claude Code 或 Claude 应用的 prompt、没有 system prompt 可改时，放进 prompt 正文；每次都要的放 CLAUDE.md 或项目指令
- 官方写明可以调整的条目（例如 4.31、4.37、4.40、4.46）和要求逐字使用的条目（4.51），`references/snippets.md` 里逐条标了出来

---

## 6. 三种请求的处理差异

**写或改 prompt（场景 A/B）**：按第 0 节工作流走完，交付说明按第 0 节第 6 步的要求写。不要塞入用户没要的解释、免责声明或替代方案清单。

**审查已有 prompt**：按第 3 节逐条比对，指出**具体问题和对应修法**，不要直接给一份全新的 prompt 替代掉——除非用户要求重写。按严重度排序：会导致 API 报错的（3.1、3.2、3.8、3.12）排最前，官方写明会起反作用或被拒的（3.9、3.10、3.13）次之，其余最后。

**先问这份 prompt 是为哪个模型写的**，再查对应的官方迁移清单：

- **为 Opus 5 写的 → Opus 5.5**：照搬过来的 `effort` 值（默认档从 `high` 降到了 `medium`）；`thinking: {"type": "disabled"}`（现在是 400）；强制 `tool_choice`（现在是 400）；只渲染 `text` 块的客户端——它在工具调用之间会看起来没动静；自己拼 `messages` 的代码要保持历史只追加（thinking 块和对话绑定）；为旧模型写的例行指令（3.4 引的那次测量）
- **为 Fable 5.1 或 Opus 5.5 写的 → Fable 5.2**：3.13 那张清单；effort 重新扫描（从 Opus 5.5 过来时，不传 `effort` 的请求在 API 上会高一档）；自己拼 `messages` 的代码要过 prefix check
- **为 Sonnet 5 写的 → Sonnet 5.5**：关 thinking 改成 `between_tools`；强制 `tool_choice` 现在是 400；effort 档位重新校准过；历史同样要只追加。Anthropic 的 Sonnet 5.5 文章：prompt 里带着 `refusal steering, tool-call retry shims, or "do not be lazy,"` 这类变通写法的，先删掉并重跑评测，再调别的
- **为 Haiku 4.5 写的 → Haiku 5.5**：`budget_tokens` 换成 effort；prefill 现在是 400；非默认采样参数现在是 400；改动历史轮次会让 thinking 块失效；按 Haiku 4.5 不带思考的用量设的 `max_tokens` 可能把回复截断

以上每条只列了和写 prompt、审 prompt 最相关的几项，完整清单在各模型的 what's-new 页和迁移指南

**聊天途中临时检查（场景 C）**：走轻量流程。**不要小题大做**——不用完整跑一遍工作流，不用读参考文件。扫一眼第 3 节的硬规则，有问题就直接说"这里有个问题：X，改成 Y"，没问题就说没问题然后继续原来的话题。用户此刻要的是不被打断，不是一份审查报告。

---

## 7. 交付前自检

- [ ] 模型和 effort 已按第 2 节自己选定（没有把这个问题丢给用户，除非属于用户规则 1 的情况），交付说明里写了选的是哪一档、依据是哪条官方说法
- [ ] 没有 prefill、没有 `budget_tokens`、没有非默认采样参数
- [ ] 没有为防欠触发而堆的 `CRITICAL: You MUST` / `If in doubt, use [tool]`（**官方片段里的原文措辞不算**，例如 4.7 内含的 `you MUST read the file`）
- [ ] 目标是 Fable 5.2：没有叫它少思考的句子；审批规则点名了具体动作；没有硬性的文风上限；`low` 或 `medium` 做 agentic 工作时加了 4.37；过了 3.13 的清单
- [ ] 目标是 Opus 5.5：没有 `thinking: {"type": "disabled"}`、没有强制 `tool_choice`、没有「不要思考」类规则；effort 以档位形式写明
- [ ] 目标是 Sonnet 5.5：关前置思考用的是 `between_tools`；没有强制 `tool_choice`；`low` 下做编码、见过它不跑检查就报告完成的，加了 4.46
- [ ] 目标是 Haiku 5.5：给了搜索工具就给了当天日期（4.47）；`low` 或 `medium` 下做编码、见过它不跑检查就报告完成的，加了 4.46
- [ ] 目标是 Fable 5.2、Opus 5.5 或 Sonnet 5.5：没有要求它把推理写出来——没有"展示你的思考过程"，没有 `<thinking>` 或草稿区段，没有要它填的 `reasoning` 字段（见 3.9）。要它简短解释、或概括做了哪些操作，是可以的
- [ ] 主会话是 Fable 5.2 且任务会派子智能体：已经查过子智能体默认跑在哪个模型上，并按第 2 节写明了要用的模型
- [ ] 主会话是 Fable 5.2、又预判到要并行派大量子智能体做同一口径的批量工作：已按第 2 节「大批量处理」写明批量工作派给非 fork 的子智能体、用 `opus` 和对应的 effort（用户规则）
- [ ] **你自己写的**约束都是正面表述，且带了「为什么」（**官方片段里的原文措辞不算**——例如 4.25 开头的 `NEVER use generic AI-generated aesthetics…`）
- [ ] 每条指令都想过：**一个善意的人类会怎么误解它？**（Claude Code 团队 Cat Wu 2026-07 的公开发言，非官方文档；Fable 5.2 专页写明规则会被照字面执行，这个检查对现行模型仍然适用）
- [ ] 同一条指令没有在 system prompt 和 tool description 里重复出现（什么时候用这个工具的说明不算，见 3.7）
- [ ] 长上下文场景：材料在前、问题在后
- [ ] 输出长度在 prompt 里有明确要求，而不是交给 effort
- [ ] 有可判定的成功标准；场景 A 有模型自己能跑的验证手段
- [ ] 官方的检验（最佳实践页）：**把这份 prompt 给一个对任务所知极少的同事，他会照做吗？**

---

## 8. 故障到处方速查

片段编号指向 `references/snippets.md`。括号里的模型是这条处方出自哪个模型的页面；标「测得于」的是前代页面上的，先在自己的评测上复核；4.16、4.25 出自 Sonnet 5 页面，4.11 以及 4.6、4.15 的第一块是最佳实践页为更早的模型写的，各条目里有说明。

| 症状 | 处方 |
|---|---|
| 回复太泛 | 加具体约束、加成功标准、加 "go beyond the basics" 类修饰语 |
| 答非所问 | 补上真实目标和背景（说清"为什么"） |
| 格式不稳定 | 加 3–5 个 `<example>`；或用 Structured Outputs |
| 有开场白 | `Respond directly without preamble.` |
| 编造内容（一般场景） | 显式允许说不知道：`If the data is insufficient to draw conclusions, say so rather than speculating.`（出自 Anthropic 2025-11 的 prompt engineering 文章，不在文档页上） |
| 对没打开过的代码/文件下判断 | 片段 4.7 |
| 只给建议不动手 | 换直接动词（`Change this function` 优于 `Can you suggest some changes`）；或加片段 4.8 |
| 输出太长 | 在 prompt 里写明长度；片段 4.2、4.4（Opus 5 原文）。不要靠降 effort |
| 没做完就结束回合、列出剩余事项就停 | Fable 5.2：4.37（官方测得它去掉了大多数请示许可或提出可以继续的结尾，列出剩余事项就停的那种还在）。Opus 5.5：无人值守的循环用 4.31 加一份由模型自己更新的任务清单；Claude Code 里有人在旁时，用 `references/for-claude-code.md`「一次说清楚」里那条 CLAUDE.md 规则。Sonnet 5.5：先试更高的 effort，或 4.41。Haiku 5.5：4.49 |
| 要了不必要的审批、压着例行工作不做（Fable 5.2） | 清理积攒的谨慎类指令；点名哪些步骤不需要审批；把例外写进规则本身（`references/models.md`） |
| 只往现有结构上加、重复已有代码、旁边的问题只报告不修（Fable 5.2） | 4.38，并看 diff 而不只看总结 |
| 长 agentic 回合在客户端里看起来没动静（Fable 5.2、Opus 5.5、Sonnet 5.5） | 先查 `thinking.display`；再用 4.3 里对应模型的块；Fable 5.2 改不了 harness 时用 4.39 |
| 最终汇报读不懂、太密 | Fable 5.2：4.40，先删掉硬性文风规则。测得于 Fable 5：4.21。测得于 Fable 5.1：4.26 |
| 思考过多、延迟高 | **降 effort 档位**。Opus 5.5 页面另留了一句 prompt：`Answer directly without deliberating.`，加了要测质量。片段 4.15 的第二段出自最佳实践页，该页写明它不适用于 Fable 5.2 |
| 复杂问题推理太浅 | **提高 effort**；只有一轮需要时，场景 A 用 `ultrathink` |
| chat 回复迟迟不开口（Opus 5.5） | 考虑从 system prompt 里删掉叫它回答前仔细思考的句子；后续轮次用 4.34（用在 Fable 5.2 上先测） |
| 派的子智能体比任务需要的多 | Fable 5.2：先删委派指令，再改启动工具的 description（`references/models.md`）。Opus 5 原文：4.10 |
| 子智能体占了用量的大头 | 第 2 节「子智能体」：给它们指定更便宜的模型；让它们回报简短摘要；`/usage` 看占比 |
| `xhigh`/`max` 下做完还自己开复查轮（Sonnet 5.5） | 4.42；例行工作跑在 `high` 及以下 |
| 只想要想法，它却直接开始做（Sonnet 5.5） | 4.43 |
| 要 JSON 的推算任务答错（Sonnet 5.5） | 4.44，或 `xhigh` |
| 凭训练知识作答、该搜不搜 | Sonnet 5.5：4.45。Haiku 5.5：4.47。测得于 Fable 5.1：4.27 |
| 没跑检查就报告代码改动完成（Sonnet 5.5、Haiku 5.5） | 4.46 |
| 关着 thinking 要 JSON 时跳过工具调用（Haiku 5.5） | 这类请求改用 adaptive thinking；必须关 thinking 时 4.48 |
| chatbot 被用户磨得不守 system prompt（Haiku 5.5） | 4.50 |
| 用户中途发的消息被当成注入忽略（Sonnet 5.5、Haiku 5.5） | 用户文字不放进 `tool_result` 块，作为 user 回合送达（`references/models.md`） |
| 跨多个已连接应用时漏掉任务没点名的信息（Opus 5.5） | 4.32 |
| agent 花的时间比应该的长（Opus 5.5） | 4.33——给出时间预算，并在每条消息末尾报告已用时间 |
| 照着用户粘贴进来的文本里的指令执行（Opus 5.5） | 4.35 |
| 反复叮嘱某规则仍不生效 | 文件太长、规则被淹没 → 精简；不能有例外的规则改用确定性手段（hook、deny 规则、schema）。加强调词没用（3.3） |
| 代码审查漏报 | 4.16，把过滤移到下游 |
| 过度工程 | 4.6 |
| 做了没被要求的事、任务范围被擅自放大 | 测得于 Fable 5：4.24。Opus 5 原文：4.5。Sonnet 5.5：4.41 的第二段 |
| 长任务编造进度或谎报完成 | 测得于 Fable 5：4.17 |
| 快到上下文上限时自行收工 | 测得于 Fable 5：4.19；首选是别让模型看见剩余预算数字 |
| 一轮只发一个工具调用、小改动整个文件重写、摘要照搬原文不标引用、`xhigh`/`max` 下长交付物等很久 | 测得于 Fable 5.1：依次是 4.9 的提示句、4.28、4.30、4.29。Fable 5.2 页面没有重述这些倾向 |
| 良性请求被拒、返回 `stop_reason: "refusal"` | 看 `stop_details.category`；`reasoning_extraction` 按 3.9 改 prompt；其余见 `references/models.md` 各模型的分类器一行 |
| 改动可能不可逆 / 碰共享环境 | 4.11 |
| 为了让测试通过而写死值、绕路 | 4.12 |
| 报告全是短 bullet、读不成文章 | 4.13 |
| 需要并行读多个文件 / 工具调用可以并发 | 4.9 |
| 数学公式渲染成 LaTeX 但环境不支持 | 4.14 |
| 请求本身没说清"为了什么" | 测得于 Fable 5：4.20 |
| agent 需要跨会话积累经验 | 测得于 Fable 5：4.22 |
| 长时异步 agent 要把内容原样送到用户面前 | 测得于 Fable 5：4.23 |
| 前端一股 AI 味 | 4.25；Opus 5.5 上点名具体要避开的样式——4.36 |
| 应用可能被问到儿童用药剂量 | 4.51，取对应模型的那一块，逐字使用 |
| 编码任务结束后留下一堆临时文件 | 4.52 |

### 长度控制

长度写在 prompt 里，不交给 effort。官方依据：Fable 5.2 专页 `Effort barely changes how long the model's replies are. To change the length and tone of a reply, use your prompt.`；Opus 5 页面写明降 effort 不能可靠缩短可见输出。成本页的建议是 `Ask for the answer you will read, not the one that looks thorough`。

现成措辞：片段 4.2（通用简洁指令和末尾的 `<tone_preference>` 短提醒）、片段 4.4（写成文件的交付物），都是 Opus 5 页面的原文。给 Fable 5.2 写具体的文风上限时留意 3.13：硬性的句长限制可能让它的文字一顿一顿的。

---

## 9. 元原则（本文件没覆盖到的情况下靠这些判断）

1. **最小信息集**：能完整界定期望行为的最少 token。
2. **高度要合适**：既不写死脆弱的硬逻辑，也不给模糊到没有信号的空话。
3. **多给上下文，少给指令。**
4. **描述你想要的样子，而不是禁止你不想要的样子。**
5. **有工具时也看工具设计**：命名、参数、返回值和错误信息的具体做法见 `references/agents.md`。
6. **能确定性执行的，不要用 prompt**：该用 hook、schema、类型约束的地方别写"请务必"。
7. **模型变强，脚手架变负担**：定期回头删掉当初为弱模型加的东西。
8. **改完要测**：拿真实任务跑，不要凭感觉判断改好了。
