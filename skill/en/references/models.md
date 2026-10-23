# Tuning by model: Opus 5.5 and Fable 5.1

> Snippet numbers (4.2, 4.17, …) refer to `snippets.md`.
>
> The backticked and fenced English text in the Prescription column below is **verbatim official wording** — copy it as is rather than translating or tidying it. The text around it is this file's summary.

## Contents

- Claude Opus 5.5 — stated for Opus 5.5; stated for Opus 5 and not restated for Opus 5.5
- Claude Fable 5.1 / Mythos 5.1 — stated for Fable 5.1; stated for Fable 5 and not restated for Fable 5.1
- Where the two models invert: verification
- Effort reference

## Claude Opus 5.5

Released September 22, 2026. Default effort **`medium`**; adaptive thinking always on and **cannot be disabled**; 1M token context window; 128K max output; $4 / $20 per MTok. Model ID `claude-opus-5-5`. Claude Opus 5 is now on the legacy list (still available) at $5 / $25.

**How the Opus 5 guidance carries over.** The Opus 5.5 prompting page says near the top: `Existing Claude Opus 5 prompts should perform well without changes, and the patterns in Prompting Claude Opus 5 remain a reasonable starting point.` The general prompting page adds a caveat for the model-specific techniques it lists, which this file applies here as well: `Where a technique names a specific model, treat it as measured on that model and re-check it against your own evals before applying it to another.` So this file keeps the two apart, exactly as it does for the Fable line: the first table is what the Opus 5.5 pages state; the second is what the Opus 5 page states and the Opus 5.5 pages neither restate nor retract. The migration guide's own instruction points the same way — `Instructions tuned for Claude Opus 5's behavior might no longer be needed`.

### Stated for Opus 5.5

| Known behavior | Prescription |
|---|---|
| **Default effort is `medium`**, where Opus 5's was `high` — a request that omits `effort` runs one tier lower than it did | Set `effort` explicitly and re-run your sweep. In Anthropic's testing, Opus 5.5 at `medium` **matches or exceeds Opus 5 at `high`** on coding and knowledge-work evaluations, and on several coding evaluations `low` comes close to Opus 5 at `high` at much lower cost |
| **Tends to think more per turn at a given level** than Opus 5, most of all at `xhigh` and `max` | Do not carry the Opus 5 value over — expect longer turns and more output tokens if you do. Reserve `xhigh` and `max` for work where you have **measured** a quality gain. To get less thinking, **lower the effort level first**: officially that works more reliably than prompt instructions |
| Thinking counts toward `max_tokens` even when the thinking text is not returned, so a limit sized for Opus 5 with thinking off can cut replies off | Leave headroom. For the long turns agentic coding can produce, `a max_tokens of 128,000, the model's maximum, has worked well in Anthropic's testing` |
| On long tasks with several parts it keeps the user updated as it works, and **some updates end the turn with text rather than a tool call** (`stop_reason: "end_turn"`). An unattended loop that reads that as the end of the task stops there | Treat a text-only end of turn as a report, not proof of completion. Keep the task's parts in a checklist the model updates, and if a turn ends with items still open and no blocker stated, send a short user message naming them (4.31 carries both the continuation message and the system-prompt addition). Stop after two or three automatic continuations. If something the model started is still running (a background command or subagent), wait for it and return its output as the next user message |
| **Text between tool calls comes back in `thinking` blocks**, not `text` blocks — at the default `thinking.display: "omitted"` their text is empty, so a client that renders only `text` blocks can look silent through a long agentic turn | Check the client first: set `display: "updates"` (beta, `thinking-display-updates-2026-08-18`) to receive a summary of each note. Then 4.3's Opus 5.5 block for cadence; 4.23's tool for content the user must get verbatim; and the quiet-turn reminder in 4.3 if turns still go silent. This is a **response-shape change from Opus 5** that fails no request |
| In workflow automation across several connected apps, **tends to get to work quickly** and on loosely specified tasks misses information the request did not point to | 4.32. In Anthropic's testing on multi-app automation tasks it completed noticeably more of them correctly with this instruction, at both `medium` and `max` effort, at the cost of slightly more tool calls and tokens. Because it tells the model to act on what it finds, keep untrusted content out of the records it searches |
| **Highly responsive to information about elapsed time** — usable to speed up a multi-agent setup | 4.33. Give a time budget and report elapsed time against it at the end of each message (`elapsed 340s / 1200s`); it paces to finish inside the budget and usually well before, so set it somewhat above the time you actually want spent. The budget is advisory — keep your own timeout for a hard stop |
| In chat, **thinks at length before answering**, which delays the first token; on later turns it sometimes goes back over an earlier answer even for a short follow-up | Consider removing system-prompt lines telling it to think carefully before answering — in Anthropic's testing in a chat product, removing such a line made replies start sooner with no clear quality decline. For the follow-up case, 4.34. Leave 4.34 out where you want it to keep re-examining earlier work |
| Resists indirect prompt injection better than any earlier Opus model, and with the right context is also robust against instructions inside **text the user pasted** — but it needs to be told which text that is | 4.35: wrap each pasted block in tags carrying the same short random ID, and add the accompanying system-prompt note. The tags are plain text and can be imitated, so treat this as one guardrail among others |
| Asked for frontend work with no design direction, falls back on a few default styles; a general "avoid a generic AI look" mostly swaps one default for another | 4.36 — name the specific patterns to avoid. Work iteratively: check which styles the first result used instead, and extend the list |
| **Reads charts, diagrams and screenshots considerably more precisely than Opus 5 without tools** | Re-test whether you still need vision scaffolding built for earlier models. For the densest inputs two things still add accuracy: higher-resolution images (most of all for technical drawings), and image-processing tools — a container holding the raw images with PIL/OpenCV, or, if that is too much overhead, a cropping tool alone, which still helps. It uses these tools more effectively at higher effort. Without tools, raising effort improves technical drawings but does little for charts |
| Safety classifiers include **biology, cybersecurity and reasoning extraction**. **Biology is new if you are coming from Opus 5**; the what's-new page introduces `reasoning_extraction` with `As on Claude Opus 5` | Handle `stop_reason: "refusal"` and configure fallback — but note that server-side fallback **does not retry** a `reasoning_extraction` decline; it returns it to you. Finding vulnerabilities in source code is allowed; high-risk dual-use cybersecurity activities are not. Everyday health and educational questions are unaffected by the biology classifier |
| A prompt that pushes the model to reproduce its reasoning in the response text can be declined with `reasoning_extraction` | See SKILL.md 3.9. Remove those instructions, set `display: "summarized"`, and read the reasoning from the thinking blocks instead. You can still ask for a short explanation of the answer or a summary of the actions taken |

**If your Opus 5 integration ran with thinking disabled**, Opus 5.5 does not accept `thinking: {"type": "disabled"}` (SKILL.md 3.2) and four changes go with the migration:

- **Start at `low` effort and measure.** At `low` the model keeps its thinking short; how often it skips thinking altogether depends on your prompts, so measure latency and quality on your own traffic and move to `medium` if quality drops. If time to first token still matters after that, officially a system prompt line such as `Answer directly without deliberating.` can reduce thinking further — measure quality when you add it, because less thinking can lower it
- **Remove instructions that stood in for thinking.** A prompt that asked the model to write its reasoning into the response as a substitute for thinking should drop that instruction and read [summarized thinking](https://platform.claude.com/docs/en/build-with-claude/thinking#summarized-thinking) blocks instead (`display: "summarized"`); such a prompt can otherwise be declined with `reasoning_extraction`
- **Re-test the thinking-disabled mitigations**, and **remove the no-thinking rule either way** (SKILL.md 3.10)
- **Read the response by block type**, not by position — a response may or may not begin with a `thinking` block

**Harness-level rules stated for Opus 5.5** (API only; see `agents.md`): keep the conversation history append-only, since editing anything before a thinking block invalidates it; change effort mid-conversation with per-message effort rather than the top-level value; forced `tool_choice` returns 400 (SKILL.md 3.12); and on the Claude API and Claude Models on Google Cloud (Google managed) computer use requires the `computer_toolset_20260801` toolset — the earlier `computer_20251124` tool returns 400, though it keeps working on Amazon Bedrock.

### Stated for Opus 5 and not restated for Opus 5.5

Each prescription below comes from the Opus 5 page (quoted text is verbatim), whose patterns the Opus 5.5 page says remain a reasonable starting point; whether the *behavior* still shows on Opus 5.5 is not stated either way. Treat these as measured on Opus 5 and re-check them on your own evals. Two of them point the opposite way from something Opus 5.5 *is* documented to do, and those are flagged.

| Known behavior (Opus 5) | Prescription |
|---|---|
| User-facing responses run longer than prior Opus models'; **lowering effort does not reliably shorten visible output** | Prompt for conciseness explicitly (4.2). Note the tension: Opus 5.5 officially `tends to complete the same task with fewer tokens` than Opus 5, so re-measure before assuming the length problem is still there |
| Narrates readily during agentic work; per-message output is often long | Describe the cadence you want (4.3). On Opus 5.5 the documented problem is the **opposite** — narration arrives in `thinking` blocks that are empty by default, so turns can look silent. Check `thinking.display` before adding anything |
| Files written to disk (reports, Markdown, summaries) are often longer than on prior models | Add length calibration (4.4) |
| **Verifies its own work without being told to** | **Remove** every verification instruction — SKILL.md 3.4, which explains why this is an Opus 5 rule and not an Opus 5.5 one. The best practices page names Opus 5 as the exception to its general "ask Claude to self-check" advice, and names no other model |
| Can expand task scope, adding steps that were not requested | For narrow tasks, constrain scope explicitly (4.5) |
| Delegates to subagents more readily than prior models | Give explicit criteria or a deterministic cap (4.10). In Claude Code and the Agent SDK the caps are `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH`, `CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS` and the SDK's `max_budget_usd` (Claude Code 2.1.217 or later); Claude Code adds a delegation instruction of its own on Opus 5 only with the `claude_code` system prompt preset, so with a custom or omitted system prompt add one yourself |
| Narrates corrections to its own earlier statements more than prior models | `Only correct an earlier statement when the error would change the user's code, conclusions, or decisions. State corrections plainly and briefly, then continue the task. For slips that change nothing for the user, make the fix and move on without noting it.` |
| Might follow "only report high-severity issues" **literally** in code review | Use 4.16; move filtering downstream. Early testers reported *stronger* code review on Opus 5.5 — more bugs caught, fewer false alarms — but the literal-instruction behavior is neither restated nor retracted, so the move still applies |
| `low` and `medium` produce strong quality at a fraction of the tokens and latency of higher settings | Superseded for Opus 5.5 by its own effort section above: the default is already `medium`, and the step-down target is `low` |

**Documented only on Opus 5** (thinking can be disabled there at `high` effort or below; on Opus 5.5 it cannot be disabled, and the Opus 5.5 page says these artifacts appear on Opus 5 only when thinking is disabled and to check whether you still need the instruction): with thinking disabled Opus 5 occasionally writes a tool call into user-facing text instead of emitting a `tool_use` block — the call never runs, and in agentic loops the leaked text stays in history and affects later turns — and it can emit `<thinking>` or other internal XML tags into visible output. The primary mitigation is to keep thinking enabled; for integrations that must keep it disabled, one combined instruction mitigates both:

```
When you use a tool, you may say a brief sentence first. If no tool can express what the user asked for, say so instead of guessing. Do not include internal or system XML tags in your response.
```

Officially, for most tasks thinking enabled at `low` effort performs better than thinking disabled at similar cost — which on Opus 5.5 is the only option anyway.

## Claude Fable 5.1 / Mythos 5.1

Default effort `high`; thinking is always on and cannot be turned off (adaptive thinking is the only mode). Claude Mythos 5.1 is the same model, offered only to organizations verified through Anthropic's verification programs (such as the Cyber Verification Program), with the same specs, pricing and prompting guidance; the one stated difference that affects prompting or harness code is that Mythos 5.1 does not run the append-only conversation check on thinking blocks (availability differs as well). Released September 1, 2026; Fable 5 is now listed as a legacy model, still available. In Claude Code, `claude-fable-5-1` is the default Fable model since 2.1.257 (2026-09-01).

Positioning: officially **Anthropic's most capable model open to all customers**. The official selection advice is the other half of the picture: **for most workloads, start with Claude Opus 5.5; use Claude Fable 5.1 for demanding reasoning and long-horizon agentic work, or when your evals on Claude Opus 5.5 at higher effort still fall short.** The selection matrix names its use cases as agent sessions that run for hours, multistep deep research, and analysis carried through to a finished document, spreadsheet, or deck. "Most capable" and what you should reach for by default are not the same question. Its capability gains over Fable 5 are largest at higher effort and concentrate in agentic coding over long sessions, document/spreadsheet/slide work, multistep research, vision, long-context work, and computer use.

**How the Fable 5 guidance carries over.** The Fable 5.1 prompting page says near the top: `Your existing Claude Fable 5 prompts should perform well on Claude Fable 5.1 without changes, but a handful of behavioral differences are worth knowing about.` With the same best-practices caveat quoted above, this file splits the Fable material the same way.

### Stated for Fable 5.1

| Known behavior | Prescription |
|---|---|
| **Fewer user-facing updates during long tool-calling turns** than Fable 5, more so at higher effort and in longer tool chains: the agent goes quiet for minutes at a time, or the final message covers only the last step | Do this in order: make sure updates reach your client at all (they arrive as progress-update `thinking` blocks, empty under the default `thinking.display: "omitted"` — set `"updates"`, beta); remove prompt lines that suppress narration ("hold all findings for the final response"); only then add the 4.3 Fable 5.1 block. Comparative latency is officially "Slower," so adjust timeouts and progress indicators as you would have for Fable 5 |
| On complex asynchronous workloads, sometimes **describes the next step instead of doing it** ("Next, I'll …") or asks permission for a step the request already covered — users have to reply "continue" | 4.18, both blocks together; the first alone keeps most of the effect. The opening sentence carries much of it — keep it as written |
| When asked to implement an open-ended feature, delivers what was asked **and sometimes more**: it might fix nearby code, extend behavior the task did not mention, or commit more test files than the change warrants | The 4.6 Fable 5.1 block. Officially, unrequested additions and committed test code drop substantially with no measurable change in task success |
| In coding and computer-use loops where the next independent calls are only implied, may issue **one tool call per turn** where Fable 5 batched several — costs tokens and round trips, not quality | The one-sentence nudge in 4.9, re-sent each turn as a turn-scoped system message (beta) or after the `tool_result` blocks |
| **Denser prose** in places than Fable 5: longer sentences, fewer paragraph breaks | 4.26. The verbosity snippets (4.2–4.4) address length, not density |
| Tends to use **less formatting in chat** than earlier models — less bold, fewer headers and lists — so anti-formatting rules written for those models can suppress structure the content needs | Remove them, or replace them with the 4.13 Fable 5.1 block that says when formatting is appropriate |
| When summarizing documents, more likely to reproduce source passages **without marking them as quotations** | 4.30 — one complete worked example in the system prompt |
| At `low` effort, **answers from memory more often** instead of calling search or retrieval | Raise effort for the affected turns (per-message effort, beta), or 4.27 |
| More likely to **rewrite a whole file** for a small change — usually the same result, but unless the file is short or most of it is changing, more output tokens and time | 4.28 |
| At `xhigh`/`max`, may **draft a long deliverable in thinking and then write it again** as the reply | Run such requests at `high`; if you must go higher, set `max_tokens` for thinking plus reply and append 4.29 (API only) |
| Prompts, skills and tool descriptions that ask it to write out its thinking or reasoning can be declined with `reasoning_extraction` — the thinking page names Fable 5.1, Opus 5.5, Opus 5, Sonnet 5.5 and Fable 5 | Ask for a short explanation or a summary of the actions taken instead, or read the reasoning from summarized thinking blocks (`display: "summarized"`). See SKILL.md 3.9 |
| Safety classifiers cover the same `stop_details` categories as Fable 5 — in the Fable 5.1 migration guide's words, `a broader set than Claude Opus 5's classifiers` — and the guide points to the refusal category table for the full set (`cyber`, `bio`, `frontier_llm`, `reasoning_extraction`, `general_harms`) rather than listing it per model. The Opus 5.5 pages also describe a broader set than Opus 5's and name biology, cybersecurity and reasoning extraction. Benign work can trigger them; finding vulnerabilities in source code is permitted; false positives are fewer than Fable 5's at launch | Handle `stop_reason: "refusal"` and configure fallback. To reduce false positives: ask "Are there any bugs in this program?" rather than "Does this program compile without errors?"; give context for lesser-known languages; keep base64-encoded data out of tool output |
| Forced tool use (`tool_choice` `any` / `tool`) returns 400 | See SKILL.md 3.12 — now true on Opus 5.5 as well |
| **Lead agent idles while subagents run** (the page's own symptom line) | Let it keep working: the tool that starts a subagent returns immediately, results come back in a later `user` message, and a separate tool waits when the lead wants to. Officially, on coding tasks this lowers average time to completion at similar quality, token usage, and cost; the model still often chooses to wait |
| Dense charts and images | For the full benefit, give it a container with the raw images and PIL/OpenCV so it can iteratively analyze, crop, and verify; if a container is too much overhead, an image-cropping tool alone delivers most of the uplift |

Harness-level rules stated for Fable 5.1 (API only; see `agents.md`): keep the conversation history append-only, since editing earlier turns invalidates its thinking blocks (Mythos 5.1 does not run that check); change effort mid-conversation with per-message effort rather than the top-level value; re-run the effort sweep even if you ran one on Fable 5.

### Stated for Fable 5 and not restated for Fable 5.1

Each prescription below comes from the Fable 5 page (quoted text is verbatim); the Fable 5.1 page says existing Fable 5 prompts should perform well on Fable 5.1 without changes; whether the *behavior* still shows on Fable 5.1 is not stated either way. Treat these as measured on Fable 5 and re-check them on your own evals.

| Known behavior (Fable 5) | Prescription |
|---|---|
| **Individual requests on hard tasks can run for many minutes at higher effort; autonomous runs can extend for hours** | Adjust client timeouts, streaming, and progress indicators before migrating; consider checking on runs asynchronously rather than blocking. The Fable 5.1 pages corroborate the scale ("quiet for minutes at a time"; comparative latency "Slower") without restating the sentence |
| Overplans when the task is ambiguous | `When you have enough information to act, act. Do not re-derive facts already established in the conversation, re-litigate a decision the user has already made, or narrate options you will not pursue in user-facing messages. If you are weighing a choice, give a recommendation, not an exhaustive survey. This does not apply to thinking blocks.` |
| At higher effort, performs unrequested tidying or refactoring | The **Fable 5 variant** in 4.6. (The Fable 5.1 block in the same entry is the measured 5.1 prescription for the adjacent behavior — unrequested fixes and extra test files) |
| **Instruction following is strong enough to steer a class of behavior with one brief instruction** | Short instructions instead of long enumerations (SKILL.md 3.3) |
| Can fabricate progress on long autonomous runs | 4.17 — "nearly eliminated fabricated status reports" was measured on Fable 5 |
| Can occasionally take unrequested actions (drafting an email, creating defensive git branches) | 4.24 standalone; on Fable 5.1 both halves of it ship inside the 4.18 block |
| Dispatches parallel subagents more readily | **Encourage it**, but give explicit delegation criteria; prefer asynchronous communication over blocking; long-lived subagents save time and cost through cache reads. (The Fable 5.1 page's harness advice on this is in the first table) |
| Can occasionally suggest a new session, most often when shown a remaining-token countdown | 4.19; better still, do not surface the countdown. (The Fable 5.1 autonomous block in 4.18 adds `Do not stop because the context or session is long.`) |
| **Skills written for prior models are often too prescriptive and can degrade output quality** | Review, and consider removing older instructions if default performance is better. Fable 5 also updates skills on the fly based on what it learns |

**Scaffolding advice (Fable 5 page, not restated for Fable 5.1):** start at the top of your difficulty range — pick a task harder than what you would assign to prior models, and have it scope the work, ask clarifying questions, and execute.

## Where the two models invert: verification

The single easiest thing to get wrong. Two official positions look contradictory; they address different situations.

| Situation | Do | Basis |
|---|---|---|
| **Opus 5, ordinary tasks** | **Remove** all verification and re-check instructions | It verifies itself; keeping them causes over-verification. Stated for Opus 5 and for no other model |
| **Opus 5.5, ordinary tasks** | **Do not add them reflexively; measure before removing inherited ones** | The Opus 5.5 pages neither restate the over-verification finding nor retract it, and the best practices page's general advice still favours a self-check. See SKILL.md 3.4 |
| **Fable 5.1, long autonomous runs** | **Add** explicit periodic self-checking | Official wording, published on the Fable 5 page and not restated or retracted on the Fable 5.1 page (which says existing Fable 5 prompts should perform well without changes): `Establish a method for checking your own work at an interval of [X] as you build. Run this every [X interval], verifying your work with subagents against the specification.` |
| **Opus 5.5, unattended long runs** | **Add a completion check** | Against early stops, the Opus 5.5 page suggests keeping the task's parts in a checklist the model updates; you can also state the completion condition up front and `have a separate, smaller model check the conversation against it at each end of turn`. This checks completion, not quality — a different problem from the over-verification 3.4 is about |

The reconciling detail: officially (Fable 5 page, same paragraph), **separate, fresh-context verifier subagents tend to outperform self-critique**. On Fable long runs, then, what you add is a clean context checking the work; on Opus 5 the page says to remove verification instructions, `use a subagent to verify` among them, because they cause over-verification there. These apply to different models and are not in conflict.

(This also does not conflict with 4.10's "do not use subagents to verify or double-check your own work." Read 4.10 on the same freshness axis: what it forbids is **delegating self-review** — handing a subagent the model's own work and its own criteria, which just pays twice for a judgment Opus 5 already makes. That a **separate verifier with independent criteria in a fresh context** remains useful is this skill's reading, not a statement on any page — and the Opus 5 page lists "use a subagent to verify" among the verification instructions to remove, so on Opus 5 add one only where you have measured that it helps. What the Fable line's long runs add is **periodicity**: run that check on an interval instead of once at the end.)

## Effort reference

`max` > `xhigh` > `high` > `medium` > `low`

**The default is model-specific: `medium` on Opus 5.5, `high` on Fable 5.1.** Officially, `"medium"` on Claude Opus 5.5 and Claude Haiku 5.5, `"high"` on other models. That difference is the single most consequential thing on this page — a request that omits `effort` runs one tier lower on Opus 5.5 than the same request did on Opus 5.

- **Opus 5.5**: start at `medium`, the default. Step up to `xhigh` or `max` only for work where you have **measured** a quality gain — officially, `Reserve xhigh and max for work where you've measured a quality gain`. In Anthropic's testing **Opus 5.5 at `medium` matches or exceeds Opus 5 at `high`** on coding and knowledge-work evaluations, so the equivalent of an old `high` setting is the new default, not `high`; and on several coding evaluations `low` comes close to Opus 5 at `high` at much lower cost, so step down to `low` where evals show quality holds
- **Fable 5.1**: start at `high`, the default. Step up to `xhigh` or `max` for the most capability-sensitive agentic and coding work, and step down to `medium`/`low` for routine or latency-sensitive work once your evals show quality holds. Officially its gains over Fable 5 show across effort levels and are largest at the higher settings; at `medium` results roughly match Fable 5 at lower cost, and at `low` it is often competitive with Opus and Sonnet models on cost per task while scoring higher — so include it in the comparison wherever you would otherwise run a smaller model at a higher effort level

Officially, `xhigh` is described as extended capability for long-horizon work: long-running agentic and coding tasks **over 30 minutes, with token budgets in the millions**. That is the concrete bar for "demanding." `low` is described as most efficient, for simpler tasks needing the best speed and lowest cost — subagents are the named example.

**If you carried effort settings over from another model, run a fresh effort sweep on your own evals rather than reusing them.** Stated for Opus 5.5 in the strongest form — `test several levels against your own evals rather than carrying over the setting you used on Claude Opus 5` — and for Fable 5.1 on its own page: re-run the sweep even if you already ran one on Fable 5, because effort level names do not correspond to the same amount of thinking across models.

Two ordering rules, both official:

- **Tuning effort is often a better lever than switching models.** Run the ladder on the model you already have before reaching for a different one
- If the model thinks more often than the workload needs, **lower the effort level before reaching for prompt-based steering**. Prompt wording is the second lever, not the first, and its effectiveness can be sensitive to exact wording

Points that matter:

- **Setting `effort` to the model's default produces exactly the same behavior as omitting the parameter** — `"medium"` on Opus 5.5, `"high"` on Fable 5.1
- Effort is a behavioral signal, not a strict token budget; it affects text, tool calls, and thinking alike
- **Lower effort means fewer and terser tool calls**; lower levels tend to combine operations and skip preamble, and higher levels might make more tool calls, explain the plan first, and give detailed summaries
- At a given level **Opus 5.5 tends to think more per turn than Opus 5**, most of all at `xhigh` and `max`
- Changing the **top-level** effort between requests invalidates the prompt cache. Opus 5.5, Fable 5.1, Mythos 5.1 and Opus 5 (and Sonnet 5.5 and Haiku 5.5) support a **per-message effort change** (beta, `mid-conversation-output-config-2026-07-01`: a `role: "system"` message with empty `content` and `output_config.effort`) that preserves the cache. Placed directly after a user turn with new input it sets the level for the reply to that turn; anywhere else it takes effect from the next user turn with new input — a user turn holding only `tool_result` blocks does not count — and the new level holds until a later message changes it. On Fable 5.1 that form is also officially preferred because a top-level change steers the model less reliably — its earlier replies were written at the previous level and it tends to stay consistent with them. Fable 5 does not support it (400)
- Leave headroom in `max_tokens` — it is a hard limit on thinking plus response text combined, and thinking counts toward it even when the thinking text is not returned to you. On Opus 5.5, for the long turns agentic coding can produce, `a max_tokens of 128,000, the model's maximum, has worked well in Anthropic's testing`, and at `xhigh` or `max` the migration guide's starting point is 64k; on Fable 5.1 the effort page says to set a large value at `high` and above. Relevant only when calling the API directly
- In Claude Code, `/effort` is per-model, and a newly released model such as Opus 5.5 starts at its own default until the user picks a level (2.1.280)
