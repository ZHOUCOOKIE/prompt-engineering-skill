# 场景 A：写给 Claude Code 执行的任务 prompt

用户在 Claude desktop 里说需求，你产出一段 prompt，他复制到 Claude Code（含 VS Code 插件的图形窗口）里执行。

## 目录

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
- 项目配置放哪：七种方法取舍表
- 把手工复查固化成验证回路
- 四种循环
- 五种常见失败模式

---

## 第一优先：给一个它自己能跑的检查

这是官方最佳实践里排第一的一条，也是这个场景和场景 B 最大的区别。

官方原文的第二人称说的是读者，在这里指的是**你的用户**而不是你，所以下面这段是改写而非原文：

> Claude 在"看起来做完了"的时候就会停。如果没有它能运行的检查，"看起来做完了"就是唯一的信号，而**用户自己就变成了验证回路**——每个错误都要等他去发现。给它一个能产出 pass/fail 的东西，这个回路就自己闭合了。

检查可以是任何能返回模型读得懂的信号的东西：测试套件、构建的退出码、linter、把输出和 fixture 做 diff 的脚本、和设计稿对比的浏览器截图。

**写 prompt 时的对照**：

| 差 | 好 |
|---|---|
| `implement a function that validates email addresses` | `write a validateEmail function. example test cases: user@example.com is true, invalid is false, user@.com is false. run the tests after implementing` |
| `make the dashboard look better` | `[paste screenshot] implement this design. take a screenshot of the result and compare it to the original. list differences and fix them` |
| `the build is failing` | `the build fails with this error: [paste error]. fix it and verify the build succeeds. address the root cause, don't suppress the error` |

> **注意与 SKILL.md 3.4 的区别**：3.4 说的是删掉"让模型自己再看一遍"这类主观复查指令。这里给的是**外部确定性检查**——跑测试、看退出码、对比截图。前者要删，后者必须给。

还要求它**给出证据**而不是断言成功：测试输出、跑了什么命令返回了什么、结果截图。审阅证据比自己重跑一遍快，而且对用户没盯着的会话同样有效。

## 这个检查「卡」得有多硬

检查存在之后还有第二个决定，而这个决定光靠 prompt 做不到。官方给了四档，用配置成本换这趟运行需要多少人盯着：

| 档位 | 是什么 | 什么时候值得配 |
|---|---|---|
| **写在一条 prompt 里** | 在同一条消息里要求它跑检查并迭代 | 任何任务，立刻可用。你正在写的这条 prompt 就是这一档 |
| **贯穿整个会话** | 把检查设成 `/goal` 条件——每轮结束后由独立评估器复查 | 用户希望它一直跑到标准达成为止 |
| **确定性闸门** | `Stop` hook 把检查当脚本跑，不通过就不许结束这一轮（默认连续续跑 8 次后 Claude Code 会放行下一次阻拦——见 hooks 参考页） | 这个检查不能被跳过 |
| **换个人评** | 验证子智能体——或者一个会复核自己发现的 dynamic workflow——让干活的和打分的不是同一个。只要查正确性，Claude Code 自带的 `/code-review` skill 会在干净的子智能体里审查当前 diff | 正确性比速度重要 |

**这一档位在交付说明里值得点一句。** 一条 prompt 天然只能做到第一档；如果用户说的是无人值守的长跑，就告诉他真正让它「没人看着也能正确收工」的是 `/goal` 或 `Stop` hook。

## 如果这个检查还不存在怎么办

上面那条规则的前提是"有一个可跑的检查"。但**最常见的 Claude Code 请求恰恰是没有的**——老代码、覆盖率低、出过事故、不敢动。这时候不要跳过验证直接改，而是**把建立检查本身作为第一阶段**：

1. **先写钉住现状的测试（characterization test）**：不管当前行为对不对，先把它**如实断言下来**，包括你认为是 bug 的行为。目的是建立一张"改动前后行为是否一致"的网，不是判断对错
2. **可疑行为照实钉住，另外标记**：发现疑似 bug 时，测试里仍按当前实际行为断言，同时加注释标明"这里可能是 bug，见 X"。**这一条必须在 prompt 里写明**——一个善意的读者被告知"写测试钉住当前行为"时，很自然会顺手把 bug 修掉，那样网就失效了
3. **网建好之后冻结**：后续重构阶段**不允许修改这些测试**。测试变了就说明行为变了，而重构的定义就是行为不变。确实需要改测试时，必须单独提出来说明理由
4. **然后才开始重构**，每一步都跑这张网

对应的 prompt 结构是分阶段并在阶段之间设检查点，而不是一句"重构这个模块"。行为变更（修 bug、改逻辑）应该是**重构完成之后的独立一轮**，不要和重构混在一起——混在一起时测试报红你分不清是重构错了还是行为改了。

---

## 一次说清楚，不要指望多轮补充

Opus 5 专页写的是 `performs best when given the complete task specification up front and left to run`，而 Opus 5.5 专页写明 Opus 5 的做法仍是合理起点。Sonnet 5 专页（现已是 legacy 模型的页面）把道理讲得更完整：`Providing well-specified, clear, and accurate task descriptions upfront can help maximize autonomy and intelligence while minimizing extra token usage after user turns`；反过来，`ambiguous or underspecified prompts conveyed progressively over multiple user turns tend to relatively reduce token efficiency and sometimes performance`。

这条对场景 A 尤其重要，因为用户是**复制粘贴一次就走**，中途不一定在旁边。

---

## 具体到什么程度

| 策略 | 差 | 好 |
|---|---|---|
| **划定范围**：指明哪个文件、什么场景、测试偏好 | `add tests for foo.py` | `write a test for foo.py covering the edge case where the user is logged out. avoid mocks.` |
| **指向来源**：告诉它去哪儿找答案 | `why does ExecutionFactory have such a weird api?` | `look through ExecutionFactory's git history and summarize how its api came to be` |
| **引用现有模式**：指出代码库里的样板 | `add a calendar widget` | `look at how existing widgets are implemented on the home page to understand the patterns. HotDogWidget.php is a good example. follow the pattern to implement a new calendar widget...` |
| **描述症状**：症状加可能位置加"修好"长什么样 | `fix the login bug` | `users report that login fails after session timeout. check the auth flow in src/auth/, especially token refresh. write a failing test that reproduces the issue, then fix it` |

模糊 prompt 也有用武之地——探索阶段、可以承受走偏时。`what would you improve in this file?` 可能问出你想不到要问的东西。但那是探索，不是执行。

**提供丰富输入的几种方式**（写进 prompt 时告诉用户可以这么用）：`@` 引用文件、直接粘贴截图、给文档 URL、`cat error.log | claude -p "explain this error"` 管道输入、或者让 Claude 自己去取（bash、MCP 工具、读文件）。

## 只有用户能提供的值怎么处理

场景 A 里总有一些东西**既推断不出、也没法提前问**：仓库路径、测试命令、那两次线上事故到底是什么、内部系统的约定。这些往往还是整份 prompt 里价值最高的输入。

做法是在 prompt 正文里留**显式占位**，并给出留空时的兜底：

```
两次线上事故的具体表现：【填：如果记得就写在这里】
如果上面留空，先从 git log 和 issue 里检索相关修复记录，把你推断出的事故场景列出来让我确认，再继续。
```

两个要点：占位要**长得像占位**（用【】或类似标记，让用户一眼扫到），以及**必须给兜底指令**——只留空不给兜底，模型会自己编一个或者直接忽略。填了更好，没填也能跑。

---

## 探索、规划、实现、提交

任务范围不明或改动跨多文件时，prompt 应当引导走这四步，而不是直接开写：

1. **探索**（plan mode）：只读不改，先理解现状
2. **规划**：产出详细实施计划
3. **实现**：按计划写，并对照计划验证
4. **提交**：描述性 commit message，开 PR

**什么时候跳过**：范围清楚且改动很小时（改错字、加一行日志、重命名变量）直接做。官方判据很实用——**如果你能用一句话描述这个 diff，就跳过规划。**

**"对照计划验证"算不算 SKILL.md 3.4 要删的自我验证？** 取决于这份计划有没有经过外部确认：

- 模型自己写完计划、没人看过就自己对照着检查 → **算自我验证**，价值有限
- 计划**经用户看过并确认**（或写进了 SPEC.md 这类文件、成为约定） → **算外部依据**，对照它验证是有效的

所以四阶段流程里的"规划"之后应当有一个确认动作。写 prompt 时把这个检查点显式写出来：让它出完计划**停下来等确认**，再进入实现。用户不在场、无法确认时，就不要写"对照计划验证"，改为依赖测试等真正的外部检查。

规划最有价值的时候：你不确定该怎么做、改动跨多个文件、或者你对要改的代码不熟。

---

## 大特性：让它先面试用户生成 SPEC

需求较大且用户自己也没完全想清楚时，产出的 prompt 可以是这个：

```text
I want to build [brief description]. Interview me in detail using the AskUserQuestion tool.

Ask about technical implementation, UI/UX, edge cases, concerns, and tradeoffs. Don't ask obvious questions, dig into the hard parts I might not have considered.

Keep interviewing until we've covered everything, then write a complete spec to SPEC.md.
```

然后**开一个全新会话**执行这份 spec——新会话上下文干净、完全聚焦实现。

官方的判断值得转告用户：**把 spec 打磨精确所花的时间，回报高于盯着实现过程看的时间。** 最有用的 spec 是自包含的：点名涉及的文件和接口、写明什么不在范围内、并以一个端到端验证步骤收尾，用它证明这个功能确实可用。

---

## 上下文是第一约束

Claude Code 的大部分最佳实践都源自同一个约束：**上下文窗口填得快，而且填满后性能下降。** 一次调试或代码库探索可能就消耗几万 token。

写 prompt 时能做的：

- **把探索隔离出去**：`Use subagents to investigate how our authentication system handles token refresh, and whether we have any existing OAuth utilities I should reuse.` 子智能体在独立上下文里读一堆文件，只回报摘要

> **子智能体的统一判据**（这条散落在几处，这里说全）：**大范围只读探索要鼓励用，让模型在自己的上下文里重看一遍自己的产出不要用。** 换到**干净上下文**的独立 verifier 子智能体是例外，Fable 5 页指出这类 verifier 往往优于自我批评——上文「换个人评」那一档、以及本文末尾对抗式审查那段说的都是这一种。
>
> 所以：前者是它最大的价值——把读几十个文件的开销挡在主上下文外面；后者基本是模型自己已经会做的事，派子智能体去做只会增加成本、不提升结果（这条是对 Opus 5 写明的；在 Opus 5.5 上把它当成需要实测的事，见 SKILL.md 3.4）。介于两者之间的判断标准是片段 4.10：几次工具调用就能自己做完的，不要派。

- **不要写成开放式调研**：`investigate X` 不划范围会让它读几百个文件把上下文填满
- 提醒用户：不相关的任务之间用 `/clear`；同一个问题纠正两次还不对，就 `/clear` 重来并把学到的写进新 prompt

---

## 这次的 prompt vs 该沉淀进项目配置

判断很简单：**这次任务特有的信息写进 prompt；每次都成立的约定沉淀进项目配置。**

如果你发现自己在 prompt 里写"我们项目用 pnpm 不用 npm"、"提交前要跑 lint"这类**每次都要说一遍**的话，就该建议用户把它放进 CLAUDE.md 或 hook，而不是每次复制粘贴。

---

## 项目配置放哪：七种方法取舍表

Claude Code 团队的文章列了七种方法；这里 CLAUDE.md 分成根目录和子目录两行。

| 手段 | 加载时机 | 上下文成本 | 适合放什么 |
|---|---|---|---|
| **CLAUDE.md（根）** | 会话开始，全程驻留，compaction 后重新读入 | **高**（始终加载） | 构建命令、目录结构、monorepo 布局、团队约定。**控制在 200 行以内**，且要有人负责审查变更 |
| **CLAUDE.md（子目录）** | 触及该目录时按需加载 | 低 | 目录级约定 |
| **Rules** | 会话开始（无 scope）或访问文件时（有 scope） | 中 | 具体约束。**加 `paths:` frontmatter 做路径限定**以降低成本 |
| **Skills** | 名称加描述在开头，正文在调用时 | 低 | 流程性工作流：部署清单、发版流程、审查流程 |
| **Subagents** | 名称加描述在开头，正文调用时加载，只返回最终消息 | 低（隔离上下文） | 只需要结论的侧任务：日志分析、依赖审计、深度搜索 |
| **Hooks** | 生命周期事件触发，绕过 compaction | 低（在主上下文之外） | **确定性**自动化：lint、发通知、阻断特定命令 |
| **Output styles** | 会话开始，永不压缩 | **高** | 只在需要大幅改变 Claude 角色时用 |
| **追加 system prompt** | 会话开始，以 CLI 参数传入；永不压缩，只对那一次调用生效 | 中（同一会话首个请求之后会被缓存） | 语气、回复长度、格式偏好 |

**反模式**：

- ✗ 把 "Every time X, always do Y" 写进 CLAUDE.md → 改用 hook。**advisory（建议性）的指令永远不如 deterministic（确定性）的执行**
- ✗ "Never do this" 这类约束 → 改用 `PreToolUse` hook 或权限配置
- ✗ 30 行的操作流程写进 CLAUDE.md → 移到 skill
- ✗ 只跟 API 相关的规则却不限定路径 → 加 `paths:` frontmatter
- ✗ 个人偏好写进项目共享文件 → 用用户级本地配置

**CLAUDE.md 写什么**：模型猜不到的 bash 命令、与默认不同的代码风格、测试方式、仓库礼仪（分支命名、PR 约定）、项目特有的架构决策、环境怪癖、非显而易见的坑。

**不写什么**：读代码就知道的、语言通用惯例、详细 API 文档（给链接）、频繁变动的信息、逐文件说明、`write clean code` 这类空话。

**自检**：对每一行问：*删掉它会不会让 Claude 犯错？* 不会就删。
**故障信号**：明明有规则禁止，Claude 还是反复做你不想要的事，多半是**文件太长、规则被淹没了**。如果它总是跳过某一条指令，官方建议只给那一行加上 "IMPORTANT" 这类强调——强调的行多了，哪一行都不再显眼。
**工具**：`/doctor` 命令可以帮你精简 skills 和 CLAUDE.md。

**Skills 怎么写**（三层渐进式披露）：

| 层 | 内容 | 何时加载 |
|---|---|---|
| 元数据 | YAML frontmatter，只有 `name` 和 `description` | 启动时预载入系统提示词 |
| 主指令 | `SKILL.md` 正文 | Claude 判断相关时 |
| 附加文件 | 从 SKILL.md 引用的其他文件 | 只在真正需要那个场景时 |

`name` 和 `description` 是激活的关键，没触发要先改这两句。正文臃肿就拆文件。有帮助时就打包可执行代码：用 token 生成来排序列表，远比直接跑一个排序算法昂贵。**Fable 5 专页写明，为旧模型写的 skill 对它往往过于规定化，可能降低输出质量** —— 面向 Fable 系列时要定期删减。（Fable 5.1 专页没有重述这条；它写明现有 Fable 5 prompt 应当无需修改就能表现良好，而通用最佳实践页说模型专属的结论要当作在那个模型上测得的。Opus 5.5 的迁移指南对 Opus 系列给的是同一个方向：`Instructions tuned for Claude Opus 5's behavior might no longer be needed`。）

---

## 把手工复查固化成验证回路

判据：**任何用户不得不一遍遍手动执行的检查，都是固化成验证回路的候选。** 尤其是通用 linter 抓不到的确定性规则，官方举的例子是 `reject any migration that drops a column without a backfill step`。

| 模式 | 触发 | 适合 |
|---|---|---|
| **Standalone 独立** | 手动调用 | 不普遍适用的横切检查：安全扫描、无障碍审计、许可证核查 |
| **Embedded 内嵌** | 在产出 skill 的流程里自动跑 | 只属于某个特定任务的检查 |
| **Chained 串联** | 一个 skill 完成后调下一个 | 端到端交接；也是给**不可修改的 skill** 加验证的办法 |
| **On every PR** | CI 上跑 | 把个人习惯变成团队基础设施，不再依赖作者自觉 |

**坑**：内嵌只对可编辑的 skill 有效。内置或插件管理的 skill 会自动更新，改动会被覆盖，那种情况必须用串联。

---

## 四种循环

| 类型 | 命令 | 适合 |
|---|---|---|
| **Turn-based** | 手动逐轮 | 短任务 |
| **Goal-based** | `/goal get the homepage Lighthouse score to 90 or above, stop after 5 tries` | 有**可验证退出标准**、单轮不够的任务 |
| **Time-based** | `/loop 5m check my PR, address review comments, and fix failing CI` | 周期性工作、对接外部系统 |
| **Proactive** | `/schedule every hour: check #project-feedback for bug reports` | 定义明确的持续工作流：bug 报告、issue 分诊、迁移 |

控成本：**确定性的工作用脚本做**（跑脚本比逐 token 推理便宜），周期匹配实际变化频率，大规模跑之前先试点。官方结论——**模型和 effort 档位属于循环成本上最大的几个杠杆。**

---

## 五种常见失败模式

值得随 prompt 一并转告用户、提醒避开：

| 失败模式 | 症状 | 解法 |
|---|---|---|
| **大杂烩会话** | 一个任务问到一半岔到别的再岔回来 | 不相关任务之间 `/clear` |
| **反复纠正** | 纠正一次不对，再纠正还不对 | **两次失败就 `/clear`**，带着学到的写一个更好的初始 prompt |
| **过度规定的 CLAUDE.md** | 文件太长，重要规则被淹没 | 狠心修剪 |
| **信任但没验证** | 看起来合理但边界情况没处理 | 永远提供验证手段；**验证不了的东西不要发布** |
| **无边界探索** | 让它去 "investigate" 却不给范围，结果读了几百个文件 | 收窄范围，或用 subagent 隔离 |

**对抗式审查的一个重要提醒**：让一个子智能体在干净上下文里审查 diff 很有效，但官方同时警告——**被要求找问题的审查者通常也会报出一些问题，即使工作本身没问题**，因为那是它被要求做的事。追着每条发现改会导致过度工程。所以审查 prompt 要写明：**只报影响正确性或既定需求的问题，其余视为可选。**这是一条过滤指令，只适用于**由你来做过滤**的场合。如果发现要交给下游的排序或分诊步骤，改用片段 4.16、让审查者全报——Opus 5 页提醒过：审查 prompt 设了严重度门槛时，模型 `might follow that instruction literally and report less`（见 `references/models.md`），而 Opus 5.5 的页面对这个倾向既没重述也没撤回。（早期测试者反馈 Opus 5.5 的代码审查强于 Opus 5——bug 找得更多、误报更少——这是相信它的发现的理由，不是让严重度门槛在上游先过滤一遍的理由。）
