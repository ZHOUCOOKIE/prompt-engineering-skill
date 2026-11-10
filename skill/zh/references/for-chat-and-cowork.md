# 场景 B：写给 Claude 应用（chat 或 Cowork）执行的任务 prompt

用户在 Claude desktop 里说需求，产出的 prompt 在 Claude 应用里直接跑。非代码任务：研究、分析、写作、整理资料、做文档表格幻灯片、日程、信息汇总。

## 目录

- chat 和 Cowork 现在是同一个对话
- 交代任务时官方建议写明的三样东西
- 模型、effort 和用量
- 输出长度写在 prompt 里
- 材料怎么摆
- 用示例固定格式、语气和结构
- 研究类任务
- 交付物与文件
- 定时任务
- 不要写进去的东西
- 一个可直接套用的骨架
- 信源

---

## chat 和 Cowork 现在是同一个对话

支持站「Claude Cowork and chat are one Claude」的说法：新体验把两者合并成一个对话，`you don't need to decide which option better suits your task before getting started`——说出你要什么，Claude 可以决定用哪种工具；以前只能在 Cowork 里做的事，现在任何对话里都能做。它正在分批推出，先是 Pro、Max、Ultra（网页、桌面和手机）；同一个计划的账号看到它的时间也不一样。消息框里还显示 "Chat" 和 "Cowork" 两个选项的账号还没换过去，照原来的方式用，支持站那篇文章里讲的大部分事情在那里也能做。

所以写 prompt 时不用替用户选模式。按下面这些官方写明的能力来写：

| 能做什么 | 官方的说明 | 写 prompt 时意味着什么 |
|---|---|---|
| 交给它一整个任务 | 描述你要的结果，Claude 自己走完各步：搜网页、读文件、跑代码、把结果拼起来。不需要你把工作拆成步骤 | 描述结果，不必把工作拆成步骤；顺序或完整性要紧时再给步骤（最佳实践页） |
| 你走开，它接着做 | 更复杂的任务在云端继续跑，合上电脑或离开页面也不停；回来时结果在对话里等着 | 用到本机文件或本机应用的任务需要 Claude Desktop 开着 |
| 拿回成品文件 | 文档、带可用公式的表格、能在 PowerPoint 里打开的演示稿 | 说清要哪种文件 |
| 设计稿、演示稿和文档（artifacts） | 付费计划上可以要 design、deck 或 doc：Claude Design、Claude Slides、Claude Docs（三者都是 beta），可以自己编辑、让 Claude 改，再用链接分享或导出 | 要给别人看、要分享的东西可以走这条路 |
| 用已连接的应用 | Google Drive、Gmail、Microsoft 365、Slack 等，执行任务途中按需取用 | 说清材料在哪个应用里。Opus 5.5 做跨多个已连接应用的任务时，官方给的那句（先翻相关来源再动手）见片段 4.32 |
| 本机文件和文件夹 | 在 Claude Desktop 里给 Claude 某个文件夹的访问权，它可以在那里读、整理和创建文件 | 要存到本机就写明文件夹，并提醒用户先授权 |
| 浏览和 computer use | 用 Claude Desktop 内置浏览器或 Claude in Chrome 打开网站、读页面、点击、填表；computer use 在 Pro、Max、Ultra 上是 beta | 内置浏览器和 computer use 只在 Claude Desktop 开着时可用 |
| 定时任务 | 让 Claude 按计划跑一个任务；可以在云端跑，电脑关着也继续 | 见「定时任务」一节 |
| skills 和 plugins | 教 Claude 你喜欢某件事怎么做，之后遇到这类任务它就照做 | 遇到这类任务就该照做的步骤，可以一次教成 skill |

**它多久问你一次**由消息框里的权限设置决定：Auto（Pro、Max、Ultra、Team 上的默认）下 Claude 不为每一步停下来问，动作前有自动安全检查；Manual 下它在动作前询问。支持站的提醒：有实际后果的工作——发消息、改重要文件——要守在旁边看它做了什么。

仓库级操作、git、构建部署属于 Claude Code，那是场景 A。

## 交代任务时官方建议写明的三样东西

支持站原话：`describe what you need, the way you'd brief a colleague`。写上这三样结果更好：

- **想要的结果**：最后要拿到什么，例如 "a one-page summary"、"a spreadsheet with a tab for each region"
- **格式**：结果以什么形式交付，例如 Word 文档、幻灯片、或一段能直接贴进 Slack 的话
- **Claude 需要的输入**：该从哪些文件、链接或应用入手

支持站另外两篇文章的写法提示：写作类请求把要求、受众和要点一次说全，要编辑的整篇文字放在一条消息里发，不要拆开；研究和分析类请求一开始就说清研究问题和关注范围，把相关数据放进一条结构清楚的消息（「Usage limit best practices」）。项目指令保持简短，只放项目的总体背景、关键准则和 Claude 的角色，任务专属的指令留在对话里（「How do usage and length limits work?」）。

## 模型、effort 和用量

模型和 effort 在发送按钮旁的模型菜单里选，选哪一档按 SKILL.md 第 2 节。支持站对各档的说法：Low 和 Medium 适合例行任务，也让用量撑得更久；High 在质量和速度之间最均衡；Extra high 面向长时间的编码和 agentic 任务；Max 最彻底。每个模型的推荐档在菜单里标着 "Default"。

同一篇文章按任务给的建议（"Choose the right settings for your task"）：日常任务用默认值就行，简单问题、基本的信息查询和一般的写作不需要额外的 effort，更低的 effort 让用量撑得更久；复杂任务提高 effort、打开 thinking，或两者都做；`For complex coding and agentic tasks on Opus 4.7 or newer, try Extra high (xhigh) first`；最难、对正确性要求最高的工作选 Max，并预期更长的响应时间；不管选哪种设置，把问题说具体。其中 xhigh 那一句和模型页不一致：Opus 5.5 和 Fable 5.2 的页面都把 `xhigh` 留给实测到提升的工作，Sonnet 5.5 和 Haiku 5.5 的页面也只在评测显示质量提升时才用 `xhigh`。这个 skill 按模型页来选（SKILL.md 第 2 节）；任务是复杂编码或 agentic 工作时，交付说明里把支持站这句话也告诉用户。

Fable 5.2、Opus 5.5、Sonnet 5.5、Haiku 5.5 在 Claude 应用里都关不掉 thinking，想让它少思考就选更低的 effort。

用量：在 Claude 里做的一切都计入计划的用量上限，会搜网页、跑代码、创建文件的较长 agentic 任务通常比一个快问题用得多；effort 越高，越快到上限。Fable 模型的额度规则见 `models.md`「订阅用量上限」；做文档、表格、幻灯片时选哪个模型，两处官方材料的例子不同：选型文章把日常的放在 Sonnet 5.5，复杂的财务模型或要从一大堆材料里做出来的文档放在 Opus 5.5；平台的选型矩阵把要一直做到成品文档、表格或幻灯片的分析列在 Fable 5.2 的例子里。支持站给 Claude 应用写的省用量做法（把相关问题合进一条消息、事先给足背景、例行任务用更低的 effort、关掉用不上的已连接应用、不需要最新信息时让 Claude 不要搜网页、长对话快到上限时开新对话）见 `models.md`「官方推荐的省用量做法」。

---

## 输出长度写在 prompt 里

在 Fable 5.2 和 Opus 5 上，不要指望靠 effort 缩短回复。Fable 5.2 专页：`Effort barely changes how long the model's replies are. To change the length and tone of a reply, use your prompt.` Opus 5 页面写明降 effort 不能可靠缩短可见输出，并说它写到磁盘的文件 `are often longer than on prior models`。Opus 5.5 的页面没有重述这两条，只说它 `tends to complete the same task with fewer tokens`。

现成措辞见片段 4.2 和 4.4（Opus 5 页面原文）。目标是 Fable 5.2 时，硬性的句长上限它会照得很紧，可能让文字变得一顿一顿的，官方的做法是先删掉不再需要的文风规则；最终汇报用片段 4.40。

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

## 用示例固定格式、语气和结构

最佳实践页：`Examples are one of the most reliable ways to steer Claude's output format, tone, and structure.` 每次都按同一个格式输出的任务（周报、会议纪要）就是这种场合。该页的要求：

- 3–5 个，用 `<example>` 包裹（多个时外面再套 `<examples>`）
- **相关**：贴近真实用例
- **多样**：覆盖边界情况，避免模型学到意外的模式
- 可以让 Claude 评估你的示例是否相关、是否够多样，或者基于你给的那几个再生成一些（`You can also ask Claude to evaluate your examples for relevance and diversity, or to generate additional ones based on your initial set`）

---

## 研究类任务

最佳实践页给研究任务列了三条：给出清楚的成功标准，定义什么算一个成功的答案；鼓励查证来源，让 Claude 跨多个来源核实信息；复杂的研究任务用结构化的做法。第三条它给了一段示例 prompt：

```
Search for this information in a structured way. As you gather data, develop several competing hypotheses. Track your confidence levels in your progress notes to improve calibration. Regularly self-critique your approach and plan. Update a hypothesis tree or research notes file to persist information and provide transparency. Break down this complex research task systematically.
```

按目标模型再加：

- **Sonnet 5.5**：有时凭训练知识作答，而搜一下就能发现细节已经变了。给了搜索工具时用片段 4.45，并删掉 prompt 里压制工具使用的话
- **Haiku 5.5**：给了搜索工具就同时给当天日期，见片段 4.47
- **Fable 5.2**：选型矩阵把多步深度研究列在它名下，但成本页在一项研究基准（DeepResearch Bench II，各自在 Messages API 的默认档：Fable 5.2 是 `high`，Opus 5.5 是 `medium`）上测得它的成本是 Opus 5.5 的 2.9 倍、分数低 1.9 分，官方的话是 `Measure what a more expensive model buys on your own workload`。交付说明里把这两条都告诉用户
- **Opus 5.5**：Anthropic 的 Opus 5.5 使用文章建议在研究和分析类请求里加一句 `Mark anything you couldn't confirm, and say where you looked`

**允许它说不知道。** 下面这句出自 Anthropic 2025-11 那篇 prompt engineering 文章（"Give permission to Claude to express uncertainty" 一节的例子），不在文档页上：

```
If the data is insufficient to draw conclusions, say so rather than speculating.
```

---

## 交付物与文件

支持站写明 Claude 能创建文档、带可用公式的表格、能在 PowerPoint 里打开的演示稿；这些文件出现在对话旁边，可以下载，也可以让 Claude 继续改。写 prompt 时把格式说出来（见上文「三样东西」）。

Anthropic 的 Opus 5.5 使用文章（2026-09-22，写的是 Opus 5.5）里和这一节有关的三条：要表格或文档时，要成品文件而不是提纲；图表、示意图、截图或幻灯片直接附上，不要把数字重新打一遍，并问一个具体的问题；想让它查一份长计划、报告或演示稿里的错，文章的写法是 `Check this deck for anything that contradicts itself: numbers, dates and names. Quote each problem and say where it is.`

要给别人看或要分享的东西，付费计划上还可以要 design、deck 或 doc（Claude Design、Claude Slides、Claude Docs，beta），用链接分享或导出。

要把文件存到自己电脑上的某个文件夹：在 Claude Desktop 里给 Claude 那个文件夹的访问权。Claude 只有在 Claude Desktop 开着时才能碰本机文件，默认在永久删除任何文件之前会先问。

## 定时任务

支持站：可以让 Claude 按计划跑一个任务，例如每周一早上汇总收件箱；定时任务可以在云端跑，电脑关着也继续。用户的需求本质是"每周给我一份 X"时，告诉他可以设成定时任务。

任务在跑的时候用户可以从手机上看进度、回答 Claude 的问题、调整方向（支持站「Check in from your phone」）。

---

## 不要写进去的东西

1. **不分模型一律加或一律删的自我验证指令**——各模型的官方说法不一样，见 SKILL.md 3.4
2. **人设铺垫**（"你是一位有二十年经验的资深分析师"）——最佳实践页说角色有用，一句话就起作用（`Even a single sentence makes a difference`）；长篇的人设是另一回事，Anthropic 2025-11 那篇文章说过度的角色设定往往没必要，要影响视角就直接说关注什么，文中的例子是 `Analyze this investment portfolio, focusing on risk tolerance and long-term growth potential`
3. **一长串"不要"**——改成正面描述加原因。见 SKILL.md 3.5
4. **叫它把思考过程写出来**——在 Fable 5.2、Opus 5.5、Sonnet 5.5 上可能被拒。见 SKILL.md 3.9
5. **目标是 Fable 5.2 时**：叫它少思考的句子、硬性的文风上限（见 SKILL.md 3.13）；笼统地叫它放手去做，官方在一个模拟设置里测得没有带来变化（见 `models.md` 的 Fable 5.2 表）

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

<task>
要做什么，以及这是为谁做的、他拿到之后要用来干什么。
</task>

<requirements>
1. ……
2. ……
（正面表述，每条带上为什么）
</requirements>

<output>
形态：（对话内容 / 文档 / 表格 / 演示稿 / 可分享的 design、deck 或 doc）
长度：（写明）
结构：（章节或字段）
</output>

<examples>
  <example>……</example>
</examples>
（想固定格式、语气或结构时加）
```

最后加一句**成功标准**——什么样的产出算做对了。`<task>` 里写受众和用途，出自 Anthropic 2025-11 那篇文章的 "Provide context and motivation" 一节（`Explaining the purpose or audience for the output`、`Describing how the output will be used`）。

---

## 信源

本文件中关于 Claude 应用能力的描述，依据 support.claude.com 的「Claude Cowork and chat are one Claude」「Change the model, effort, and thinking settings」「Claude Fable models on your plan」「Usage limit best practices」和「How do usage and length limits work?」。prompt 技法部分依据官方 prompting best practices 和各模型的 prompting 专页，出自 Anthropic 文章的六处（2025-11 的 prompt engineering 文章三处、Opus 5.5 使用文章两处、选型文章一处）已在行内写明。产品能力会随版本变化，判断不准时以用户实际看到的界面为准。
