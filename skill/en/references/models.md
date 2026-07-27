# Tuning by model: Opus 5 and Fable 5

> Snippet numbers (4.2, 4.17, …) refer to `snippets.md`.
>
> Every English block in the Prescription column below is **verbatim official wording** — copy it as is rather than translating or tidying it.

## Claude Opus 5

Default effort `high`; thinking on by default; 1M token context window as both default and maximum.

| Known behavior | Prescription |
|---|---|
| User-facing responses run longer than prior Opus models'; **lowering effort does not reliably shorten visible output** | Prompt for conciseness explicitly (4.2) |
| Narrates readily during agentic work; per-message output is often long | Describe the cadence you want (4.3) |
| Files written to disk (reports, Markdown, summaries) run long | Add length calibration (4.4) |
| **Verifies its own work without being told to** | **Remove** every verification instruction (SKILL.md 3.4). Official wording is remove, not rewrite |
| Expands task scope, adding steps that were not requested | Constrain scope explicitly (4.5) |
| Delegates to subagents more readily than prior models | Give explicit criteria or a deterministic cap (4.10) |
| Narrates corrections to its own earlier statements more than prior models | `Only correct an earlier statement when the error would change the user's code, conclusions, or decisions. State corrections plainly and briefly, then continue the task. For slips that change nothing for the user, make the fix and move on without noting it.` |
| Follows "only report high-severity issues" **literally** in code review | Use 4.16; move filtering downstream |
| `low` and `medium` produce strong quality at a fraction of the tokens and latency of higher settings | **Start at the default `high`** and step up to `xhigh` only for demanding coding and agentic work. Use `low`/`medium` liberally as the primary control for token cost and latency wherever your evals show quality holds. If you carried settings over from an earlier model, run a fresh effort sweep |
| With thinking disabled, occasionally writes a tool call into user-facing text instead of emitting a `tool_use` block (the call never runs, and in agentic loops the leaked text stays in history and affects later turns) | **Keep thinking enabled** and control cost with lower effort. Officially: for most tasks, thinking enabled at `low` effort performs better than thinking disabled at similar cost |
| With thinking disabled, can emit `<thinking>` or other internal XML tags into visible output | Remove any rule telling the model not to think — it increases leakage, and naming the thinking tags specifically works less well than the general form |

For integrations that must keep thinking disabled, official guidance is **one combined instruction that mitigates both artifacts**:

```
When you use a tool, you may say a brief sentence first. If no tool can express what the user asked for, say so instead of guessing. Do not include internal or system XML tags in your response.
```

Thinking can be disabled only at effort `high` or below; a request setting `thinking: {"type": "disabled"}` at `xhigh` or `max` returns a 400 error. For most tasks, thinking enabled at `low` effort performs better than thinking disabled at similar cost. At `xhigh`/`max`, leave headroom in `max_tokens`; officially, starting at 64k and tuning from there is a reasonable default. This applies only when calling the API directly.

## Claude Fable 5 / Mythos 5

Default effort `high`; thinking is always on and cannot be configured.

Positioning: officially **Anthropic's most capable widely released model**. Aimed at problems that were previously too complex, long-running, or ambiguous for prior models; particularly effective at end-to-end work that takes a person hours, days, or weeks. The teams seeing the best outcomes apply it to their hardest unsolved problems; testing it only on simpler workloads tends to undersell its capability range.

> Note the split in the official model-selection advice: **start with Opus 5 for complex agentic coding and enterprise work; use Fable 5 for workloads that need the highest available capability.** "Most capable" and "what you should reach for by default" are not the same question.

| Known behavior | Prescription |
|---|---|
| **Individual requests can run for many minutes; autonomous runs can extend for hours** | Adjust client timeouts, streaming, and progress indicators before migrating; consider checking on runs asynchronously rather than blocking |
| Overplans when the task is ambiguous | `When you have enough information to act, act. Do not re-derive facts already established in the conversation, re-litigate a decision the user has already made, or narrate options you will not pursue in user-facing messages. If you are weighing a choice, give a recommendation, not an exhaustive survey. This does not apply to thinking blocks.` |
| At higher effort, performs unrequested tidying or refactoring | Use the **Fable 5 variant** in 4.6 |
| **Instruction following is strong enough to steer a class of behavior with one brief instruction** | Use short instructions instead of long enumerations |
| Can fabricate progress on long autonomous runs | **4.17 — the highest-value single snippet for this model** |
| Takes unrequested actions (drafting an email, creating defensive git branches) | Define boundaries explicitly (4.24) |
| Dispatches parallel subagents more readily | **Encourage it**, but give explicit delegation criteria; prefer asynchronous communication over blocking; long-lived subagents save time and cost through cache reads |
| Deep into a long session, occasionally ends a turn stating intent without issuing the tool call | A "continue" suffices; for autonomous pipelines use 4.18 |
| Suggests a new session when shown a remaining-token countdown | 4.19; better still, do not surface the countdown |
| **Skills written for prior models are often too prescriptive and can degrade output quality** | Review and prune. Fable 5 also updates skills on the fly based on what it learns |
| Instructing it to reproduce its reasoning triggers refusals | See SKILL.md 3.9 |

**Scaffolding advice:** start at the top of your difficulty range — pick a task harder than what you would assign to prior models, and have it scope the work, ask clarifying questions, and execute.

## Where the two models invert: verification

The single easiest thing to get wrong. Two official positions look contradictory; they address different situations.

| Situation | Do | Basis |
|---|---|---|
| **Opus 5, ordinary tasks** | **Remove** all verification and re-check instructions | It verifies itself; keeping them causes over-verification |
| **Fable 5, long autonomous runs** | **Add** explicit periodic self-checking | Official wording: `Establish a method for checking your own work at an interval of [X] as you build. Run this every [X interval], verifying your work with subagents against the specification.` |

The reconciling detail: officially, **separate, fresh-context verifier subagents tend to outperform self-critique**. So what you remove is "have the model look at its own work again"; what you add is "have a clean context check it." These are not in conflict.

(This also does not conflict with 4.10's "do not use subagents to verify or double-check your own work." Read 4.10 on the same freshness axis: what it forbids is **delegating self-review** — handing a subagent the model's own work and its own criteria, which just pays twice for a judgment Opus 5 already makes. A **separate verifier with independent criteria in a fresh context** stays permitted on both models. What Fable 5's long runs add is not permission but **periodicity**: run that check on an interval instead of once at the end.)

## Effort reference

`max` > `xhigh` > `high` (default) > `medium` > `low`

**Both models start at the default `high`; what differs is when to step up:**

- **Opus 5**: step up to `xhigh` for **demanding** coding and agentic work, or `max` when a task justifies unconstrained token spending. Use `low`/`medium` liberally as the primary control for token cost and response time wherever your evals show quality holds
- **Fable 5**: step up to `xhigh` only for the most capability-sensitive workloads, and step down to `medium`/`low` for routine work — its lower effort settings often exceed `xhigh` performance on prior models

Officially, `xhigh` is described as extended capability for long-horizon work: long-running agentic and coding tasks **over 30 minutes, with token budgets in the millions**. That is the concrete bar for "demanding." `low` is described as most efficient, for simpler tasks needing the best speed and lowest cost — subagents are the named example.

**If you carried effort settings over from an earlier model, run a fresh effort sweep on your own evals rather than reusing them.** This is stated for both models.

Points that matter:

- `effort: "high"` produces exactly the same behavior as omitting the parameter
- Effort is a behavioral signal, not a strict token budget; it affects text, tool calls, and thinking alike
- **Lower effort directly reduces the number of tool calls**, combines operations, and skips preamble
- Do not change effort mid-session with prompt caching enabled — it invalidates the cache
- Leave headroom in `max_tokens` — it is a hard limit on thinking plus response text combined. On Opus 5 this matters at `xhigh`/`max`, where official guidance is to start at 64k and tune from there; on Fable 5 it is called out at `high` and `xhigh`. Relevant only when calling the API directly
