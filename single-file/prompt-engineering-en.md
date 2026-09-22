# Writing prompts — single-file edition


**Models in scope:** Claude Opus 5 and Claude Fable 5.1 (including Mythos 5.1). Guidance specific to earlier models is out of scope for this file. Several Fable 5.1 prescriptions are still published only on the Fable 5 page; they are kept because the Fable 5.1 page says existing Fable 5 prompts should perform well on Fable 5.1 without changes, and each is marked as such at the point of use.

**Why this skill exists:** several widely repeated prompt techniques are now obsolete on these models, rejected by the API, or actively reversed in effect. Your training instincts probably still carry some of them. **When instinct conflicts with Section 3, this file wins.**

**Basis:** Anthropic's official documentation and official/employee public material, snapshot 2026-09-03. For specific API parameters, confirm against the live docs — those are the only surface that tracks model updates.

---


> **Relationship to the skill**: this file is the **merged single-file edition** of the `prompt-engineering` skill, generated from its SKILL.md and five reference files. The two are kept consistent.
> If you have the skill installed, use the skill — it loads on demand and costs less context. This file is for cases where you do not: pasting into another session, another model, or offline reading.


## Contents

| # | Section |
|---|---|
| **0** | Workflow |
| **1** | Classify the scenario first |
| **2** | Select model and effort yourself |
| **3** | Obsolete techniques (hard rules) |
| **4** | Official prompt snippet library |
| **5** | General skeleton and structure |
| **6** | Tuning by model: Opus 5 and Fable 5.1 |
| **7** | Scenario A: task prompts destined for Claude Code |
| **8** | Scenario B: task prompts for desktop chat or a Cowork session |
| **9** | Agents and long-running tasks: context engineering |
| **10** | Three request types, handled differently |
| **11** | Pre-delivery checklist |
| **12** | Symptom to prescription |
| **13** | Meta-principles (for anything this file does not cover) |

> Snippet numbers 4.1–4.30 all live in Section 4; every cross-reference points there.

---

## 0. Workflow

1. **Classify the target scenario** (Section 1) — determines whether to read Section 7 or Section 8
2. **Select model and effort yourself** (Section 2) — do not hand this question back to the user
3. **Fill in missing variables** (end of Section 1) — ask only what you genuinely cannot proceed without
4. **Draft**: read the matching scenario section (7 or 8), and Section 4 when you need a ready-made block
5. **Run the Section 11 checklist**
6. **Deliver**: the prompt itself, plus a delivery note covering five things — which model and effort you chose, which assumptions you made on the user's behalf, what is most likely to need tuning, whether anything can only be supplied by the user, and — **if the prompt embeds official English snippets, a reminder not to translate them**. Two or three sentences when that fits; **when there are many assumptions, err on the side of a longer note rather than hiding them** — the user cannot judge whether to change an assumption they cannot see.

---

## 1. Classify the scenario first

The user describes work in Claude desktop, and the resulting prompt goes to one of two places. **Decide which, because the writing differs substantially.**

| Signal | Scenario | Read this |
|---|---|---|
| Codebase, files, git, running tests, refactoring, bug fixing, scaffolding a project; the user says they will paste it into Claude Code or VS Code | **A: a task prompt for Claude Code** | Section 7 |
| Research, analysis, writing, organizing source material, producing documents/spreadsheets/decks, scheduling, information synthesis; run directly in a desktop chat or Cowork session | **B: a task prompt for chat or Cowork** | Section 8 |
| The user drops a prompt into an ongoing conversation and asks whether it looks right | **C: an inline check** | See Section 10, lightweight path |

When it is not obvious, decide by the **verb** in the request: touching code is A, everything else is B. When both apply (for example, "analyze this repo's tech debt and write it up"), decide by the **final deliverable** — a code change is A, a document is B.

### Filling in variables

Ask only what you genuinely cannot proceed without. When something is missing but reasonably inferable from context, **assume it and state the assumption on delivery** rather than handing the question back.

"Must be settled" below means **this value must have a definite answer**, not that you must ask a question. Infer where you can, and record what you inferred in the delivery note.

| Variable | Requirement |
|---|---|
| **Success criteria** | **Must be settled.** Without a decidable success criterion you cannot write a good prompt and cannot check your own work |
| **Output shape and length** | **Must be settled** when the deliverable is a document or report. Claude Opus 5's default responses run long; unstated, they will overrun |
| **Available tools or files** | Must be settled for Scenario A (which repo, which files); for Scenario B, depends on whether source material must be read |
| Target model and effort | **Do not ask.** Decide it yourself per Section 2 and report the decision |

Ask only when a value is **neither inferable nor safe to guess** — that is, when guessing wrong would invalidate the whole prompt. When several such values exist, use AskUserQuestion to ask them **all at once**.

---

## 2. Select model and effort yourself

This step is mandatory before drafting, and the conclusion belongs in the delivery note.

### Choosing the model

| Task profile | Choose | Basis |
|---|---|---|
| Complex agentic coding, multi-file features, larger refactors, end-to-end feature work | **Opus 5** | Official positioning: strongest on difficult coding tasks; it completes full tasks rather than leaving stubs or placeholders |
| Code review and bug-finding | **Opus 5** | High precision and recall; accuracy holds at lower effort settings |
| Enterprise knowledge work, documents, spreadsheets, decks, long-context analysis | **Opus 5** | 1M token context window as both default and maximum, with instruction following consistent throughout the window |
| **Demanding reasoning and long-horizon agentic work**: agent sessions that run for hours, multistep deep research, analysis carried through to a finished document, spreadsheet, or deck; or when the user's evals on Opus 5 at higher effort still fall short | **Fable 5.1** | The official selection advice (models overview, and the choosing-a-model matrix). Officially Anthropic's most capable widely released model; its gains over Fable 5 are largest at higher effort and concentrate in agentic coding over long sessions, document/spreadsheet/slide work, multistep research, vision, long context, and computer use |
| Work that touches the safety-classifier categories — benign work in those areas can also trigger them | **Plan for refusals**: `cyber` on both models; `bio`, `frontier_llm`, `general_harms` (and `reasoning_extraction`) on Fable 5.1 only | Opus 5 runs cybersecurity classifiers only; Fable 5.1's cover the same five categories as Fable 5. A declined request returns `stop_reason: "refusal"`, so configure fallback (for Fable 5.1: Opus 4.8 or Opus 5). Finding vulnerabilities in source code is permitted |

Default to **Opus 5** — officially, most workloads start there. Recommend Fable 5.1 for demanding reasoning and long-horizon agentic work, or when Opus 5 at higher effort still falls short on the user's own evals; and warn the user that it is the slowest model in the lineup and, during long tool-calling turns, can go quiet for minutes at a time, which changes how timeouts and interaction are handled.

### Choosing effort

**Both models start at the default `high`.** The decision you are actually making is when to step *up*.

| Model | Start | Step up when | Step down when |
|---|---|---|---|
| **Opus 5** | **`high`** (the default) | The coding or agentic work is **demanding** → `xhigh`. A task that justifies unconstrained token spending → `max` | Cost or latency matters and your evals show quality holds → `medium`/`low`, used liberally as the primary control for token cost and response time. They produce strong quality at a fraction of the tokens and latency of higher settings |
| **Fable 5.1** | **`high`** (the default) | The most capability-sensitive agentic and coding work → `xhigh` or `max`. Its gains over Fable 5 are largest at the higher settings | Routine or latency-sensitive work once your evals show quality holds → `medium`/`low`. Officially, `medium` roughly matches Fable 5 at lower cost, and `low` is often competitive with Opus and Sonnet models on cost per task while scoring higher — include it wherever you would otherwise run a smaller model at a higher effort level |

If you are carrying effort settings over from an earlier model, the official instruction is to **run a fresh effort sweep on your own evals** rather than reusing them — on Fable 5.1 even if you already ran one on Fable 5, because effort level names do not correspond to the same amount of thinking across models.

**What "demanding agentic work" means** (this is the hinge of the step-up decision — settle it first). Two conditions, and you need both:

1. **Agentic**: the task requires the model to **decide its own next step and drive forward through repeated tool calls**. Typical markers are many tool-call rounds, plans that change based on intermediate results, and long single runs. Writing code, running tests and fixing failures, and repeatedly searching and cross-checking across many sources all count. By contrast, when the material is already assembled and the job is to produce one artifact in a pass (write a report, build a sheet, translate, rewrite), it is **not** agentic even when the artifact is large.
2. **Demanding**: the run is long. Officially `xhigh` is for long-running agentic and coding tasks — over 30 minutes, with token budgets in the millions. A three-tool-call bug fix is agentic but not demanding.

**When it is unclear whether a task is agentic, use this tiebreaker: who decides where to look?** If the user assembled the material and the model just reads through it, not agentic. If the model decides what to go find, and then decides what to look for next based on what it found, agentic.

**When in doubt, stay at `high`.** Being one tier low on a task that turns out to be demanding costs less than running every routine task at `xhigh`.

**Where these parameters live** (what you can actually set differs by surface):

| | Scenario A: Claude Code | Scenario B: chat / Cowork | Raw API |
|---|---|---|---|
| effort | Set in the UI or via a slash command | Set in the UI | `output_config: {effort}` |
| thinking | Managed by the client | Managed by the client | `thinking: {type: "adaptive"}` |
| `max_tokens` | **Not reachable** | **Not reachable** | You set it; officially, at `xhigh`/`max`, starting at 64k and tuning from there is a reasonable default |

So for Scenario A and B delivery notes, **naming the model and the effort tier is enough** — do not write `max_tokens` or `thinking` into them, because the user has no such control on that surface and it only causes confusion. The API-field details below apply only when calling the API directly.

Things you need to know:

- `effort: "high"` produces **exactly the same behavior as omitting the parameter entirely**
- Effort is a **behavioral signal, not a strict token budget**; it affects all tokens in the response — text, tool calls, and thinking
- **Lower effort directly reduces the number of tool calls**, combines operations, and skips preamble; higher effort means more tool calls, plans explained before acting, and detailed summaries
- **On Opus 5, effort controls how much the model thinks, not how much it says.** To shorten responses, prompt for it (snippet 4.2); lowering effort does not reliably work. (The Fable 5.1 pages make no such statement; its documented text-shape problem is density, snippet 4.26)
- Changing the **top-level** effort between requests invalidates the prompt cache. Fable 5.1 and Opus 5 support a per-message effort change (beta) that preserves it — API only, see Section 6
- At `xhigh`/`max`, leave headroom in `max_tokens` (it is a hard limit on thinking plus response text combined)

Model-specific behaviors and their prescriptions are in Section 6.

---

## 3. Obsolete techniques (hard rules)

These are not style preferences. Each one either errors at the API layer or reverses the behavior you intended.

### 3.1 Do not use prefill (prefilled assistant responses)
**Neither Opus 5 nor Fable 5.1 supports it.** Prefilling the last assistant turn returns a 400 error on both (the Fable 5.1 what's-new page lists it under behaviors unchanged from Fable 5).

The official replacements are **structured outputs, system prompt instructions, or `output_config.format`**. In practice: for structured output use Structured Outputs, `output_config.format`, or state the schema directly; to eliminate preamble write `Respond directly without preamble. Do not start with phrases like 'Here is...', 'Based on...', etc.` (verbatim from the prompting best practices page, "Migrating away from prefilled responses"); for classification use a tool with an enum field; for continuations move the prior text into the user message.

### 3.2 Do not use `budget_tokens`
Neither model supports manual extended thinking; setting it returns 400. On Opus 5 use `thinking: {type: "adaptive"}` with `output_config: {effort: "..."}`. On Fable 5.1 thinking is always on and cannot be configured — `{"type": "enabled"}` with `budget_tokens` and `{"type": "disabled"}` both return 400; omit `thinking` or send `{"type": "adaptive"}`, and control depth with `effort` alone. For a hard ceiling use `max_tokens`.

### 3.3 Do not stack instructions where context would do
The core principle: **give context, not a long list of instructions.**

The Fable 5 page states it plainly — instruction following is improved enough that you can steer most behaviors with a brief instruction rather than enumerating each behavior by name — and the Fable 5.1 page, which says existing Fable 5 prompts should perform well on Fable 5.1 without changes, does not retract it. On that basis the Claude Code team cut their system prompt by more than 80% with no loss of performance.

Rather than enumerating ten "do not X" clauses, write one positive description and state the intent. Change `Default to using [tool]` to `Use [tool] when it would enhance your understanding of the problem`. Express how a tool should be used through the tool description and its parameter design, not through repeated reminders in the system prompt.

> One exception: Claude Code's best practices still say that if Claude keeps skipping one instruction in **CLAUDE.md**, you can add emphasis such as `IMPORTANT` to that line alone — and that if you emphasize many lines, none of them stands out. The same page warns that if CLAUDE.md is too long, Claude ignores half of it because important rules get lost in the noise.

### 3.4 Do not tell Opus 5 to "check its work again"
Opus 5 verifies its own work without being told to. Keeping `double-check your answer` / `re-verify before responding` / `include a final verification step` causes **over-verification**: wasted tokens with no gain in quality.

The official wording is to **remove these instructions rather than rewriting them**. The same applies to legacy harness scaffolding that adds separate verification steps.

**What you are removing is "have the model look at its own work again," not "check against external facts."**

| Remove (model self-review) | Keep (checking against external ground truth) |
|---|---|
| "check once more that nothing is missing" | Running tests, build exit codes, screenshot comparison (Scenario A) |
| "re-verify your answer" | Reconciling what source A says against source B, and stating which you took (Scenario B) |
| "use a subagent to double-check your own work" — that is, one given the model's own work *and its own criteria* | Every figure must trace to a specific location in a source document. A **fresh-context verifier with independent criteria** is not what this row removes — see Section 6 |

The test is simple: **if the basis for the check comes from the model itself, remove it; if it comes from something externally verifiable, keep it.** In Scenario B research and reporting tasks, cross-source reconciliation *is* the task, not self-verification — do not delete it by mistake.

Long autonomous runs on Fable 5.1 go the other way and want explicit periodic verification — see Section 6 (the instruction is published on the Fable 5 page and carries over).

### 3.5 Do not stack negative constraints
`Never do X` lands with unusual force on Claude, and when it collides with a later user instruction the model gets badly confused. Give a positive description with a reason:

- ✗ `NEVER use ellipses`
- ✓ `Your response will be read aloud by a text-to-speech engine, so never use ellipses since the text-to-speech engine will not know how to pronounce them.`

The second version still contains "never" — but it hangs on a **reason**. Once the model knows why, it can decide for itself in edge cases the rule never covered.

### 3.6 Do not substitute absolute rules for judgment
- ✗ `Never write multi-paragraph docstrings or multi-line comment blocks — one short line max`
- ✓ `Write code that reads like the surrounding code: match its comment density, naming, and idiom`

### 3.7 Do not repeat an instruction across the system prompt and a tool description
Guidance on how to use a tool belongs **in the tool description, once**.

> **One exception: the invocation trigger.** Snippet 4.23 is the official case — the Fable 5 page notes that Fable 5 rarely calls `send_to_user` from the tool description alone, so the official guidance pairs the tool with a short system-prompt line telling it *when* to reach for the tool. What this rule forbids is restating the same **usage guidance** in two places, not pairing a tool description with a trigger the model demonstrably needs.

### 3.8 Do not rely on `temperature` for variety
On both models in scope, non-default `temperature`, `top_p`, or `top_k` values return 400 (stated in the Opus 5 migration guide and, for Fable 5.1, under behaviors unchanged from Fable 5). Get variety from the prompt instead — the most effective move is to have the model **propose options before committing**:
```
Before building, propose 4 distinct visual directions tailored to this brief (each as: bg hex / accent hex / typeface, plus a one-line rationale). Ask the user to pick one, then implement only that direction.
```
(Snippet 4.25 — verbatim from the Claude Sonnet 5 page. Copy it as is.)

### 3.9 Do not ask the model to reproduce or display its internal reasoning
Instructions like "show your thinking" or "explain your reasoning process" **can trigger** the `reasoning_extraction` refusal category on **Fable 5.1** (stated for Fable 5.1 and Fable 5 alike on the thinking page), raising the rate of refused requests and fallbacks. For reasoning visibility, read the structured `thinking` blocks from adaptive thinking instead; to surface progress during a long run, use a send-to-user tool (snippet 4.23).

### 3.10 Do not write "do not think" or "do not reason"
**With thinking disabled**, Opus 5 can emit `<thinking>` and other internal XML tags into its visible response, and a system-prompt rule telling it not to think **increases** that leakage. (Opus 5 runs thinking-on by default and cannot disable it at `xhigh`/`max`, so this only applies to integrations that deliberately turn thinking off.) Use the general form, and do not name the thinking tags specifically — naming them works less well:
```
Do not include internal or system XML tags in your response.
```

### 3.11 Audit inherited prompts for prefill
A prompt you did not write yourself — an older template, a pre-4.6 codebase, anything generated by tooling you do not control — can still end with a partial assistant turn. That returns 400 on both models in scope. **Re-run this section over anything inherited**, and check the trailing turn first. Claude Code ships a `prompt-audit` subcommand in its `claude-api` skill "for auditing prompts and tool descriptions for patterns written for older models" (changelog 2.1.221, 2026-08-04); it is a first pass, not a substitute for this section.

### 3.12 Do not force tool use on Fable 5.1
`tool_choice` of type `any` or `tool` returns a 400 `invalid_request_error` on Fable 5.1 and Mythos 5.1 (thinking is always on, and a forced call would skip it); `auto` and `none` are unchanged, and Opus 5 is unaffected. For schema-valid JSON keep `tool_choice: {"type": "auto"}` and set `strict: true` with strict tool use, or move the schema to structured outputs. To make the model call a tool rather than reply in text, state in the prompt when the tool applies — the official example is `Use the get_weather tool to answer` — which Fable 5.1 follows reliably.

---

## 4. Official prompt snippet library

> Every English snippet below **except 4.1** is **verbatim text from Anthropic's official documentation**, checked character by character; 4.1 is a decision table, not a quotation. Every quoted row links to the page it came from — the docs change, so treat the live page as authoritative.
>
> **Two entries come from a page outside this document's stated scope.** 4.16 and 4.25 are verbatim from the **Claude Sonnet 5** page. Sonnet 5 is a current model, not a superseded one, and both blocks are model-agnostic prompt text — but the provenance is flagged at each entry so you can decide for yourself.
>
> **Seven entries and one variant are published only on the Fable 5 page.** 4.17, 4.19–4.24, and the Fable 5 variant in 4.6 are not republished on the Fable 5.1 page. They are kept because that page opens with `Your existing Claude Fable 5 prompts should perform well on Claude Fable 5.1 without changes`, and each of them says so at the point of use. 4.18 is the one entry the Fable 5.1 page rewrote; it carries the new wording. 4.26–4.30 are new with Fable 5.1.

### Section map

- How to use these
- Snippet index — the table of contents for 4.1–4.30
- The snippets, 4.1–4.30

### How to use these

**Two categories, handled differently:**

| Category | Which | How to use |
|---|---|---|
| **Behavior-tuned** — wording Anthropic tuned to correct a specific model behavior | 4.2, 4.3, 4.4, 4.5, 4.6, 4.7, 4.15, 4.16, 4.17, 4.18, 4.19, 4.21, 4.24, 4.25, 4.26, 4.27, 4.28, 4.29 | **Copy the English verbatim, do not change a word.** Changing it discards the tuning it was built for |
| **General policy** — the content is the policy; the wording carries nothing special | 4.8, 4.9, 4.10, 4.11, 4.12, 4.13, 4.14, 4.20, 4.22, 4.23, 4.30 | Prefer copying verbatim; but when the original context does not match your task (4.10 describes multi-file code investigation, say, and your task is reading source material for a report), **rewrite it to fit**, preserving the criterion |

**Entries with a block per model.** 4.3, 4.6, 4.9 and 4.13 carry an Opus 5 or general block and a separate Fable 5.1 block, each labeled with the model it was measured on. Pick the block for your target model; do not stack both. (4.18 is different: both of its blocks are Fable 5.1 wording, and the official instruction is to apply both.)

**Working in a non-English prompt:**

- Place the English **as a whole block**; do not splice it into surrounding sentences. Put it inside the section it governs, immediately after that heading
- **Tell the user these blocks are official wording and should not be translated or "tidied up"** — seeing mixed languages, users naturally want to unify them, and that breaks the tuning
- If you need only one or two blocks and they are "general policy" above, express them in the user's language rather than mixing for two sentences

### Snippet index

| # | Purpose | Source |
|---|---|---|
| 4.1 | Whether to give few-shot examples | decision table, not a quotation |
| 4.2 | Controlling verbosity (essential for Opus 5) | [Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) |
| 4.3 | Controlling agent narration cadence (essential for Opus 5; Fable 5.1 block for the opposite problem) | [Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5); Fable 5.1 block from [Prompting Claude Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) |
| 4.4 | Controlling written deliverable length | [Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) |
| 4.5 | Constraining task scope | [Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) |
| 4.6 | Reducing over-engineering (with a Fable 5 variant and a Fable 5.1 block) | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices); Fable 5 variant from [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) (carries over, see entry); Fable 5.1 block from [Prompting Claude Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) |
| 4.7 | Preventing hallucination about code | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| 4.8 | Act by default / suggest by default | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| 4.9 | Parallel tool calls (with a Fable 5.1 nudge for agent loops) | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices); nudge from [Prompting Claude Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) |
| 4.10 | Limiting subagent overuse | [Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) |
| 4.11 | Safety guardrails for irreversible actions | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| 4.12 | Preventing solutions written only to pass tests | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| 4.13 | Suppressing markdown in favor of flowing prose (Fable 5.1 block for the opposite problem) | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices); Fable 5.1 block from [Prompting Claude Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) |
| 4.14 | Disabling LaTeX | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| 4.15 | Curbing overthinking | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| 4.16 | Code review: coverage over filtering | wording from [Prompting Claude Sonnet 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-sonnet-5); diagnosis from [Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) |
| 4.17 | Progress truthfulness on long runs (measured on Fable 5) | [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) — not republished for Fable 5.1; carries over, see entry |
| 4.18 | Unattended autonomous operation (Fable 5.1 wording) | [Prompting Claude Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) |
| 4.19 | Reassurance about context budget | [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) — not republished for Fable 5.1; carries over, see entry |
| 4.20 | The "give the reason, not only the request" template | [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) — not republished for Fable 5.1; carries over, see entry |
| 4.21 | Readability of the final report | [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) — not republished for Fable 5.1; carries over, see entry |
| 4.22 | Memory-writing conventions | [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) — not republished for Fable 5.1; carries over, see entry |
| 4.23 | A send-to-user tool | [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) — not republished for Fable 5.1; carries over, see entry |
| 4.24 | Defining behavioral boundaries | [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) — on Fable 5.1 both halves of it ship inside 4.18, see entry |
| 4.25 | Front-end design (avoiding the AI look) | [Prompting Claude Sonnet 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-sonnet-5) — **not** the longer `frontend_aesthetics` block on the best practices page |
| 4.26 | Writing density: removing mannered prose (Fable 5.1) | [Prompting Claude Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) |
| 4.27 | Search triggering at low effort (Fable 5.1) | [Prompting Claude Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) |
| 4.28 | Targeted edits over whole-file rewrites (Fable 5.1) | [Prompting Claude Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) |
| 4.29 | Leaving room for long outputs at `xhigh`/`max` (Fable 5.1, API only) | [Prompting Claude Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) |
| 4.30 | Quoting retrieved sources when summarizing (Fable 5.1) | [Prompting Claude Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) |

---

### 4.1 Few-shot examples: decide first whether to use any

The most misapplied decision in the set. **Decide by situation:**

| Goal | Give examples? | Basis |
|---|---|---|
| Lock output **format, tone, or structure** (a fixed JSON shape, a fixed voice) | **Yes** — 3–5, diverse, wrapped in `<example>` | Official docs: "Examples are one of the most reliable ways to steer Claude's output format, tone, and structure" |
| An agent system prompt where the model should **exercise judgment and creativity** | **No** — improve the tool interface instead | Claude Code team: "removing examples was extremely helpful, because it was just more creative than the examples we gave it" |
| Demonstrate a **reasoning pattern** | Use `<thinking>` tags inside the few-shot examples; the model generalizes that style to its own thinking blocks | Official docs |

Requirements when you do give them: **relevant** (mirror the actual use case), **diverse** (cover edge cases and vary enough that Claude does not pick up unintended patterns), **structured** (wrapped in `<example>` / `<examples>` tags).

### 4.2 Controlling verbosity (essential for Opus 5)

```
Keep responses focused, brief, and concise. Keep disclaimers and caveats short, and spend most of the response on the main answer. When asked to explain something, give a high-level summary unless an in-depth explanation is specifically requested.
```

In a long system prompt, pair it with a short reminder near the **end**:

```
<tone_preference>
Keep outputs reasonably concise.
</tone_preference>
```

### 4.3 Controlling agent narration cadence (essential for Opus 5)

```
Before your first tool call, say in one sentence what you're about to do. While working, give a brief update only when you find something important or change direction. When you finish, lead with the outcome: your first sentence should answer "what happened" or "what did you find," with supporting detail after it for readers who want it.
```

> The same lever applies in the other direction: **positive examples of the communication style you want** tend to be more effective than instructions about what not to do.

**Fable 5.1 runs the opposite way.** Its default is *fewer* user-facing updates during long tool-calling turns than Fable 5, more so at higher effort and in longer tool chains: the agent goes quiet for minutes, or the final message covers only the last step. The official order of operations, before adding any prompt text: first check that updates reach your client at all — they arrive as progress-update `thinking` blocks, which are empty under the default `thinking.display` of `"omitted"`; set `display: "updates"` (beta, `thinking-display-updates-2026-08-18` header) or `"summarized"`. Second, remove prompt lines written for chattier models that suppress narration, such as "hold all findings for the final response." Only then, if you still want more — pair programming, other human-in-the-loop work — add a short system prompt line:

```
Before you start, say in a line what you're about to do; brief updates while you work help the user follow along. Close with a short recap that stands on its own — what you found, what you did, and what's next — so a reader who only sees the last message has the full picture.
```

If your product collapses or hides tool output, tell the model, or it may run commands to "show" the user output your UI never displays. Deliver the note as a turn-scoped system message (`clear_at: "next_user_message"`, beta):

```
Only you see that command's output — the user's terminal shows at most a few lines of it. If the user needs to read any of it, put it in your reply.
```

### 4.4 Controlling written deliverable length

```
Match the length of written documents to what the task needs: cover the substance, but do not pad with filler sections, redundant summaries, or boilerplate.
```

### 4.5 Constraining task scope

```
Deliver what was asked, at the scope intended. Make routine judgment calls yourself, and check in only when different readings of the request would lead to materially different work. If the request seems mistaken or a better approach exists, say so in a sentence and continue with the task as asked rather than quietly narrowing, widening, or transforming it. Finish the whole task, and stop short of actions that are clearly beyond what was asked.
```

### 4.6 Reducing over-engineering

```
Avoid over-engineering. Only make changes that are directly requested or clearly necessary. Keep solutions simple and focused:

- Scope: Don't add features, refactor code, or make "improvements" beyond what was asked. A bug fix doesn't need surrounding code cleaned up. A simple feature doesn't need extra configurability.

- Documentation: Don't add docstrings, comments, or type annotations to code you didn't change. Only add comments where the logic isn't self-evident.

- Defensive coding: Don't add error handling, fallbacks, or validation for scenarios that can't happen. Trust internal code and framework guarantees. Only validate at system boundaries (user input, external APIs).

- Abstractions: Don't create helpers, utilities, or abstractions for one-time operations. Don't design for hypothetical future requirements. The right amount of complexity is the minimum needed for the current task.
```

**Fable 5 variant** (Fable 5 page; adds the feature-flag clause and is aimed at unrequested tidying at higher effort. Not republished on the Fable 5.1 page, which says existing Fable 5 prompts should perform well on Fable 5.1 without changes):

```
Don't add features, refactor, or introduce abstractions beyond what the task requires. A bug fix doesn't need surrounding cleanup and a one-shot operation usually doesn't need a helper. Don't design for hypothetical future requirements: do the simplest thing that works well. Avoid premature abstraction and half-finished implementations. Don't add error handling, fallbacks, or validation for scenarios that cannot happen. Trust internal code and framework guarantees. Only validate at system boundaries (user input, external APIs). Don't use feature flags or backwards-compatibility shims when you can just change the code.
```

**Fable 5.1 block** (Fable 5.1 page, "Keep changes and tests to what the task asks for"). The adjacent behavior on Fable 5.1 is delivering what was asked *and sometimes more*: fixing nearby code, extending behavior the task did not mention, committing more test files than the change warrants. Officially, with this instruction unrequested additions and committed test code drop substantially with no measurable change in task success:

```
If, while working or testing, you find a pre-existing bug, a performance concern, or behavior the task doesn't mention, don't fix, optimize or extend it in this change unless the requested behavior cannot work without it; report it as a follow-up in your summary. Where the task is ambiguous, implement the reading its wording and the surrounding code most directly support, state that assumption in your summary, and don't build for the other readings as well. Verify your work however you like; scratch scripts and quick checks need not be kept. Commit tests only where the task asks for them or this repository already keeps tests for this kind of change, sized like the neighboring test files — roughly one focused test per stated behavior — and don't turn scratch checks into additional permanent test files. This is about extras only: implement every behavior the task asks for, completely.
```

### 4.7 Preventing hallucination about code

```
<investigate_before_answering>
Never speculate about code you have not opened. If the user references a specific file, you MUST read the file before answering. Make sure to investigate and read relevant files BEFORE answering questions about the codebase. Never make any claims about code before investigating unless you are certain of the correct answer - give grounded and hallucination-free answers.
</investigate_before_answering>
```

### 4.8 Act by default / suggest by default

Act by default:

```
<default_to_action>
By default, implement changes rather than only suggesting them. If the user's intent is unclear, infer the most useful likely action and proceed, using tools to discover any missing details instead of guessing. Try to infer the user's intent about whether a tool call (e.g., file edit or read) is intended or not, and act accordingly.
</default_to_action>
```

Suggest only:

```
<do_not_act_before_instructions>
Do not jump into implementation or change files unless clearly instructed to make changes. When the user's intent is ambiguous, default to providing information, doing research, and providing recommendations rather than taking action. Only proceed with edits, modifications, or implementations when the user explicitly requests them.
</do_not_act_before_instructions>
```

> Also: when you want action, use a direct verb. `Change this function to improve its performance.` beats `Can you suggest some changes to improve this function?` — the latter often produces suggestions only.

### 4.9 Parallel tool calls

```
<use_parallel_tool_calls>
If you intend to call multiple tools and there are no dependencies between the tool calls, make all of the independent tool calls in parallel. Prioritize calling tools simultaneously whenever the actions can be done in parallel rather than sequentially. For example, when reading 3 files, run 3 tool calls in parallel to read all 3 files into context at the same time. Maximize use of parallel tool calls where possible to increase speed and efficiency. However, if some tool calls depend on previous calls to inform dependent values like the parameters, do NOT call these tools in parallel and instead call them sequentially. Never use placeholders or guess missing parameters in tool calls.
</use_parallel_tool_calls>
```

**Fable 5.1 nudge for agent loops.** Fable 5.1 issues parallel calls as expected when a request names several things to fetch. The exception is coding and computer-use loops where the next independent calls are implied by the task rather than requested: there it may issue one call per turn where Fable 5 batched several. This does not affect answer quality, but each extra turn costs tokens, a round trip, and wall-clock time. The official fix is one sentence at the end of the current request:

```
First privately list what you need next; then request every item that doesn't depend on another's result in this one response.
```

The best practices page says the same for long agent loops on Fable 5.1 (send the parallel-calls instruction as a turn-scoped system message after each round of tool results). Send a fresh copy each time you return tool results, as a turn-scoped system message after that user message (`role: "system"`, `clear_at: "next_user_message"`; beta header `mid-conversation-system-clear-at-2026-08-21`), and leave the earlier copies in place byte-for-byte — once a later user message exists the API clears them, and deleting them is an edit to earlier turns. Without the beta, put the sentence in a text block after the `tool_result` blocks in the same user message.

### 4.10 Limiting subagent overuse

```
Delegate to a subagent only for large tasks that are genuinely independent and parallelizable, such as a wide multi-file investigation. Do not delegate work you can finish yourself in a handful of tool calls, and do not use subagents to verify or double-check your own work. If one subagent can complete the task, use one rather than several, and keep spawn counts low.
```

### 4.11 Safety guardrails for irreversible actions

```
Consider the reversibility and potential impact of your actions. You are encouraged to take local, reversible actions like editing files or running tests, but for actions that are hard to reverse, affect shared systems, or could be destructive, ask the user before proceeding.

Examples of actions that warrant confirmation:
- Destructive operations: deleting files or branches, dropping database tables, rm -rf
- Hard to reverse operations: git push --force, git reset --hard, amending published commits
- Operations visible to others: pushing code, commenting on PRs/issues, sending messages, modifying shared infrastructure

When encountering obstacles, do not use destructive actions as a shortcut. For example, don't bypass safety checks (e.g. --no-verify) or discard unfamiliar files that may be in-progress work.
```

### 4.12 Preventing solutions written only to pass tests

```
Please write a high-quality, general-purpose solution using the standard tools available. Do not create helper scripts or workarounds to accomplish the task more efficiently. Implement a solution that works correctly for all valid inputs, not just the test cases. Do not hard-code values or create solutions that only work for specific test inputs. Instead, implement the actual logic that solves the problem generally.

Focus on understanding the problem requirements and implementing the correct algorithm. Tests are there to verify correctness, not to define the solution. Provide a principled implementation that follows best practices and software design principles.

If the task is unreasonable or infeasible, or if any of the tests are incorrect, please inform me rather than working around them. The solution should be robust, maintainable, and extendable.
```

### 4.13 Suppressing markdown in favor of flowing prose

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

**Fable 5.1 goes the other way.** Earlier models overused bullets and bold in chat; Fable 5.1 uses bold less and is less likely to reach for headers, lists, or quotation marks, so anti-formatting rules written for those models can suppress structure the content needs. The best practices page says so about the block above itself: on Fable 5.1 "a block like this can suppress structure the content needs. Remove it, or replace it with the shorter rule" — the rule being one that says when specific formatting is appropriate, such as:

```
Use lists and bullet points when asked to, or when the content is multifaceted enough that they help with clarity. If the person explicitly requests minimal formatting, always format your responses without bullet points, headers, lists, or bold emphasis, as requested. In conversational, personal, or emotional exchanges, keep to plain prose.
```

### 4.14 Disabling LaTeX

```
Format your response in plain text only. Do not use LaTeX, MathJax, or any markup notation such as \( \), $, or \frac{}{}. Write all math expressions using standard text characters (e.g., "/" for division, "*" for multiplication, and "^" for exponents).
```

### 4.15 Curbing overthinking

```
When you're deciding how to approach a problem, choose an approach and commit to it. Avoid revisiting decisions unless you encounter new information that directly contradicts your reasoning. If you're weighing two approaches, pick one and see it through. You can always course-correct later if the chosen approach fails.
```

To reduce how often adaptive thinking triggers:

```
Thinking adds latency and should only be used when it will meaningfully improve answer quality - typically for problems that require multistep reasoning. When in doubt, respond directly.
```

### 4.16 Code review: coverage over filtering

```
Report every issue you find, including ones you are uncertain about or consider low-severity. Do not filter for importance or confidence at this stage - a separate verification step will do that. Your goal here is coverage: it is better to surface a finding that later gets filtered out than to silently drop a real bug. For each finding, include your confidence level and an estimated severity so a downstream filter can rank them.
```

> **Provenance:** the block above is verbatim from the **Claude Sonnet 5** page ("Code review harnesses"), not the Opus 5 page. It is included because the Opus 5 page independently prescribes the same move in prose — `ask it to report everything and filter in a separate pass instead` — without publishing wording for it. The block is model-agnostic prompt text; use it on Opus 5 with that caveat in mind.
>
> Basis: Opus 5 follows "only report high-severity issues" or "be conservative" **literally** — it investigates the code just as thoroughly and finds the bugs, then does not report findings it judges below your stated bar. This shows up as precision rising while measured recall falls. Move the filtering to a separate downstream step.

### 4.17 Progress truthfulness on long runs (measured on Fable 5)

From the Fable 5 page ("Ground progress claims during long runs"): in Anthropic's testing on Fable 5 this nearly eliminated fabricated status reports, even on tasks designed to elicit them. The Fable 5.1 page does not republish it and does not restate the behavior; it says existing Fable 5 prompts should perform well on Fable 5.1 without changes, so the block is kept for Fable 5.1 on that basis:

```
Before reporting progress, audit each claim against a tool result from this session. Only report work you can point to evidence for; if something is not yet verified, say so explicitly. Report outcomes faithfully: if tests fail, say so with the output; if a step was skipped, say that; when something is done and verified, state it plainly without hedging.
```

### 4.18 Unattended autonomous operation (Fable 5.1 wording)

The Fable 5.1 page rewrote this entry, so the Fable 5 wording is retired here. The behavior: on complex asynchronous workloads the model sometimes describes what it would do next instead of doing it ("Next, I'll …") or stops to ask permission for a step the original request already covered ("Shall I apply this?"). Officially, two system prompt additions together mitigate this — **apply both**; if you must limit prompt length, use only the first, which keeps most of the effect. The opening sentence, which tells the model the user is not watching, carries much of the effect: **keep it as written**. If your product needs the model to stop for specific confirmations, add a sentence after it listing them. The block can also make the model less likely to ask about ambiguous requests, so check that trade-off on your own tasks.

```
You are operating autonomously. The user is not watching in real time and cannot answer questions mid-task, so asking 'Want me to…?' or 'Shall I…?' will block the work. For reversible actions that follow from the original request, proceed without asking. Stop only for destructive actions or genuine scope changes the user must decide. Offering follow-ups after the task is done is fine; asking permission before doing the work is not.

Exception: when the user is describing a problem, asking a question, or thinking out loud rather than requesting a change, the deliverable is your assessment. Report your findings and stop. Don't apply a fix until they ask for one.

Before ending your turn, check your last paragraph. If it is a plan, an analysis, a question, a list of next steps, or a promise about work you have not done ('I'll…', 'let me know when…'), do that work now with tool calls. That includes retrying after errors and gathering missing information yourself. Do not stop because the context or session is long. End your turn only when the task is complete or you are blocked on input only the user can provide.

Before running a command that changes system state (such as restarts, deletes, or config edits), check that the evidence actually supports that specific action. A signal that pattern-matches to a known failure may have a different cause.
```

The second defines the user's request as the scope of the deliverable:

```
# Delivering work
The user's request — or the plan they approved — sets the scope, and the scope is the deliverable: don't quietly narrow, widen, or swap it. Read ambiguity the way a careful colleague would: make routine judgment calls yourself, and check in only when different readings would lead to materially different work. If you see a real problem with the task as specified, say so in a sentence or two and keep building under stated assumptions; if the user hears the concern and reaffirms, that is their decision, so deliver the full request.

If a question comes up partway, first do everything that doesn't depend on the answer; then state the assumption you made, or — when going ahead on a wrong guess would be unsafe or would make the work useless — put the question at the end of a turn that also delivers that progress. If one part turns out to be blocked, complete every other part in full and say exactly what you left out and why — the whole task is the deliverable, and scaling it down is the user's call, not yours. A step you have decided on is something to run, not to announce: describing the next step and ending the turn leaves it undone until the user replies.

Keep changes to what the request needs. Something else you notice worth doing — cleanup or documentation the task didn't call for, a change to a file the task didn't require — is a suggestion to make at the end, not a change to make; actions clearly beyond what the ask implies, and risky or destructive ones, still need the user's go-ahead.
```

The "Exception:" paragraph and the last paragraph of the first block are the two halves of 4.24; do not add 4.24 on top of this block.

### 4.19 Reassurance about context budget

From the Fable 5 page ("Rare cases of context-budget concern"): when the harness shows the model a remaining-token countdown, it may propose a new session or trim its own work. **The first choice is not to expose the remaining budget at all**; when you must, add the block below. Not republished on the Fable 5.1 page, which says existing Fable 5 prompts should perform well on Fable 5.1 without changes; the Fable 5.1 autonomous block in 4.18 carries its own sentence on this, `Do not stop because the context or session is long.`

```
You have ample context remaining. Do not stop, summarize, or suggest a new session on account of context limits. Continue the work.
```

### 4.20 The "give the reason, not only the request" template

From the Fable 5 page ("Give the reason, not only the request"); worth adopting as a default. Not republished on the Fable 5.1 page, which says existing Fable 5 prompts should perform well on Fable 5.1 without changes:

```
I'm working on [the larger task] for [who it's for]. They need [what the output enables]. With that in mind: [request].
```

### 4.21 Readability of the final report (long agent sessions)

From the Fable 5 page ("Readability when communicating with the user"). Not republished on the Fable 5.1 page, which says existing Fable 5 prompts should perform well on Fable 5.1 without changes; note that the Fable 5.1 page describes the model's *default* prose as denser rather than shorthand-heavy, which is 4.26's problem, not this one:

```
Terse shorthand is fine between tool calls (that's you thinking out loud, and brevity there is good). Your final summary is different: it's for a reader who didn't see any of that.

If you've been working for a while without the user watching (overnight, across many tool calls, since they last spoke), your final message is their first look at any of it. Write it as a re-grounding, not a continuation of your working thread: the outcome first, then the one or two things you need from them, each explained as if new. The vocabulary you built up while working is yours, not theirs; leave it behind unless you re-introduce it.

When you write the summary at the end, drop the working shorthand. Write complete sentences. Spell out terms. Don't use arrow chains, hyphen-stacked compounds, or labels you made up earlier. When you mention files, commits, flags, or other identifiers, give each one its own plain-language clause. Open with the outcome: one sentence on what happened or what you found. Then the supporting detail. If you have to choose between short and clear, choose clear.
```

### 4.22 Memory-writing conventions

From the Fable 5 page ("Construct a memory system"). Not republished on the Fable 5.1 page, which says existing Fable 5 prompts should perform well on Fable 5.1 without changes:

```
Store one lesson per file with a one-line summary at the top. Record corrections and confirmed approaches alike, including why they mattered. Don't save what the repo or chat history already records; update an existing note rather than creating a duplicate; delete notes that turn out to be wrong.
```

To bootstrap a memory store from past sessions:

```
Reflect on the previous sessions we've had together. Use subagents to identify core themes and lessons, and store them in [X]. Make sure you know to reference [X] for future use.
```

### 4.23 A send-to-user tool (long async agents)

A long-running async agent needs a channel that puts content in front of the user verbatim **without ending the turn**. Tool inputs are never summarized, so the content arrives intact. From the Fable 5 page ("Create a send-to-user tool"); not republished on the Fable 5.1 page, which says existing Fable 5 prompts should perform well on Fable 5.1 without changes. On Fable 5.1, check first that the model's own progress updates reach your client (see 4.3) — this tool is for content that must arrive verbatim, not for routine narration.

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

**Defining the tool is not enough** — officially, without an instruction in the system prompt, Fable 5 rarely calls it:

```
Between tool calls, when you have content the user must read verbatim (a partial deliverable, a direct answer to their question), call the send_to_user tool with that content. Use send_to_user only for user-facing content, not for narration or reasoning.
```

### 4.24 Defining behavioral boundaries

From the Fable 5 page ("State the boundaries"), where the behavior is described as occasionally taking unrequested actions — drafting an email when none was asked for, creating defensive git-branch backups. On Fable 5.1 both halves of this block ship inside the 4.18 autonomous block (its "Exception:" paragraph and its last paragraph, with `(such as restarts, deletes, or config edits)`); use this standalone form when you want the boundary without the autonomous framing, and do not stack the two:

```
When the user is describing a problem, asking a question, or thinking out loud rather than requesting a change, the deliverable is your assessment. Report your findings and stop. Don't apply a fix until they ask for one. Before running a command that changes system state (restarts, deletes, config edits), check that the evidence actually supports that specific action. A signal that pattern-matches to a known failure may have a different cause.
```

> The second half is easy to overlook and matters most for autonomous runs: **a signal that pattern-matches to a known failure may have a different cause.** Confirm the evidence supports that specific action before changing system state.

### 4.25 Front-end design (avoiding the AI look)

> **Provenance:** both blocks below are verbatim from the **Claude Sonnet 5** page ("Design and frontend defaults"). The prompting best practices page carries a *different and much longer* `frontend_aesthetics` block — if you want that one, take it from the page rather than from here.

```
<frontend_aesthetics>
NEVER use generic AI-generated aesthetics like overused font families (Inter, Roboto, Arial, system fonts), cliched color schemes (particularly purple gradients on white or dark backgrounds), predictable layouts and component patterns, and cookie-cutter design that lacks context-specific character. Use unique fonts, cohesive colors and themes, and animations for effects and micro-interactions.
</frontend_aesthetics>
```

For variety without `temperature`:

```
Before building, propose 4 distinct visual directions tailored to this brief (each as: bg hex / accent hex / typeface, plus a one-line rationale). Ask the user to pick one, then implement only that direction.
```

### 4.26 Writing density: removing mannered prose (Fable 5.1)

Fable 5.1's writing is officially "a step up from earlier Claude models, with fewer stock phrases and less unexplained jargon," but in some cases its prose is denser than Fable 5's: sentences run longer and there are fewer paragraph breaks. This is a different problem from length (4.2–4.4 do not reach it). The official fix is an instruction that defines the anti-pattern, added to a user message (preferred) or the system prompt:

```
Mannered prose substitutes metaphor and flourish for direct statement. Instead of "a parameter worth varying," the mannered writer produces "a dial worth turning." Instead of "this point still matters," they write "this point earns its keep." The phrases exist to display the writer, not to convey the idea, and readers can tell. That is why mannered prose irritates: it makes the reader work harder so the writer can perform. It is also imprecise. Metaphors drag in connotations the writer did not choose and cannot control. The fix is to say what you mean. When a literal phrase is available, use it.
```

The short version also tends to work:

```
Please remove all mannered prose.
```

### 4.27 Search triggering at low effort (Fable 5.1)

At `low` effort, Fable 5.1 is less likely than Fable 5 to call a search or retrieval tool, and more likely to answer from memory. In some cases the simplest fix is to raise effort for the affected turns rather than the whole conversation (per-message effort, beta — see Section 6). Otherwise, in the system prompt, say that recognizing a name is not the same as knowing its current state, and that such names should be searched as the user wrote them:

```
When a query centers on a name you do not confidently recognize, or recognize from a fast-moving area like AI models and developer tools where the landscape shifts within months, the name itself is the thing to verify: search before answering, and include the name as the user wrote it in at least one query alongside any reformulations. This holds even when you have some background on it — partial background is exactly what makes an out-of-date answer sound authoritative, so familiarity is not a reason to skip the search.
```

### 4.28 Targeted edits over whole-file rewrites (Fable 5.1)

Fable 5.1 is more likely than Fable 5 to rewrite an entire text file rather than make a targeted edit. The resulting file is usually the same, but unless the file is short or most of it is changing, a rewrite costs more output tokens and time. Append to the system prompt or the first user message; officially it brings Fable 5.1 back in line with Fable 5 for small and medium changes:

```
The number of tokens used to edit files is best minimized, all else being equal. Therefore, when it will not affect the end result, try to surgically edit a file rather than rewrite the entire thing.
```

### 4.29 Leaving room for long outputs at `xhigh`/`max` (Fable 5.1, API only)

At `xhigh` and especially `max`, Fable 5.1 can think for longer before it starts writing. When a single request asks for a long deliverable — a full rewrite of a long document, say — it may draft much of it in its thinking and then write it out again as the reply: a longer wait and more output tokens. The official first choice is to run such requests at `high` and move up only where you have measured a quality gain. If you do run them at `xhigh` or `max`: set `max_tokens` to leave room for the thinking *and* the reply, and append the following to the end of the user message, replacing `[max_tokens]` with the request's actual value (for example 64,000). It makes the thinking much shorter on prose and code requests:

```
Everything produced in one reply, including any reasoning or drafting done before the reply, counts toward a single limit of about [max_tokens] tokens. If that limit is reached before the reply is finished, the person receives a cut-off response and has to start over. Composing an entire output or deliverable in full as reasoning and then again as a reply would double the length of the turn without improving the result, so don't do that.

Instead, when the person has asked for a long or effort-intensive deliverable such as a multi-section document, a large table or dataset, or a complete code file, spend extra effort on understanding the request, checking the inputs the answer depends on, settling the structure and other difficult decisions, and otherwise using the reasoning space to reason and the output space to write an output. Usually it is not needed to draft an output multiple times.
```

### 4.30 Quoting retrieved sources when summarizing (Fable 5.1)

When summarizing documents, Fable 5.1 is more likely than Fable 5 to reproduce passages of the source text without marking them as quotations. The official fix is one complete example of a correct response in the system prompt — the user's request, the response, and a sentence explaining why the response is correct:

```
<example>
<user>look up how the Riverton Ledger and the Coast Dispatch each covered the Harbor Bridge closure and compare their reporting</user>
<response>
[web_search: Harbor Bridge closure Riverton Ledger]
[web_search: Harbor Bridge closure Coast Dispatch]
Both outlets agree on the basics: the bridge closed on March 3 after inspectors found cracked welds, and the state expects repairs to take about eight months. Where they differ is emphasis. The Ledger treats it as a local-economy story. The Dispatch frames it as a funding failure; its editorial calls the closure "entirely foreseeable." Read together, the Ledger explains who is affected now and the Dispatch explains how it came to this — neither account alone gives the whole picture.
</response>
<rationale>CORRECT: The response is organized around where the two outlets agree and differ, not as a walk through either article. Each outlet's reporting is conveyed in one or two sentences of the assistant's own indirect speech. One short marked phrase from one source; every other claim is reworded. The response is still specific and complete.</rationale>
</example>
```

Replace the two `[web_search: ...]` lines with your own tool's name, so the model reads them as templated tool output rather than literal text to emit.

---

## 5. General skeleton and structure

**Principle: the smallest set of information that fully outlines your expected behavior.** Take what you need; do not pad the structure for its own sake.

```
<role_and_goal>
One or two sentences. Who it is and what it must achieve. Not a résumé.
</role_and_goal>

<background_information>
Domain knowledge, business constraints, and terminology the model cannot infer.
Not general knowledge it already has.
</background_information>

<instructions>
The things that must happen, numbered when order or completeness matters.
Positive descriptions, each with its reason.
</instructions>

## Output description
Shape, length, format. Stated as what to do, not what to avoid.
```

**Structuring:** XML tags or Markdown headers. Use consistent, descriptive tag names, and nest them when the content has a natural hierarchy. The pattern in the official library runs one way only: **every tagged block is a standing policy**, but plenty of standing policies are left plain. Length is not the criterion — the shortest tagged block is one sentence. A tag earns its place when the block needs to be **addressed as one named unit**, set apart from what surrounds it or referred to from elsewhere.

**Long-context placement (20k+ tokens):** put long material **at the top** and the query and instructions **at the end**. Queries at the end can improve response quality by up to 30 percent in tests, especially with complex, multidocument inputs. Wrap multiple documents in `<document>` tags with `<source>` and `<document_content>` subtags.

### Whether to give few-shot examples

The single most misapplied decision. The official documentation and the Claude Code team appear to contradict each other; they are addressing different situations.

| Goal | Give examples? | Basis |
|---|---|---|
| Lock output **format, tone, or structure** | **Yes** — 3–5, diverse, wrapped in `<example>` tags | Official docs: "Examples are one of the most reliable ways to steer Claude's output format, tone, and structure" |
| An agent system prompt where the model should **exercise judgment and creativity** | **No** — improve the tool interface design instead | Claude Code team: "removing examples was extremely helpful, because it was just more creative than the examples we gave it" |
| Demonstrate a **reasoning pattern** | Use `<thinking>` tags inside the few-shot examples | Official docs |

**One line to remember: examples constrain. Use them when you want constraint; drop them when you want creativity.**

---

### When you use an English snippet

Producing a non-English prompt with embedded official English blocks is the normal case, and it is handled the same way every time:

- Place the English **as a whole block**; do not splice it into surrounding sentences. Put it inside the section it governs, immediately after that section's heading
- **Tell the user in the delivery note that these blocks are official wording and should not be translated or "tidied up."** Seeing mixed languages, a user will naturally want to unify them, and doing so breaks the tuning
- If you need only one or two blocks and they are marked "general policy" in Section 4, express them in the user's language instead of mixing for the sake of two sentences

Which snippets must be verbatim and which may be adapted is in the grading table at the top of Section 4.

---

## 6. Tuning by model: Opus 5 and Fable 5.1

> Snippet numbers (4.2, 4.17, …) refer to Section 4.
>
> Every English block in the Prescription column below is **verbatim official wording** — copy it as is rather than translating or tidying it.

### Section map

- Claude Opus 5
- Claude Fable 5.1 / Mythos 5.1 — stated for Fable 5.1; stated for Fable 5 and not restated for Fable 5.1
- Where the two models invert: verification
- Effort reference

### Claude Opus 5

Default effort `high`; thinking on by default; 1M token context window as both default and maximum.

| Known behavior | Prescription |
|---|---|
| User-facing responses run longer than prior Opus models'; **lowering effort does not reliably shorten visible output** | Prompt for conciseness explicitly (4.2) |
| Narrates readily during agentic work; per-message output is often long | Describe the cadence you want (4.3) |
| Files written to disk (reports, Markdown, summaries) run long | Add length calibration (4.4) |
| **Verifies its own work without being told to** | **Remove** every verification instruction (3.4). Official wording is remove, not rewrite |
| Expands task scope, adding steps that were not requested | Constrain scope explicitly (4.5) |
| Delegates to subagents more readily than prior models | Give explicit criteria or a deterministic cap (4.10). In Claude Code and the Agent SDK the caps are `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH`, `CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS` and the SDK's `max_budget_usd` (Claude Code 2.1.217 or later); Claude Code adds a delegation instruction of its own on Opus 5 only with the `claude_code` system prompt preset, so with a custom or omitted system prompt add one yourself |
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

### Claude Fable 5.1 / Mythos 5.1

Default effort `high`; thinking is always on and cannot be configured (adaptive thinking is the only mode). Claude Mythos 5.1 is the same model, offered to Project Glasswing participants only, with the same specs, pricing and prompting guidance; the one stated difference that affects prompting or harness code is that Mythos 5.1 does not run the append-only conversation check on thinking blocks (availability differs as well). Released September 1, 2026; Fable 5 is now listed as a legacy model, still available. In Claude Code, `claude-fable-5-1` is the default Fable model since 2.1.257 (2026-09-01).

Positioning: officially **Anthropic's most capable widely released model**. The official selection advice is the other half of the picture: **for most workloads, start with Claude Opus 5; use Claude Fable 5.1 for demanding reasoning and long-horizon agentic work, or when your evals on Claude Opus 5 at higher effort still fall short.** The selection matrix names its use cases as agent sessions that run for hours, multistep deep research, and analysis carried through to a finished document, spreadsheet, or deck. "Most capable" and "what you should reach for by default" are not the same question. Its capability gains over Fable 5 are largest at higher effort and concentrate in agentic coding over long sessions, document/spreadsheet/slide work, multistep research, vision, long-context work, and computer use.

**How the Fable 5 guidance carries over.** The Fable 5.1 prompting page opens with `Your existing Claude Fable 5 prompts should perform well on Claude Fable 5.1 without changes, but a handful of behavioral differences are worth knowing about.` The general prompting page adds the caveat that applies to every model-specific technique: `Where a technique names a specific model, treat it as measured on that model and re-check it against your own evals before applying it to another.` So this file keeps the two apart: the first table is what the Fable 5.1 pages state; the second is what the Fable 5 page states and the Fable 5.1 pages neither restate nor retract.

#### Stated for Fable 5.1

| Known behavior | Prescription |
|---|---|
| **Fewer user-facing updates during long tool-calling turns** than Fable 5, more so at higher effort and in longer tool chains: the agent goes quiet for minutes at a time, or the final message covers only the last step | Do this in order: make sure updates reach your client at all (they arrive as progress-update `thinking` blocks, empty under the default `thinking.display: "omitted"` — set `"updates"`, beta); remove prompt lines that suppress narration ("hold all findings for the final response"); only then add the 4.3 Fable 5.1 block. Comparative latency is officially "Slower," so adjust timeouts and progress indicators as you would have for Fable 5 |
| On complex asynchronous workloads, **describes the next step instead of doing it** ("Next, I'll …") or asks permission for a step the request already covered — users have to reply "continue" | 4.18, both blocks together; the first alone keeps most of the effect. The opening sentence carries much of it — keep it as written |
| Delivers what was asked **and sometimes more**: fixes nearby code, extends behavior the task did not mention, commits more test files than the change warrants | The 4.6 Fable 5.1 block. Officially, unrequested additions and committed test code drop substantially with no measurable change in task success |
| In coding and computer-use loops where the next independent calls are only implied, may issue **one tool call per turn** where Fable 5 batched several — costs tokens and round trips, not quality | The one-sentence nudge in 4.9, re-sent each turn as a turn-scoped system message (beta) or after the `tool_result` blocks |
| **Denser prose** in places than Fable 5: longer sentences, fewer paragraph breaks | 4.26. The verbosity snippets (4.2–4.4) address length, not density |
| Uses **less formatting in chat** than earlier models — less bold, fewer headers and lists — so anti-formatting rules written for those models suppress structure the content needs | Remove them, or replace them with the 4.13 Fable 5.1 block that says when formatting is appropriate |
| When summarizing documents, more likely to reproduce source passages **without marking them as quotations** | 4.30 — one complete worked example in the system prompt |
| At `low` effort, **answers from memory** instead of calling search or retrieval | Raise effort for the affected turns (per-message effort, beta), or 4.27 |
| More likely to **rewrite a whole file** for a small change — same result, more output tokens and time | 4.28 |
| At `xhigh`/`max`, may **draft a long deliverable in thinking and then write it again** as the reply | Run such requests at `high`; if you must go higher, set `max_tokens` for thinking plus reply and append 4.29 (API only) |
| Instructing it to reproduce its reasoning triggers `reasoning_extraction` refusals — stated for Fable 5.1 and Fable 5 alike | See 3.9 |
| Safety classifiers cover the same categories as Fable 5 (`cyber`, `bio`, `frontier_llm`, `reasoning_extraction`, `general_harms`) — a broader set than Opus 5's cybersecurity-only classifiers. Benign work can trigger them; finding vulnerabilities in source code is permitted; false positives are fewer than Fable 5's at launch | Handle `stop_reason: "refusal"` and configure fallback (permitted targets: Opus 4.8 and Opus 5). To reduce false positives: ask "Are there any bugs in this program?" rather than "Does this program compile without errors?"; give context for lesser-known languages; keep base64-encoded data out of tool output |
| Forced tool use (`tool_choice` `any` / `tool`) returns 400 | See 3.12 |
| **Lead agent idles while subagents run** (the page's own symptom line) | Let it keep working: the tool that starts a subagent returns immediately, results come back in a later `user` message, and a separate tool waits when the lead wants to. Officially this lowers average time to completion at similar quality, token usage, and cost; the model still often chooses to wait |
| Dense charts and images | Give it a crop-and-zoom tool (or a container with the raw images and PIL/OpenCV) so it can iteratively analyze, crop, and verify — the crop tool alone delivers most of the uplift |

Harness-level rules stated for Fable 5.1 (API only; see Section 9): keep the conversation history append-only, since editing earlier turns invalidates its thinking blocks (Mythos 5.1 does not run that check); change effort mid-conversation with per-message effort rather than the top-level value; re-run the effort sweep even if you ran one on Fable 5.

#### Stated for Fable 5 and not restated for Fable 5.1

Each prescription below is official Fable 5 wording, which the Fable 5.1 page says carries over; whether the *behavior* still shows on Fable 5.1 is not stated either way. Treat these as measured on Fable 5 and re-check them on your own evals.

| Known behavior (Fable 5) | Prescription |
|---|---|
| **Individual requests can run for many minutes; autonomous runs can extend for hours** | Adjust client timeouts, streaming, and progress indicators before migrating; consider checking on runs asynchronously rather than blocking. The Fable 5.1 pages corroborate the scale ("quiet for minutes at a time"; comparative latency "Slower") without restating the sentence |
| Overplans when the task is ambiguous | `When you have enough information to act, act. Do not re-derive facts already established in the conversation, re-litigate a decision the user has already made, or narrate options you will not pursue in user-facing messages. If you are weighing a choice, give a recommendation, not an exhaustive survey. This does not apply to thinking blocks.` |
| At higher effort, performs unrequested tidying or refactoring | The **Fable 5 variant** in 4.6. (The Fable 5.1 block in the same entry is the measured 5.1 prescription for the adjacent behavior — unrequested fixes and extra test files) |
| **Instruction following is strong enough to steer a class of behavior with one brief instruction** | Short instructions instead of long enumerations (3.3) |
| Can fabricate progress on long autonomous runs | 4.17 — "nearly eliminated fabricated status reports" was measured on Fable 5 |
| Takes unrequested actions (drafting an email, creating defensive git branches) | 4.24 standalone; on Fable 5.1 both halves of it ship inside the 4.18 block |
| Dispatches parallel subagents more readily | **Encourage it**, but give explicit delegation criteria; prefer asynchronous communication over blocking; long-lived subagents save time and cost through cache reads. (The Fable 5.1 page's harness advice on this is in the first table) |
| Suggests a new session when shown a remaining-token countdown | 4.19; better still, do not surface the countdown. (The Fable 5.1 autonomous block in 4.18 adds `Do not stop because the context or session is long.`) |
| **Skills written for prior models are often too prescriptive and can degrade output quality** | Review and prune. Fable 5 also updates skills on the fly based on what it learns |

**Scaffolding advice (Fable 5 page, not restated for Fable 5.1):** start at the top of your difficulty range — pick a task harder than what you would assign to prior models, and have it scope the work, ask clarifying questions, and execute.

### Where the two models invert: verification

The single easiest thing to get wrong. Two official positions look contradictory; they address different situations.

| Situation | Do | Basis |
|---|---|---|
| **Opus 5, ordinary tasks** | **Remove** all verification and re-check instructions | It verifies itself; keeping them causes over-verification |
| **Fable 5.1, long autonomous runs** | **Add** explicit periodic self-checking | Official wording, published on the Fable 5 page and not restated or retracted on the Fable 5.1 page (which says Fable 5 prompts carry over unchanged): `Establish a method for checking your own work at an interval of [X] as you build. Run this every [X interval], verifying your work with subagents against the specification.` |

The reconciling detail: officially (Fable 5 page, same paragraph), **separate, fresh-context verifier subagents tend to outperform self-critique**. So what you remove is "have the model look at its own work again"; what you add is "have a clean context check it." These are not in conflict.

(This also does not conflict with 4.10's "do not use subagents to verify or double-check your own work." Read 4.10 on the same freshness axis: what it forbids is **delegating self-review** — handing a subagent the model's own work and its own criteria, which just pays twice for a judgment Opus 5 already makes. A **separate verifier with independent criteria in a fresh context** stays permitted on both models. What the Fable line's long runs add is not permission but **periodicity**: run that check on an interval instead of once at the end.)

### Effort reference

`max` > `xhigh` > `high` (default) > `medium` > `low`

**Both models start at the default `high`; what differs is when to step up:**

- **Opus 5**: step up to `xhigh` for **demanding** coding and agentic work, or `max` when a task justifies unconstrained token spending. Use `low`/`medium` liberally as the primary control for token cost and response time wherever your evals show quality holds
- **Fable 5.1**: step up to `xhigh` or `max` for the most capability-sensitive agentic and coding work, and step down to `medium`/`low` for routine or latency-sensitive work once your evals show quality holds. Officially its gains over Fable 5 show across effort levels and are largest at the higher settings; at `medium` results roughly match Fable 5 at lower cost, and at `low` it is often competitive with Opus and Sonnet models on cost per task while scoring higher — so include it wherever you would otherwise run a smaller model at a higher effort level

Officially, `xhigh` is described as extended capability for long-horizon work: long-running agentic and coding tasks **over 30 minutes, with token budgets in the millions**. That is the concrete bar for "demanding." `low` is described as most efficient, for simpler tasks needing the best speed and lowest cost — subagents are the named example.

**If you carried effort settings over from an earlier model, run a fresh effort sweep on your own evals rather than reusing them.** Stated for Opus 5 on the effort page, and for Fable 5.1 on its own page in stronger form: re-run the sweep even if you already ran one on Fable 5, because effort level names do not correspond to the same amount of thinking across models.

Two ordering rules, both official:

- **Tuning effort is often a better lever than switching models.** Run the ladder on the model you already have before reaching for a different one
- If the model thinks more often than the workload needs, **lower the effort level before reaching for prompt-based steering**. Prompt wording is the second lever, not the first, and its effectiveness is sensitive to exact phrasing

Points that matter:

- `effort: "high"` produces exactly the same behavior as omitting the parameter
- Effort is a behavioral signal, not a strict token budget; it affects text, tool calls, and thinking alike
- **Lower effort directly reduces the number of tool calls**, combines operations, and skips preamble
- Changing the **top-level** effort between requests invalidates the prompt cache. Fable 5.1, Mythos 5.1 and Opus 5 support a **per-message effort change** (beta, `mid-conversation-output-config-2026-07-01`: a `role: "system"` message with empty `content` and `output_config.effort`) that takes effect from the next user turn and preserves the cache. On Fable 5.1 that form is also officially preferred because a top-level change steers the model less reliably — its earlier replies were written at the previous level and it tends to stay consistent with them. Fable 5 does not support it (400)
- Leave headroom in `max_tokens` — it is a hard limit on thinking plus response text combined. On Opus 5 this matters at `xhigh`/`max`, where official guidance is to start at 64k and tune from there; on Fable 5.1 it is called out at `high` and above. Relevant only when calling the API directly

---

## 7. Scenario A: task prompts destined for Claude Code

The user describes work in Claude desktop; you produce a prompt they paste into Claude Code (including the VS Code extension's graphical window).

### Section map

- First priority: give it a check it can run
- How hard the check gates the stop
- When the check does not exist yet
- Say it all at once
- How specific to be
- Values only the user can supply
- Explore, plan, implement, commit
- Larger features: have it interview the user into a SPEC
- Context is the binding constraint
- This prompt versus project configuration
- Where project configuration goes: seven mechanisms
- Turning manual checks into verification loops
- Four kinds of loops
- Five common failure patterns

---

### First priority: give it a check it can run

This is the first rule in the official best practices, and it is the largest difference between this scenario and Scenario B.

Adapted from the official wording — the second person there addresses the reader, which here means your user rather than you:

> Claude stops when the work looks done. Without a check it can run, "looks done" is the only signal available, and **the user becomes the verification loop** — every mistake waits for them to notice it. Give Claude something that produces a pass or fail, and the loop closes on its own.

The check is anything that returns a signal the model can read: a test suite, a build exit code, a linter, a script that diffs output against a fixture, or a browser screenshot compared against a design.

**Contrasts to use when writing the prompt:**

| Weak | Strong |
|---|---|
| `implement a function that validates email addresses` | `write a validateEmail function. example test cases: user@example.com is true, invalid is false, user@.com is false. run the tests after implementing` |
| `make the dashboard look better` | `[paste screenshot] implement this design. take a screenshot of the result and compare it to the original. list differences and fix them` |
| `the build is failing` | `the build fails with this error: [paste error]. fix it and verify the build succeeds. address the root cause, don't suppress the error` |

> **Distinguish this from 3.4.** That section is about removing subjective self-review instructions. This is about supplying **external deterministic checks** — running tests, reading exit codes, comparing screenshots. Remove the former; you must supply the latter.

Also require **evidence rather than assertion**: the test output, the command run and what it returned, a screenshot of the result. Reviewing evidence is faster than re-running the verification yourself, and it works for sessions nobody was watching.

### How hard the check gates the stop

Once the check exists, there is a second decision the prompt alone cannot make. Official guidance gives four levels, trading setup effort for how little attention the run needs:

| Level | What it is | When it earns the setup |
|---|---|---|
| **In one prompt** | Ask it to run the check and iterate in the same message | Any task, today. This is what the prompt you are writing does |
| **Across a session** | Set the check as a `/goal` condition — a separate evaluator re-checks after every turn | The user wants it to keep going until the criterion holds |
| **A deterministic gate** | A `Stop` hook runs the check as a script and blocks the turn from ending until it passes (Claude Code overrides after 8 consecutive blocks) | The check must not be skippable |
| **A second opinion** | A verification subagent — or a dynamic workflow that checks its own findings — so the agent doing the work is not the one grading it. For a plain correctness check, Claude Code's bundled `/code-review` skill reviews the current diff in a fresh subagent | Correctness matters more than speed |

**Mention the level in the delivery note when it matters.** A prompt is level one by definition; if the user described an unattended run, say that `/goal` or a `Stop` hook is what actually lets it finish correctly without them.

### When the check does not exist yet

The rule above assumes a runnable check exists. **The most common Claude Code request is exactly the one where no such check exists** — legacy code, low coverage, a past incident, nobody willing to touch it. Do not skip verification and start editing. Make **building the check the first phase**:

1. **Write characterization tests that pin current behavior.** Regardless of whether that behavior is correct, assert it as it stands — including behavior you believe is a bug. The goal is a net that detects whether behavior changed, not a judgment about what is right
2. **Pin suspected bugs as they are, and mark them separately.** Where you suspect a bug, still assert the actual current behavior, and add a comment flagging it. **This must be stated explicitly in the prompt** — a well-intentioned reader told to "write tests that pin current behavior" will very naturally fix the bugs they find along the way, which destroys the net
3. **Freeze the net once it exists.** The refactor phase must not modify these tests. A changed test means changed behavior, and refactoring is by definition behavior-preserving. When a test genuinely must change, that has to be raised separately with a reason
4. **Only then refactor**, running the net at every step

The corresponding prompt structure is phased with checkpoints between phases, not a single line saying "refactor this module." Behavior changes — fixing the bugs, altering logic — belong in a **separate later round**. If the two are mixed together, a red test leaves you unable to tell whether the refactor broke something or the behavior simply changed.

### Say it all at once

Official guidance is explicit: **specifying the task, intent, and relevant constraints upfront in the first turn maximizes autonomy and intelligence while minimizing extra token usage after user turns.** Ambiguous or underspecified prompts delivered progressively over multiple turns tend to reduce token efficiency and sometimes performance.

This matters more here than anywhere else, because the user pastes once and walks away.

### How specific to be

| Strategy | Weak | Strong |
|---|---|---|
| **Scope the task**: which file, which scenario, testing preferences | `add tests for foo.py` | `write a test for foo.py covering the edge case where the user is logged out. avoid mocks.` |
| **Point to sources**: where the answer lives | `why does ExecutionFactory have such a weird api?` | `look through ExecutionFactory's git history and summarize how its api came to be` |
| **Reference existing patterns**: point at a model in the codebase | `add a calendar widget` | `look at how existing widgets are implemented on the home page to understand the patterns. HotDogWidget.php is a good example. follow the pattern to implement a new calendar widget...` |
| **Describe the symptom**: symptom, likely location, and what "fixed" looks like | `fix the login bug` | `users report that login fails after session timeout. check the auth flow in src/auth/, especially token refresh. write a failing test that reproduces the issue, then fix it` |

Vague prompts have their place — exploration, when you can afford to course-correct. `what would you improve in this file?` surfaces things you would not have thought to ask. That is exploration, not execution.

**Ways to supply rich input** (mention these to the user): `@` to reference files, paste images directly, give documentation URLs, pipe data in with `cat error.log | claude`, or let Claude fetch what it needs through bash, MCP tools, and file reads.

### Values only the user can supply

Scenario A always has values that are **neither inferable nor askable in advance**: the repo path, the test command, what the two production incidents actually were, internal conventions. These are often the highest-value inputs in the whole prompt.

Leave an **explicit placeholder** in the prompt body, with a fallback for when it is left blank:

```
What the two production incidents looked like: 【fill in if you remember】
If the line above is blank, first search git log and the issue tracker for related fixes,
list the incident scenarios you infer, and confirm them with me before continuing.
```

Two requirements: the placeholder must **look like a placeholder** (use a bracket convention the user's eye catches), and **a fallback is mandatory** — a blank with no fallback gets filled in with invented content, or silently ignored. Filled in is better; blank still runs.

### Explore, plan, implement, commit

When scope is unclear or the change spans files, the prompt should walk through four phases rather than starting to write:

1. **Explore** (plan mode): read without changing, understand the current state
2. **Plan**: produce a detailed implementation plan
3. **Implement**: write it, verifying against the plan
4. **Commit**: descriptive message, open a PR

**When to skip:** when scope is clear and the change is small — a typo, a log line, a rename. The official test is useful: **if you could describe the diff in one sentence, skip the plan.**

Planning earns its cost when you are uncertain about the approach, when the change modifies multiple files, or when you are unfamiliar with the code.

**Does "verify against the plan" count as the self-verification 3.4 removes?** It depends on whether the plan was externally confirmed:

- The model wrote the plan, nobody reviewed it, and it now checks itself against it → **that is self-verification**, and of limited value
- The plan was **reviewed and confirmed by the user** (or written to a SPEC.md and adopted as an agreement) → **that is an external basis**, and checking against it works

So the four-phase flow needs a confirmation step after planning. Write that checkpoint explicitly: have it produce the plan and **stop for confirmation** before implementing. When the user will not be present to confirm, drop "verify against the plan" and rely on tests and other genuinely external checks.

### Larger features: have it interview the user into a SPEC

When the request is large and the user has not fully thought it through, the prompt you hand them can be this one:

```text
I want to build [brief description]. Interview me in detail using the AskUserQuestion tool.

Ask about technical implementation, UI/UX, edge cases, concerns, and tradeoffs. Don't ask obvious questions, dig into the hard parts I might not have considered.

Keep interviewing until we've covered everything, then write a complete spec to SPEC.md.
```

Then execute that spec in a **fresh session** — clean context, focused entirely on implementation.

Two official points worth passing on: **time spent making the spec precise pays off more than time spent watching the implementation**, and the most useful specs are **self-contained** — they name the files and interfaces involved, state what is out of scope, and end with an end-to-end verification step that proves the feature works.

---

### Context is the binding constraint

Most of the Claude Code best practices derive from one constraint: **the context window fills fast, and performance degrades as it fills.** A single debugging session or codebase exploration can consume tens of thousands of tokens.

What the prompt can do about it:

- **Push exploration outside the main context**: `Use subagents to investigate how our authentication system handles token refresh, and whether we have any existing OAuth utilities I should reuse.` The subagent reads many files in its own context and reports back a summary

> **The unified subagent criterion** (this is scattered across sources; stated fully here): **encourage them for wide read-only exploration, forbid them for re-reviewing the model's own work in its own context.** A *separate, fresh-context* verifier is the exception, and officially outperforms self-critique — that is what the "second opinion" level above and the adversarial-review note at the end of this section both refer to.
>
> So: The first is their greatest value — keeping the cost of reading dozens of files out of the main context. The second is something Opus 5 already does, so delegating it merely doubles the cost. For everything in between, snippet 4.10 is the test: if you can finish it yourself in a handful of tool calls, do not delegate.

- **Do not write open-ended investigations**: `investigate X` without a boundary reads hundreds of files and fills the context
- Remind the user: `/clear` between unrelated tasks; after two failed corrections on the same issue, `/clear` and rewrite the prompt with what was learned

### This prompt versus project configuration

The test is simple: **information specific to this task goes in the prompt; conventions that hold every time get sedimented into project configuration.**

If you find yourself writing "we use pnpm, not npm" or "run lint before committing" — things that have to be said **every single time** — recommend the user move them into CLAUDE.md or a hook rather than pasting them again.

### Where project configuration goes: seven mechanisms

| Mechanism | Loaded | Context cost | Best for |
|---|---|---|---|
| **CLAUDE.md (root)** | Session start, stays throughout, re-read after compaction | **High** (always loaded) | Build commands, directory layout, monorepo structure, team conventions. **Keep under 200 lines**, with an owner who reviews changes |
| **CLAUDE.md (subdirectory)** | On demand when that directory is touched | Low | Directory-specific conventions |
| **Rules** | Session start (unscoped) or on file access (scoped) | Medium | Specific constraints. **Add `paths:` frontmatter** to scope them and cut the cost |
| **Skills** | Name and description at start, body on invocation | Low | Procedural workflows: deploy checklists, release processes, review processes |
| **Subagents** | Name and description at start, body when called; only the final message returns | Low (isolated context) | Side tasks where you only want the conclusion: log analysis, dependency audits, deep search |
| **Hooks** | Fire on lifecycle events, bypass compaction | Low (outside main context) | **Deterministic** automation: linters, notifications, blocking specific commands |
| **Output styles** | Session start, never compacted | **High** | Only when Claude's role needs to change substantially |

**Anti-patterns:**

- ✗ "Every time X, always do Y" in CLAUDE.md → use a hook. **Advisory instructions are always weaker than deterministic execution**
- ✗ "Never do this" constraints → use a `PreToolUse` hook or permission configuration
- ✗ A 30-line procedure in CLAUDE.md → move it to a skill
- ✗ An API-specific rule with no path scoping → add `paths:` frontmatter
- ✗ Personal preferences in a shared project file → use the user-level local equivalent

**What belongs in CLAUDE.md:** bash commands Claude cannot guess, code style that differs from defaults, testing instructions, repository etiquette (branch naming, PR conventions), project-specific architectural decisions, environment quirks, non-obvious gotchas.

**What does not:** anything readable from the code, standard language conventions, detailed API documentation (link it), frequently changing information, file-by-file descriptions, platitudes like "write clean code."

**Self-test:** for each line ask, *would removing this cause Claude to make mistakes?* If not, cut it.
**Diagnostic signal:** when a rule keeps being ignored despite repetition, the file is usually **too long and the rule is buried**, not insufficiently emphatic.
**Tool:** `/doctor` helps rightsize skills and CLAUDE.md.

**How to write a skill** (three-tier progressive disclosure):

| Tier | Content | Loaded |
|---|---|---|
| Metadata | YAML frontmatter, only `name` and `description` | Preloaded into the system prompt at startup |
| Primary instructions | The `SKILL.md` body | When Claude judges the skill relevant |
| Bundled files | Files referenced from SKILL.md | Only when that specific scenario arises |

`name` and `description` are what drive activation — if it does not trigger, fix those two lines first. Split the body when it gets unwieldy. Bundle executable code where it helps: sorting a list via token generation is far more expensive than simply running a sorting algorithm. **The Fable 5 page states that skills developed for prior models are often too prescriptive for it and can degrade output quality** — prune periodically when targeting the Fable line. (The Fable 5.1 page does not restate this; it says existing Fable 5 prompts should perform well on Fable 5.1 without changes, and the general prompting page says to treat a model-specific finding as measured on that model.)

### Turning manual checks into verification loops

The test: **anything the user keeps having to enforce by hand is a candidate for turning into a loop.** Especially deterministic rules a generic linter will not catch — the official example is "reject any migration that drops a column without a backfill step."

| Mode | Trigger | Best for |
|---|---|---|
| **Standalone** | Invoked deliberately | Cross-cutting checks that do not apply universally: security scans, accessibility audits, license verification |
| **Embedded** | Fires automatically inside the producing skill | Checks that belong to one specific task |
| **Chained** | One skill calls another on completion | End-to-end handoff; also the only way to add verification to a **skill you cannot modify** |
| **On every PR** | CI | Turns a personal habit into team infrastructure, removing dependence on author diligence |

**Trap:** embedding only works on editable skills. Built-in or plugin-managed skills update automatically and would overwrite your change — those must use chaining.

### Four kinds of loops

| Type | Command | Best for |
|---|---|---|
| **Turn-based** | Manual prompting | Short tasks |
| **Goal-based** | `/goal get the homepage Lighthouse score to 90 or above, stop after 5 tries` | Tasks with a **verifiable exit criterion** where one turn is not enough |
| **Time-based** | `/loop 5m check my PR, address review comments, and fix failing CI` | Recurring work, interfacing with external systems |
| **Proactive** | `/schedule every hour: check #project-feedback for bug reports` | Well-defined recurring streams: bug reports, issue triage, migrations |

Cost control: **use scripts for deterministic work** (running a script costs less than reasoning through it token by token); match intervals to how fast the underlying state actually changes; pilot before running at scale. The official conclusion: **model and effort selection are among the biggest levers on loop cost.**

### Five common failure patterns

Worth passing on to the user alongside the prompt:

| Pattern | Symptom | Fix |
|---|---|---|
| **The kitchen sink session** | One task, then something unrelated, then back again | `/clear` between unrelated tasks |
| **Correcting over and over** | Corrected once, still wrong, corrected again | **After two failed corrections, `/clear`** and write a better initial prompt incorporating what you learned |
| **The over-specified CLAUDE.md** | Too long; important rules get lost in the noise | Prune ruthlessly |
| **The trust-then-verify gap** | Plausible-looking implementation that misses edge cases | Always provide verification. **If you cannot verify it, do not ship it** |
| **The infinite exploration** | "Investigate this" without scope; hundreds of files read | Narrow the scope, or isolate it in a subagent |

**One important caveat on adversarial review.** Having a subagent review the diff in a fresh context is effective, but the official guidance also warns: **a reviewer prompted to find gaps will usually report some, even when the work is sound, because that is what it was asked to do.** Chasing every finding leads to over-engineering. So the review prompt must say: **flag only gaps that affect correctness or the stated requirements, and treat the rest as optional.** This is a filtering instruction, so it belongs only where **you** are the filter. If the findings feed a downstream ranking or triage step instead, use snippet 4.16 and let the reviewer report everything — Section 6 warns that Opus 5 applies a stated severity bar literally and silently drops whatever falls below it.

---

## 8. Scenario B: task prompts for desktop chat or a Cowork session

The user describes work in Claude desktop and the resulting prompt runs directly in **desktop chat** or a **Cowork session**. Non-coding work: research, analysis, writing, organizing source material, producing documents, spreadsheets and decks, scheduling, information synthesis.

### Section map

- Chat or Cowork first
- Output length must be pinned
- Where the material goes
- Give examples when format must be locked
- Research tasks
- Cowork: deliverables and files
- Cowork: skills that take over automatically
- What not to put in
- A skeleton you can lift
- Reusable and recurring prompts
- Sourcing note for this section

---

### Chat or Cowork first

The two are positioned differently, and the writing follows. The table below follows Anthropic's own description of the desktop modes (the third, Claude Code, is Scenario A and is covered in Section 7).

| | desktop chat | Cowork session |
|---|---|---|
| **Positioning** | Quick exchanges and iterative work: asking, brainstorming, drafting, working through a problem back and forth | Work that takes real effort: pulling information from many sources, making sense of it, producing something finished |
| **Local files** | Desktop connectors reach local tools; **cannot** access multiple sources systematically | **Reads and writes local files in connected folders directly**, no manual upload or download |
| **Output** | Content in the conversation | **Finished deliverables**: Excel with functional formulas, PowerPoint, formatted documents |
| **Duration** | Not suited to sustained work | Runs autonomously for a long time; the user can leave. Supports **scheduled recurring tasks** |
| **Parallelism** | None | **Sub-agent coordination**, dividing complex work into parallel workstreams |
| **Extensibility** | Connectors | Connectors, skills, plugins, Claude in Chrome |

**One boundary that matters:** Cowork **cannot** access your full file system or run terminal commands the way Claude Code can — it works inside a protected environment, on connected folders only. So repository-level operations, git, and build or deploy work are not Scenario B; those are Scenario A.

**How to decide:** if the deliverable is "content I can just read," chat is enough; if it is "a file I want to keep" or the work needs systematic sourcing across many materials, Cowork.

When the user does not say and the task clearly needs file output or bulk material handling, **default to writing for Cowork and say so in the delivery note**.

**Two Cowork capabilities worth using deliberately:**

- **Scheduled recurring tasks.** When the underlying need is "give me an X every Monday," write the prompt as a reusable task description and tell the user to set it up as a scheduled task rather than running it by hand each time.
  **A scheduled task means nobody is watching.** Any "shall I continue?" stalls the whole thing until next week. So a recurring prompt must say **how to keep going when something goes wrong** — missing material, an unopenable file, an unresolvable source conflict: record it and continue, then collect them under a "needs your decision" section in the output rather than stopping to ask. (Snippet 4.18 is the full treatment; for something light like a weekly report you do not need its two blocks, just this rule.)
  **One more rule that recurring deliverables need:** have it read the previous cycle's output first and **report only the delta** — what changed, whether last cycle's stated next steps happened, what is still open. Without this, every cycle becomes a flat restatement and the reader has to diff it themselves. This holds for weekly reports, standup notes, status updates, digests, and changelogs.
- **Parallel subagents.** With a lot of material, the prompt can say explicitly: process each source in a parallel subagent and have each return only a structured summary. Faster, and it keeps the main context clean.

### Output length must be pinned

This is the most frequent problem in Scenario B. Opus 5's user-facing responses **run longer than prior models' by default**, and lowering effort does not reliably shorten them — effort governs how much it thinks, not how much it says.

So anything meant to be read needs an explicit length constraint. **How to choose and what stacks is in Section 12, "Choosing a length control"**; in short:

- Pick one primary mechanism — **give a number when you can** ("300–400 words", "3–6 bullets, each under 15 words", not "keep it brief"); fall back to snippet 4.2 only when a number is impossible
- When the deliverable is written to a file, **add** snippet 4.4 (a word count usually constrains the body but not the padding sections the model adds on its own)
- When the system prompt is long, **append** a short `<tone_preference>` reminder at the end

### Where the material goes

When the user wants Claude to work through a body of source material — documents, emails, chat logs, saved web pages:

**Long material at the top, the query and instructions at the end.** Queries at the end can improve response quality by up to 30 percent in tests, especially with complex, multidocument inputs.

```xml
<documents>
  <document index="1">
    <source>annual_report_2025.pdf</source>
    <document_content>
      {{MATERIAL}}
    </document_content>
  </document>
  <document index="2">
    <source>competitor_analysis_q2.xlsx</source>
    <document_content>
      {{MATERIAL}}
    </document_content>
  </document>
</documents>

(query and instructions go here, last)
```

**One more technique for long documents — grounding in quotes:** ask the model to extract the relevant quotes into `<quotes>` tags first, then answer from those. This helps it focus on the relevant content and ignore the rest, an effect that is clearest on "find the supporting evidence across this pile" tasks.

### Give examples when format must be locked

A large share of Scenario B work is "produce this same shape every time" — weekly reports, meeting notes, retrospectives, briefings, cards. This is exactly the "give examples" branch of Section 5.

- 3–5 of them, wrapped in `<example>`
- Make them **diverse**, covering edge cases, so the model does not pick up unintended patterns
- The most valuable examples are **paired**: raw material → finished output. That teaches the compression ratio and the selection criteria, not just the shape
- If the user has real past samples, have them drop two or three in — this is the single highest-return step

**When the user has no samples yet** (common in a one-shot delivery where you cannot ask): **give one paired example you wrote, and do not manufacture three to five.** Invented examples in bulk teach an imagined house style as though it were the standard, and the user will find that hard to detect later. Give one that demonstrates the compression ratio and the selection criteria, leave a clearly marked slot in the prompt, and tell the user in the delivery note: **swapping in two or three real ones is the biggest single improvement available.**

Conversely, when the task is open-ended creation or brainstorming, or you want the model to choose its own angle, **do not give examples** — they narrow the exploration space to their own neighborhood.

### Research tasks

For work that requires sourcing and synthesizing across many sources, there is a ready-made official block. **Copy it verbatim, including the self-critique sentence** — see the carve-out below it:

```
Search for this information in a structured way. As you gather data, develop several competing hypotheses. Track your confidence levels in your progress notes to improve calibration. Regularly self-critique your approach and plan. Update a hypothesis tree or research notes file to persist information and provide transparency. Break down this complex research task systematically.
```

> **Carve-out for 3.4.** That block contains `Regularly self-critique your approach and plan`, and 3.4 says to remove self-verification instructions on Opus 5. This is the same kind of exception as `you MUST read the file` inside snippet 4.7: it is **tuned official wording, so copy it as is**. The two are not the same instruction — 3.4 removes *end-of-answer self-review*, a last look at a finished deliverable; this sentence is *mid-run course correction*, re-aiming the search while evidence is still coming in. If the task is a single-pass write-up rather than a multi-step search, that sentence has nothing to correct, and you may drop that one line.

Three more: **give clear success criteria** (what constitutes a successful answer to this question), **ask for source verification** across multiple sources, and **explicitly allow it to say it does not know**:

```
If the data is insufficient to draw conclusions, say so rather than speculating.
```

That last one matters especially for research — without an exit, the model states uncertain things with unwarranted confidence.

### Cowork: deliverables and files

Cowork produces real files. **Name the file type you want**, because it determines the path taken:

| Deliverable | How to say it |
|---|---|
| Editable document | "output as .docx" |
| Data table | "output as .xlsx, header row first, columns are …" |
| Presentation | "build a .pptx, X slides, each slide …" |
| A shareable page, dashboard, or tracker | "build it as a single self-contained HTML file" |
| Just text for me to read | Say so explicitly: "give it to me in the conversation, do not create a file" |

**Whether it lands on disk:** if the user wants the file saved to a specific folder on their machine, name the **target folder** in the prompt; otherwise it is delivered in the conversation only.

**Bulk processing:** with multiple files, state the processing order and naming rule, and **what to do with the ones it cannot handle** — skip and record, or stop and ask. Without that rule, behavior on a bad file is unpredictable.

### Cowork: skills that take over automatically

Cowork ships output-format skills (docx, xlsx, pptx, pdf, and others) that engage automatically when the corresponding file type is involved. That means:

- **Do not teach it how to generate a .docx** in the prompt — that is the skill's job, and writing it is redundant (3.3)
- What the prompt should specify is the **content requirements**: structure, terminology, data sources, length
- But **do state style preferences** — "use the company template," "avoid the default palette" — because the skill cannot guess those

One ordering rule worth writing in: **gather the facts fully before touching the file mechanics.** The official guidance for output-format skills is that research comes first; starting the document machinery early anchors on layout while the correct content is not yet in hand.

### What not to put in

The three things most often over-written in Scenario B:

1. **Self-verification instructions** ("check it once more," "double-check your answer") — Opus 5 does this on its own; adding them causes over-verification. See 3.4. Note the distinction that section draws: **reconciling sources against each other is the task, not self-verification** — do not delete that by mistake
2. **Résumé personas** ("you are a senior analyst with twenty years of experience") — no benefit on current models, pure token cost. To shift the perspective, say what to focus on — the official example is `Analyze this investment portfolio, focusing on risk tolerance and long-term growth potential` rather than assigning a role
3. **A long list of "do not"s** — rewrite as positive statements with reasons. See 3.5

### A skeleton you can lift

```
(long material first, if any)
<documents>
  <document index="1">
    <source>…</source>
    <document_content>…</document_content>
  </document>
</documents>

<variables_this_cycle>   (only for a recurring template — see the last section)
Date range: …
Material location: …
</variables_this_cycle>

<task>
One sentence on what to do, who it is for, and what they will do with the output.
</task>

<requirements>
1. …
2. …
(positive statements, each with its reason)
</requirements>

<output>
Shape: (conversation / .docx / .xlsx / .pptx / single-file HTML)
Length: (a specific number)
Structure: (sections or fields)
</output>

<examples>
  <example>…</example>
</examples>
(only when format must be locked)
```

Close with a **success criterion** — what counts as having done this correctly. Without it you cannot check your own work and the user cannot judge whether to send it back.

### Reusable and recurring prompts

When the user says "I want to fix this and use it every week," write a template rather than a one-off instruction. Add a variables block that collects everything that changes each cycle in one place, so the user edits only that block:

```
<variables_this_cycle>
Date range: 2026-07-20 ~ 2026-07-26
Material location: (folder path)
Previous output: (path to last cycle's file, for the delta comparison)
</variables_this_cycle>
```

**Place it immediately before `<task>`** — that is, after the material, in the slot the skeleton above shows. This keeps the long-context rule intact (material first, instructions last) and still puts the lines the user edits each cycle right next to the instruction they parameterize.

---

### Sourcing note for this section

The description in this section of chat and Cowork capability boundaries follows Anthropic's official Cowork overview and its comparison of the desktop modes; the prompting technique sections follow the official prompting best practices. Product capability details can change between releases — when in doubt, defer to what the user actually sees in their interface.

---

## 9. Agents and long-running tasks: context engineering

> Snippet numbers refer to Section 4.
>
> The English blocks in this section are **verbatim official wording** from Anthropic's long-running-agent guidance (see the repository's SOURCES.md) — copy them as is rather than translating or tidying them.

### Section map

- Context is the first constraint
- Multi-window workflows
- State management
- Compaction awareness
- Complex research
- Let the model orchestrate itself
- Design context to maximize cache hits
- When to promote a bash action to a dedicated tool
- Tool design beats prompt text

### Context is the first constraint

Performance degrades as the context fills — context rot. The target is not "tell it everything you know" but **the smallest set of high-signal tokens that maximize the likelihood of the desired outcome**.

### Multi-window workflows

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

### State management

- Structured data (test results, task status) → JSON
- Progress notes → freeform text
- Versions and checkpoints → **git**. Current models are especially effective at using git to track state across sessions
- Ask explicitly for incremental progress

### Compaction awareness

```
Your context window will be automatically compacted as it approaches its limit, allowing you to continue working indefinitely from where you left off. Therefore, do not stop tasks early due to token budget concerns. As you approach your token budget limit, save your current progress and state to memory before the context window refreshes. Always be as persistent and autonomous as possible and complete tasks fully, even if the end of your budget is approaching. Never artificially stop any task early regardless of the context remaining.
```

**If you compact on the client side** (Fable 5.1 page): server-side compaction already tells the model what to keep; for your own summarization step, the official instruction is below. The simplest shape on Fable 5.1 is to replace the whole history with one summary message plus the new user turn and replay nothing else — no thinking blocks carry over, so nothing fails the append-only check. And because cache reads are cheaper on Fable 5.1, compacting early to save cost may no longer be the right trade-off; experiment with later compaction points.

```
Summarize the transcript inside <summary></summary> tags. Include relevant information in the summary such that this conversation will be continued by a new context window without needing to redo work or be reprovided with relevant constraints or context. Be sure to preserve: (1) any difficulties or problems that came up, and how they were handled or resolved; (2) any possibilities, options, or approaches that were raised, tried, or set aside, and why; (3) anything that was asked for, decided, agreed, ruled out, or established as a preference, constraint, or boundary — stated exactly; (4) exactly where things stand now — what has been covered, settled, or completed so far; (5) anything still open, unresolved, promised, or expected to happen next; (6) specific details that would be hard to reconstruct — names, numbers, dates, exact wording, links or references — kept exactly. Be complete on these even at the cost of length; keep everything else concise. Weight the two voices differently: keep what the user said, asked for, shared, or established carefully and close to their own words; your own explanations and reasoning can be condensed much further, to what they concluded or produced — as long as nothing in the six items above is dropped.
```

### Complex research

```
Search for this information in a structured way. As you gather data, develop several competing hypotheses. Track your confidence levels in your progress notes to improve calibration. Regularly self-critique your approach and plan. Update a hypothesis tree or research notes file to persist information and provide transparency. Break down this complex research task systematically.
```

### Let the model orchestrate itself

The highest-leverage change at the harness level. **Do not route every tool result back through the context window as tokens**, and do not hardcode filtering logic into the tool design.

Give the model code execution (bash, a language REPL) so it can write logic that expresses the tool calls and filters the output itself — **only the execution results reach its attention**. Official figure: on BrowseComp web-browsing tasks, giving the model the ability to filter its own tool outputs raised accuracy from **45.3% to 61.6%**.

Two more in the same direction:

- **Let it manage its own context**: progressive disclosure through skills (frontmatter resident, body on demand), context editing to drop stale tool results, subagents for isolated contexts (BrowseComp +2.8%)
- **Let it persist its own context**: compaction plus memory folders (agentic search 60.4% → 67.2%)

> Provenance: these three figures come from Anthropic's experiments on an **earlier model generation**. The article's own argument is that the trend continues as models improve — the stronger the model, the more decisions you should hand back to it.

### Design context to maximize cache hits

**Cached tokens are 10% the cost of base input tokens** — 2.5% on Fable 5.1 and Mythos 5.1 — which makes ordering a pure cost lever.

| Practice | Guidance |
|---|---|
| **Static first** | Place stable content (system prompts, tool definitions) before dynamic material |
| **Append in messages, do not edit the prompt** | Inject reminders by appending to messages rather than editing the system prompt |
| **Do not switch models mid-session** | Caches are model-specific |
| **Adding or removing tools invalidates the cache** | Use tool search for dynamic discovery instead of changing tool definitions |
| **Move breakpoints** | In multi-turn applications, move the cache breakpoint to the latest messages |

> Direct consequence: **changing the top-level effort between requests invalidates the cache** for the same reason. Fable 5.1, Mythos 5.1 and Opus 5 support a per-message effort change (beta) that preserves it — see Section 6.

**The mechanism behind row two.** Prompt caching hashes the request prefix in order — `tools`, then `system`, then `messages` — and a hit requires a byte-exact prefix match. Appending one sentence to the system prompt therefore misses the cache for the system prompt *and every cached message after it*. When an instruction only becomes relevant partway through a run, append a `{"role": "system"}` message at that point in `messages` rather than editing the top-level `system` field: the cached prefix stays intact, and the instruction still applies as a system instruction, taking precedence over the top-level field for the turns that follow it. Available on Fable 5.1, Mythos 5.1, Fable 5, Mythos 5, Opus 4.8 and Opus 5 with no beta header; **not available on Sonnet 5**, where the top-level `system` field is the only option. A `role: "system"` message can also be scoped to one turn with `clear_at: "next_user_message"` (beta, `mid-conversation-system-clear-at-2026-08-21`): it carries system-prompt authority for the current turn, stops rendering once a later user message exists, stays in the array so nothing earlier changes, and costs no input tokens once cleared — the official home for per-turn reminders in a tool loop.

**Keep the history append-only (Fable 5.1).** Append each assistant turn exactly as returned, thinking blocks included, and do not edit earlier turns between requests. On Fable 5.1, a thinking block is valid only in the exact conversation that produced it: for accounts created on or after August 31, 2026, replaying one after its prefix (system prompt, tool list, or any earlier message) has changed returns a 400, or drops the block if you set `thinking.block_binding.prefix_mismatch_behavior: "drop_block"` (beta, `thinking-binding-controls-2026-08-01`); future models are expected to enforce this for all accounts. The edits that trip it are the same ones that restart the cache — injecting and removing per-turn reminders, summarizing older turns in place, changing the system prompt mid-session — so the fixes are the same: turn-scoped system messages for reminders, mid-conversation system messages and tool changes instead of rewriting `system` or `tools`, and server-side compaction or context editing for trimming. Removing a leading run of thinking blocks (oldest first), moving `cache_control` markers, and changing `effort` between requests keep later blocks valid.

### When to promote a bash action to a dedicated tool

Bash has broad leverage but no observability — the harness cannot intercept, gate, or audit it. Four cases warrant a dedicated declarative tool:

1. **Security boundaries**: hard-to-reverse actions that need user confirmation (calling an external API)
2. **Write operations needing staleness checks**: preventing overwrites of files that changed
3. **User-facing presentation**: modal displays, multiple options
4. **Structured logging, tracing, and replay**

For low-stakes work there is another path: use a secondary Claude instance to judge whether a bash command is safe — this is what Claude Code's auto mode does, and it reduces the number of dedicated tools needed.

### Tool design beats prompt text

Where tools exist, **changing the tool is more effective than changing the prompt**.

- **Low cardinality**: every tool has a distinct function from every other tool, so Claude can easily distinguish when to call each. The test is blunt — **if engineers cannot definitively identify which tool to use in a situation, agents will not either**
- **Semantically meaningful parameter names**: `user_id` rather than `user`; resolving arbitrary UUIDs to interpretable language significantly improves precision
- **Namespacing** by service and resource type (`asana_projects_search`, `asana_users_search`)
- **Token-efficient returns**: pagination, range selection, filtering, truncation, with sensible defaults (Claude Code's default cap is 25,000 tokens)
- **Offer a `ResponseFormat` enum** (`concise` / `detailed`) — the official example shows 206 tokens versus 72, a 65% reduction
- **Actionable error messages**: say what to change and show a correctly formatted example, rather than returning a stack trace
- **Write tool descriptions as onboarding for a new team member**: make implicit context explicit — specialized query formats, terminology, resource relationships

---

## 10. Three request types, handled differently

**Writing or revising a prompt (Scenario A/B):** run the Section 0 workflow, and write the delivery note to the Section 0 step 6 spec. Do not add explanations, disclaimers, or alternative-approach lists the user did not ask for.

**Auditing an existing prompt:** compare against Section 3 point by point and report **the specific problem and the specific fix**. Do not replace their prompt with a new one unless they asked for a rewrite. Order findings by severity: things that cause API errors (3.1, 3.2, 3.8, 3.12) first, behavior-reversing issues (3.4, 3.9, 3.10) next, style last.

**Inline check mid-conversation (Scenario C):** go light. **Do not make a production of it** — do not run the full workflow, do not read the whole file. Scan the Section 3 hard rules; if something is wrong say "this line is a problem: X, change it to Y"; if nothing is wrong say so and return to what you were discussing. The user wants to get back to what they were doing, not to receive an audit report.

---

## 11. Pre-delivery checklist

- [ ] You chose the model and effort yourself, and said so in the delivery note
- [ ] If official English snippets are embedded: the delivery note tells the user not to translate or rewrite them
- [ ] No prefill, no `budget_tokens`
- [ ] No `CRITICAL: You MUST` / `If in doubt, use [tool]` stacked to prevent undertriggering (**wording inside official snippets does not count** — for example 4.7 contains `you MUST read the file`, which is tuned official text, copy it as is)
- [ ] Targeting Opus 5: **no** self-verification or double-check instructions (external deterministic checks do not count; nor does `Regularly self-critique your approach and plan` inside the official research block — see Section 8)
- [ ] Targeting Fable 5.1: no "show your reasoning" instructions; no forced `tool_choice`; long autonomous runs carry the 4.18 autonomous block (4.17's progress-audit rule stays available; it was measured on Fable 5)
- [ ] Every constraint **you wrote** is stated positively and carries its reason (**wording inside official snippets does not count** — 4.6, 4.13 and 4.25 are stacked negatives by design; 4.25 opens `NEVER use generic AI-generated aesthetics…`, which is tuned official text, copy it as is)
- [ ] Every instruction has been read against: **how would a well-intentioned human misinterpret this?**
- [ ] No instruction repeated across the system prompt and a tool description (**a tool-invocation trigger the model needs does not count** — for example the system-prompt line shipped with snippet 4.23; see 3.7)
- [ ] The examples decision follows the Section 5 table — not added or removed reflexively
- [ ] Long-context case: material first, query last
- [ ] Output length is explicitly specified (especially necessary for Opus 5)
- [ ] There is a decidable success criterion; Scenario A has a check the model can run itself
- [ ] Read it through and cut every sentence whose removal would change nothing
- [ ] Golden test: **hand this prompt to a colleague with no context on the task — could they follow it?**

---

## 12. Symptom to prescription

Snippet numbers refer to Section 4.

| Symptom | Prescription |
|---|---|
| Generic response | Add specific constraints, add the success criterion, add a modifier like "go beyond the basics" |
| Answers the wrong question | Supply the real goal and background — say why |
| Unstable format | Add 3–5 `<example>` blocks; or use Structured Outputs |
| Preamble | `Respond directly without preamble.` |
| Fabricates content (general) | Explicitly permit not knowing: `If the data is insufficient to draw conclusions, say so rather than speculating.` |
| Makes claims about code or files it never opened | Snippet 4.7 (essential for any codebase task) |
| Suggests instead of acting | Use a direct verb (`Change this function` beats `Can you suggest changes`); or add snippet 4.8 |
| Response too long (prose) | See "Choosing a length control" below. **On Opus 5, lowering effort does not reliably work** |
| Agent narration too chatty mid-run | Snippet 4.3. This is a separate problem from prose length; the length controls do not reach it |
| Over-engineering | Snippet 4.6 |
| Subagent overuse | Snippet 4.10 |
| Too much thinking, high latency | Snippet 4.15; or lower effort |
| Shallow reasoning on hard problems | **Raise effort** rather than prompting around it |
| A rule keeps being ignored despite repeated reminders | The prompt is too long and the rule is buried → cut it down, or move to a deterministic mechanism (hook, tool design, schema) |
| Code review under-reports | Snippet 4.16; move filtering downstream. The adversarial-review caveat in Section 7 is the opposite case — no downstream filter, so the reviewer must filter |
| Long task fabricates progress or overstates completion | Snippet 4.17 |
| Stops to ask questions during an unattended run, or ends the turn on "Next, I'll …" | Snippet 4.18 |
| Agent goes quiet for minutes during long tool runs (Fable 5.1) | Check `thinking.display` first, remove narration-suppressing lines, then the 4.3 Fable 5.1 block |
| One tool call per turn in an agent loop (Fable 5.1) | The 4.9 Fable 5.1 nudge, re-sent each turn |
| Prose too dense — long sentences, few paragraph breaks (Fable 5.1) | Snippet 4.26; this is not a length problem |
| Chat replies carry less structure than the content needs (Fable 5.1) | Remove anti-formatting rules; the 4.13 Fable 5.1 block |
| Summaries reproduce source text without marking quotations (Fable 5.1) | Snippet 4.30 |
| Answers from memory instead of searching at `low` effort (Fable 5.1) | Raise effort for those turns, or snippet 4.27 |
| Rewrites whole files for small edits (Fable 5.1) | Snippet 4.28 |
| Long deliverable at `xhigh`/`max` takes very long or hits `max_tokens` (Fable 5.1) | Run it at `high`; otherwise snippet 4.29 |
| Fixes nearby bugs or commits extra test files nobody asked for (Fable 5.1) | The 4.6 Fable 5.1 block |
| Benign coding request comes back `stop_reason: "refusal"` (Fable 5.1) | Ask "Are there any bugs in this program?" rather than "Does this program compile without errors?"; give context for lesser-known languages; keep base64 out of tool output |
| Wraps up early as context fills | Snippet 4.19; better still, do not show the model a remaining-budget countdown |
| Final summary of a long session is unreadable | Snippet 4.21 |
| Did things nobody asked for | Snippet 4.24; scope creep specifically → snippet 4.5 |
| Front-end output has the generic AI look | Snippet 4.25 |
| Change may be irreversible / touches shared environments | Snippet 4.11 |
| Bends the code to make tests pass | Snippet 4.12 |
| Report is all short bullets, unreadable as prose | Snippet 4.13 |
| Needs to read many files at once / calls can run concurrently | Snippet 4.9 |
| Math renders as LaTeX in an environment that cannot display it | Snippet 4.14 |
| The request never states what it is for | The template in snippet 4.20 |
| Agent needs to accumulate lessons across sessions | Snippet 4.22 |
| Long async agent must deliver content to the user verbatim | Snippet 4.23 |

### Choosing a length control

**Pick one primary mechanism.** Stacking them works against the minimal-information principle.

- **If you can give a number, give a number** ("300–400 words", "3–6 bullets, each under 15 words"). Decidable and most effective. First choice
- **Only when you cannot** (the output shape itself is not fixed) fall back to snippet 4.2's general conciseness instruction

**Two supplements that may be added on top** — they govern different problems and do not conflict with the primary mechanism:

- When the deliverable is **written to a file**, add snippet 4.4. Files run longer than conversational replies, and a word count usually only constrains the body, not the padding sections the model adds on its own. **When you gave a number and it writes a file, use both**
- When the system prompt is long and you are worried the length requirement gets buried, append a short `<tone_preference>` reminder **at the end**

In one line: **a number or 4.2, pick one; 4.4 and the trailing reminder stack as needed.**

---

## 13. Meta-principles (for anything this file does not cover)

1. **Minimal information set**: the fewest tokens that fully outline the expected behavior.
2. **The right altitude**: neither brittle hardcoded logic nor guidance so vague it carries no signal.
3. **More context, fewer instructions.**
4. **Describe what you want, rather than forbidding what you do not.**
5. **When there are tools, fix the tools first**: interface design shapes behavior more than prompt text does.
6. **Anything that can be enforced deterministically should not be a prompt**: use a hook, a schema, or a type constraint instead of writing "please make sure to."
7. **As the model gets stronger, scaffolding becomes dead weight**: periodically go back and delete what you added for a weaker model.
8. **Test after changing**: run it on a real task rather than judging by feel.
