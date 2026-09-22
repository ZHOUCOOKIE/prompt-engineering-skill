---
name: prompt-engineering
description: Writes, rewrites, reviews, and audits prompts and system prompts using Anthropic's current official guidance, and selects the right model and effort level for the task before drafting. Use when the user asks to write a prompt, improve or fix a prompt, design a system prompt, or check a prompt before running a task — including phrasings like "write me a prompt for", "make this prompt better", "design a system prompt", "check my prompt before I run this", "turn these requirements into a prompt". Applies whether the resulting prompt will be pasted into Claude Code for a coding task or run in a desktop chat or Cowork session for a non-coding task. Also use when the user pastes a prompt and asks why it is not working, when they are authoring an agent system prompt or CLAUDE.md, or when they ask which model or effort level to use. Several widely repeated prompt techniques are now obsolete or rejected by the API, so consult this skill even when prompting from memory feels sufficient.
---

# Writing prompts

**Models in scope:** Claude Opus 5 and Claude Fable 5.1 (including Mythos 5.1). Guidance specific to earlier models is out of scope for this file. Several Fable 5.1 prescriptions are still published only on the Fable 5 page; they are kept because the Fable 5.1 page says existing Fable 5 prompts should perform well on Fable 5.1 without changes, and each is marked as such at the point of use.

**Why this skill exists:** several widely repeated prompt techniques are now obsolete on these models, rejected by the API, or actively reversed in effect. Your training instincts probably still carry some of them. **When instinct conflicts with Section 3, this file wins.**

**Basis:** Anthropic's official documentation and official/employee public material, snapshot 2026-09-03. For specific API parameters, confirm against the live docs — those are the only surface that tracks model updates.

---

## 0. Workflow

1. **Classify the target scenario** (Section 1) — determines which target reference to read
2. **Select model and effort yourself** (Section 2) — do not hand this question back to the user
3. **Fill in missing variables** (end of Section 1) — ask only what you genuinely cannot proceed without
4. **Draft**: read the matching target file, and `references/snippets.md` when you need a ready-made block
5. **Run the Section 7 checklist**
6. **Deliver**: the prompt itself, plus a delivery note covering five things — which model and effort you chose, which assumptions you made on the user's behalf, what is most likely to need tuning, whether anything can only be supplied by the user, and — **if the prompt embeds official English snippets, a reminder not to translate them**. Two or three sentences when that fits; **when there are many assumptions, err on the side of a longer note rather than hiding them** — the user cannot judge whether to change an assumption they cannot see.

---

## 1. Classify the scenario first

The user describes work in Claude desktop, and the resulting prompt goes to one of two places. **Decide which, because the writing differs substantially.**

| Signal | Scenario | Read this |
|---|---|---|
| Codebase, files, git, running tests, refactoring, bug fixing, scaffolding a project; the user says they will paste it into Claude Code or VS Code | **A: a task prompt for Claude Code** | `references/for-claude-code.md` |
| Research, analysis, writing, organizing source material, producing documents/spreadsheets/decks, scheduling, information synthesis; run directly in a desktop chat or Cowork session | **B: a task prompt for chat or Cowork** | `references/for-chat-and-cowork.md` |
| The user drops a prompt into an ongoing conversation and asks whether it looks right | **C: an inline check** | See Section 6, lightweight path |

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
- Changing the **top-level** effort between requests invalidates the prompt cache. Fable 5.1 and Opus 5 support a per-message effort change (beta) that preserves it — API only, see `references/models.md`
- At `xhigh`/`max`, leave headroom in `max_tokens` (it is a hard limit on thinking plus response text combined)

Model-specific behaviors and their prescriptions are in `references/models.md`.

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
| "use a subagent to double-check your own work" — that is, one given the model's own work *and its own criteria* | Every figure must trace to a specific location in a source document. A **fresh-context verifier with independent criteria** is not what this row removes — see `references/models.md` |

The test is simple: **if the basis for the check comes from the model itself, remove it; if it comes from something externally verifiable, keep it.** In Scenario B research and reporting tasks, cross-source reconciliation *is* the task, not self-verification — do not delete it by mistake.

Long autonomous runs on Fable 5.1 go the other way and want explicit periodic verification — see `references/models.md` (the instruction is published on the Fable 5 page and carries over).

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

## 4. General skeleton and structure

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

## 5. Reference files

Read as needed; do not load everything. All are linked directly from this file.

| File | When to read |
|---|---|
| **`references/for-claude-code.md`** | Scenario A. How to write a Claude Code task prompt, the verification affordance you must supply, the explore-plan-implement flow, and what belongs in CLAUDE.md versus this one prompt |
| **`references/for-chat-and-cowork.md`** | Scenario B. Writing for desktop chat and Cowork sessions, material placement, deliverable shapes, and the Cowork-specific file and skill capabilities |
| **`references/models.md`** | After choosing the model. Known behaviors of Opus 5 and Fable 5.1 with their prescriptions — split into what the Fable 5.1 pages state and what only the Fable 5 page states — and the one place where their verification strategies invert |
| **`references/snippets.md`** | When you need a ready-made block. 30 numbered entries, 29 of them **verbatim** English text from the official docs, with a note on which must be copied word for word and which may be adapted |
| **`references/agents.md`** | When the prompt will drive a long-running autonomous agent. Context engineering, multi-window workflows, cache hits, letting the model orchestrate itself |

### When you use an English snippet

Producing a non-English prompt with embedded official English blocks is the normal case, and it is handled the same way every time:

- Place the English **as a whole block**; do not splice it into surrounding sentences. Put it inside the section it governs, immediately after that section's heading
- **Tell the user in the delivery note that these blocks are official wording and should not be translated or "tidied up."** Seeing mixed languages, a user will naturally want to unify them, and doing so breaks the tuning
- If you need only one or two blocks and they are marked "general policy" in `references/snippets.md`, express them in the user's language instead of mixing for the sake of two sentences

Which snippets must be verbatim and which may be adapted is in the grading table at the top of `references/snippets.md`.

---

## 6. Three request types, handled differently

**Writing or revising a prompt (Scenario A/B):** run the Section 0 workflow, and write the delivery note to the Section 0 step 6 spec. Do not add explanations, disclaimers, or alternative-approach lists the user did not ask for.

**Auditing an existing prompt:** compare against Section 3 point by point and report **the specific problem and the specific fix**. Do not replace their prompt with a new one unless they asked for a rewrite. Order findings by severity: things that cause API errors (3.1, 3.2, 3.8, 3.12) first, behavior-reversing issues (3.4, 3.9, 3.10) next, style last.

**Inline check mid-conversation (Scenario C):** go light. **Do not make a production of it** — do not run the full workflow, do not read the reference files. Scan the Section 3 hard rules; if something is wrong say "this line is a problem: X, change it to Y"; if nothing is wrong say so and return to what you were discussing. The user wants to get back to what they were doing, not to receive an audit report.

---

## 7. Pre-delivery checklist

- [ ] You chose the model and effort yourself, and said so in the delivery note
- [ ] If official English snippets are embedded: the delivery note tells the user not to translate or rewrite them
- [ ] No prefill, no `budget_tokens`
- [ ] No `CRITICAL: You MUST` / `If in doubt, use [tool]` stacked to prevent undertriggering (**wording inside official snippets does not count** — for example 4.7 contains `you MUST read the file`, which is tuned official text, copy it as is)
- [ ] Targeting Opus 5: **no** self-verification or double-check instructions (external deterministic checks do not count; nor does `Regularly self-critique your approach and plan` inside the official research block — see `references/for-chat-and-cowork.md`)
- [ ] Targeting Fable 5.1: no "show your reasoning" instructions; no forced `tool_choice`; long autonomous runs carry the 4.18 autonomous block (4.17's progress-audit rule stays available; it was measured on Fable 5)
- [ ] Every constraint **you wrote** is stated positively and carries its reason (**wording inside official snippets does not count** — 4.6, 4.13 and 4.25 are stacked negatives by design; 4.25 opens `NEVER use generic AI-generated aesthetics…`, which is tuned official text, copy it as is)
- [ ] Every instruction has been read against: **how would a well-intentioned human misinterpret this?**
- [ ] No instruction repeated across the system prompt and a tool description (**a tool-invocation trigger the model needs does not count** — for example the system-prompt line shipped with snippet 4.23; see 3.7)
- [ ] The examples decision follows the Section 4 table — not added or removed reflexively
- [ ] Long-context case: material first, query last
- [ ] Output length is explicitly specified (especially necessary for Opus 5)
- [ ] There is a decidable success criterion; Scenario A has a check the model can run itself
- [ ] Read it through and cut every sentence whose removal would change nothing
- [ ] Golden test: **hand this prompt to a colleague with no context on the task — could they follow it?**

---

## 8. Symptom to prescription

Snippet numbers refer to `references/snippets.md`.

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
| Code review under-reports | Snippet 4.16; move filtering downstream. The adversarial-review caveat in `references/for-claude-code.md` is the opposite case — no downstream filter, so the reviewer must filter |
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

## 9. Meta-principles (for anything this file does not cover)

1. **Minimal information set**: the fewest tokens that fully outline the expected behavior.
2. **The right altitude**: neither brittle hardcoded logic nor guidance so vague it carries no signal.
3. **More context, fewer instructions.**
4. **Describe what you want, rather than forbidding what you do not.**
5. **When there are tools, fix the tools first**: interface design shapes behavior more than prompt text does.
6. **Anything that can be enforced deterministically should not be a prompt**: use a hook, a schema, or a type constraint instead of writing "please make sure to."
7. **As the model gets stronger, scaffolding becomes dead weight**: periodically go back and delete what you added for a weaker model.
8. **Test after changing**: run it on a real task rather than judging by feel.
