# 官方提示词片段库

> 除 4.1 外，所有英文片段均为 Anthropic 官方文档**逐字原文**，已逐字符核对；4.1 是一张判断表，不是引文。凡是引文的条目都标了出处页面，方便你回原文核对——文档会更新，以线上为准。

## 怎么用这些片段

**分两类，处理方式不同：**

| 类别 | 哪些 | 怎么用 |
|---|---|---|
| **行为调校型**——官方为纠正某个具体模型行为而调过措辞 | 4.2、4.3、4.4、4.5、4.6、4.7、4.15、4.16、4.17、4.18、4.19、4.21、4.24、4.25 | **原文英文照抄，一个词都不要改**。改了就失去了它被调校出来的效果 |
| **通用策略型**——内容是策略本身，措辞没有特殊性 | 4.8、4.9、4.10、4.11、4.12、4.13、4.14、4.20、4.22、4.23 | 优先照抄；但如果原文的语境和你的任务对不上（比如 4.10 讲的是多文件代码调查，而你的任务是读材料写报告），**可以改写成贴合语境的表述**，保留其判据即可 |

**中英混排怎么处理**（最常见的情况：用户要中文 prompt，而片段是英文）：

- 英文片段**成段放置**，不要拆开插进中文句子里
- 放在它管辖的那一节内部，紧跟中文小标题之后
- 在交付说明里**明确告诉用户：这几段英文是官方原文，请不要翻译或"顺手改通顺"**——用户看到中文里夹英文，很自然会想去统一语言，一改就失效
- 如果整份 prompt 只需要一两个片段，且片段属于上表的"通用策略型"，可以直接用中文表达，避免为了两句话让整份 prompt 变成中英混杂

**中文部分是使用说明，不要放进 prompt。**

## 目录

| 编号 | 用途 | 出处 |
|---|---|---|
| 4.1 | 示例（few-shot）该不该给 | 决策表，非引用 |
| 4.2 | 控制冗长（Opus 5 必备） | [Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) |
| 4.3 | 控制 Agent 播报节奏（Opus 5 必备） | [Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) |
| 4.4 | 控制书面交付物长度 | [Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) |
| 4.5 | 约束任务范围 | [Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) |
| 4.6 | 减少过度工程（含 Fable 5 加强版） | [Prompting best practices（Fable 5 加强版出自 Prompting Claude Fable 5）](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| 4.7 | 防幻觉（代码场景） | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| 4.8 | 默认动手 / 默认不动手 | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| 4.9 | 并行工具调用 | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| 4.10 | 限制子智能体滥用 | [Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) |
| 4.11 | 安全护栏（不可逆操作） | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| 4.12 | 防止只为通过测试而写死 | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| 4.13 | 抑制 Markdown、改为流畅散文 | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| 4.14 | 关闭 LaTeX | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| 4.15 | 抑制过度思考 | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| 4.16 | 代码审查（要覆盖率而非过滤） | [Prompting Claude Opus 5 的诊断 + 官方推荐措辞](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) |
| 4.17 | 长任务进度真实性（Fable 5 强烈推荐） | [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) |
| 4.18 | 无人值守自主运行 | [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) |
| 4.19 | 上下文预算安抚 | [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) |
| 4.20 | 「给理由，不只给请求」模板 | [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) |
| 4.21 | 最终汇报的可读性 | [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) |
| 4.22 | 记忆系统的写入规范 | [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) |
| 4.23 | send_to_user 工具 | [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) |
| 4.24 | 划定行为边界 | [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) |
| 4.25 | 前端设计（避免 AI 味） | [官方模型专页的 frontend_aesthetics 片段](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |

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

**Fable 5 专用加强版**（其官方页给的版本，多了 feature flag / 向后兼容一句，专治高 effort 下的擅自整理）：

```
Don't add features, refactor, or introduce abstractions beyond what the task requires. A bug fix doesn't need surrounding cleanup and a one-shot operation usually doesn't need a helper. Don't design for hypothetical future requirements: do the simplest thing that works well. Avoid premature abstraction and half-finished implementations. Don't add error handling, fallbacks, or validation for scenarios that cannot happen. Trust internal code and framework guarantees. Only validate at system boundaries (user input, external APIs). Don't use feature flags or backwards-compatibility shims when you can just change the code.
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

> 依据：Opus 5 会**字面执行**"只报高危"、"保守一点"这类指令——它会同样深入地调查代码、同样找到 bug，然后把自己判定为低于你所设门槛的发现**不报出来**。表现为精确率上升、召回率下降。把过滤动作移到独立的下游步骤。

### 4.17 长任务进度真实性（Fable 5 强烈推荐）

Anthropic 测试中，这段几乎消除了编造的进度汇报，即便是刻意诱导它编造的任务：

```
Before reporting progress, audit each claim against a tool result from this session. Only report work you can point to evidence for; if something is not yet verified, say so explicitly. Report outcomes faithfully: if tests fail, say so with the output; if a step was skipped, say that; when something is done and verified, state it plainly without hedging.
```

### 4.18 无人值守自主运行

```
You are operating autonomously. The user is not watching in real time and cannot answer questions mid-task, so asking "Want me to…?" or "Shall I…?" will block the work. For reversible actions that follow from the original request, proceed without asking. Offering follow-ups after the task is done is fine; asking permission after already discussing with the user before doing the work is not. Before ending your turn, check your last paragraph. If it is a plan, an analysis, a question, a list of next steps, or a promise about work you have not done ("I'll…", "let me know when…"), do that work now with tool calls. End your turn only when the task is complete or you are blocked on input only the user can provide.
```

配套的"何时才该停下来问"：
```
Pause for the user only when the work genuinely requires them: a destructive or irreversible action, a real scope change, or input that only they can provide. If you hit one of these, ask and end the turn, rather than ending on a promise.
```

### 4.19 上下文预算安抚（防止提前收工）

当 harness 会向模型显示剩余 token 倒计时，模型可能提议开新会话或自行缩减工作。**首选是不要向模型暴露剩余预算数字**；必须暴露时加：

```
You have ample context remaining. Do not stop, summarize, or suggest a new session on account of context limits. Continue the work.
```

### 4.20 「给理由，不只给请求」模板

Fable 5 页给的通用请求模板，值得作为默认写法：

```
I'm working on [the larger task] for [who it's for]. They need [what the output enables]. With that in mind: [request].
```

### 4.21 最终汇报的可读性（长 agent 会话）

```
Terse shorthand is fine between tool calls (that's you thinking out loud, and brevity there is good). Your final summary is different: it's for a reader who didn't see any of that.

If you've been working for a while without the user watching (overnight, across many tool calls, since they last spoke), your final message is their first look at any of it. Write it as a re-grounding, not a continuation of your working thread: the outcome first, then the one or two things you need from them, each explained as if new. The vocabulary you built up while working is yours, not theirs; leave it behind unless you re-introduce it.

When you write the summary at the end, drop the working shorthand. Write complete sentences. Spell out terms. Don't use arrow chains, hyphen-stacked compounds, or labels you made up earlier. When you mention files, commits, flags, or other identifiers, give each one its own plain-language clause. Open with the outcome: one sentence on what happened or what you found. Then the supporting detail. If you have to choose between short and clear, choose clear.
```

### 4.22 记忆系统的写入规范

```
Store one lesson per file with a one-line summary at the top. Record corrections and confirmed approaches alike, including why they mattered. Don't save what the repo or chat history already records; update an existing note rather than creating a duplicate; delete notes that turn out to be wrong.
```

从历史会话冷启动记忆库：
```
Reflect on the previous sessions we've had together. Use subagents to identify core themes and lessons, and store them in [X]. Make sure you know to reference [X] for future use.
```

### 4.23 send_to_user 工具（长时异步 agent）

长时间异步运行的 agent 需要一个「不结束回合就把内容原样送到用户面前」的通道。工具输入永远不会被摘要，所以内容能完整抵达。

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

**只定义工具不够**——不在 system prompt 里加引导语，Fable 5 很少会调用它：
```
Between tool calls, when you have content the user must read verbatim (a partial deliverable, a direct answer to their question), call the send_to_user tool with that content. Use send_to_user only for user-facing content, not for narration or reasoning.
```

### 4.24 划定行为边界（防止未被要求的动作）

```
When the user is describing a problem, asking a question, or thinking out loud rather than requesting a change, the deliverable is your assessment. Report your findings and stop. Don't apply a fix until they ask for one. Before running a command that changes system state (restarts, deletes, config edits), check that the evidence actually supports that specific action. A signal that pattern-matches to a known failure may have a different cause.
```

> 后半句常被忽略，但对自主运行很关键：**"这个现象长得像某个已知故障"不等于"就是那个故障"**，动系统状态前要确认证据真的支持这个具体动作。

### 4.25 前端设计（避免 AI 味）

```
<frontend_aesthetics>
NEVER use generic AI-generated aesthetics like overused font families (Inter, Roboto, Arial, system fonts), cliched color schemes (particularly purple gradients on white or dark backgrounds), predictable layouts and component patterns, and cookie-cutter design that lacks context-specific character. Use unique fonts, cohesive colors and themes, and animations for effects and micro-interactions.
</frontend_aesthetics>
```

产生多样性（不要靠 temperature，用这招）：
```
Before building, propose 4 distinct visual directions tailored to this brief (each as: bg hex / accent hex / typeface, plus a one-line rationale). Ask the user to pick one, then implement only that direction.
```
