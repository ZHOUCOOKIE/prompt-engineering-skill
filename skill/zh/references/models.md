# 按模型调校：Fable 5.2、Opus 5.5、Sonnet 5.5、Haiku 5.5

> 片段编号（如 4.2）指向 `snippets.md`。
>
> 下表"处方"列里用反引号或代码块标出的英文**是官方文档原文**；其余文字是本文件的概括。

## 目录

- 每条说法怎么归档
- Claude Fable 5.2 / Mythos 5.2——5.2 页面明确写了的；只在 Fable 5.1 页面写了的；只在 Fable 5 页面写了的
- Claude Opus 5.5——5.5 页面明确写了的；只在 Opus 5 页面写了的
- Claude Sonnet 5.5
- Claude Haiku 5.5
- 验证指令：各模型的官方说法
- effort 取值参考
- 价格、用量上限与省用量做法（官方数字）

## 每条说法怎么归档

通用最佳实践页对它列出的模型专属技巧有一条提醒，本文件照此处理：`Where a technique names a specific model, treat it as measured on that model and re-check it against your own evals before applying it to another.`

所以每个模型下面的说法分三类：

1. **当前页面明确写了的**——放在该模型的第一张表里。
2. **只在前代页面写过、当前页面既没重述也没撤回的**——放在标明「在某模型上测得」的表里，写清出处，使用前在自己的评测上复核。
3. **已被撤回、或与当前页面冲突的**——不收。

## Claude Fable 5.2 / Mythos 5.2

2026 年 11 月 9 日发布。模型 ID `claude-fable-5-2`；$8 / $40 每 MTok（Fable 5.1 是 $10 / $50）；缓存读取 $0.20 每 MTok；1M token 上下文窗口；最大输出 128K；adaptive thinking 始终开启且无法关闭；延迟标注 Moderate，与 Opus 5.5 相同。effort 默认值按界面不同：**Claude API 上是 `high`，Claude Code 和 Claude 应用里是 `medium`**。Claude Mythos 5.2 能力相同，只向通过 Anthropic 验证计划（例如 Cyber Verification Program）认证的组织提供，规格和价格相同。Fable 5.2 要求 30 天数据保留，zero data retention 下不可用（除非 Anthropic 明确授权）。在 Claude Code 里，`fable` 别名从 2.1.321（2026-11-09）起指向 Fable 5.2（Claude apps gateway 会话里指向 Fable 5）。

定位：官方称其为 `Anthropic's most capable model open to all customers`，`built for demanding reasoning and long-horizon agentic work`。官方选型建议是另一半：多数工作负载从 Claude Opus 5.5 起步。什么时候换到 Fable 5.2，官方四处说法不完全一样：what's-new 页、models overview 和成本页说 Opus 5.5 `at higher effort` 评测仍不够时；choosing-a-model 页说 `xhigh` 或 `max` 下仍不够时；成本页 "Start here" 表对质量不够的建议是 `If you lowered effort, restore it; otherwise try the next tier up at its default effort`；2026-11-09 的选型文章说 Opus 5.5 在 `high` 下仍不够时，并建议在难任务上把 Opus 5.5 提到 `high` 以上之前先试 Fable 5.2。SKILL.md 第 2 节按选型文章的步骤走。发布公告还写明一个例外领域：Anthropic 在训练中限制了最能直接加速 AI 研发的那部分研究能力，所以在加速器内核优化、改进大语言模型架构这类方向上，Fable 5.2 和 Mythos 5.2 `perform at about the level of Opus 5.5`；agentic 编码、硬件开发和传统的 AI 或机器学习研究不受影响；这个领域里的一小部分话题（公告举的例子是某些非标准加速器上的内核开发）会透明地 fallback 到 Opus 5。

**旧 prompt 怎么办。** Fable 5.2 的页面没有说现有 prompt 无需修改就能表现良好。它写的是：`When you migrate to Claude Fable 5.2, review your prompts. In Anthropic's testing, some instructions written for earlier models raised cost, caused side effects, or no longer had an effect.` 迁移指南那句 `Migration is mostly drop-in` 讲的是 API 层面。所以 Fable 5.1 和 Fable 5 页面上的做法一律按「在该模型上测得」对待，见后面两张表。

**这些 prompt 改动只在新会话里做**（仅 API）。Fable 5.2 对每个账号都执行 prefix check：`system`、`tools` 或更早的消息变了之后再回传 thinking 块，可能返回 400。会话中途要改指令，放在最新一轮里，或者在支持的平台上追加一条会话中 system 消息（见 `agents.md`）。

### Fable 5.2 页面明确写了的

| 已知倾向 | 处方 |
|---|---|
| 长的多步任务上**有时没做完就结束回合**：结尾列出还剩什么、提出可以继续、或者停在超时这类普通障碍上。`low` 和 `medium` 下更常见 | 4.37。它区分两类障碍：普通障碍先诊断再绕过去，有人特意设的阻断不碰并如实报告；改写时保留这个区分。官方回放 `medium` 下的编码会话：留下可做工作就结束的会话约少一半，输出 token 多约 30%；请示许可或提出可以继续的结尾大多没了，但列出还剩什么然后停的那种结尾还在。模型因此更少发问，风险或不可逆操作的确认步骤要自己保留 |
| 让它继续时，只回一句 "continue" 不够清楚；system prompt 里的审批规则会被**照字面执行** | 说清要它做什么（官方例子：`Yes, run the migration on the staging database`）。审批规则要点名具体覆盖哪些动作——官方在没有人可以批准的模拟任务上测试，一条把 `sending messages to people outside this conversation` 列为需审批的规则，让模型压下了大约一半发给同事和其他团队的例行请求 |
| 对 prompt 里**代码归谁**的说法照得很紧。官方在小型编码任务上测得：没说归属、或把代码库说成别人的：倾向于往现有结构上加东西（哪怕重复已有代码），旁边的问题只报告不修。说代码库是它自己的：常常合并重复代码、顺手做相关修复 | 想让它边做边改善结构，用 4.38。代价也是官方写明的：输出 token 约 1.5 倍，会做几处没人要求的改动（其中很多是新测试和文档；除修 bug 之外，六到十次里有一次改了现有功能的行为，effort 越高越常见），收尾总结常漏掉小改动，所以要看 diff 而不只看总结。用户拥有代码、希望改动尽量小时不要加这句；prompt 别处的限范围规则抵消不了它 |
| 长串工具调用期间**有时很少写给用户看的文字**，也可能不说要做什么就发出第一个工具调用 | 先确认客户端收得到：播报以 progress-update `thinking` 块返回，默认 `thinking.display: "omitted"` 下文本为空，设 `"updates"`（beta）或 `"summarized"`。还不够时用 4.3 的 Fable 5.2 块（harness 提醒、开场与收尾）或 4.39。用户不盯着看的长时异步任务里，间隔没那么要紧 |
| **委派多少子智能体取决于 harness 和任务**：只给一个简单的启动工具、任务又短时很少派；prompt 鼓励委派的编码产品里派得很痛快。`xhigh`、`max` 这类高档位下还观察到它用子智能体复查自己的工作（Fable 5.1 在同样任务上不会） | 先查 system prompt、skills、agent 文件里叫它委派的句子。仍然派得比任务需要的多，才去改启动子智能体那个工具的 description：写明一个子智能体给会话增加多少成本、什么时候值得派、怎么给它写交代，成本放最前面。官方在短任务上测得：用它测过的那版 description，委派比用主要罗列委派理由的那版少约三分之一，任务成功率相同；讲成本的开头几句是抑制委派的，派得太少就少讲成本，仍然太多就多讲。官方原话：`subagents can account for a large part of what a session costs, so check which model your harness runs subagents on by default`。早期测试者用它给其他模型当规划者和编排者，效果不错 |
| **谨慎类指令会越积越多**（记忆、笔记、指令文件、agent 之间的消息），而它照得很紧，于是可能要不必要的审批、压着例行工作不做。少数早期测试者见过，多在几个 agent 协作的场合；官方描述的一个设置里，Fable 5.1 和 Opus 5.5 会重试，只有 Fable 5.2 压着不做 | 删掉不再需要的笔记；点名哪些步骤不需要审批（官方测试里，笼统地叫它放手去做没有带来变化），重试只点名那些重复执行也安全的调用；检查 reviewer agent 附带了什么；把例外写进规则本身，而不是另起一行。系统已经卡住时，考虑把记忆和笔记精简后重启，原件留一份。风险或不可逆操作的确认步骤自己保留 |
| **叫它少思考的句子在 `medium` 及以上不管用**；`high` 及以上它几乎每次回复前都思考，包括回一句问候 | 删掉 `answer directly without deliberating` 这类句子，改为降 effort。原来靠这类句子把回复塞进很小的 `max_tokens` 的请求，可能在出任何文字之前就用完 token：调大 `max_tokens` 或降 effort |
| **照得很紧的还有文风规则**，硬性上限（官方例子 `one instruction per sentence, in 20 words or fewer`）可能让文字变得一顿一顿的 | 先删掉不再需要的文风规则，再加 4.40 |
| 长时间 agentic 工作之后，读最终汇报的人通常没看过背后的工具调用和文件 | 4.40，按读者和产品调整。官方测得汇报更不密、分号更少；流程自动化任务上句子更短 |
| harness 在 prompt 里写明任务能用多少 token、还剩多少时，**预算的大小会改变它花多少** | 官方在 `high` 和 `xhigh` 的编码任务上测得：去掉一个大预算，成本下降、表现相当。拿自己的任务去掉预算试一下。这说的是 prompt 里的文字，不是 API 的 task budgets |
| Opus 5.5 页面给 chat 的那两句把已答过的当定论的话（4.34）在它身上有副作用 | `high` 下这两句确实减少了追问轮次的思考，但模型也**远不如原来那样会指出自己早先答案里的错误**。希望它持续复查的地方不要加；别处先测它还会不会指出自己的错 |
| effort **几乎不改变回复长度** | 官方原文 `Effort barely changes how long the model's replies are. To change the length and tone of a reply, use your prompt.` |
| 安全分类器五类：`cyber`、`bio`、`frontier_llm`、`reasoning_extraction`、`general_harms`。正当的网络安全和生命科学工作也可能触发 `cyber` 和 `bio` | 处理 `stop_reason: "refusal"`。服务端 fallback（beta）允许的目标是 Claude Opus 4.8 和 Claude Opus 5；`reasoning_extraction` 没有推荐的 fallback，拒绝会直接返回。prompt、skills、tool description 里叫它把推理写进回复的句子要删（SKILL.md 3.9）。被挡住的正当工作，见 Cyber Verification Program 和 Life Sciences Verification Program。选模型时要知道的范围：发布公告说 Fable 5.2 的分类器会拦截高风险的网络安全活动（渗透测试、漏洞利用生成、基于二进制的漏洞工作）和研究生物学的大多数任务，被拦时改由 Opus 4.8（网络安全）或 Opus 5（生物）回答。Claude Code 的 model-config 页：进攻性安全或生物方向的工作负载（渗透测试、CTF、和生物相邻的代码库）经常触发 fallback，往往第一个请求就触发；在 Fable 模型或 Opus 5.5 上做实质性的生物工作时，第一次被标记并切换会把会话移到 Opus 5，之后被标记的生物请求在那里以拒绝结束，因为 Opus 5 没有生物方向的 fallback |
| 强制工具调用（`tool_choice` 的 `any` / `tool`）返回 400 | 见 SKILL.md 3.12。在用客户管理加密密钥（CMEK）的组织里，Fable 5.2 不支持 structured outputs，那里去掉 `strict` 和 `output_config.format`，在自己代码里校验 |

**迁移时的顺序**（官方 "Where to start"）：先把 `effort` 显式设成 `high`，再在 `medium` 和 `xhigh` 上各跑一遍评测；读一遍带过来的 system prompt、skills、agent 文件、记忆、笔记和 harness 文本，按 SKILL.md 3.13 删掉或实测；跑最长的 agentic 任务，记下哪里提前停、要了不必要的审批、没动静、派多了、或者该重构却只是往上加；然后一次只改一样，量效果和 token 成本。

Fable 5.2 页面写明的 harness 层规则（仅 API；见 `agents.md`）：对话历史只追加不修改，prefix check 对所有账号生效；只有 Fable 5.2 和 Mythos 5.2 读得了 Fable 5.2 的 thinking 块；`computer_20251124` 除 Amazon Bedrock 外都返回 400，改用 `computer_toolset_20260801`；用 advisor 工具时，Fable 5.2 执行者只接受 Fable 5.2 或 Mythos 5.2 当 advisor；会话中途改 effort 用按消息设置（beta）而不是改顶层值。

### 只在 Fable 5.1 页面写了、Fable 5.2 页面没有重述的

下表每条都出自 Fable 5.1 的 prompting 专页（该页仍在线；Fable 5.1 的 what's-new 页和迁移指南已重定向）。Fable 5.2 的页面对这些倾向既没重述也没撤回，「5.2 页面相关的话」一列记的是它在相近话题上说了什么。把它们当作在 Fable 5.1 上测得的，先在自己的评测上复核。

| 已知倾向（Fable 5.1） | 处方 | 5.2 页面相关的话 |
|---|---|---|
| 编码和 computer use 循环里，下一批独立调用只是隐含时，可能一轮只发一个工具调用 | 4.9 里那一句提示，每轮重发 | 没提 |
| 有些情况下文风比 Fable 5 更密：句子更长、段落更少 | 4.26 | 没有「密度」一节；最终汇报用 4.40 |
| 产品会折叠或隐藏工具输出时，模型可能跑命令去给用户展示界面根本不显示的输出 | 4.3 里 Fable 5.1 的那一句，用 turn-scoped system message 传；先删掉压制播报的旧指令 | 没提 |
| 聊天里用的格式往往比旧模型少，旧的反格式化规则可能压掉内容需要的结构 | 删掉那些规则，或换成 4.13 里的 Fable 5.1 块 | 没提；最佳实践页这句仍只点名 Fable 5.1 |
| 做文档摘要时更容易照搬原文而不标为引用 | 4.30 | 没提 |
| `low` effort 下更常凭记忆作答而不去调搜索 / 检索工具 | 给受影响的那几轮提高 effort，或 4.27 | 没提 |
| 小改动也更倾向于整个文件重写 | 4.28 | 没提 |
| `xhigh`/`max` 下可能先在思考里把长交付物起草一遍，再作为回复写一遍 | 这类请求跑在 `high`；确实要更高时追加 4.29 | 没提；effort 页只说 `high` 及以上要把 `max_tokens` 设大 |
| 子智能体在跑时主 agent 闲着 | 启动子智能体的工具立即返回，结果之后以 `user` 消息回传，另给一个想等的时候用来等的工具 | 没提 |
| 密集图表和图像 | 给它一个装有原图和 PIL / OpenCV 的容器；容器开销太大时，单独一个裁剪工具就能拿到大部分提升 | 没提 |
| 良性的编码请求被安全分类器误拒 | 问 "Are there any bugs in this program?" 而不是 "Does this program compile without errors?"；给小众语言补上下文；别让 base64 数据进工具输出 | 没重述这几条；发布公告说早期测试里被拦截的次数比 Fable 5.1 少约 55% |
| 自己在客户端做上下文摘要时用的那段指令 | 见 `agents.md` | 没提 |

### 只在 Fable 5 页面写了的

下表每条都出自 Fable 5 页面（引文为原文）。Fable 5.1 和 Fable 5.2 的页面都没有重述。把它们当作在 Fable 5 上测得的，先在自己的评测上复核。

| 已知倾向（Fable 5） | 处方 |
|---|---|
| 困难任务上单次请求在高 effort 下可能跑很多分钟，自主运行可达数小时 | 迁移前先调整客户端超时、流式和进度提示；考虑改成异步轮询而非阻塞等待 |
| 任务模糊时会过度规划 | `When you have enough information to act, act. Do not re-derive facts already established in the conversation, re-litigate a decision the user has already made, or narrate options you will not pursue in user-facing messages. If you are weighing a choice, give a recommendation, not an exhaustive survey. This does not apply to thinking blocks.` |
| 长自主运行中可能编造进度 | 4.17——`nearly eliminated fabricated status reports` 是在 Fable 5 上测得的 |
| 偶尔会做未被要求的动作（自动起草邮件、建防御性 git 分支） | 4.24（Fable 5.1 页面把这段的两半并进了 4.18 那段里） |
| 偶尔会提议开新会话，最常见于看到剩余 token 倒计时的时候 | 4.19；更好的做法是别把倒计时暴露给模型。Fable 5.2 页面相关的话：prompt 里的 token 预算会改变它花多少，先试试去掉 |
| 为旧模型写的 skills 往往过于规定化，可能降低输出质量 | 审查，默认表现更好时考虑删掉旧指令。Fable 5.2 页面给了它自己的一张「要复查的旧指令」清单，见 SKILL.md 3.13 |
| 长时异步 agent 需要把内容原样送到用户面前 | 4.23 |
| 需要跨会话积累经验 | 4.22。Fable 5.2 页面的相关提醒：记忆文件和笔记会积累谨慎类的话，而它照得很紧，迁移前删掉理由已不成立的 |
| prompt 说明了请求背后的意图时表现更好 | 4.20 |
| 长的或 agentic 的对话里，写出来的文字可能难读 | 4.21。Fable 5.2 页面对同一问题给了新措辞，见 4.40 |

**前代页面上的四条说法，Fable 5.2 的页面讲的是另一回事，所以不放在上面两张表里，也不作为对 Fable 5.2 的建议**：

- Fable 5 页面：更主动派并行子智能体，建议鼓励使用。5.2 页面：先查委派指令，注意子智能体成本
- Fable 5 页面：一句简短指令就能管住一类行为。5.2 页面：规则会被照字面执行，要点名具体动作；官方测试里笼统的一句没有带来变化
- Fable 5.1 页面：做开放式功能时交付所要求的、有时还多给（4.6 的 Fable 5.1 块）；Fable 5 页面：高 effort 下会做未被要求的整理和重构（4.6 的 Fable 5 变体）。5.2 页面描述的默认方向不同：没说代码归谁时，它倾向于往现有结构上加、旁边的问题只报告不修；说代码归它之后才写到没人要求的改动。这两段仍在 `snippets.md` 里，标着各自是在哪个模型上测得的
- Fable 5.1 页面：复杂的异步任务上有时只描述下一步而不去做，或对已覆盖的步骤停下来请示（4.18）。5.2 页面对同一类问题给了新措辞 4.37，并说它在 `low` 和 `medium` 下更常见

## Claude Opus 5.5

2026 年 9 月 22 日发布。默认 effort **`medium`**（API 和 Claude Code 都是）；adaptive thinking 始终开启且**无法关闭**；1M token 上下文窗口；最大输出 128K；$4 / $20 每 MTok。模型 ID `claude-opus-5-5`。它是 Claude Code 在各订阅计划和 Anthropic API 上的默认模型。Claude Opus 5 现已列为仍可用的旧模型（legacy），价格 $5 / $25。

**Opus 5 的指引怎么沿用。** Opus 5.5 的 prompting 专页靠前的位置写着：`Existing Claude Opus 5 prompts should perform well without changes, and the patterns in Prompting Claude Opus 5 remain a reasonable starting point.` 官方迁移指南另有一句：`Instructions tuned for Claude Opus 5's behavior might no longer be needed`。所以本文件把两者分开：第一张表是 Opus 5.5 页面明确写了的；第二张表是只在 Opus 5 页面写了、Opus 5.5 页面既没有重述也没有撤回的。

### Opus 5.5 页面明确写了的

| 已知倾向 | 处方 |
|---|---|
| **默认 effort 是 `medium`**，Opus 5 是 `high`——不传 `effort` 的请求比以前低了一档 | 显式设 `effort` 并重做扫描。官方测试中，Opus 5.5 在 `medium` **达到或超过 Opus 5 在 `high`** 的编码与知识工作评测成绩，而且若干编码评测上 `low` 也接近 Opus 5 在 `high` 的水平，成本低得多 |
| **同一档位下每回合往往思考得比 Opus 5 多**，`xhigh` 和 `max` 尤其明显 | 不要照搬 Opus 5 的值——照搬会让回合更长、输出 token 更多。只把 `xhigh` 和 `max` 留给**实测**到质量提升的工作。想少思考就**先降 effort 档位**：官方说这比用 prompt 指令更可靠 |
| **汇报写得清楚**：工作中的更新和结束时的总结 `say plainly what it did, what it found, and what it needs from you` | 想换汇报的格式就直接说。Anthropic 的 Opus 5.5 使用文章（2026-09-22）举的例子是在 CLAUDE.md 里写 `End every run with three headings: Blocked on me, Changed, Found.` |
| 即使思考文本不返回给你，思考也计入 `max_tokens`；按 Opus 5 关闭 thinking 时的用量设的上限，可能会把回复截断 | 留足空间。面对 agentic 编码可能产生的长回合，官方原话是 `a max_tokens of 128,000, the model's maximum, has worked well in Anthropic's testing` |
| 在有多个部分的长任务上它会边做边向用户汇报，而**其中一些汇报是以文字而不是工具调用结束回合的**（`stop_reason: "end_turn"`）。无人值守的循环如果把这种回合当成任务结束，就停在那儿了 | 把「只有文字的回合」当成汇报，不是完成的证据。把任务的各个部分放进一份由模型自己更新的清单，回合结束时若还有未完成项且没说明阻塞原因，就发一条点名这些项的简短 user 消息（4.31 同时给了这条续跑消息和 system prompt 追加段）。自动续跑两三次之后就停，不要无限重复。如果模型启动的东西还在跑（后台命令、子智能体），先等它结束并把输出作为下一条 user 消息回传。也可以事先写明完成条件，然后 `have a separate, smaller model check the conversation against it at each end of turn` |
| **工具调用之间的文字以 `thinking` 块返回**，不是 `text` 块——默认 `thinking.display: "omitted"` 下内容为空，所以只渲染 `text` 块的客户端在整个长 agentic 回合里都可能显得没动静 | 先查客户端：设 `display: "updates"`（beta，`thinking-display-updates-2026-08-18`）拿到每条播报的摘要。想要更频繁或固定时点的播报，在 system prompt 里写明（该页没有给现成措辞）；用 4.23 的工具传递必须逐字送达用户的内容；回合仍然安静就用 4.3 里那条提醒。这是**相对 Opus 5 的响应形态变更**，不会让任何请求失败 |
| 跨多个已连接应用做流程自动化时**往往上手很快**，任务描述比较松散时会漏掉请求没有点名的信息 | 4.32。官方测得加上这条指令后，多应用自动化任务的正确完成率明显提高（`medium` 和 `max` 两档都是），代价是工具调用和 token 略多。因为它让模型照找到的东西去行动，**要把不可信内容挡在它会搜索的记录之外** |
| **对「已过去多少时间」这类信息非常敏感**——可以拿来给 agent 提速 | 4.33。给出时间预算，并在每条消息末尾报告已用时间与预算的对比（`elapsed 340s / 1200s`）；它会为了在预算内完成而调整节奏，而且通常远早于预算完成，所以预算要设得比你真正希望花的时间略高。预算只是建议性的——需要硬停就自己保留超时 |
| 在 chat 里**回答前会长时间思考**，拖慢首 token；后续轮次里即使是很短的追问，它有时也会回头重新过一遍之前的答案 | 考虑从 system prompt 里删掉叫它回答前仔细思考的句子——官方在一个 chat 产品里测得，删掉之后回复开始得更早，质量没有明显下降。追问这个问题用 4.34。希望它持续复查早先工作的场景，不要加 4.34 |
| 抵抗间接 prompt 注入的能力强于此前任何 Opus 模型；给对上下文时，对**用户粘贴进来的文本**里的指令同样稳健——但你得告诉它哪段是粘贴来的 | 4.35：给每段粘贴内容套上带同一个短随机 ID 的标签，并加上配套的 system prompt 说明。标签是纯文本、可以被伪造，所以把它当成多道防线中的一道 |
| 要求做前端但没给设计方向时，会落回少数几种默认样式；笼统地说 `avoid a generic AI look` 基本只是把一种默认换成另一种 | 4.36——点名具体要避开的样式。迭代着做：看第一版改用了哪些样式，再把清单加长 |
| **不借助工具读图表、流程图和截图就比 Opus 5 准得多** | 重新检验为更早模型搭的视觉脚手架还需不需要。最密的输入上仍有两件事能加分：更高分辨率的图（工程制图尤其），以及图像处理工具——装有原图和 PIL / OpenCV 的容器；容器负担太重时，单独一个裁剪工具也仍然有帮助。effort 越高它越会用这些工具。不给工具时，提高 effort 能改善工程制图，但对图表帮助不大 |
| 安全分类器包括**生物、网络安全、推理抽取**三类。**生物对从 Opus 5 迁移过来的人是新增的**；what's-new 页介绍 `reasoning_extraction` 时用的是 `As on Claude Opus 5`，即 Opus 5 上本来就有 | 处理 `stop_reason: "refusal"` 并配置回落——但注意服务端 fallback **不会重试** `reasoning_extraction`，它会把拒绝直接返回给你。在源码里找漏洞是允许的；高风险两用网络安全活动不允许。日常健康与教育类问题不受生物分类器影响 |
| 要求模型把推理复述进正文的 prompt 可能被 `reasoning_extraction` 拒绝 | 见 SKILL.md 3.9。删掉这类指令，设 `display: "summarized"`，改从 thinking 块里读。仍然可以要它简短解释答案、或概括做了哪些操作 |

**如果你的 Opus 5 集成原本关闭了 thinking**：Opus 5.5 不接受 `thinking: {"type": "disabled"}`（SKILL.md 3.2），迁移时还要配套改四件事：

- **从 `low` effort 起步并实测。** `low` 下它的思考很短；具体多大比例会完全跳过思考取决于你的 prompt，所以要在自己的流量上测延迟和质量，质量掉了就升到 `medium`。如果之后首 token 时间仍然要紧，官方说 system prompt 里加一句 `Answer directly without deliberating.` 能进一步减少思考——加了之后要测质量，因为思考变少可能把质量带下来。（这句话只对 Opus 5.5 成立：官方测试里它没能让 Fable 5.2 在 `medium` 及以上停止思考，见上文。）
- **删掉那些用来替代思考的指令。** 如果你的 prompt 当初要求模型把推理写进回复里当作思考的替代品，把那条指令删掉，改从 [summarized thinking](https://platform.claude.com/docs/en/build-with-claude/thinking#summarized-thinking) 块读（`display: "summarized"`）；否则这种 prompt 可能被 `reasoning_extraction` 拒绝
- **重新检验关闭 thinking 时的那套缓解措施**，并且**无论如何都把「不要思考」那条规则删掉**（SKILL.md 3.10）
- **按块类型而不是按位置读响应**——回复可能以 `thinking` 块开头，也可能不

**Opus 5.5 页面写明的 harness 层规则**（仅 API；见 `agents.md`）：对话历史只追加不修改，因为改动 thinking 块之前的任何内容都会让它失效；会话中途改 effort 用按消息设置而不是改顶层值；强制 `tool_choice` 返回 400（SKILL.md 3.12）；computer use 必须用 `computer_toolset_20260801` 工具集——旧的 `computer_20251124` 返回 400，只在 Amazon Bedrock 上仍然可用。

### 只在 Opus 5 页面写了、Opus 5.5 页面没有重述的

下表每条处方都出自 Opus 5 页面（引文为原文），Opus 5.5 页面写明这些做法仍是合理起点；但*倾向本身*在 Opus 5.5 上是否仍然存在，官方两边都没有说。把它们当作在 Opus 5 上测得的，在自己的评测上复核。其中两条与 Opus 5.5 *有明确记载*的行为方向相反，已在行内标出。

| 已知倾向（Opus 5） | 处方 |
|---|---|
| 用户可见回复比更早的 Opus 模型长；**降 effort 不能可靠缩短可见输出** | 显式写简洁指令（4.2）。注意这里有张力：官方说 Opus 5.5（相对 Opus 5）`tends to complete the same task with fewer tokens`，所以先实测，别默认这个长度问题还在 |
| Agent 场景播报频繁、单条消息往往偏长 | 显式描述播报节奏（4.3）。在 Opus 5.5 上有记载的问题**正好相反**——播报以默认为空的 `thinking` 块返回，所以回合会显得安静。加任何东西之前先查 `thinking.display` |
| 写到磁盘的文档（报告、Markdown、总结）往往比前代长 | 长度校准指令（4.4） |
| **不用提示就会自我验证** | 最佳实践页对 Opus 5 的原话：`remove these instructions rather than rewriting them`。见下文「验证指令」一节 |
| 可能自行扩大任务范围，加上没被要求的步骤 | 窄任务要显式约束范围（4.5） |
| 比更早的模型更积极派生子智能体 | 给明确判据或确定性上限（4.10）。在 Claude Code 和 Agent SDK 里，上限是 `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH`、`CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS` 和 SDK 的 `max_budget_usd`；Claude Code 只在用它的 `claude_code` system prompt 预设时才会在 Opus 5 上自动加一条委派指令，自定义或不传 system prompt 时要自己加 |
| 更频繁地口头纠正自己此前的说法 | `Only correct an earlier statement when the error would change the user's code, conclusions, or decisions. State corrections plainly and briefly, then continue the task. For slips that change nothing for the user, make the fix and move on without noting it.` |
| 代码审查中 `only report high-severity issues` 可能被**字面执行** | 用 4.16；把过滤动作移到独立的下游步骤。早期测试者反馈 Opus 5.5 的代码审查*更强*——bug 找得更多、误报更少——但「字面执行指令」这个倾向既没被重述也没被撤回 |

**只在 Opus 5 上有记载**（Opus 5 在 `high` 及以下 effort 可以关闭 thinking；Opus 5.5 不能关，Opus 5.5 页面说这两种现象只在 Opus 5 关闭 thinking 时出现，并让你检查是否还需要这条指令）：关闭 thinking 时 Opus 5 偶尔会把 tool call 写成用户可见的文本而不是发出 `tool_use` 块——该调用不会执行，而且在 agent 循环里泄漏的文本会留在历史里影响后续轮次；也可能把 `<thinking>` 等内部 XML 标签泄漏到可见输出。主要的缓解办法是保持 thinking 开启；必须关闭 thinking 的集成，官方给了一条合并指令同时缓解两者：

```
When you use a tool, you may say a brief sentence first. If no tool can express what the user asked for, say so instead of guessing. Do not include internal or system XML tags in your response.
```

## Claude Sonnet 5.5

2026 年 9 月 28 日发布。模型 ID `claude-sonnet-5-5`；$2 / $10 每 MTok；延迟标注 Fast；1M token 上下文窗口；最大输出 128K。effort 默认值：**Claude API 上是 `high`，Claude Code 里是 `medium`**。官方定位 `The best combination of speed and intelligence`；专页同时写明 `For the hardest long-horizon work, an Opus model is the better choice.`

**Sonnet 5 的指引怎么沿用。** 专页原文：`Existing Claude Sonnet 5 prompts should perform well without changes, and the patterns in Prompting Claude Sonnet 5 remain a reasonable starting point.` 片段 4.16 和 4.25 出自 Sonnet 5 页面，依据就是这句。从 Sonnet 5 页面取用的只有这两段和 `for-claude-code.md` 里关于一次说清的那句，所以没有单独列「在 Sonnet 5 上测得」的表，三处各自标了出处。

**它和 Fable 5.2、Opus 5.5 不一样的 API 行为**（影响 SKILL.md 第 3 节怎么套用）：`thinking: {"type": "disabled"}` 在每个 effort 档都返回 400；想关掉前置思考，传 `thinking: {"type": "between_tools"}`，只在 `high` 及以下可用。强制 `tool_choice` 返回 400。advisor tool：Sonnet 5.5 执行者接受 Fable 和 Mythos 各代、Opus 5.5、Opus 5 或 Sonnet 5.5 自己做 advisor；Opus 4.8、Opus 4.7、Sonnet 5 和 Haiku 5.5 做 advisor 会返回 400。

| 已知倾向 | 处方 |
|---|---|
| effort 档位**重新校准过**，同一档位的思考量和 Sonnet 5 不一样 | 重跑扫描。起点 `high`；agentic 编码和多步工具使用，说清了的任务从 `medium` 起、更难更长的升 `high`；chat 和延迟敏感的工作从 `medium` 或 `low` 起。`xhigh` 和 `max` 留给实测到质量提升的工作。agentic 编码把 `max_tokens` 设成 128,000 并用流式 |
| 想让它少思考 | 降 effort。官方原话：`Asking the model in the system prompt to think less doesn't reliably reduce its thinking.` |
| `low` 和 `medium` 下做 agentic 编码时，**有时没做完就停下来确认**（确认计划、问一个自己能答的问题、做完一部分问要不要继续） | 先试更高的 effort。不想动 effort 时用 4.41。用了之后这两档的会话会更长、更贵；风险操作的规则自己保留 |
| 编码时**倾向于加没人要求的测试、文档和小的辅助文件**，各档都有，越高越多 | 多数团队会欢迎。只想要明确要求的改动，就只加 4.41 的第二段 |
| `xhigh` 和 `max` 下**做完之后可能自己开几轮复查和验证**，有子智能体时还会派 reviewer，也可能顺手做相关修复 | 例行工作跑在 `high` 及以下。想要高档位的细致、又想让它只用在任务本身，用 4.42。官方在 `max` 的编码任务上测得：不再派 reviewer 子智能体，会话成本约降三分之一，质量不变 |
| 请求很开放时（`show me what you can do with this`），**可能直接开始做**演示稿、报告或视频，而你只想要想法 | 在请求里说先要想法或计划，或用 4.43 |
| 要 JSON 答案、但任务需要几步推算时（汇总数字、套规则、排序），**常常不思考就答**，`low` 和 `medium` 尤其 | 能用 structured outputs 就用。然后用 4.44，或者跑在 `xhigh`；这类请求用 adaptive thinking 而不是 `between_tools` |
| 工具调用之间的播报超过一两句时以 progress-update `thinking` 块返回，默认文本为空，客户端可能显得没动静 | 设 `display: "updates"`（beta）；用 `between_tools` 时不要发 `display`，这些说明本来就带摘要文字返回，带上会返回 400。删掉 `hold all findings for the final response` 这类旧指令。想要固定时点的播报，在 system prompt 里说。长回合仍然太安静，用 4.3 的 Opus 5.5 那条提醒（两页原文相同），发两三次就停 |
| chat 和知识工作里，**有时凭训练知识作答**，而搜一下就能发现细节已经变了 | 先删掉压制工具使用的话（`only use tools when strictly necessary`、`minimize tool calls`），再用 4.45 |
| 有时**把用户中途发来的真消息当成注入**，尤其是它紧跟在工具结果后面、或放在 `tool_result` 块里的时候 | 用户文字永远不要放进 `tool_result` 块；中途的用户输入作为 user 回合送达，接在最后一个 `tool_result` 之后的文本块里；harness 的提醒放在另一条会话中 system 消息里；交互式会话里不要在工具结果后面加自己的 token 倒计时 |
| `low` 下**有时没跑能检验改动的检查就报告完成** | 4.46。官方测得跳过或敷衍的检查变得很少，任务质量没有可测的变化，成本只略高 |
| 偶尔把工具名的大小写写错（声明的是 `Bash`，它调 `bash`），或把参数名写得略有出入 | harness 不要当成致命错误：匹配无歧义就接受，或返回 `is_error: true` 并写明期望的准确名字 |
| 密集图表和工程制图 | 给它裁剪、缩放或对图跑代码的手段。图表上各档都有帮助；工程制图只在 `high` 及以上有帮助。图表上加工具比升 effort 更管用：官方测得带工具跑 `high` 比不带工具跑 `max` 读得更准，成本只是一小部分 |
| 安全分类器五类：`cyber`、`bio`、`frontier_llm`、`reasoning_extraction`、`general_harms` | 服务端 fallback（beta）只把 `cyber` 和 `frontier_llm` 的拒绝重试到 Claude Sonnet 5。叫它把推理写进回复的指令要删 |

## Claude Haiku 5.5

2026 年 10 月 7 日发布。模型 ID `claude-haiku-5-5`；价格分两档：prompt 不超过 100,000 token 时 $0.10 / $0.50 每 MTok，超过时 $0.50 / $2.50；延迟标注 Fastest；1M token 上下文窗口；最大输出 128K；默认 effort `medium`（API 和 Claude Code 都是）。官方定位：`For high-volume, latency-sensitive tasks such as classification, extraction, and routing`；选型矩阵把子智能体任务列在它名下。专页原文：`Existing Claude Haiku 4.5 prompts should perform well without changes.` 两点和成本有关（what's-new 页）：它把此前所有 assistant 回合的 thinking 块留在上下文里，这些块按输入 token 计，所以多轮对话的输入 token 更多；同样的输入文本在 Haiku 5.5 上产生的 token 比 Haiku 4.5 多约 30%。

**它和另外三个模型不一样的 API 行为**（影响 SKILL.md 第 3 节怎么套用）：`thinking: {"type": "disabled"}` 在 `low`、`medium`、`high` 下可用，`xhigh` 和 `max` 下返回 400。**接受强制 `tool_choice`**，但这时回复直接以工具调用开头、没有 `thinking` 块。拒绝类别是四类，**没有 `reasoning_extraction`**。prefill、`budget_tokens`、非默认采样参数和其他模型一样返回 400。

**专页建议加进 system prompt 的文字只在新会话里加**（仅 API）：`system` 变了之后回传 thinking 块可能返回 400。

| 已知倾向 | 处方 |
|---|---|
| 它是第一个有 effort 档位的 Haiku；effort 取代了 Haiku 4.5 的 `budget_tokens` | 起点 `medium`。`low` 用于 chat、短工具任务和简单的高吞吐请求——长 agent prompt 里它在这一档更容易跳过搜索、提前停、跳过检查。`high` 用于知识工作、较长的 agent 任务和严格的指令遵循。`xhigh` 和 `max` 只在评测显示质量提升值得这个成本时用，并且要同时在 Sonnet 5.5 上跑评测，比较表现、成本和速度 |
| 想让它少思考 | 降 effort。官方测试中，在 prompt 里叫它直接回答并没有让它停止思考。也可以在 `high` 及以下关掉 thinking |
| `xhigh` 的多轮 chat 里**有时把整个答案写在思考里**，回合结束时没有可见文字 | 检查每个回复是不是空的 |
| 给了搜索工具却不知道今天几号；`low` 和长 system prompt 下**有时该搜不搜** | 4.47：给今天的日期，再接那段搜索提醒。system prompt 短时可以不加搜索提醒：官方测得短 prompt 配 `medium` 时，只给日期就让它更常去搜。不要写 `search for any present-day factual question, regardless of how confident you are` 这种一刀切的指令——官方测得它让一半不需要搜索的 prompt 也去搜了，正确答案并没有变多 |
| 关掉 thinking、又用 structured outputs 要 JSON 时，**可能跳过它需要的工具调用** | 三选一：这类请求用 adaptive thinking；必须调工具的请求去掉 `output_config.format`；用 `tool_choice` 强制调用。必须关 thinking 时加 4.48 |
| system prompt 很长的编码 agent 在 `low` 下**有时提前停下，把任务交还给用户** | 4.49。升 effort 也能减少提前停：官方测得从 `low` 升到 `medium` 大致减半，但每次尝试的输出 token 多了一倍以上 |
| `low` 和 `medium` 下**有时没跑检查就报告代码改动完成** | 4.46（与 Sonnet 5.5 页面的原文相同）。官方测得它更常检查改动、表现提高，代价是 token 更多 |
| 用户中途发来的消息如果放在 `tool_result` 块里、或作为紧跟工具结果的会话中 system 消息到达，可能被当成不可信文本忽略 | 同 Sonnet 5.5：用户文字不进 `tool_result` 块，作为 user 回合送达；harness 提醒单独放 |
| 部署成 chatbot 或客服助手时，用户争辩、软磨、说有人批准了例外，**可能不再守 system prompt** | 4.50，与其他 prompt 注入防护一起用。指令遵循最要紧时再配 `high` |
| 有时在用户看得到的回复里写出**像推理的文字**，关掉 thinking 或 `low` 下更常见 | 换成 adaptive thinking 加 `medium` |
| 安全分类器四类：`cyber`、`frontier_llm`、`bio`、`general_harms`；从 Haiku 4.5 过来的话这些拒绝是新增的 | 它没有服务端 fallback，在客户端处理 `stop_reason: "refusal"`。把同一个请求再发一次通常还是被拒 |

## 验证指令：各模型的官方说法

这里按模型列各页面的说法；反引号里的是原文，其余是概括。

| 模型 | 官方说法 | 出处 |
|---|---|---|
| 通用建议 | `Ask Claude to self-check. Append something like "Before you finish, verify your answer against [test criteria]." This catches errors reliably, especially for coding and math.` | 最佳实践页 |
| Opus 5（legacy） | 上面这条建议的唯一例外：`it verifies its own work well without explicit instruction`，沿用的验证指令 `can cause over-verification, adding tokens and latency. On Claude Opus 5, remove these instructions rather than rewriting them.` | 最佳实践页；Opus 5 专页 |
| Opus 5.5 | 页面对过度验证既没重述也没撤回。无人值守的长跑另有一条**完成度**检查：任务清单，或事先写明完成条件后 `have a separate, smaller model check the conversation against it at each end of turn`——它查的是有没有做完 | Opus 5.5 专页 |
| Fable 模型（Claude Code 里） | `Skip the verification reminders: it verifies its own work with less prompting, so reminders to test or check are usually unnecessary.` | Claude Code model-config「Work with Fable」 |
| Fable 5.2 | `xhigh`、`max` 这类高档位下观察到它用子智能体复查自己的工作；同一段提醒子智能体可能占会话成本的很大一部分 | Fable 5.2 专页 |
| Sonnet 5.5 | 一般会在报告完成前检查；`low` 下有时不跑检查就报告完成 → 加 4.46。`xhigh`、`max` 下反过来，可能自己开几轮复查 → 4.42 | Sonnet 5.5 专页 |
| Haiku 5.5 | `low` 和 `medium` 下有时不跑检查就报告完成 → 加 4.46 | Haiku 5.5 专页 |
| Claude Code 里任何模型 | 给它一个能跑、结果是 pass 或 fail 的检查；并要它给出证据（跑了什么命令、输出是什么） | Claude Code 最佳实践 |

Fable 5 页面曾建议长自主运行时按间隔自查、并说独立的干净上下文 verifier 子智能体 `tend to outperform self-critique`。Fable 5.2 的页面没有重述，而 Claude Code 文档对 Fable 模型现在写的是上表那句 `Skip the verification reminders`，所以本文件不再把周期自查作为对 Fable 5.2 的建议。

## effort 取值参考

`max` > `xhigh` > `high` > `medium` > `low`。四个现行模型五档都支持。

**默认值按模型、也按界面不同：**

| | Fable 5.2 | Opus 5.5 | Sonnet 5.5 | Haiku 5.5 |
|---|---|---|---|---|
| Claude API | `high` | `medium` | `high` | `medium` |
| Claude Code | `medium` | `medium` | `medium` | `medium` |

Claude 应用里每个模型的推荐档在菜单里标着 "Default"；发布公告说 Fable 5.2 在 Claude 应用和 Claude Code 里默认 `medium`。支持站那篇讲应用设置的文章另有一句：Opus 4.7 及更新模型上的复杂编码和 agentic 任务，`try Extra high (xhigh) first`。这和 Opus 5.5、Fable 5.2 各自页面的说法（`xhigh` 留给实测到提升的工作）不一致，Sonnet 5.5 和 Haiku 5.5 的页面同样只在评测显示质量提升时才用 `xhigh`；本文件按各模型自己的页面；场景 B 的任务是复杂编码或 agentic 工作时，交付说明里把两种说法都告诉用户。

**effort 页对各档的通用定义**（各模型自己的建议与它不同时，以各模型的为准）：

| 档位 | 官方描述 | 典型用途 |
|---|---|---|
| `max` | `Absolute maximum capability with no constraints on token spending` | 需要最深推理和最彻底分析的任务 |
| `xhigh` | `Extended capability for long-horizon work` | `Long-running agentic and coding tasks (over 30 minutes) with token budgets in the millions` |
| `high` | `Spends as many tokens as the task needs for excellent results` | 复杂推理、困难的编码问题、agentic 任务 |
| `medium` | `Balanced approach with moderate token savings` | 需要在速度、成本和表现之间平衡的 agentic 任务 |
| `low` | `Most efficient. Significant token savings with some capability reduction` | `Simpler tasks that need the best speed and lowest costs, such as subagents` |

**各模型自己的建议：**

- **Fable 5.2**（Fable 5.2 专页 "Calibrate effort"）：`Start at high, the default on the Claude API`，评测显示质量不掉的地方再降到 `medium`（Claude Code 和 Claude 应用的默认档就是 `medium`，选型文章的任务表也从 `medium` 起，见 SKILL.md 第 2 节）。`low` 适合快速来回、说清了的任务和高吞吐工作，比如又快又聪明的子智能体，官方测试中它在 `low` 的分数接近 Fable 5.1 在更高档位的分数。`medium` 适合交互式编码、chat 和日常知识工作。`high` 适合困难的编码、研究、agentic 工作和长时间无人值守的运行，回合明显比 `medium` 久。`xhigh` 用于最难的长时间编码和 agentic 工作，且测到了提升；官方内部编码评测上它分数略升、token 多得多。`max` 用来测模型上限，或用于时间和 token 都无所谓的少数任务，最适合科学发现和数学这类开放问题；effort 页说它比 `xhigh` 更慢、更贵，换来的往往只是很小的提升。官方的总括：`Most of the gain in capability is already present at high`，`Above high, gains are uneven and depend on the task`。低档位下它更容易提前停，`low` 或 `medium` 做 agentic 工作时加 4.37。一个流程里可以混用档位：官方的例子是用 `medium` 规划和迭代，到长时间无人值守的那一段再升档。
- **Opus 5.5**：从默认的 `medium` 起步。只有在自己的评测上**实测**到质量提升才上 `xhigh` 或 `max`——官方原话是 `Reserve xhigh and max for work where you've measured a quality gain`。官方测试中 Opus 5.5 在 `medium` 的编码与知识工作评测成绩达到或超过 Opus 5 在 `high`；若干编码评测上 `low` 也接近 Opus 5 在 `high` 的水平，成本低得多。
- **Sonnet 5.5** 和 **Haiku 5.5**：见各自一节的第一行。

**Claude Code 文档对各档的说法**（对会话通用；四个现行模型在 Claude Code 里都从 `medium` 起）：

- `medium`：`plenty for everyday feature work with a clear scope`
- 升到 `high` 或 `xhigh`：对错取决于容易漏掉的情况——不是你写的代码里的 bug、flaky test、要找出旧结构所有读取方的 schema 变更。这时 Claude 会检查更多边界情况、在回答前验证更多工作，多出来的时间和 token 花在这里
- 降到 `low`：来回很快、每个结果你反正都要看——头脑风暴、初稿、重命名这类小改动
- `max`：留给想让 Claude 自己啃下来的难题，比如找安全漏洞；普通任务上它容易想过头，花得更多、答案未必比 `high` 好
- 只有下一条消息需要更深的推理时，在那条 prompt 里写 `ultrathink`，不用改设置

官方给出的两条先后顺序：

- **调 effort 往往比换模型更有效**（官方原文 `Tuning effort is often a better lever than switching models`）：先在手上这个模型上把档位跑一遍，再考虑换模型
- 如果模型思考的频率超出任务需要，**先降 effort 档位**。Fable 5.2、Sonnet 5.5、Haiku 5.5 的页面都写明用 prompt 叫它少思考不可靠；Opus 5.5 的页面说降 effort 比 prompt 指令更可靠

要点：

- **把 `effort` 设成该模型的默认值，与完全不传该参数行为完全相同**（API）
- effort 是**行为信号而非硬预算**，影响正文、工具调用和思考的全部 token
- **低 effort 意味着工具调用更少、更简短**；低档位往往会合并操作、省去前言，高档位则可能更多工具调用、先讲计划、给详细总结
- **如果 effort 设置是从别的模型沿用过来的，在自己的评测上重新做一次扫描。** Fable 5.2 专页：`rerun your effort sweep instead of carrying over a setting you tuned on Claude Fable 5.1 or Claude Opus 5.5`
- 在请求之间改**顶层** effort 会让 prompt cache 失效。四个现行模型都支持**按消息改 effort**（beta，header `mid-conversation-output-config-2026-07-01`：一条 `content` 为空、带 `output_config.effort` 的 `role: "system"` 消息），缓存不失效，各平台的支持情况见 effort 页。紧跟在一条带新输入的 user 消息之后时，它决定的是对这条消息的回复用什么档位；放在别处时，从下一条带新输入的 user 消息起生效——只装着 `tool_result` 块的 user 消息不算——新档位一直保持到后面有消息再改它。Sonnet 5.5 用 `between_tools`、或 Haiku 5.5 关掉 thinking 时，按消息改 effort 返回 400
- 上一条是 Claude API 的规则。Claude Code 里（prompt caching 页）：用 API key 或订阅时，在 Fable 5.1 及更新的 Fable 模型、Opus 5.5、Sonnet 5.5 和 Haiku 5.5 上改 effort 默认不丢缓存；在 Amazon Bedrock、Google Cloud 的 Agent Platform 或 Claude apps gateway 上，改 effort 之后下一个请求读整段对话历史都不命中缓存
- 要给 `max_tokens` 留足空间——它是思考加正文的**总**上限，而且即使思考文本不返回给你，思考也计入这个上限。Opus 5.5：长的 agentic 编码回合 128,000 `has worked well in Anthropic's testing`。Sonnet 5.5：agentic 编码设 128,000 并用流式。Fable 5.2：`high` 及以上设大。成本页的通用建议：agentic 工作设 64,000，单次被截断代价很高时设 128,000。Claude Code 和 Claude 应用里这个值由客户端管

## 价格、用量上限与省用量做法（官方数字）

本节只收官方页面写了的内容，出处标在每条后面。

### API 价格（每百万 token，美元）

| 模型 | 输入 | 输出 | 缓存读取 | Batch（输入 / 输出） | Fast mode（输入 / 输出） |
|---|---|---|---|---|---|
| Fable 5.2 / Mythos 5.2 | $8 | $40 | $0.20 | $4 / $20 | $16 / $80 |
| Opus 5.5 | $4 | $20 | $0.20 | $2 / $10 | $8 / $40 |
| Sonnet 5.5 | $2 | $10 | $0.10 | $1 / $5 | 不提供 |
| Haiku 5.5（prompt ≤ 100,000 token） | $0.10 | $0.50 | $0.01 | $0.05 / $0.25 | 不提供 |
| Haiku 5.5（prompt > 100,000 token） | $0.50 | $2.50 | $0.05 | $0.25 / $1.25 | 不提供 |

出处：pricing 页、各模型 overview 和 what's-new 页。Batch 是五折，适合能等 24 小时以内的工作。Fast mode 输出速度最高 2.5 倍，价格是标准价的两倍；在订阅上它只走 usage credits。输出 token 的价格是输入的五倍，而且 `in an agent loop every token the model writes comes back as input on every later turn`（成本页）。

### effort 对 token 和耗时的影响

官方没有给出「每升一档多花百分之几」的通用倍数。它给的是具体测量（成本页 "Tune effort"，除注明外均为 Anthropic 内部测量，官方自己标注为 `directional, not guarantees`）：

- **Opus 5.5，SWE-bench Pro 子集**：`low` 解出 87.4%、每任务 $0.11；`medium` 92.8%、$0.21；`high` 95.3%、$0.29；`xhigh` 96.7%（单次运行）、$0.74
- **Opus 5.5，`low` 对比 `medium`**：典型任务耗时在 DRACO 上少 84%、HLE 上少 30%、内部物理题上少 47%，分数分别低 13.3、3.4、14.0 分。内部缺陷搜索基准上 `low` 比 `medium` 低 34 分，成本约为五分之一
- **Fable 5.2，DeepResearch Bench II**：`low` 54.5%、每任务 $1.49；`medium` 55.0%、$2.16；`high` 57.7%、$4.82。`high` 比 `low` 高 3.3 分，成本是 3.2 倍
- **Fable 5.2，缺陷搜索**：`low` 比 `high` 低 4.3 分，成本少 37%
- **Fable 5.2 专页**：从 `medium` 到 `high` 这一步在官方跟踪的多数编码和知识工作评测上提高了分数，同时用更多 token 和时间；`xhigh` 在内部编码评测上 `raised scores slightly and used many more tokens`；`max` 的回合慢，token 比 `xhigh` 多得多
- **Sonnet 5.5**：`medium` 比默认的 `high` 在 SWE-bench Pro 子集上低约 7 分，每任务成本约为 59%；在内部 agentic 编码基准上是 62.7% 对 77.8%（每任务一次尝试）
- **Haiku 5.5**：长 agent prompt 里从 `low` 升到 `medium`，提前停大致减半，每次尝试的输出 token 多一倍以上（出自 Haiku 5.5 专页，不是成本页）

个别 prompt 片段的 token 代价也是官方给的：4.37 输出 token 多约 30%；4.38 约 1.5 倍；4.3 里 Fable 5.2 的开场与收尾那段每任务多约 20%。

### 模型之间的成本比较

出处是成本页（"Compare models on cost per task"、"Upgrade the model"、"Orchestrator strategy"）和发布公告。官方的总原则：`Compare models on cost per completed task, not per token`；同样的文本在 Claude 4.7 及之后的模型上多约 30% 的 token，所以按 token 比会让新模型显得更贵。

- SWE-bench Pro 子集（463 题）：Fable 5.2 在 `high` 解出 96.3%，每个解出的任务 $1.39；在 `low` 解出 94.6%、$0.39。Opus 5.5 默认档 93.1%、$0.22；`high` 95.5%、$0.30；`xhigh` 96.5%、$0.75。官方的话：`At a similar pass rate, Claude Opus 5.5 still costs less per solved task on these problems`
- Opus 5.5 在 `low` 与 Sonnet 5.5 在默认档解出的比例大致相同，每个解出的任务贵约 7%
- DeepResearch Bench II（研究类；对每个模型都评了分的 44 个任务，Messages API 上的默认 effort）：Opus 5.5 比 Sonnet 5.5 高 2.4 分，每任务成本是 1.5 倍（在全部 50 个任务上，Opus 5.5 没有可测地领先 Sonnet 5.5）；Fable 5.2 的成本是 Sonnet 5.5 的 4.3 倍，分数没有可测的提升；是 Opus 5.5 的 2.9 倍，分数低 1.9 分。对比 Fable 5.1，它在各档位成本低 20% 到 61%，分数低 4.7 到 7.0 分。成本页紧挨着这组数字的建议：`Measure what a more expensive model buys on your own workload`，以及 `Price the tail of your workload, not the median`（拿最难的那十分之一任务来比，而不是典型任务）
- 对比 Fable 5.1 的另外两组数字（成本页）：SWE-bench Pro 子集上，Fable 5.1 默认档解出 92.4%、每个解出的任务 $1.17，Fable 5.2 在 `high` 解出 96.3%、$1.39——解得更多，每个解出的任务更贵；缺陷搜索基准上，Fable 5.2 在每一档都比 Fable 5.1 分数高、成本低（`low` 高 3.6 分、便宜 32%；`medium` 高 3.8 分、便宜 37%；`high` 高 6.1 分、便宜 21%）
- 缺陷搜索基准（工作量超出一个上下文窗口）：Fable 5.2 单独做，`low` 到 `high` 比 Opus 5.5 单独在 `high` 高 1.6 到 5.9 分，成本是 1.5 到 2.4 倍
- 内部 agentic 编码基准：Opus 5.5 在 `medium` 86.6%、每次尝试 $0.84；Fable 5.2 在 `medium` 88.9%（单次运行）、$1.34
- GPQA Diamond：Haiku 5.5 在 `medium` 准确率 85%，Opus 5.5 和 Sonnet 5.5 各 91%；每个正确答案的成本约为 Opus 5.5 的二十分之一、Sonnet 5.5 的十分之一
- 发布公告（anthropic.com）里的数字：FrontierCode 2.0 上（公告注明是早期测试）Fable 5.2 在 `high` 与 Opus 5.5 在 `max` 大致持平，每任务成本约一半；WANDR 上 Fable 5.2 在 `medium` 追平 Opus 5.5 的最高分，每任务成本约为 40%；Chartography 上 Fable 5.2 在 `medium` 领先 Opus 5.5 在 `max`，每任务成本不到十分之一；Humanity's Last Exam 上 Fable 5.2 在 `low` 追平 Opus 5.5 的最高分，每任务成本不到十分之一

### 子智能体与多模型

- 每个子智能体自己发请求，`count toward the same usage limits as your main conversation`（子智能体页）。`Running several sessions or subagents at once multiplies token usage`（agents 页）
- 官方测得委派只在两种情况下划算：有很长成本尾巴的例行工作（在更早的模型上测得；官方要求这种情况先测准确率和成本）；超出一个上下文窗口的工作。工作是一条前后依赖的链、或者一个上下文装得下又没有长成本尾巴时，编排者模型自己用更低的 effort 更划算——在更早的模型上测过的每个案例里都是如此（成本页 "Orchestrator strategy"）
- 超出一个上下文窗口的工作（内部缺陷搜索基准）：Opus 5.5 配 Sonnet 5.5 子智能体，比每部分一个 Opus 5.5 会话便宜约三分之一，分数相当。把编排者换成 Fable 5.2，成本高 19%，分数相当。同一基准上 Opus 5.5 配 Haiku 5.5 子智能体的平均 F1 是 0.677，配 Sonnet 5.5 是 0.831（成本页，基准说明第 8 条）
- 编码任务上的 Haiku 5.5 子智能体：`an alternative to lowering effort, not better`——加到 `medium` 的 Opus 5.5 上，每次尝试成本降 32%，通过率降约 4 个百分点
- agent team 在队友跑 plan mode 时 `use approximately 7x more tokens than standard sessions`（Claude Code 成本页）。同一页的其他建议：团队保持小（token 用量大致和团队规模成正比；agent teams 页的起点是 `Start with 3-5 teammates for most workflows`）、派生队友的 prompt 保持聚焦、队友做完就关掉
- advisor 模式（测的是 Opus 5.5 在 `high` 配 Fable 5.1 advisor 等几组配对；Fable 5.2 执行者接受哪些 advisor 见上文 Fable 5.2 一节）：测过的几组配对里，执行者经常去咨询的那一组分数高 1.7 分（在运行间噪声边缘），成本约 2.1 倍；执行者很少或从不咨询的几组，分数在噪声范围内或更低，其中一组低 7 分（成本页）。advisor tool 页自己的说法方向不同：给做复杂任务的 Sonnet 加一个更高档的 advisor，`Opus keeps total cost similar or lower; Claude Fable 5.2 maximizes the quality lift`；编码任务上，`medium` 档的 Sonnet 执行者配 Opus advisor `achieves intelligence comparable to Sonnet at default effort, at lower cost`。Claude Code 的 advisor 页：订阅计划上 advisor 的用量计入计划上限，Fable advisor 在 Fable 走 usage credits 的计划上记到 usage credits
- 缓存寿命：订阅上 prompt cache 大约保留一小时，API key、云厂商或 usage credits 上是五分钟；子智能体和 workflow agent 默认是五分钟。fork 复用主会话的缓存，所以同样上下文的任务比全新的子智能体便宜（Claude Code 最佳实践、workflows 页、子智能体页）

### 订阅用量上限

- Pro 和 Max：一个五小时会话上限加一个每周上限。Ultra：每周上限；**通过验证的 Ultra 账号没有五小时上限**，没通过验证前有一个临时上限。Max 和 Ultra 另有一个单独的 Fable 每周上限（支持站「Use Claude Code with your Pro, Max, or Ultra plan」「What is the Ultra plan?」）
- Ultra 每周用量是 Pro 的 10 倍。各计划只用 Pro 的倍数描述，官方没有给出绝对的 token 或消息数（支持站「What is the Ultra plan?」）
- 上限在 Claude 和 Claude Code 之间共用，IDE 里的用量也算在同一个上限里（支持站「Use Claude Code with your Pro, Max, or Ultra plan」）
- Max 和 Ultra 上，Fable 模型最多可以用掉每周上限的 50%，而且 `use them faster than other Claude models`。到了 Fable 上限后，可以用 usage credits 继续用 Fable，或者换到别的模型在计划内继续。Pro 和 Team 的标准席位上，除限时的发布优惠外，Fable 模型不含在计划额度里，从一开始就走按量付费的 usage credits（支持站「Claude Fable models on your plan」）
- 用量同时计入会话上限和每周上限；Claude Code 错误参考页的原话：`A single burst of heavy activity, such as a large workflow fanout, can exhaust the weekly allowance before the session window resets.` 开着 ultracode 的会话每个请求用的 token 更多，会更早到会话或每周上限（workflows 页）
- Ultra 的支持页另写明：为管理容量，可能适用额外的用量限制
- effort 越高，`you'll reach your usage limits faster`（支持站「Change the model, effort, and thinking settings」）

### 到了上限会怎样

- 请求会被挡住，直到上限重置。会话上限和每周上限对所有模型共用，`so switching models doesn't restore access`；Opus 或 Sonnet 系列各自的上限只管对那个系列的请求，换到系列外的模型可以继续（错误参考页）；到了单独的 Fable 每周上限，同样可以换到别的模型继续用计划内的额度（见上）。开着的交互式会话默认会等到重置后自动接着做（`autoContinueAtUsageLimit` 默认 `true`；settings 页）。其他做法：开 usage credits（按 API 价格计）；关掉了自动继续的，可以在 `/rate-limit-options` 里手动选等待（Claude Code 命令页）
- 无人值守的会话可以设 `CLAUDE_CODE_RETRY_WATCHDOG=1`：它会一直重试容量错误，响应带了重置时间时就等到上限重置，`so a session that hits a usage limit waits out the remaining window`；例外是报告 spend limit 或 usage credits 用尽的 429，遇到它会立刻失败（环境变量页）
- **没人守着的会话里用 Fable**（错误参考页）：后台会话、Remote Control 会话、agent team 队友会话和 Agent SDK 托管的会话里，Fable 走 usage credits 的确认提示没人回答就会关掉，这一轮以 `the prompt to confirm went unanswered — nothing was sent` 结束；到了 Fable 上限、或 Fable 请求开始走 usage credits 时都会这样。处理办法：到会话运行的地方回答提示，`/model` 换到不走 usage credits 的模型，或把 `dialogExpiry` 设长
- 在五小时上限处，如果 Claude Code 正在回复中途，可能短暂继续把手上这一步做完，界面显示 `Usage limit reached · wrapping up`；这只对已经在进行的回复有效，而且这项功能在逐步推出，账号上可能还没有。通过验证的 Ultra 账号没有五小时上限，所以不涉及（支持站「Use Claude Code with your Pro, Max, or Ultra plan」）
- **子智能体**因 API 错误（官方举的例子包括 usage limit）结束时：前台子智能体如果已经产出了文字，Agent 工具返回这段部分输出，并注明它被打断、没做完；什么都没产出、或只产出了工具调用的，以 `Agent terminated early due to an API error` 失败。后台子智能体标记为失败，回报里带它最后的输出，`so partial work isn't lost`。错误消除后可以让 Claude 重试，或用 SendMessage 续跑那个子智能体（Explore 和 Plan 不能续跑）（子智能体页 "API errors in subagents"）
- **dynamic workflow** 里的 agent 撞到 claude.ai 用量上限时，这次运行会暂停而不是失败：撞到上限的 agent 等重置，不再启动新的 agent，重置后自动继续。条件：交互式会话、用 claude.ai 订阅登录、`autoContinueAtUsageLimit` 开着、上限在 24 小时内重置（每周上限的重置可能更远）、这次运行此前等待不超过两次；用 `claude -p` 或 Agent SDK 的非交互模式、后台会话、Remote Control 会话和 agent team 队友会话里不会暂停（workflows 页；需要 Claude Code 2.1.271 及以上）
- **`/goal`** 在 claude.ai 用量上限这类重试也只会重复的失败之后暂停，并给出以 `Goal paused` 开头的提示；会话如果在等上限重置后自动继续，重置后 Claude 接着朝目标做（goal 页）
- **limit reset**（支持站「What is a limit reset?」「Claude Fable 5.2 launch offers」）：符合条件的计划偶尔会得到重置额度，不必等到撞上限就能用。Fable limit reset 恢复的是这一周花在 Fable 模型上的用量。按钮 "Reset for free" 在网页、Claude Desktop 和手机应用的 Settings > Usage 里，Claude Code 的终端和 IDE 里目前没有；用量上限是整个账号共用的，所以恢复的用量在 Claude Code 里同样可用。Fable 5.2 发布时给 Max、Ultra 和 premium 席位的是一到三次 Fable limit reset（按订阅时长），2027-03-08 到期

官方页面没有描述 dynamic workflow 之外的子智能体在上限处自己暂停再继续；按子智能体页，它以错误结束，之后可以重试或续跑。主会话自己撞到上限时正在跑的子智能体，文档页没有专门描述；changelog 有两条相关记录：后台 agent 会在已经让主会话停下的同一个上限上失败（2.1.292 的修复说明），它们的通知会写明是被用量上限停下的（2.1.299）。

### 官方推荐的省用量做法

- `/usage` 看用量花在哪：订阅计划上它按 skill、子智能体、插件、MCP server 给出占比，并对占近期用量 10% 以上的行为给出建议（Claude Code 成本页、最佳实践页）
- 不相关的任务之间 `/clear`；长会话 `/compact` 并说明要保留什么（`/compact` 本身是一次大请求）（Claude Code 成本页、最佳实践页）
- Claude Code 的 prompt caching 页：`Pick your model and effort level at the top of a session, then save /compact for natural breaks between tasks`。会话中途换模型，下一个请求整段重新计算；第一次打开 fast mode，下一个请求按 fast mode 费率不命中缓存地读完整段对话，所以要开就在会话开始时开；`opusplan` 下每次进出 plan mode 都是一次换模型，缓存从头开始。想彻底放弃走错的一段路，用 `/rewind` 回到更早的一轮，它回到的是已经缓存的前缀。选型文章给换模型的建议是挑自然的间歇：先 `/compact`，或带一份简短的书面计划开新会话，让新模型上要付缓存写入价的第一轮小一些
- 模型：`Sonnet handles most coding tasks well and costs less than Opus`；简单的子智能体任务在定义里写 `model: haiku`；想让某个自定义子智能体留在小模型上，在定义里设 `model`（Claude Code 成本页、model-config 页）
- 对简单任务降 effort（`/effort` 或 `/model` 里调）（Claude Code 成本页）
- 写具体的 prompt：`Vague requests like "improve this codebase" trigger broad scanning`，具体的请求让 Claude 少读文件；复杂任务先用 plan mode，方向错了的返工很贵；在 prompt 里给验证目标（Claude Code 成本页 "Reduce token usage"）
- Claude 应用里（支持站「Usage limit best practices」「How do usage and length limits work?」）：把相关的问题合进一条消息、事先给足背景；例行任务选更低的 effort；关掉这次对话用不上的已连接应用，不需要最新信息时让 Claude 不要搜网页；对话很长、快到上限时开一个新对话；项目里的内容有缓存，重复使用时少计用量
- 让子智能体回报简短摘要，`so twenty results don't fill your context anyway`（Claude Code 最佳实践）
- CLAUDE.md 控制在 200 行以内，专门流程挪到 skills；关掉不用的 MCP server；能用 CLI 工具就用 CLI（Claude Code 成本页）
- 注意后台还在发请求的东西：`/loop`、goal 每轮之后的检查、Claude 自己派的子智能体（Claude Code 最佳实践）
- API 上的几条（成本页）：`In Anthropic's measurements, prompt caching was usually the largest lever`；没人等结果的请求走 Batch API，官方称批处理是无人值守 agent 工作里仅次于缓存的第二大免费杠杆；用不到的工具定义延迟加载（tool search；在更早的模型上测得）；要让模型计算的表格用 Files API 上传并配 code execution，不要贴进 prompt（在更早的模型上测得：贴进去每个请求约 91,000 输入 token，25 题答对 7 题；上传后 25 题全对，成本约十二分之一）；Haiku 5.5 的长对话在 prompt 超过 100,000 token 之前压缩，超过之后请求里的每个 token 贵 5 倍，输出也算
- 产出能校验时（API）：`low` 跑一遍、失败的用 `high` 重跑。官方在 Opus 5.5 的 SWE-bench Pro 运行上算得：成本比全部 `high` 低约 40%，通过率相同。前提是有可靠的失败信号，失败的任务要花两次的时间（成本页 "Re-run failures at higher effort"）
- 要更短的答案：三种最终答案格式（一行、两行、六节备忘录）正确率在噪声范围内相同，每次运行成本分别是 $0.49、$0.57、$1.40。官方的话：`Ask for the answer you will read, not the one that looks thorough`（成本页，在更早的模型上测得）
- 告诉模型时间要紧并在每次请求里给出已用时间：单个 Opus 5.5 agent 在 `medium` 下典型 DRACO 研究任务耗时少 47%、成本少 60%，平均分低 4.1 分（成本页；片段 4.33）
