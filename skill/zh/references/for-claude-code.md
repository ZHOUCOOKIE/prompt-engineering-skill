# 场景 A：写给 Claude Code 执行的任务 prompt

用户在 Claude desktop 里说需求，你产出一段 prompt，他复制到 Claude Code（终端、VS Code 插件的图形窗口，或 Claude Desktop 应用的 Code 标签页）里执行。

**用户在 Desktop 应用里跑时**（Claude Code 的 Desktop 页）：Code 标签页只有交互式会话，没有 `claude -p` 和 `--max-turns`、`--max-budget-usd` 这类为脚本设计的参数，`dontAsk` 权限模式只在 CLI 里有，agent teams 也只在 CLI 里有（官方说 Desktop 里的多 agent 工作用 dynamic workflow）。模型和 effort 在输入框下面那一行的模型选择器里选。下文用到这些东西的建议，对 Desktop 用户换成这里能用的做法，交付说明里不要让他去敲用不了的命令。

这里的主要依据是 Claude Code 文档的 Best practices、Create custom subagents、Model configuration 和 Manage costs effectively 几页；出自别处的会在行内写明。

## 目录

- 只告诉它看不见的东西
- 第一优先：给一个它自己能跑的检查
- 这个检查「卡」得有多硬
- 一次说清楚
- 具体到什么程度
- 需求没想清楚：让它先面试用户写 SPEC
- 大改动先要计划
- 子智能体：模型、effort、并发和出错时
- 这次的 prompt vs 该沉淀进项目配置
- 确定性指令和非确定性指令
- 项目配置放哪：七种方法取舍表
- 把手工复查固化成验证回路
- 四种循环
- 无人值守运行前的五个问题
- 可以停掉的旧做法
- 会话没按预期走时

---

## 只告诉它看不见的东西

Claude 开始任务时先读代码，文件布局、约定、测试框架、现有组件怎么搭的，它自己找得到，通常比你解释更快。它找不到的是从来不在仓库里的信息：团队在会上做的决定、只在某个人脑子里的部署流程、集成测试需要的环境变量。

官方给的判据，写任何东西之前先问两句：它读代码能找到吗？它真的会去找吗？两个都是「是」，就不写。

这条同时管 prompt、CLAUDE.md 和 skill。

## 第一优先：给一个它自己能跑的检查

Claude Code 最佳实践的说法，换成对你的用户说的话：Claude 在工作看起来做完了的时候停。没有能运行的东西告诉它是不是真做完了，这就是它唯一的依据，实际上用户成了测试套件，每个错误都要等他碰巧发现。给它一个结果是 pass 或 fail 的东西，它就能自己闭合这个回路：改、跑检查、读输出、修、再跑。

检查可以是任何 Claude 读得懂结果的东西：一个测试文件、构建的退出码、linter、把输出和 fixture 做 diff 的脚本、摆在设计稿旁边的浏览器截图。仓库里已经有其中一样的话，CLAUDE.md 里最有用的一行就是跑它的命令。

还要让它**给出证据**：跑了什么命令、输出是什么，或者截图。官方原话：`"All tests pass" on its own tells you very little`；读输出比自己重跑一遍快，对用户没盯着的会话来说，这也是唯一的证据。

Claude 说做完之后，还可以在会话里跑 `/verify`：它会构建并运行应用、看结果，能抓到测试和类型检查漏掉的那类问题。

## 这个检查「卡」得有多硬

官方给了四档，从 Claude 自己判断到 Claude Code 强制执行：

| 档位 | 是什么 | 官方的说法 |
|---|---|---|
| **写在 prompt 里** | `run the auth tests after, and keep going until they pass` | Claude 自己判断什么时候满意。任何任务都能用，不需要配置 |
| **设成 goal** | `/goal all tests in tests/auth pass and lint is clean`：每轮之后由另一个模型对照会话检查条件。评估器只看对话里出现的内容，不自己跑命令或读文件，所以条件要写成 Claude 自己的输出能证明的东西；goal 不改变权限模式，要让它无人值守地跑，在 auto mode 里用 | 在会话里自己盯着时很少需要。它的用处在长时间的后台运行：Claude 有时在还有未完成项时以进度总结结束回合，goal 会把它送回去 |
| **`Stop` hook** | Claude 想结束回合时跑你的脚本，脚本以 exit code 2 退出就把它送回去干活 | 由 Claude Code 强制执行。连续拦截八次且中间没有工具调用时，Claude Code 会放行下一次；用 `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP` 调这个上限 |
| **换个没写这段代码的来评** | `/code-review` 在一个没看过你这场对话的全新子智能体里审查 diff | 它不受产生这段代码的那套推理影响 |

用户说的是没人盯着的长时间运行时，在交付说明里告诉他官方的说法：这种运行靠 `/goal` 或 `Stop` hook 来强制。

想让审查对照计划而不是泛泛地找 bug，就自己写审查 prompt。官方例子：

```text
use a subagent to review the rate limiter diff against SPEC.md, check every requirement is implemented and the listed edge cases have tests, report gaps that affect correctness and skip style
```

官方特别提醒不要去掉 `skip style`：`A reviewer that's asked to find problems will always find something.` 每条发现都改，最后会多出一堆抽象层和给不可能发生的情况写的测试。官方这个例子里，子智能体审查完直接向你汇报。片段 4.16 出自 Sonnet 5 页面的 "Code review harnesses" 一节，做法相反：让审查者全报，把按置信度和严重度的过滤移出找问题这一步；该页说后面没有真正的第二步也能用，想让模型在一遍里自己过滤，就把报告门槛写具体。Opus 5 页面也提醒过：审查 prompt 设了严重度门槛时，模型 `might follow that instruction literally and report less`（在 Opus 5 上测得，Opus 5.5 的页面既没重述也没撤回，见 `models.md`）。

---

## 一次说清楚

Claude Code 最佳实践的说法：说出文件名、描述症状、事先讲清修好了是什么样，`you usually save yourself a round of corrections later`。在两个合理做法之间有偏好时也一样——Claude 没法从代码里读出你的偏好，在意就事先说。

两个前代页面讲得更直接，它们的后继页面都写明前代的做法仍是合理起点：Opus 5 专页 `performs best when given the complete task specification up front and left to run`；Sonnet 5 专页 `Providing well-specified, clear, and accurate task descriptions upfront can help maximize autonomy and intelligence while minimizing extra token usage after user turns`，反过来，`ambiguous or underspecified prompts conveyed progressively over multiple user turns tend to relatively reduce token efficiency and sometimes performance`。

Opus 5.5 自己也有一篇这样说的官方文章（Anthropic 2026-09-22「Getting the most out of Opus 5.5 in Claude and Claude Code」）：把整个任务放在一条消息里，说出终点线，例如 "the tests pass" 或 "every endpoint is migrated"。文章还给了一条写进 CLAUDE.md 的规则，用在 Opus 5.5 的长任务反复停下来汇报、问要不要继续的时候，并说明要按自己的项目改写：

```text
When a step doesn't need my input, keep going. Put status notes in the same message as your next action.
Stop and ask only when you can't continue without me, or before anything destructive: deleting data, force-pushing, or changing anything outside this repository.
```

文章的配套说明：停下来问要不要继续时，回一句 continue；经常发生的话，这条规则有帮助。规则的最后一行保留了有风险或难撤销的操作之前的确认，破坏性命令的权限确认也要留着。结对编程时可以反过来，要它开始前给一行计划、结束时给一个简短回顾，同样写进 CLAUDE.md。

同一篇文章给 Opus 5.5 长任务的另外两条。一是让它把任务清单记在文件里、边做边更新，文章的写法是 `Keep a checklist in TASKS.md. Tick each item when it's done, and add anything new you find.`；理由是长任务会填满上下文窗口，Claude Code 随后会摘要更早的轮次，而文件里的清单不受影响。二是大范围的审计、迁移或审查，让它拆给子智能体并检查每个结果，文章的例子：

```text
Audit every service in services/ for the retry bug in the linked issue.
Give each service to its own subagent. When a subagent reports back, check its evidence before you accept it.
Finish with one table: service, affected yes or no, and the evidence.
```

让 Fable 5.2 继续干活或批准某一步时，同样要说清：Fable 5.2 专页要求说明你要它做什么，而不是只发一句 continue；它的例子是在原本会回 "go" 或点个赞的地方，回 `Yes, run the migration on the staging database`。

---

## 具体到什么程度

官方对照（Claude Code 最佳实践）：

| 不这样写 | 这样写 |
|---|---|
| `fix the login bug` | `users report login fails after the session times out. look at token refresh in src/auth/, write a failing test that reproduces it, then fix it` |
| `add a calendar widget` | `follow the pattern in HotDogWidget.php and build a calendar widget that lets you pick a month and page between years. no new libraries` |

**手上已经有它需要的东西，就直接给，不要描述**：`@` 加路径把文件内容带进消息；截图直接粘进 prompt；命令输出从 shell 管道进来，例如 `cat error.log | claude -p "what's going on here"`。支持站的 Claude Code 指南补了一条和用量有关的提醒：`@` 前缀会把整个文件连同它的 CLAUDE.md 树注入上下文，只写路径则让 Claude 自己打开、有选择地读；日志和堆栈先裁到相关的二三十行再贴。

**三条关于措辞的官方说明**：

- `think step by step`、`be thorough` 这类话是普通的 prompt 文字，不是关键词。控制它想多深的主要是 effort；这类措辞只能在当前档位之内推一把
- 仍然管用的一句是 `be ambitious`：Claude 倾向于保守地读请求，做出满足它的最小东西。想要更完整的版本就说出来，这是在告诉它**范围**，而范围只有你知道。官方例子：

```text
build the settings page from the mockup. be ambitious: empty and error states, the save flow wired up, keyboard navigation, not just the layout
```

- 在探索而不是修某个具体问题时，`what would you improve in this file?` 这种模糊 prompt 完全可以，代价是途中多纠正几次

## 需求没想清楚：让它先面试用户写 SPEC

用户自己也没完全想清楚时，不必先想完再写 prompt。官方例子：

```text
I want to add rate limiting to the public API but haven't thought it through yet. Interview me until you could write the spec, then write it to SPEC.md.
```

Claude 会就没定下来的部分提多选题，一直问到答案够写成 spec。`SPEC.md` 有了之后，**开一个全新会话**照它来做：上下文干净，而且有一份写下来的完成定义。之后做审查时指给审查者看的也是这个文件。

## 大改动先要计划

支持站的 Claude Code 指南：凡是会动两三个以上文件的改动，切到 Plan Mode，或者直接问 `Before changing anything, list the files you'll touch and what you'll do in each.`，看完清单用大白话纠正，再让它执行。理由是成本：一份计划几百 token，一个改错了要回滚重来的 400 行 diff 是几千 token 的两倍，还要加上解释哪里错了的那几轮。Claude Code 成本页的说法相同：复杂任务先用 plan mode，方向错了的返工很贵。

同一份指南给的模型搭配：用 Opus 写计划、用 Sonnet 执行，内置为 `/model opusplan`。Anthropic 的文章「What a task costs on Opus 5.5」（2026-09-25，写于 Sonnet 5.5 发布之前）提醒：`opusplan` 把代码编辑放到了 Sonnet 上，和把代码编辑留在主模型的建议相反，`Measure it on your own tasks before you make it a default`。Claude Code 的 prompt caching 页另写明，`opusplan` 下每次进出 plan mode 都是一次换模型，缓存从头开始。

---

## 子智能体：模型、effort、并发和出错时

选哪个模型见 SKILL.md 第 2 节。这里是 Claude Code 文档写明的机制，写 prompt 时照这些来。

**模型按这个顺序定**（子智能体页 "Choose a model"）：

1. 这次调用传的 `model` 参数
2. 子智能体定义里的 `model` 字段，`inherit` 表示用主会话的模型
3. `CLAUDE_CODE_SUBAGENT_MODEL` 环境变量
4. 主会话的模型

要点：

- Agent 工具每次调用的 `model` 只接受别名：`sonnet`、`opus`、`haiku`、`fable`，不能传完整模型 ID 或 `inherit`。定义文件里的 `model` 字段可以写别名、完整 ID 或 `inherit`
- 每次调用传的或定义 `model` 字段里的系列别名（如 `opus`），在主会话模型属于同一个系列时，解析成主会话那个确切的模型。`CLAUDE_CODE_SUBAGENT_MODEL` 里的别名总是解析成别名指向的版本
- 别名指向哪个版本看 provider（Model configuration 页的表）：Anthropic API 上 `opus`、`sonnet`、`haiku` 是 Opus 5.5、Sonnet 5.5、Haiku 5.5；Claude Platform on AWS 上是 Opus 5.5、Sonnet 4.6、Haiku 4.5；Amazon Bedrock 和 Google Cloud 的 Agent Platform 上是 Opus 5.5、Sonnet 4.5、Haiku 4.5；Microsoft Foundry 上是 Opus 4.6、Sonnet 4.5、Haiku 4.5。没设 `ANTHROPIC_DEFAULT_FABLE_MODEL` 时，`fable` 别名在 Claude apps gateway 会话里指向 Fable 5，其他地方指向 Fable 5.2
- 每次调用传的 `model` 在这个子智能体被续跑或收到后续消息时仍然生效
- 这个顺序从 v2.1.251 起成立，更早的版本里环境变量排在最前。三种情况下每次调用传的 `model` 不算数：开了 `CLAUDE_CODE_SUBAGENT_MODEL_FORCE` 时，Claude 启动子智能体时不能传模型，定义里的 `model` 字段也被忽略；组织的 `availableModels` 白名单拦下的值会被换成别的模型；装了的 mod 在 `agent.spawn` hook 里设了模型时，用 mod 设的那个
- 用 `/model` 切换主会话模型，也会影响之后启动的、继承主会话模型的子智能体
- 想知道某个子智能体实际跑在哪个模型上，用 `/tasks`：它在子智能体那一行显示模型，设了 effort 的话也显示
- **fork 不走这个顺序**。fork 继承整个对话，跑在主会话的模型上，和主会话共用 prompt cache；官方说，需要同样上下文的任务上它比全新的子智能体便宜。fork 模式默认开着（v2.1.302 起；交互式终端会话从 v2.1.232 起），开着时 Claude 可以自己把侧任务 fork 出去。所以主会话在 Fable 上时，fork 出去的任务也跑在 Fable 上。不想让 Claude 自己 fork：在 `permissions.deny` 里加 `"Agent(fork)"`，或把 `CLAUDE_CODE_FORK_SUBAGENT` 设为 `0`

**内置子智能体用什么模型**：

| 子智能体 | 模型 |
|---|---|
| Explore | 主会话的模型。**主会话是 Fable 时**：用订阅、Console 账号或 `ANTHROPIC_BASE_URL` 网关连接的，跑在 `opus` 别名指向的模型上；在 Amazon Bedrock、Google Cloud、Microsoft Foundry、Claude Platform on AWS 或 Claude apps gateway 上，留在主会话的模型 |
| Plan | 继承主会话 |
| general-purpose | 设了 `CLAUDE_CODE_SUBAGENT_MODEL` 且没有别的来源指定模型时用它，否则是主会话的模型 |
| statusline-setup / claude-code-guide | Sonnet / Haiku |
| claude | 自己不指定模型；Claude 把它当子智能体派出时走上面的顺序 |

只设 `CLAUDE_CODE_SUBAGENT_MODEL` 不会改变 Explore 和 Plan 的模型。想把 Explore 放到更便宜的模型上，官方的办法是定义一个同名的 `Explore` 子智能体并写 `model: haiku`。

**effort**（子智能体页 "Choose an effort level"）：

- 定义文件里可以设 `effort` 字段（`low`、`medium`、`high`、`xhigh`、`max`，可用档位取决于模型）
- 要求 Claude 以某个档位跑非 fork 的子智能体时，它可以在那次调用上传 `effort` 参数，覆盖定义里的字段（需要 Claude Code 2.1.292 及以上）。`CLAUDE_CODE_EFFORT_LEVEL` 环境变量优先于这两者
- 都没设时：和主会话同一个模型的子智能体用会话的档位；在别的模型上，会话的档位不适用，改用那个模型保存的档位或它的默认档（changelog 2.1.317）。`CLAUDE_CODE_SUBAGENT_EFFORT` 可以给没有别处指定档位的子智能体统一设一个（只见于 changelog 2.1.316，环境变量参考页上没有）

**定义文件改了什么时候生效**：Claude Code 监视 `~/.claude/agents/` 和 `.claude/agents/`，原文是 `Claude Code detects the change within a few seconds and the next delegation uses the updated definition, with no restart needed`。三种情况仍要重启：会话开始后才新建的 `agents` 目录里的第一个文件；用 `--add-dir` 或 `/add-dir` 加进来的目录里的 `.claude/agents/`；用 `--disable-slash-commands` 启动的会话。

**并发**（子智能体页 "Concurrent subagent limit"）：默认一个会话里同时在跑的子智能体到 20 个时，再用 Agent 工具派一个会失败并返回 `Concurrent subagent limit reached`，在跑的数量降下来后才能继续派。用 `CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS` 改这个上限（任意正整数，只能调不能关）。开了 ultracode 的会话不受这个上限约束。一个会话总共能派多少个没有上限。嵌套深度默认三层，用 `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH` 改。

官方没有给「该同时跑几个」的建议数字。它给的是规模上的分工：子智能体适合 `A few delegated tasks per turn`；要协调几十到几百个 agent 时用 dynamic workflow，它有自己的上限（默认同时 16 个，每次运行最多 1,000 个），并且计划安排的 agent 超过 25 个或预计 token 超过 150 万时会显示 `Large workflow` 警告。同时 16 个是默认值，Claude Code 可用的 CPU 少时更少，用 `CLAUDE_CODE_WORKFLOW_MAX_CONCURRENT_AGENTS` 改（1 到 256）。

workflow 里每个 agent 的模型按和子智能体相同的顺序定：脚本给某个阶段写的模型算作「这次调用传的」那一项，都没指定时跑在会话的模型上。Dynamic workflows 页给的控制模型成本的两条：平时会切到小模型做例行工作的，大规模运行前先看一眼 `/model`；描述任务时让 Claude 给不需要最强模型的阶段用小一点的模型。想在大任务前估一下花费，先在一小片上跑，例如一个目录而不是整个仓库。同一页还写明：开着 ultracode 的会话里每个请求用的 token 更多，在订阅上会更早到会话上限或每周上限。

**怎么让一条 prompt 用上 workflow**（同一页）：在 prompt 里带上关键词 `ultracode`，或者直接写 "use a workflow"、"run a workflow"，Claude 把这种直接的要求当作同样的许可。关键词只在用户自己输入的 prompt 里算数，用 `-p` 传入的 prompt、定时任务的 prompt 不会因此启动 workflow。size guideline 是给 Claude 的建议而不是上限，prompt 要求别的规模时以 prompt 为准。该页的示例 prompt 之一：

```text
use a workflow to audit every route handler under src/routes/ for missing authentication checks, and adversarially verify each finding before reporting it
```

Claude Code 最佳实践另给了两种批量做法。一个大的机械改动要跨很多文件时用 `/batch`，它的例子是 `/batch migrate every handler to the v2 response helpers`：把工作拆成 5 到 30 个单元并先给你看计划，批准后每个单元在自己的 worktree 里做，开 PR 或汇报结果。用脚本循环调 `claude -p` 时：`Try it on three files first, see what goes wrong, fix the prompt, then run the rest.` 脚本里加了 `--bare` 时（Headless 页说它是脚本调用的推荐模式，以后会成为 `-p` 的默认），Claude Code 跳过对 CLAUDE.md、hooks、skills、自定义子智能体、插件和 MCP server 的自动发现，要用的得在命令行上显式传进去，hooks 在 bare 模式下始终不运行；也不能跑后台子智能体。所以这种 prompt 要自带检查命令和必须遵守的规则。

**结果怎么回来**：主会话派出的子智能体默认在后台跑，结果在之后某一轮以完成通知的形式到达。fork 模式开着时（默认），在本机的交互式终端、VS Code 和桌面应用会话里 Claude 不能等一个子智能体跑完再继续；要让它等，先关掉 fork 模式（`CLAUDE_CODE_FORK_SUBAGENT=0`）。子智能体完成时返回的结果会进主会话的上下文，官方的提醒：`Running many subagents that each return detailed results can consume significant context, and each subagent spends tokens of its own while it runs.` 所以交代里要它们回报简短摘要。

**交代里要带什么**：非 fork 的子智能体从全新的上下文开始，看不到对话历史、已经调用过的 skill 和 Claude 读过的文件；它会加载 CLAUDE.md；内置的 Explore 和 Plan 不加载主会话那几层，定义里设了 `omitClaudeMd` 的子智能体只加载 managed policy 那一层（定义本身来自 managed settings 时一层都不加载）。这几种情况下，处理到某个子目录里的文件时，那里的 CLAUDE.md 仍会加载。子智能体页的说法：主会话读这些子智能体的结果时仍有完整的 CLAUDE.md，所以大多数规则不必传到子智能体；必须传到的规则（它举的例子是忽略 `vendor/` 目录），在让 Claude 委派的那条 prompt 里重述。Agent SDK 的子智能体页：`The only content you pass from parent to subagent is the Agent tool's prompt string, so include any file paths, error messages, or decisions the subagent needs directly in that prompt.`

**出错和撞到用量上限时**（子智能体页 "API errors in subagents"）：子智能体因 API 错误结束——官方举的例子是 usage limit 或反复的服务器错误——会把失败回报给 Claude。前台子智能体如果已经产出了文字，Agent 工具返回这段部分输出，并注明它被打断、没做完；什么都没产出、或只产出了工具调用的，以 `Agent terminated early due to an API error` 失败。后台子智能体标记为失败，Claude 收到的消息里写明 API 错误并带上它最后的输出，`so partial work isn't lost`。错误消除后，让 Claude 重试，或用 `SendMessage` 续跑那个子智能体——续跑的子智能体保留完整的对话历史。Explore 和 Plan 是一次性的，不能续跑。子智能体的 transcript 独立于主会话保存，重启 Claude Code 后恢复同一个会话仍可续跑。

**子智能体中途换了模型**：回报以 `⚠ This agent switched to <model> because its earlier model could not be used.` 开头，说明它是在 fallback 模型上跑完的（配置了 fallback model chain、且遇到链条覆盖的失败时会这样，例如它的模型不可用）。请求被安全分类器标记时，子智能体不会停下来问。有 fallback 模型可用时：`/config` 里选的是自动切换、或还没选过，这个请求在 fallback 模型上重跑，主会话的模型不变；选的是每次询问，子智能体以 `Agent terminated early due to an API error` 失败并带上拒绝信息。没有 fallback 模型可用时，两种选择下子智能体都以同样的方式失败。从 2.1.319 起，安全措施拦下一个子智能体之后，Claude 被告知的做法是用更窄的 prompt 重试或者告诉你，而不是自己去做那件事（changelog）。

dynamic workflow 不一样：其中的 agent 撞到 claude.ai 用量上限时，运行暂停而不是让那个 agent 失败：撞到上限的 agent 等待重置，不再启动新的 agent，重置后运行自己继续（条件见 `models.md`「到了上限会怎样」）。

**什么时候派，官方怎么说**（Claude Code 最佳实践 "Know what subagents cost"）。值得的四种情况：同时跑彼此独立的工作；把长输出（完整的测试运行、大范围搜索）挡在主会话外；要一个没看过代码怎么写出来的 reviewer；给某个任务比主会话更少的工具。代价主要是 token，有时还有时间：全新的子智能体要先收集主会话已经有的上下文。同一页的话：`by default it's the same model (or a smaller one) on the same task. You get parallelism or a fresh pair of eyes out of it, but not a smarter answer.`

缓存也算在代价里：prompt cache 在订阅上大约保持一小时，在 API key、云 provider 或 usage credits 上是五分钟，而 `Subagents get five minutes unless you change it`。workflow 的 agent 同样默认五分钟，订阅上也是；把 `subagentPromptCacheTtl` 设成 `1h` 可以保持一小时，1 小时的缓存写入按更高的费率计。

Fable 5.2 专页对 prompt 的要求：system prompt、skills、agent 文件里叫它委派的句子先查一遍；派得比任务需要的多，就删掉不再需要的。

**写子智能体定义时**（子智能体页）：Claude 靠每个子智能体的 `description` 决定什么时候委派，所以要写清它该在什么时候用，并且具体到能和别的子智能体区分开。description 要短：自定义子智能体的 description 合计超过 15,000 token 时，Claude Code 启动时会给警告；细节挪进子智能体自己的 system prompt，它只在那个子智能体运行时才加载。同一页说，在 description 里写 "use proactively" 这类话会鼓励 Claude 主动委派；目标是 Fable 5.2 时把它和上一段放在一起看，Fable 5.2 专页写的是 prompt 鼓励委派的编码产品里它派得很痛快。要保证某个子智能体在这次任务里一定运行，用 @ 点名它；只在话里提到它的名字，用不用仍由 Claude 决定。

---

## 这次的 prompt vs 该沉淀进项目配置

判断很简单：**这次任务特有的信息写进 prompt；每次都成立的约定沉淀进项目配置。**

如果你发现自己在 prompt 里写"我们项目用 pnpm 不用 npm"、"提交前要跑 lint"这类**每次都要说一遍**的话，就该建议用户把它放进 CLAUDE.md 或 hook，而不是每次复制粘贴。

## 确定性指令和非确定性指令

Claude Code 最佳实践把每条指令分成两种：非确定性的由模型读了之后自己决定怎么办（prompt、CLAUDE.md、skill），通常会照做，但没有保证；确定性的由 Claude Code 这个 harness 执行（hook、deny 规则、managed setting），不管模型怎么决定都会生效。

| | 非确定性（Claude 决定） | 确定性（Claude Code 强制） |
|---|---|---|
| 做什么 | prompt、CLAUDE.md、Claude 自己会用的 skill | hook、只有你能启动的 skill |
| 能碰什么 | auto mode 的分类器 | deny 规则、strict mode 的沙箱、managed settings |
| 什么时候算完 | 要证据、`/goal`、`/code-review` | 测试、类型检查、`Stop` hook、CI |
| 花多少 | effort 档位、Claude 自己的节奏 | `-p` 下的 `--max-turns` 或 `--max-budget-usd`、你设的 spend limit |
| 谁说了算 | Claude 对自己做了什么的总结 | 分支保护、批准合并的人 |

官方的取舍：大多数时候让 Claude 决定——几乎每条规范都有例外，眼前有实际代码时它通常比你事先写的规则判断得更好。确定性那一种留给**零例外**的事：`a hook is for a rule with zero exceptions`，有例外的偏好（官方例子 `Prefer named exports`）留在 CLAUDE.md。反过来，如果你给只是偏好的东西配了 hook，删掉，让 Claude 决定。

---

## 项目配置放哪：七种方法取舍表

Anthropic 的「Steering Claude Code」一文列了七种方法；这里 CLAUDE.md 分成根目录和子目录两行。

| 手段 | 加载时机 | 上下文成本 | 适合放什么 |
|---|---|---|---|
| **CLAUDE.md（根）** | 会话开始，全程驻留，compaction 后重新读入 | **高**（始终加载） | 确认工作用的命令（测试、类型检查、lint、怎么只跑一个测试文件）、代码里有误导性的地方、不在仓库里的决定、已经纠正过两次的事。**控制在 200 行以内** |
| **CLAUDE.md（子目录）** | 触及该目录时按需加载 | 低 | 目录级约定 |
| **Rules** | 会话开始（无 scope）或访问文件时（有 scope） | 中 | 只在代码库某一部分才要紧的规则。**加 `paths:` 过滤**，只在 Claude 真的在那里干活时加载 |
| **Skills** | 名称加描述在开头，正文在调用时 | 低 | 团队的流程和知识：发版流程、事故手册、API 文档怎么写。有副作用的流程加 `disable-model-invocation: true`，只在你自己输入命令时才跑 |
| **Subagents** | 名称加描述在开头，正文调用时加载，只返回最终消息 | 对主上下文低；它自己的请求照样计用量 | 只需要结论的侧任务 |
| **Hooks** | 生命周期事件触发 | 低（在主上下文之外） | 零例外的规则和确定性自动化：格式化、阻断对某个目录的编辑。要真正强制，用 `command` 或 `http` 类型的 hook；多数事件上，hook 自己失败或超时时仍会放行，想在这种时候也拦住就设 `onFailure: "block"`（2.1.295 起）。用来执行策略的 hook 以退出码 2 退出：退出码 1 又没有有效 JSON 输出，算非阻断错误 |
| **Output styles** | 会话开始，compaction 后仍然生效 | **高** | 每次回复都要的语气、长度或格式，或者让 Claude 以软件工程师之外的角色工作 |
| **追加 system prompt** | 以 CLI 参数在启动时传入 | 中 | 脚本和自动化里需要放在 system prompt 层的指令（`--append-system-prompt`） |

**CLAUDE.md 写什么**（最佳实践 "Keep CLAUDE.md short"）：怎么确认这个仓库里的工作——确切的测试、类型检查、lint 命令，以及怎么只跑一个测试文件；代码有误导性的地方——到处都是但新代码不许照抄的旧模式、看着没用其实在用的目录；不在仓库里的决定；已经不得不纠正过两次的事。

**怎么写**（Memory 页）：Claude 把 CLAUDE.md 当作上下文，不是强制执行的配置，所以怎么写决定了它被遵守得多可靠。指令要具体到能验证：写 "Use 2-space indentation"，不写 "Format code properly"。两条指令互相矛盾时，Claude 可能随便挑一条，所以定期清掉过时的和冲突的。CLAUDE.md 的内容是作为 system prompt 之后的一条 user 消息送达的，不是 system prompt 的一部分。

**不写什么**：Claude 读代码就能得出的东西。官方把在 CLAUDE.md 里描述架构和文件布局列进了可以停掉的旧做法。个人偏好放 `CLAUDE.local.md` 并加进 `.gitignore`。仓库用 `AGENTS.md` 当项目指令时要留意（Memory 页）：工作目录或上层只要有 `CLAUDE.md` 或 `CLAUDE.local.md`，Claude 默认就只读 CLAUDE.md 这些文件，不再读 `AGENTS.md`；要两者都生效，在 CLAUDE.md 里导入它。

**某一行老被忽略时**：加 IMPORTANT 多半没用（SKILL.md 3.3）。要么文件太长、那一行被淹没了，缩短文件；要么它是不能有例外的规则，做成 hook。

**工具**：`/init` 从代码库生成一个起始版本，之后主要是删行；`/doctor` 建议删掉 Claude 能从代码里得出的内容；`/doctor prompt-audit` 标出为旧模型写的、指向已不存在的文件的、或互相矛盾的指令；会话里 `/context all` 确认文件加载了。

**Skills 怎么写**（三层渐进式披露，出自 Anthropic 的 Agent Skills 一文）：

| 层 | 内容 | 何时加载 |
|---|---|---|
| 元数据 | YAML frontmatter，只有 `name` 和 `description` | 启动时预载入系统提示词 |
| 主指令 | `SKILL.md` 正文 | Claude 判断相关时 |
| 附加文件 | 从 SKILL.md 引用的其他文件 | 只在真正需要那个场景时 |

CLAUDE.md 的那条规则对 skill 同样适用：只写 Claude 自己找不到的。Skills 页对正文的写法是 `State what to do rather than narrating how or why`，理由是 skill 加载后一直留在上下文里，每一行都是反复付出的 token；这和给 prompt 里的约束附上原因（SKILL.md 3.5）方向不同，写 skill 正文时按 Skills 页。迁移到 Fable 5.2 时，skills 和 agent 文件也在 Fable 5.2 专页要求复查的范围里（SKILL.md 3.13）。

Claude Code 文档里两条和截断有关的写法提示：skill 列表里 `description` 和 `when_to_use` 合计超过 1,536 个字符的部分会被截掉，所以把最主要的使用场景写在最前（Skills 页）；compaction 之后重新注入的 skill 正文，大的会按每个 skill 的上限截断并保留文件开头，所以最重要的指令放在 `SKILL.md` 靠前的位置（Context window 页）。

---

## 把手工复查固化成验证回路

出自 Anthropic 的「Building verification loops in Claude Code with skills」一文。判据：任何用户不得不一遍遍手动执行的检查，都是固化成验证回路的候选，尤其是通用 linter 抓不到的确定性规则，文中的例子是 `Reject any migration that drops a column without a backfill step`。

| 模式 | 触发 | 适合 |
|---|---|---|
| **Standalone 独立** | 手动调用 | 不普遍适用的横切检查：安全扫描、无障碍审计、许可证核查 |
| **Embedded 内嵌** | 在产出 skill 的流程里自动跑 | 只属于某个特定任务的检查 |
| **Chained 串联** | 一个 skill 完成后调下一个 | 端到端交接；也是给**不可修改的 skill** 加验证的办法 |
| **On every PR** | CI 上跑 | 把个人习惯变成团队基础设施，不再依赖作者自觉 |

内嵌只对可编辑的 skill 有效。内置或插件管理的 skill 会自动更新，改动会被覆盖，那种情况用串联。

## 四种循环

出自 Anthropic 的「Getting started with loops」一文。

| 类型 | 命令 | 适合 |
|---|---|---|
| **Turn-based** | 手动逐轮 | 短任务 |
| **Goal-based** | `/goal get the homepage Lighthouse score to 90 or above, stop after 5 tries` | 有**可验证退出标准**、单轮不够的任务 |
| **Time-based** | `/loop 5m check my PR, address review comments, and fix failing CI` | 周期性工作、对接外部系统 |
| **Proactive** | `/schedule every hour: check #project-feedback for bug reports` | 定义明确的持续工作流：bug 报告、issue 分诊、迁移 |

控成本：确定性的工作用脚本做，周期匹配实际变化频率，大规模跑之前先试点。文中原话：`Your model and effort level choices are among the biggest levers on what a loop costs.` 它给 proactive 循环写的省用量办法是 `Routing routines to smaller, faster models and using the most capable model for judgment calls`。Claude Code 最佳实践补充：`/loop` 的每次运行都是同一个会话里的又一轮，所以不要让它在一个很长的对话里一直跑。

## 无人值守运行前的五个问题

Claude Code 最佳实践 "Let Claude run without you"。让一个任务自己跑之前，针对这个具体任务过一遍：

1. 它知道做完是什么样吗？一个 goal 条件、一份 spec 文件、或一个它得跑绿的失败测试
2. 它能检查自己的工作吗？测试套件、`Stop` hook、某种审查步骤
3. 它被允许碰什么？权限规则、`dontAsk`、沙箱、worktree、它可以推的那一个分支
4. 什么能让它停？`--max-turns`、`--max-budget-usd`、spend limit
5. 结果去哪儿、谁签字？一个 PR、一条评论或一份总结，然后由人批准合并、部署或其他收不回来的事

后台会话（从 agent view、`claude --bg` 或 `/bg` 启动）在 Claude 自己进入的 worktree 里改了代码时，默认会不经询问地提交，并在仓库有 remote 时推送分支，任务需要时开 draft PR；它不会推 `main` 或 `master`、不 force-push、不 merge。任务、CLAUDE.md 或 memory 里写了由你自己处理提交和推送时，以你写的为准；会话是在已有的 worktree 里启动的、或没有做隔离时，提交前仍会询问（Agent view 页）。所以给这种会话写的 prompt 要写明 git 的结束状态。

官方的理由：你盯着的时候，Claude 缺一点上下文不要紧，一两条消息内你就会发现并告诉它；睡前启动的运行里没人告诉它，第二天早上才发现它一整夜都在错误的假设上往下做。

## 可以停掉的旧做法

Claude Code 最佳实践 "What you can stop doing"。审查用户给的 Claude Code prompt 或 CLAUDE.md 时照这张表查：

| 旧做法 | 官方现在的说法 |
|---|---|
| 把上下文窗口当主要约束、激进地管理 | 多数现行模型有 1M token 窗口，compaction 自动发生，很少会用完。任务之间 `/clear` 仍然值得做：上下文聚焦，答案更好、花得更少 |
| 给老被漏掉的指令加 IMPORTANT 或全大写 | 现行模型照平实的一句话就会做，堆强调多半让它把规则用过头。不能有例外的做成 hook 或 deny 规则；被长文件淹没的，缩短文件 |
| 用子智能体保护上下文窗口 | 用完上下文很少再是问题。子智能体仍适合并行、挡住嘈杂输出、独立审查，但每一样都多花 token |
| 派给子智能体以求更好的答案 | 把同一个任务交给子智能体不会让答案更好。全新的 reviewer 有帮助，是因为它没看过你的推理，那是另一回事 |
| prompt 里写 `think hard`、`think harder`、`be thorough` | 那是 effort 管的事。这些是普通的 prompt 文字。`ultrathink` 仍然有效：它只对那一轮要更深的推理 |
| 叫它继续、循环到做完、别偷懒 | 现行模型通常不用说就会把说定的任务做下去，这类话更可能造成做过头。给它一个要通过的检查是另一回事，仍然值得。真的停早了就说 `continue`；没人盯着的运行用 `/goal` 或 `Stop` hook 强制。目标是 Fable 5.2 时另看 Fable 5.2 专页的要求：叫它继续时说清要它做什么，不要只发一句 continue（见「一次说清楚」） |
| 在 CLAUDE.md 里描述架构和文件布局 | Claude 自己会读仓库。只写不在里面的 |

## 会话没按预期走时

Claude Code 最佳实践开头的诊断，用户反馈会话没按预期走时用：

| 现象 | 多半是什么原因 | 去看哪一节 |
|---|---|---|
| 做出来的东西和你想的不一样，或者每个会话都要重新发现同一个事实 | 它缺少读仓库得不到的信息 | 只告诉它看不见的东西 |
| 它说做完了，其实没完全做完 | 它没有办法检查自己的工作 | 给一个它自己能跑的检查 |
| 它做了这个仓库里绝不能发生的事，或跳过了必须做的一步 | 光靠 prompt 保证不了 | 确定性指令 |
| 做对了，但花的时间、token 或权限确认比任务值得的多 | 某个默认值不合适 | `/usage` 看花在哪；SKILL.md 第 2 节 |

会话里的三个控制（"Steer the session"）：`Esc` 在动作中途停下并保留到那一刻为止的一切；`/rewind` 把代码、对话或两者回滚到之前的某条 prompt；同一件事纠正了两次还不对，就 `/clear`，把学到的并进一条新 prompt——第三次纠正多半只是往一段已经含有两次失败尝试的对话里再加东西。
