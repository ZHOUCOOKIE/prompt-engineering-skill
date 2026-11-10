# Agents and long-running tasks: context engineering

> The English blocks in this file are **verbatim official wording** from Anthropic (sources in `SOURCES.md`). The API details here matter only when you build requests yourself: if you use Claude Code, claude.ai, Claude Managed Agents or the Claude Agent SDK, they manage the conversation history.

## Contents

- Context is a finite resource
- Multi-window workflows
- State management
- Compaction awareness
- Complex research
- Let the model orchestrate itself
- Design context to maximize cache hits
- When to promote a bash action to a dedicated tool
- Tool design

## Context is a finite resource

Performance degrades as the context fills — context rot. The target is not telling it everything you know but **the smallest possible set of high-signal tokens that maximize the likelihood of some desired outcome**.

## Multi-window workflows

1. **Use a different prompt for the very first window.** Use it to set up the framework — write tests, create setup scripts — and use later windows to iterate on a todo list
2. **Have the model write tests in a structured format** (for example `tests.json`), and remind it: `It is unacceptable to remove or edit tests because this could lead to missing or buggy functionality.`
3. **Set up quality-of-life scripts** (`init.sh`) that start servers, run the test suite, and run linters — this prevents repeated work at the start of each fresh window
4. **When a context window is cleared, consider starting fresh instead of compacting.** Current models are extremely effective at discovering state from the local filesystem, and **in some cases** you might want to take advantage of that over compaction (the official wording is "consider" and "in some cases," not always). When you do, be prescriptive about the opening:
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
- Versions and checkpoints → git. Current models are especially effective at using git to track state across sessions
- Ask explicitly for incremental progress

## Compaction awareness

If you use Claude in an agent harness that compacts context or allows saving context to external files (as in Claude Code), the best practices page suggests adding this to your prompt so Claude can behave accordingly. The page places it under context awareness, a feature it lists for Sonnet 5, Sonnet 4.6, Sonnet 4.5 and Haiku 4.5. The Context windows page states that Claude Opus 4.7 and later Opus models, Sonnet 5.5, Fable 5.2 and Haiku 5.5 don't receive those injected budget tags, and that on these models you give the model an explicit budget with task budgets (beta). So treat this block as measured on those earlier models. Before you use it with Fable 5.2, note one more thing: the Fable 5.2 page says that a token budget the harness writes into the prompt changes how much the model spends, and suggests testing without the budget:
```
Your context window will be automatically compacted as it approaches its limit, allowing you to continue working indefinitely from where you left off. Therefore, do not stop tasks early due to token budget concerns. As you approach your token budget limit, save your current progress and state to memory before the context window refreshes. Always be as persistent and autonomous as possible and complete tasks fully, even if the end of your budget is approaching. Never artificially stop any task early regardless of the context remaining.
```

**How to compact without tripping the prefix check** (API only; preserved thinking page). Trimming on the client (dropping or summarizing the oldest turns and keeping the recent ones verbatim) is an edit to the history: the thinking blocks in the kept turns were produced while the removed content was still there, so they fail the check. Three approaches do not count as edits:

- **Compaction on demand** (beta, `compact-2026-09-04`): a separate request returns the summary, and you put the returned block in place of the messages it summarizes. The kept turns, with their thinking, can stay valid
- **Compaction at a token threshold**: when the context approaches a threshold you set, the earlier turns are summarized into a compaction block. Its `instructions` parameter can take your own summarization prompt
- **Context editing**: clears old tool results or old thinking blocks by rule, oldest first

Two things don't carry over on their own after compaction (the "Compaction and preserved thinking" page). Mid-conversation system messages in the summarized turns are summarized too, so their text instructions stop applying after the swap. A per-message effort they set doesn't carry over either: until a later message sets a level, turns run at the request's top-level `output_config.effort`, or at the model's default if that isn't set. To keep an instruction or an effort level in force, state it again in a `role: "system"` message after the first new `user` turn that follows the kept turns.

When you write the summary yourself on the client, the officially recommended approach is **simple compaction**: summarize the whole session into one user message, send only that message plus the next instruction, and replay nothing earlier. No old thinking is sent back, so nothing can fail the check. The official wording: `Claude models are trained on long-horizon tasks with this scheme, and for most workloads it performs well.` Do not compact in the middle of a tool round, and do not cut turns out of the middle of the history.

The instruction for your own summarization step is published only on the Fable 5.1 page (measured on Fable 5.1; the Fable 5.2 pages do not reprint it). The same page says that cache reads on Fable 5.1 cost 2.5 percent of the input price, that compacting early to save cost `can be the wrong trade-off between cost and intelligence`, and to experiment with later compaction points; Fable 5.2's cache reads are at the same rate, but its pages do not restate this advice:

```
Summarize the transcript inside <summary></summary> tags. Include relevant information in the summary such that this conversation will be continued by a new context window without needing to redo work or be reprovided with relevant constraints or context. Be sure to preserve: (1) any difficulties or problems that came up, and how they were handled or resolved; (2) any possibilities, options, or approaches that were raised, tried, or set aside, and why; (3) anything that was asked for, decided, agreed, ruled out, or established as a preference, constraint, or boundary — stated exactly; (4) exactly where things stand now — what has been covered, settled, or completed so far; (5) anything still open, unresolved, promised, or expected to happen next; (6) specific details that would be hard to reconstruct — names, numbers, dates, exact wording, links or references — kept exactly. Be complete on these even at the cost of length; keep everything else concise. Weight the two voices differently: keep what the user said, asked for, shared, or established carefully and close to their own words; your own explanations and reasoning can be condensed much further, to what they concluded or produced — as long as nothing in the six items above is dropped.
```

## Complex research

```
Search for this information in a structured way. As you gather data, develop several competing hypotheses. Track your confidence levels in your progress notes to improve calibration. Regularly self-critique your approach and plan. Update a hypothesis tree or research notes file to persist information and provide transparency. Break down this complex research task systematically.
```

## Let the model orchestrate itself

**Do not route every tool result back through the context window as tokens.** The article's point: hard-coded filters in the tool design can handle this, but the harness is then still making an orchestration decision that Claude is better placed to make.

Give the model code execution (bash, a language REPL) so it can write logic that expresses the tool calls and filters the output itself — **only the execution results reach its attention**. Official figure: on BrowseComp web-browsing tasks, giving Opus 4.6 the ability to filter its own tool outputs raised accuracy from **45.3% to 61.6%**.

Two more in the same direction:

- **Let it manage its own context**: progressive disclosure through skills (frontmatter resident, body on demand), context editing to drop stale tool results, subagents for isolated contexts (BrowseComp +2.8% over the best single-agent runs, Opus 4.6)
- **Let it persist its own context**: compaction (on BrowseComp, Opus 4.6 reached 84% where Sonnet 4.5 stayed flat at 43% under the same setup) plus memory folders (a memory folder lifted Sonnet 4.5 from 60.4% to 67.2% on BrowseComp-Plus)

> Provenance: these figures come from Anthropic's experiments on an **earlier model generation** (Opus 4.6 and Sonnet 4.5). The article's argument is that as models improve you can hand more decisions back to them, and that `Assumptions about what Claude can't do need to be re-tested with each step change in its capability` — so re-test on the current models rather than assume the figures carry over.

## Design context to maximize cache hits

**Cache reads cost 10% of base input tokens** (5% on Opus 5.5 and Sonnet 5.5, 2.5% on Fable 5.2 and Mythos 5.2), so this is a pure cost lever. Cache writes cost extra: a write with a 5-minute TTL is 1.25 times the base input price, and one with a 1-hour TTL is 2 times. For concurrent requests, a cache entry becomes available only after the first response starts, so wait for the first response if parallel requests need cache hits. Five principles:

| Practice | Guidance |
|---|---|
| **Static first** | Place stable content (system prompts, tool definitions) before dynamic material |
| **Append in messages, do not edit the prompt** | Inject reminders by appending to messages rather than editing the system prompt |
| **Do not switch models mid-session** | Caches are model-specific |
| **Adding or removing tools invalidates the cache** | Use tool search for dynamic discovery, or mid-conversation tool changes (below), instead of changing tool definitions |
| **Move breakpoints** | In multi-turn applications, move the cache breakpoint to the latest messages |

> Direct consequence: **changing the top-level effort between requests invalidates the cache** for the same reason. The four current models and Mythos 5.2 support a per-message effort change (beta, header `mid-conversation-output-config-2026-07-01`) that preserves the cache; see "Effort reference" in `models.md`.

**The mechanism behind row two.** Prompt caching hashes the request prefix in order — `tools`, then `system`, then `messages` — and a hit requires a byte-exact prefix match. Appending one sentence to the system prompt therefore misses the cache for the system prompt and every cached message after it. When an instruction only becomes relevant partway through a run, append a `{"role": "system"}` message at that point in `messages` rather than editing the top-level `system` field: the cached prefix stays intact, and the instruction still applies as a system instruction, taking precedence over the top-level field for the turns that follow it. Available on the four current models and Mythos 5.2, with no beta header. Placement is restricted: a system message that carries content can't be the first entry in `messages`; it must immediately follow a `user` turn (including one that carries `tool_result` blocks) or an assistant turn that ends in a server tool result, and it can't sit between a `tool_use` and its `tool_result`. A misplaced one returns a 400. Also, don't put text from outside the conversation (raw tool output, retrieved documents, web content) directly in a system message: Claude treats system content as operator instructions and follows it, so doing this gives that text operator-level authority. Keep that kind of data in `tool_result` blocks. On wording, the same page advises phrasing the system content as context, not as a command that overrides the user: state the fact (its example is `new input arrived from the user: X`) and let Claude act on it; language such as `ignore what the user said` is less effective than stating what changed. On Sonnet 5.5 and Haiku 5.5, relay the user's words mid-loop in the user turn instead: these two models can treat a system message placed right after a tool result as untrusted text and ignore it, so append the user's words as a `text` block after the last `tool_result` in the same `user` message, and keep harness reminders in a separate system message after it. The same mechanism covers tools (beta, `inline-tools-2026-09-15`): declare the full tool set in `tools` up front, then offer or withdraw a tool from a point in the conversation with `tool_addition` / `tool_removal` blocks in a `role: "system"` message — and a `tool_addition` can carry a full definition for a tool that was unknown at the first request. The `tools` array never changes, so the cached prefix stays intact. A `role: "system"` message can also be scoped to one turn with `clear_at: "next_user_message"` (beta, header `mid-conversation-system-clear-at-2026-08-21`): it carries system-prompt authority for the current turn, stops rendering once a later user message exists, stays in the array so nothing earlier changes, and costs no input tokens once cleared — the official home for per-turn reminders in a tool loop.

**Keep the history append-only (on all four current models).** Append each assistant turn exactly as returned (thinking blocks included, even empty ones), and do not edit earlier turns between requests. A thinking block is valid only against the content that came before it when it was produced: if you send it back after the `system` prompt, `tools`, or any earlier message has changed, it fails the prefix check. The best practices page's wording: `editing earlier messages, rebuilding system or tools, or summarizing older turns in place between requests invalidates every later thinking block`. What happens on a failure depends on the model and the account (Messages API, with `prefix_mismatch_behavior` unset; in the Message Batches API, where the table shows 400, the failing blocks are dropped instead):

| Model | Accounts created on or after 2026-08-31 | Accounts created earlier |
|---|---|---|
| Fable 5.2, Mythos 5.2 | 400 | 400 |
| Opus 5.5, Sonnet 5.5, Haiku 5.5 (Fable 5.1 is in this row too, which matters when you migrate from it) | 400 | No error; the invalid blocks reach the model as usual |

If you set `thinking.block_binding.prefix_mismatch_behavior: "drop_block"` (beta, `thinking-binding-controls-2026-08-01`), the API instead drops the invalid blocks and continues, and the response's `input_transformations` lists the dropped blocks. The Fable 5.2 migration guide presents this as a stopgap while you fix your code: dropping blocks can raise output tokens. The Fable 5.2 page says that in Anthropic's testing, when earlier thinking was dropped on every turn, the model thought again and used more output tokens, and that this is the worst case. For blocks from Sonnet 5.5, Haiku 5.5, Fable 5.2 and Mythos 5.2, an MCP server reached through the MCP connector returning a different tool list on the next request also counts as an edit when the toolset isn't pinned; on the Claude API the fix is to pin the server's tool list.

**So make prompt changes in new conversations only** (the Fable 5.2 and Haiku 5.5 pages both say so): adding text to the system prompt, deleting notes and rules your agents carry, and changing a tool description all change the prefix. To change instructions partway through a conversation, put them in the newest turn, or append a mid-conversation system message.

**Thinking blocks are also tied to the model that produced them.** Whether the reasoning survives a model switch depends on whether the target model can read the source model's blocks (Claude API; table from the preserved thinking page):

| Switching from | To Fable 5.2 / Mythos 5.2 | To Opus 5.5 | To Sonnet 5.5 | To Haiku 5.5 |
|---|---|---|---|---|
| Fable 5.2 / Mythos 5.2 | Kept | Lost | Lost | Lost |
| Opus 5.5 | Kept | Kept | Lost | Lost |
| Sonnet 5.5 | Kept | Kept | Kept | Lost |
| Haiku 5.5 | Kept | Kept | Kept | Kept |

The key point is that only Fable 5.2 and Mythos 5.2 can read Fable 5.2's blocks. When routing, a client-side retry or a fallback after a refusal moves a conversation from Fable 5.2 to another model (including its own fallback targets, Opus 4.8 and Opus 5), the turns after the move run without its reasoning. When a request carries a block the target model cannot read, the API drops it before the model sees it, the request succeeds as usual, and dropped blocks are not billed; the first turn after the switch may cost more and take longer. Blocks produced by Fable 5.2, Mythos 5.2, Sonnet 5.5 and Haiku 5.5 are also tied to the account that produced them: they work only in that account or in an account linked to it, and are dropped the same way when replayed from any other account.

Most edits that trip the prefix check also restart the cache — injecting and removing per-turn reminders, summarizing older turns in place, changing the system prompt mid-session — so the fixes are the same: turn-scoped system messages for reminders, mid-conversation system messages and tool changes instead of rewriting `system` or `tools`, and, for trimming context, the three approaches above that do not count as edits.

## When to promote a bash action to a dedicated tool

Bash gives broad leverage, but the harness sees only a command string — the same shape for every action. A dedicated tool gives it an action-specific hook with typed arguments it can intercept, gate, render, or audit. Four cases warrant a dedicated declarative tool:

1. **Security boundaries**: hard-to-reverse actions, such as external API calls, which can be gated by user confirmation
2. **Write operations needing staleness checks**: preventing overwrites of files that changed
3. **User-facing presentation**: modal displays, multiple options
4. **Structured logging, tracing, and replay**

(There is another path, which should only be used for tasks where users trust the general direction: use a secondary Claude instance to judge whether a bash command is safe. This is what Claude Code's auto mode does, and it can limit the need for dedicated tools. Dedicated tools can still earn their place for certain high-stakes actions.)

## Tool design

From Anthropic's articles "Writing effective tools for agents" and "Effective context engineering for AI agents":

- Each tool has a clear purpose, distinct from every other tool (`Make sure each tool you build has a clear, distinct purpose`). The test: `If a human engineer can't definitively say which tool should be used in a given situation, an AI agent can't be expected to do better`
- Unambiguous parameter names: `user_id` rather than `user`
- Meaningful identifiers in what tools return: resolving arbitrary UUIDs to interpretable language (or even a 0-indexed ID scheme) significantly improves Claude's precision in retrieval tasks by reducing hallucinations
- Namespacing by service and resource type (`asana_projects_search`, `asana_users_search`)
- Token-efficient returns: pagination, range selection, filtering, truncation, with sensible defaults (Claude Code's default cap is 25,000 tokens)
- Offer a `ResponseFormat` enum (`concise` / `detailed`) — the official example shows 206 tokens versus 72, about a third of the tokens
- Actionable error messages: say what to change and show a correctly formatted example, rather than returning a stack trace
- Write tool descriptions as onboarding for a new team member: make implicit context explicit — specialized query formats, terminology, resource relationships
