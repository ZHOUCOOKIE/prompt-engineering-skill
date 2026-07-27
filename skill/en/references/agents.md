# Agents and long-running tasks: context engineering

> Snippet numbers refer to `snippets.md`.
>
> The English blocks in this file are **verbatim official wording** from Anthropic's long-running-agent guidance (see `SOURCES.md`) — copy them as is rather than translating or tidying them.

## Context is the first constraint

Performance degrades as the context fills — context rot. The target is not "tell it everything you know" but **the smallest set of high-signal tokens that maximize the likelihood of the desired outcome**.

## Multi-window workflows

For tasks that span more than one context window:

1. **Use a different prompt for the very first window.** Use it to set up the framework — write tests, create setup scripts — and use later windows to iterate on a todo list
2. **Have the model write tests in a structured format** (for example `tests.json`), and remind it: `It is unacceptable to remove or edit tests because this could lead to missing or buggy functionality.`
3. **Set up quality-of-life scripts** (`init.sh`) that start servers, run the test suite, and run linters — this prevents repeated work at the start of each fresh window
4. **Consider starting fresh instead of compacting.** Current models are extremely effective at discovering state from the local filesystem, and **in some cases** that beats compaction (the official wording is "consider" and "in some cases," not always). When you do, be prescriptive about the opening:
   - `Call pwd; you can only read and write files in this directory.`
   - `Review progress.txt, tests.json, and the git logs.`
   - `Manually run through a fundamental integration test before moving on to implementing new features.`
5. **Provide verification tools** (browser automation, computer use). The longer and more autonomous the task, the more the model needs to verify correctness without continuous human feedback
6. **Encourage complete use of the context:**
```
This is a very long task, so it may be beneficial to plan out your work clearly. It's encouraged to spend your entire output context working on the task - just make sure you don't run out of context with significant uncommitted work. Continue working systematically until you have completed this task.
```

## State management

- Structured data (test results, task status) → JSON
- Progress notes → freeform text
- Versions and checkpoints → **git**. Current models are especially effective at using git to track state across sessions
- Ask explicitly for incremental progress

## Compaction awareness

```
Your context window will be automatically compacted as it approaches its limit, allowing you to continue working indefinitely from where you left off. Therefore, do not stop tasks early due to token budget concerns. As you approach your token budget limit, save your current progress and state to memory before the context window refreshes. Always be as persistent and autonomous as possible and complete tasks fully, even if the end of your budget is approaching. Never artificially stop any task early regardless of the context remaining.
```

## Complex research

```
Search for this information in a structured way. As you gather data, develop several competing hypotheses. Track your confidence levels in your progress notes to improve calibration. Regularly self-critique your approach and plan. Update a hypothesis tree or research notes file to persist information and provide transparency. Break down this complex research task systematically.
```

## Let the model orchestrate itself

The highest-leverage change at the harness level. **Do not route every tool result back through the context window as tokens**, and do not hardcode filtering logic into the tool design.

Give the model code execution (bash, a language REPL) so it can write logic that expresses the tool calls and filters the output itself — **only the execution results reach its attention**. Official figure: on BrowseComp web-browsing tasks, giving the model the ability to filter its own tool outputs raised accuracy from **45.3% to 61.6%**.

Two more in the same direction:

- **Let it manage its own context**: progressive disclosure through skills (frontmatter resident, body on demand), context editing to drop stale tool results, subagents for isolated contexts (BrowseComp +2.8%)
- **Let it persist its own context**: compaction plus memory folders (agentic search 60.4% → 67.2%)

> Provenance: these three figures come from Anthropic's experiments on an **earlier model generation**. The article's own argument is that the trend continues as models improve — the stronger the model, the more decisions you should hand back to it.

## Design context to maximize cache hits

**Cached tokens are 10% the cost of base input tokens**, which makes ordering a pure cost lever.

| Practice | Guidance |
|---|---|
| **Static first** | Place stable content (system prompts, tool definitions) before dynamic material |
| **Append in messages, do not edit the prompt** | Inject reminders by appending to messages rather than editing the system prompt |
| **Do not switch models mid-session** | Caches are model-specific |
| **Adding or removing tools invalidates the cache** | Use tool search for dynamic discovery instead of changing tool definitions |
| **Move breakpoints** | In multi-turn applications, move the cache breakpoint to the latest messages |

> Direct consequence: **changing effort mid-session invalidates the cache** for the same reason.

## When to promote a bash action to a dedicated tool

Bash has broad leverage but no observability — the harness cannot intercept, gate, or audit it. Four cases warrant a dedicated declarative tool:

1. **Security boundaries**: hard-to-reverse actions that need user confirmation (calling an external API)
2. **Write operations needing staleness checks**: preventing overwrites of files that changed
3. **User-facing presentation**: modal displays, multiple options
4. **Structured logging, tracing, and replay**

For low-stakes work there is another path: use a secondary Claude instance to judge whether a bash command is safe — this is what Claude Code's auto mode does, and it reduces the number of dedicated tools needed.

## Tool design beats prompt text

Where tools exist, **changing the tool is more effective than changing the prompt**.

- **Low cardinality**: every tool has a distinct function from every other tool, so Claude can easily distinguish when to call each. The test is blunt — **if engineers cannot definitively identify which tool to use in a situation, agents will not either**
- **Semantically meaningful parameter names**: `user_id` rather than `user`; resolving arbitrary UUIDs to interpretable language significantly improves precision
- **Namespacing** by service and resource type (`asana_projects_search`, `asana_users_search`)
- **Token-efficient returns**: pagination, range selection, filtering, truncation, with sensible defaults (Claude Code's default cap is 25,000 tokens)
- **Offer a `ResponseFormat` enum** (`concise` / `detailed`) — the official example shows 206 tokens versus 72, a 65% reduction
- **Actionable error messages**: say what to change and show a correctly formatted example, rather than returning a stack trace
- **Write tool descriptions as onboarding for a new team member**: make implicit context explicit — specialized query formats, terminology, resource relationships
