# Agent 与长任务：上下文工程

> 片段编号（如 4.17）指向 `snippets.md`。

### 6.1 上下文是第一约束
性能随上下文填充而下降（context rot）。目标是「能最大化达成目标概率的**最小高信噪比 token 集**」。

### 6.2 跨上下文窗口的工作流
1. **第一个上下文窗口用不同的 prompt**：先搭框架（写测试、建 setup 脚本），后续窗口再迭代 todo list
2. **让模型用结构化格式写测试**（如 `tests.json`），并提醒：`It is unacceptable to remove or edit tests because this could lead to missing or buggy functionality.`
3. **建生活质量脚本**（`init.sh`）：起服务、跑测试、跑 lint
4. **可以考虑用「全新窗口」代替 compaction**：现行模型极擅长从文件系统恢复状态，**某些场景下**这比压缩更划算（官方措辞是 "consider" / "in some cases"，不是一律如此）。这时开场要具体：
   - `Call pwd; you can only read and write files in this directory.`
   - `Review progress.txt, tests.json, and the git logs.`
   - `Manually run through a fundamental integration test before moving on to implementing new features.`
5. **提供验证工具**（Playwright MCP、computer use 等）
6. **鼓励用满上下文**：
```
This is a very long task, so it may be beneficial to plan out your work clearly. It's encouraged to spend your entire output context working on the task - just make sure you don't run out of context with significant uncommitted work. Continue working systematically until you have completed this task.
```

### 6.3 状态管理
- 结构化数据（测试结果、任务状态）→ JSON
- 进度笔记 → 自由文本
- 版本与检查点 → git（现行模型用得很好）
- 明确要求增量推进

### 6.4 上下文压缩感知
```
Your context window will be automatically compacted as it approaches its limit, allowing you to continue working indefinitely from where you left off. Therefore, do not stop tasks early due to token budget concerns. As you approach your token budget limit, save your current progress and state to memory before the context window refreshes. Always be as persistent and autonomous as possible and complete tasks fully, even if the end of your budget is approaching. Never artificially stop any task early regardless of the context remaining.
```

### 6.5 复杂研究任务
```
Search for this information in a structured way. As you gather data, develop several competing hypotheses. Track your confidence levels in your progress notes to improve calibration. Regularly self-critique your approach and plan. Update a hypothesis tree or research notes file to persist information and provide transparency. Break down this complex research task systematically.
```

### 6.6 让 Claude 自己编排，而不是把一切塞进上下文

这是 harness 层面收益最大的一条。**不要把每个工具结果都当 token 塞回上下文窗口**，也不要把过滤逻辑硬编码进工具设计。

给 Claude 代码执行能力（bash、语言 REPL），让它自己写逻辑来表达工具调用、过滤输出——**只有执行结果进入它的注意力**。官方数据：在 BrowseComp 网页浏览任务上，给模型输出过滤能力使准确率从 **45.3% 提升到 61.6%**。

同一思路的另外两条：
- **让 Claude 自己管上下文**：用 skills 做渐进式披露（frontmatter 常驻、正文按需），用 context editing 清掉过期的工具结果，用子智能体隔离上下文（BrowseComp +2.8%）
- **让 Claude 自己持久化上下文**：compaction + memory 文件夹（agentic search 任务 60.4% → 67.2%）

> 出处说明：以上三组数字来自 Anthropic 在**更早一代模型**上的实测。文章本身的论点是这个趋势会随模型变强而延续——模型越强，越该把决策交还给它。

### 6.7 设计上下文以最大化缓存命中

**缓存 token 的成本只有基础输入 token 的 10%**，所以这是纯粹的成本杠杆。五条原则：

| 做法 | 要点 |
|---|---|
| **静态在前** | 稳定内容（系统提示词、工具定义）放在动态内容之前 |
| **用消息追加而非改提示词** | 需要注入提醒时，追加到 messages 里，不要去改 system prompt |
| **不要中途换模型** | 缓存是按模型隔离的 |
| **工具增删会作废缓存** | 需要动态发现工具时用 tool search，而不是增删工具定义 |
| **移动断点** | 多轮应用里把 cache breakpoint 移到最新消息 |

> 直接后果：**中途修改 effort 会作废缓存**，同理（见 SKILL.md 第 2 节）。

### 6.8 什么时候该把 bash 动作提升为专用工具

bash 杠杆大但缺乏可观测性，harness 无法拦截、把关或审计。以下四种情况应当做成专用的声明式工具：

1. **安全边界**：难以撤销的动作，需要用户确认（如调用外部 API）
2. **写操作需要陈旧性检查**：防止覆盖已被改动的文件
3. **需要面向用户呈现**：弹窗、多选项展示
4. **需要结构化日志、追踪和回放**

（低风险工作可以走另一条路：用第二个 Claude 实例判断 bash 命令是否安全，这是 Claude Code auto 模式的做法，可以减少专用工具的数量。）

### 6.9 工具设计优先于提示词
有工具的场景下，**改工具比改提示词更有效**：
- 每个工具功能互不重叠，模型能一眼分清何时调用哪个
- 参数名语义明确（`user_id` 而非 `user`）
- 按服务和资源类型做命名空间（`asana_projects_search`）
- 返回值 token 高效：分页、范围选择、过滤、截断，并给默认上限
- 提供 `ResponseFormat` 枚举（`concise` / `detailed`）
- 错误信息要可执行：说清怎么改，附正确格式示例，而不是抛栈
- 工具描述当作给新同事的入职说明来写，把隐含前提写明
