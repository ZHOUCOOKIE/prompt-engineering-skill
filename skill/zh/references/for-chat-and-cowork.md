# 场景 B：写给 desktop chat 或 Cowork session 执行的任务 prompt

用户在 Claude desktop 里说需求，产出的 prompt 在 **desktop chat** 或 **Cowork session** 里直接跑。非代码任务：研究、分析、写作、整理资料、做文档表格幻灯片、日程、信息汇总。

## 目录

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
- 信源

---

## 先分清 chat 还是 Cowork

两者定位不同，prompt 写法跟着变。以下依据官方目前对这两个界面的说明（第三种 Claude Code 属于场景 A，见 for-claude-code.md）。Cowork 一栏依据 Cowork 概览，只有定时任务一条依据 Claude Desktop 的 Unified Claude 页和 changelog；chat 一栏依据 Claude Desktop 的 Chat 页——它是为接入第三方云的部署而写的，也是把 Chat 的限制讲清楚的那一页，其他环境里的 chat 在个别地方可能不同。

| | desktop chat | Cowork session |
|---|---|---|
| **定位** | 用来快速提问和起草的对话界面 | agentic 工作区：说清想要的结果，回头来取做好的成品——成形的文档、整理好的文件、汇总好的研究 |
| **文件与工具** | 刻意收窄的工具面：附进对话的文件、连接器、skills | **直接读写本地文件**，无需手动上传下载；访问网站用 Claude in Chrome |
| **产出** | 对话里的内容，外加它在自己的临时空间里生成、供下载的文件 | **成品交付物**：带可用公式的 Excel、PowerPoint、格式化文档 |
| **长时与周期性工作** | 不能跑定时任务，也不能转交给 agentic 会话；需求超出能力时，它会建议改用 Cowork | 自主完成多步任务；支持**定时任务** |
| **并行** | 无 | **子智能体协作**，复杂工作拆成并行工作流 |

**一条重要边界**：Cowork 在你授权的文件夹上工作，它的代码跑在隔离的工作区虚拟机里，而不是你自己的 shell。仓库级操作、git、构建部署属于 Claude Code——那是场景 A，不是场景 B。

**这两个界面正在合并。** 在接入第三方云的 Claude Desktop 上，Unified Claude（beta）把 Chat 和 Cowork 合成一个体验，带 Cowork 的 agentic 能力：在能运行 Cowork 的设备上，从首页发出的消息直接开启一个 Cowork 会话。在 2026 年 11 月 10 日之前它是可选的，之后对所有这类部署强制启用；Anthropic 现行的桌面端指南也已不再把 chat 和 Cowork 当作两种模式分开介绍。用户处在这种配置下时就没有二选一的问题——只要任务需要文件、多份材料或成品交付物，就按 Cowork 的能力来写。

**判断方法**：交付物是"一段可以直接读的内容"→ chat 就够；交付物是"一个要保存下来的文件"、或需要跨多份材料系统性取材 → Cowork。

如果用户没说，而任务明显需要产出文件或处理批量材料，**默认按 Cowork 写，并在交付说明里点明这一点**。

**两个值得在 prompt 里主动用上的 Cowork 能力**：

- **定时重复任务**：如果用户的需求本质是"每周一给我一份 X"，prompt 应该写成可复用的任务描述，并提示他设成定时任务，而不是每次手动跑。
  **定时任务意味着无人值守**：跑的时候用户不在，任何"要不要继续？"都会把整个任务卡死到下一周。所以周期性 prompt 必须写明**遇到问题怎么自己往下走**——素材缺失、文件打不开、来源冲突无法判定，一律记录下来继续做，最后在产出里单列"需要你确认"一节，而不是停下来问。（完整的无人值守片段在 Fable 5.1 上见 4.18，在 Opus 5.5 上见 4.31，但对周报这类轻量任务不必整段搬，写清这一条即可。）
  **写周期性交付物的 prompt 时还有一条必须加**：让它先读上一期的产出，然后**只写增量**——发生了什么变化、上期说要做的做了没有、哪些还挂着。不加这一条，每期都会变成从零开始的平铺罗列，读的人自己去比对差异。这对周报、站会纪要、状态更新、简报、变更日志都成立
- **子智能体并行**：材料很多时，prompt 里可以明确说"用子智能体并行处理各份材料，每个只回报结构化摘要"——这既快又省主上下文

---

## 输出长度必须写死

这是场景 B 最高频的问题。**别指望靠 effort 缩短回复。** 在 Opus 5 上，effort 控的是想多少、不是说多少；最佳实践页把这写成 Opus 5 在冗长度上的例外（effort 页说，一般情况下 effort `affects all tokens in the response`），而 Opus 5.5 和 Fable 5.1 的页面都没说降 effort 对回复长度有什么影响。所以在所有适用模型上，长度都要在 prompt 里写明。（默认输出更长这一点是对 Opus 5 写明的：Opus 5 页说它写到磁盘的文件 `are often longer than on prior models`。官方说 Opus 5.5 `tends to complete the same task with fewer tokens`，所以先实测再下结论——但长度无论如何都要写。）

所以任何面向阅读的产出，prompt 里都要有明确的长度约束。**具体怎么选、哪些能叠加，按 SKILL.md 第 8 节「长度控制怎么选」的规则**，简言之：

- 主控手段二选一——**能给数字就给数字**（"300–400 字"、"3–6 条，每条不超过 40 字"，不要写"简洁一些"），给不出数字才用片段 4.2
- 产出物写成文件时**追加**片段 4.4（数字通常只约束正文，管不住模型自行添加的填充章节）
- system prompt 很长时，末尾**追加** `<tone_preference>` 短提醒

---

## 材料怎么摆

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

**长文档还有一招 grounding in quotes**：先让它把相关引文摘进 `<quotes>` 标签，再基于引文回答。这能让它聚焦相关内容、忽略文档的其余部分。

---

## 要格式稳定就给示例

场景 B 里大量任务是"每次都按这个格式输出"——周报、会议纪要、复盘、简报、卡片。这正是 SKILL.md 第 4 节里**该给示例**的那一支。

- 3–5 个，用 `<example>` 包裹
- 要**多样**，覆盖边界情况，避免模型学到意外模式
- 最有价值的示例是**成对的**：原始素材 → 成品。这样教的是压缩比和取舍标准，不只是长什么样
- 如果用户手上有过去写好的真实样本，让他放两三个进去——这是单点回报最高的一步

**用户还没有样本时怎么办**（一次性交付、问不了用户的情况很常见）：**给一个你自己写的成对示例就够，不要硬凑 3–5 个**。理由是——凭空编出来的多个示例会把一套你臆想的"标准格式"当成规范教给模型，用户之后很难察觉哪里不对。正确做法是给一个示例说明**压缩比和取舍标准**，然后在 prompt 里留出明确位置，并在交付说明里告诉用户：**把你真实的两三期换进去，这是提升最大的一步**。

反过来，如果任务是开放式创作、头脑风暴、或者你希望它自己判断切入角度，**不要给示例**，示例会把它的探索空间收窄到示例附近。

---

## 研究类任务

需要跨多个来源查证、综合的任务，官方给了一段现成的结构化指令。**照抄，包括里面那句自我批评在内**——理由见下方的例外说明：

```
Search for this information in a structured way. As you gather data, develop several competing hypotheses. Track your confidence levels in your progress notes to improve calibration. Regularly self-critique your approach and plan. Update a hypothesis tree or research notes file to persist information and provide transparency. Break down this complex research task systematically.
```

> **这是 SKILL.md 3.4 的一处例外。** 这段里有 `Regularly self-critique your approach and plan`，而 3.4 说 Opus 5 上要删掉自我验证类指令（Opus 5.5 上则要先实测再删）。这和片段 4.7 里那句 `you MUST read the file` 属于同一类例外：**它是调校过的官方原文，照抄即可**。两者说的不是一回事——3.4 删的是*答案写完后再自审一遍*；这句是*跑的过程中随时纠偏*，证据还在陆续进来的时候调整方向。如果任务是一次性写一篇东西、而不是多步检索，那这句没有可纠的对象，可以只删掉这一行。

另外三条：**给清晰的成功标准**（什么样的答案算回答了这个问题）、**要求跨来源交叉验证**、**显式允许说不知道**。下面这句是 Anthropic 2025-11 那篇 prompt engineering 博客里那个例子的第二句（"Give permission to Claude to express uncertainty" 一节）；它不在文档页上，也没有经过特别调校，可以自由改写：

```
If the data is insufficient to draw conclusions, say so rather than speculating.
```

最后一条对研究任务特别重要——不给退出通道，模型会把不确定的东西说得很肯定。

---

## Cowork 特有：交付物与文件

Cowork 能产出真实文件。写 prompt 时**明确说出要什么文件类型**，因为这直接决定它走哪条路径：

| 交付物 | 在 prompt 里怎么说 |
|---|---|
| 可编辑文档 | "输出为 .docx" |
| 数据表 | "输出为 .xlsx，第一行表头，每列……" |
| 演示 | "做成 .pptx，X 页，每页……" |
| 可分享的页面 / 看板 / 追踪表 | "做成单文件 HTML" |
| 纯文字给我直接看 | 明说"直接在对话里给我，不要生成文件" |

**落到哪里**：如果用户要文件保存到自己电脑某个文件夹，prompt 里要写清楚**目标文件夹**，并确认这个文件夹已在 Cowork 里添加且给了编辑权限——桌面端指南说，有了这个权限，Claude 会把做好的成品存回那里。没有可写的文件夹，文件就不会落到用户自己的文件夹里。

**批量处理**：涉及多个文件时，写明处理顺序和命名规则，以及**处理不了的怎么办**（跳过并记录，还是停下来问）。不写这条，遇到坏文件时行为不可预期。

---

## Cowork 特有：会自动接管的 skill

Cowork 里装了一批输出格式 skill（docx、xlsx、pptx、pdf 等）。它们会在涉及对应文件类型时自动介入。这意味着：

- **不需要在 prompt 里教它怎么生成 docx**，那是 skill 的事，写了反而是冗余（SKILL.md 3.3）
- prompt 该说的是**内容要求**：结构、口径、数据来源、长度
- 但**要说清楚样式偏好**，比如"用公司模板"、"不要用默认配色"——这些 skill 猜不到

有一个顺序值得写进 prompt（这是本 skill 自己的建议，不是官方文档写明的规则）：**先把事实查全、再动手做文件**。过早开始搞文档机制，会过早锚定在排版上，而手里还没有正确的内容。

---

## 不要写进去的东西

场景 B 里最容易多写的三类：

1. **自我验证指令**（"最后检查一遍"、"再核对一次"）——在 Opus 5 上模型自己会做，写了造成过度验证；在 Opus 5.5 上这个结论没有被重述，所以不要条件反射式地加，也不要拿 Opus 5 的结论当 Opus 5.5 的结论。见 SKILL.md 3.4。注意 3.4 划的那条界线：**跨来源交叉核对是任务本身，不是自我验证**——研究类 prompt 里不要连它一起删掉
2. **人设铺垫**（"你是一位有二十年经验的资深分析师"）——一句话的角色就够了（最佳实践页：`Even a single sentence makes a difference`），再堆履历只是多占 token，Anthropic 2025-11 那篇文章也说过度的角色设定往往没必要。要影响视角就直接说关注什么——那篇文章的例子是 `Analyze this investment portfolio, focusing on risk tolerance and long-term growth potential`，而不是给它安一个角色
3. **一长串"不要"**——改成正面描述加原因。见 SKILL.md 3.5

---

## 一个可直接套用的骨架

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

## 可复用 / 周期性 prompt 的变量块

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

## 信源

本文件中关于 chat 与 Cowork 能力边界的描述，依据 Anthropic 的 Cowork 概览和 Claude Desktop 文档——其中的 Chat 页和 Unified Claude 页（为接入第三方云的部署而写）是把 Chat 的限制和两者合并讲清楚的页面，Claude Desktop 的 changelog 记录了工作区虚拟机和定时任务。Anthropic 的桌面端指南已不再对比各模式。prompt 技法部分依据官方 prompting best practices。涉及产品能力的具体细节可能随版本变化，判断不准时以用户实际看到的界面为准。
