# Agents and long-running tasks: context engineering

> Snippet numbers refer to `snippets.md`.
>
> The English blocks in this file are **verbatim official wording** from Anthropic's long-running-agent guidance (see `SOURCES.md`) — copy them as is rather than translating or tidying them.

## Contents

- Context is the first constraint
- Multi-window workflows
- State management
- Compaction awareness
- Complex research
- Let the model orchestrate itself
- Design context to maximize cache hits
- When to promote a bash action to a dedicated tool
- Tool design beats prompt text

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

**If you compact on the client side** (Fable 5.1 page): server-side compaction already tells the model what to keep; for your own summarization step, the official instruction is below. A third option now exists on both models in scope — **compaction on demand** (beta, `compact-2026-09-04`): send the top-level `compaction` parameter and the API returns a signed `compaction` block summarizing the conversation, which you then send first in place of the summarized messages. You choose when to compact, and the thinking blocks in the turns you keep can stay valid after the swap — which matters on Opus 5.5 precisely because its thinking blocks are tied to the conversation. The simplest shape on Fable 5.1 is to replace the whole history with one summary message plus the new user turn and replay nothing else — no thinking blocks carry over, so nothing fails the append-only check. And because cache reads are cheaper on Fable 5.1, compacting early to save cost may no longer be the right trade-off; experiment with later compaction points.

```
Summarize the transcript inside <summary></summary> tags. Include relevant information in the summary such that this conversation will be continued by a new context window without needing to redo work or be reprovided with relevant constraints or context. Be sure to preserve: (1) any difficulties or problems that came up, and how they were handled or resolved; (2) any possibilities, options, or approaches that were raised, tried, or set aside, and why; (3) anything that was asked for, decided, agreed, ruled out, or established as a preference, constraint, or boundary — stated exactly; (4) exactly where things stand now — what has been covered, settled, or completed so far; (5) anything still open, unresolved, promised, or expected to happen next; (6) specific details that would be hard to reconstruct — names, numbers, dates, exact wording, links or references — kept exactly. Be complete on these even at the cost of length; keep everything else concise. Weight the two voices differently: keep what the user said, asked for, shared, or established carefully and close to their own words; your own explanations and reasoning can be condensed much further, to what they concluded or produced — as long as nothing in the six items above is dropped.
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

**Cached tokens are 10% the cost of base input tokens** — 5% on Opus 5.5, 2.5% on Fable 5.1 and Mythos 5.1 — which makes ordering a pure cost lever.

| Practice | Guidance |
|---|---|
| **Static first** | Place stable content (system prompts, tool definitions) before dynamic material |
| **Append in messages, do not edit the prompt** | Inject reminders by appending to messages rather than editing the system prompt |
| **Do not switch models mid-session** | Caches are model-specific |
| **Adding or removing tools invalidates the cache** | Use tool search for dynamic discovery instead of changing tool definitions |
| **Move breakpoints** | In multi-turn applications, move the cache breakpoint to the latest messages |

> Direct consequence: **changing the top-level effort between requests invalidates the cache** for the same reason. Opus 5.5, Fable 5.1, Mythos 5.1 and Opus 5 support a per-message effort change (beta) that preserves it — see `models.md`.

**The mechanism behind row two.** Prompt caching hashes the request prefix in order — `tools`, then `system`, then `messages` — and a hit requires a byte-exact prefix match. Appending one sentence to the system prompt therefore misses the cache for the system prompt *and every cached message after it*. When an instruction only becomes relevant partway through a run, append a `{"role": "system"}` message at that point in `messages` rather than editing the top-level `system` field: the cached prefix stays intact, and the instruction still applies as a system instruction, taking precedence over the top-level field for the turns that follow it. Available on Fable 5.1, Mythos 5.1, Fable 5, Mythos 5, Opus 5.5, Opus 4.8 and Opus 5 with no beta header; **not available on Sonnet 5**, where the top-level `system` field is the only option. A `role: "system"` message can also be scoped to one turn with `clear_at: "next_user_message"` (beta, `mid-conversation-system-clear-at-2026-08-21`): it carries system-prompt authority for the current turn, stops rendering once a later user message exists, stays in the array so nothing earlier changes, and costs no input tokens once cleared — the official home for per-turn reminders in a tool loop.

**Keep the history append-only (Fable 5.1 and Opus 5.5).** Append each assistant turn exactly as returned, thinking blocks included, and do not edit earlier turns between requests. On both models a thinking block is valid only in the exact conversation that produced it: for accounts created on or after August 31, 2026, replaying one after its prefix (system prompt, tool list, or any earlier message) has changed returns a 400, or drops the block if you set `thinking.block_binding.prefix_mismatch_behavior: "drop_block"` (beta, `thinking-binding-controls-2026-08-01`); on older accounts, setting that field to either value opts the request in. The best practices page states it for both: `On Claude Fable 5.1 and Claude Opus 5.5, modifying the conversation before a thinking block results in an error, or in the block being dropped if you opt into that`. This is one of the **four breaking changes** when moving from Opus 5.

**Thinking blocks are also tied to the model that produced them.** Opus 5.5 reads thinking blocks from Opus 5 and earlier Opus, Sonnet and Haiku models, but not from Claude Fable or Claude Mythos models; on the Claude API, Fable 5.1 and Mythos 5.1 read Opus 5.5's blocks and no other model does. A conversation that moves from Opus 5 onto Opus 5.5, or from Opus 5.5 up to Fable 5.1 or Mythos 5.1 on the Claude API, keeps its reasoning; any other switch runs the turns after it without the previous model's reasoning. When a request carries a block the target model cannot read, the API drops it before the model sees it — the request succeeds, and dropped blocks are not billed. So if a router or fallback can move a conversation between model families, expect the reasoning not to survive it. The edits that trip it are the same ones that restart the cache — injecting and removing per-turn reminders, summarizing older turns in place, changing the system prompt mid-session — so the fixes are the same: turn-scoped system messages for reminders, mid-conversation system messages and tool changes instead of rewriting `system` or `tools`, and server-side compaction or context editing for trimming. Removing a leading run of thinking blocks (oldest first), moving `cache_control` markers, and changing `effort` between requests keep later blocks valid.

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
