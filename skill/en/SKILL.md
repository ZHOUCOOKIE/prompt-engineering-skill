---
name: prompt-engineering
description: Writes, rewrites, reviews, and audits prompts and system prompts using Anthropic's current official guidance, and selects the right model and effort level for the task before drafting. Use when the user asks to write a prompt, improve or fix a prompt, design a system prompt, or check a prompt before running a task — including phrasings like "write me a prompt for", "make this prompt better", "design a system prompt", "check my prompt before I run this", "turn these requirements into a prompt". Applies whether the resulting prompt will be pasted into Claude Code for a coding task or run in a desktop chat or Cowork session for a non-coding task. Also use when the user pastes a prompt and asks why it is not working, when they are authoring an agent system prompt or CLAUDE.md, or when they ask which model or effort level to use. Several widely repeated prompt techniques are now obsolete or rejected by the API, so consult this skill even when prompting from memory feels sufficient.
---

# Writing prompts

**Models in scope:** the four current official models: Claude Fable 5.2 (including Mythos 5.2), Claude Opus 5.5, Claude Sonnet 5.5 and Claude Haiku 5.5. Claude Fable 5.1, Fable 5, Opus 5 and Sonnet 5 are on the official legacy list (still available), and they appear here in three roles only: as the source of some wording, as the starting point of a migration, or where a current page names them (as a fallback target, or in a price or benchmark comparison). Each model-specific claim is handled in one of three ways: what the current page states explicitly is used directly; what only a predecessor's page states, and the current page neither restates nor retracts, goes into a table in `references/models.md` marked as measured on that model; what has been retracted, or conflicts with the current page, is left out.

**Why this skill exists:** several widely repeated prompt techniques are now obsolete on current models, rejected by the API, or actively reversed in effect. Your training instincts probably still carry some of them. **When instinct conflicts with Section 3, this file wins.**

**Basis:** Anthropic's official documentation, official blog and support site, snapshot 2026-11-09. The few rules marked "User rule, not official guidance" are practices set by this skill's owner, not official guidance. For specific API parameters, confirm against the live docs — those are the only surface that tracks model updates.

---

## 0. Workflow

1. **Classify the target scenario** (Section 1) — determines which target reference to read
2. **Select model and effort** (Section 2) — by default you decide; for when to ask the user, see the user rules in Section 2
3. **Fill in missing variables** (end of Section 1) — ask only what you genuinely cannot proceed without
4. **Draft**: read the matching target file, and `references/snippets.md` when you need a ready-made block
5. **Run the Section 7 checklist**
6. **Deliver**: the prompt itself, plus a delivery note covering four things: which model and effort you chose and on what basis, which assumptions you made on the user's behalf, what is most likely to need tuning, and whether anything can only be supplied by the user. Two or three sentences when that fits; **when there are many assumptions, err on the side of a longer note rather than hiding them** — the user cannot judge whether to change an assumption they cannot see.

---

## 1. Classify the scenario first

The user describes work in Claude desktop, and the resulting prompt goes to one of two places. **Decide which first.**

| Signal | Scenario | Read this |
|---|---|---|
| Codebase, files, git, running tests, refactoring, bug fixing, scaffolding a project; the user says they will paste it into Claude Code or VS Code | **A: a task prompt for Claude Code** | `references/for-claude-code.md` |
| Research, analysis, writing, organizing source material, producing documents/spreadsheets/decks, scheduling, information synthesis; run directly in the Claude app | **B: a task prompt for chat or Cowork** | `references/for-chat-and-cowork.md` |
| The user drops a prompt into an ongoing conversation and asks whether it looks right | **C: an inline check** | See Section 6, lightweight path |

When the user says where the prompt will run, go by that: non-code work run in Claude Code (reading files in bulk, extracting, summarizing) is also A. When they do not say and it is not obvious, decide by the **verb** in the request: touching code is A, everything else is B. When both apply (for example, "analyze this repo's tech debt and write it up"), decide by the **final deliverable** — a code change is A, a document is B.

### Filling in variables

Ask only what you genuinely cannot proceed without. When something is missing but reasonably inferable from context, **assume it and state the assumption on delivery**.

"Must be settled" below means **this value must have a definite answer**, not that you must ask a question. Infer where you can, and record what you inferred in the delivery note.

| Variable | Requirement |
|---|---|
| **Success criteria** | **Must be settled.** Without a decidable success criterion you cannot write a good prompt and cannot check your own work |
| **Output shape and length** | **Must be settled** when the deliverable is a document or report. Write the length into the prompt rather than counting on effort: the Fable 5.2 page states that effort barely changes reply length, and the Opus 5 page states that lowering effort does not reliably shorten visible output (see Section 8, "Length control") |
| **Available tools or files** | Must be settled for Scenario A (which repo, which files); for Scenario B, depends on whether source material must be read |
| Target model and effort | Decide it yourself per Section 2, and tell the user the decision and its basis; ask only in the situation that user rule 1 in Section 2 describes |

When you need to ask about several things, use AskUserQuestion to ask them **all at once**, not one at a time.

---

## 2. Select model and effort

This step is mandatory before drafting, and the conclusion belongs in the delivery note. This section contains only what the official material gives a basis for; where the official material gives no threshold, none is set here. The numbers and finer detail are in `references/models.md`.

**Three user rules** (User rule, not official guidance; confirmed 2026-11-09, with rule 3 revised 2026-11-10. They govern how to choose among the options the official material gives, and do not change the official basis of the tiers below):

1. **You decide model and effort by default.** Only when a real trade-off exists and cannot be inferred from context, ask the user once before you output the prompt, with the options and a recommended one.
2. **Do not deliberately save usage, and do not raise anything without a basis.** The user has an Ultra subscription: do not pick a model or level lower than the tier the task corresponds to just to save usage, and do not raise the model or level without one of the bases listed in this section.
3. **Balance quality and consumption for batch subagents.** Run the whole batch on Sonnet 5.5 subagents first (in the official measurement this scored about the same as Opus 5.5 working part by part on its own at `medium`, for less), and re-run only the items that fail the check on the main session's model; each step and its official figures are under "Bulk processing" in this section. This rule sets where batch subagents start, and doing so does not break rule 2.

The official starting point (Anthropic's model-selection article of 2026-11-09, "Which Claude model should you build with?"): `you should start most work on Opus 5.5 at its default effort level`, which is `medium`. From lowest to highest price and capability, the four current models are Haiku 5.5, Sonnet 5.5, Opus 5.5 and Fable 5.2.

### Main session: choose by task

This table is that article's "Pick a model by task", row for row. The selection matrix in the platform docs also starts from Opus 5.5 (`Most workloads start with Claude Opus 5.5`), but files two rows differently: it puts large-scale refactoring under Opus 5.5, and analysis carried through to a finished document, spreadsheet or deck under Fable 5.2.

| Task | Starting model | Starting effort | The official reason |
|---|---|---|---|
| Everyday engineering and debugging; multi-PR feature work | Opus 5.5 | `medium` (default), then `high` if that falls short | Balance of cost and capability for everyday tasks |
| Classification, routing, tagging, high-volume extraction | Haiku 5.5 | `medium` (default) | Lowest cost and latency in the model family |
| Subagents that search files, read logs, or summarize | Haiku 5.5 or Sonnet 5.5 | The model's default | Wrong answers are cheap to spot, and the main model has the judgment calls anyway |
| Chat and interactive sessions | Sonnet 5.5 (Haiku 5.5 for simple chat at volume) | `low` or `medium` | Sonnet 5.5 is fast, especially at `low` and `medium` |
| Well-scoped coding: bug fixes, feature iteration, spec-driven work | Sonnet 5.5 | `medium`, `high` for harder tasks | It handles scoped tasks well, at half the Opus 5.5 token price |
| Everyday documents, slides and spreadsheets | Sonnet 5.5 | `high` (API default; `medium` in Claude Code) | It iterates fast and has a strong eye for design. For a complex financial model, or a document built from a large pile of source material, start on Opus 5.5 |
| Agent tasks you run repeatedly: investigation, review, drafting | Sonnet 5.5 | `medium` | Well-defined, repeated tasks rarely need a bigger model |
| Computer use and vision-heavy workflows | Opus 5.5 | `medium` | These runs are long agentic loops over many screenshots and images, the kind of work Opus 5.5 is built for |
| Major refactors and migrations, architecture planning, leading many subagents | Fable 5.2 | `medium`, then `high` if that falls short (the API default is `high`) | Anthropic's most capable generally available model for technical fields |
| Deep research, scientific or quantitative analysis | Fable 5.2 | `medium`, then `high` if that falls short (the API default is `high`) | Built for the hardest technical projects in coding, finance and the sciences |
| You aren't sure yet | Opus 5.5 | `medium` (default) | Claude Code uses it by default on the subscription plans and the Anthropic API, and the platform docs recommend it as the starting point |

The same article adds detail on moving down and up. Move down to Sonnet 5.5 for well-scoped everyday work, but `Just because you have a clear spec doesn't mean you should use Sonnet`: a migration or a performance optimization can have a clear spec and still be hard enough to warrant Opus 5.5, or Fable 5.2 for a major one. Move up to Fable 5.2 in three cases: the hardest technical projects in coding, finance and the sciences; Opus 5.5 at `high` still falls short; long unattended runs. The Claude Code model-config page: no Fable model is the default on any plan, and one has to be selected explicitly with `/model fable`.

### How to adjust effort on each model

The task table gives the starting level; when a task already meets a step-up or step-down condition in the table below from the start, use that cell's level directly.

| Model | Default | When to step up | When to step down |
|---|---|---|---|
| **Haiku 5.5** | `medium` | Knowledge work, longer agent tasks, strict instruction following → `high`. `xhigh`/`max` only where evals show a quality gain, and compare against Sonnet 5.5 at the same time | Chat, short tool tasks, simple high-volume requests → `low` (in long agent prompts it is more likely at this level to skip a search, stop early, or skip a check) |
| **Sonnet 5.5** | API `high`; Claude Code `medium` | Agentic coding and multistep tool use: start well-specified tasks at `medium`, and harder or longer ones → `high`. `xhigh`/`max` only where evals show a quality gain; Anthropic's Sonnet 5.5 article says that at those two levels some tasks lose Sonnet's balance of quality, speed and cost, and in that case to consider Opus 5.5 | Chat and other latency-sensitive work → `medium` or `low` |
| **Opus 5.5** | `medium` | → `high`: work where verification matters or edge cases are likely, such as fixing a bug in an existing codebase (Claude Code docs). → `xhigh`/`max`: officially, `Reserve xhigh and max for work where you've measured a quality gain`; the model-selection article adds: on a hard task, test Fable 5.2 before you raise Opus 5.5 past `high` | Cost or latency matters and evals show quality holds → `low`; likewise quick back-and-forth where you review each result (brainstorming, a first sketch, a small change like a rename) |
| **Fable 5.2** | API `high`; Claude Code and the Claude app `medium` | `medium` suits interactive coding, chat and everyday knowledge work; → `high`: hard coding, research, agentic work and long unattended runs (officially, the step from `medium` to `high` raised scores on most of the coding and knowledge-work evaluations Anthropic tracks, and it also uses more tokens and time). → `xhigh`: the hardest long-running coding and agentic work, where a gain has been measured. `max`: for measuring the model's ceiling, or the rare task where time and tokens do not matter | Where evals show quality holds → `medium`; quick back-and-forth, well-specified tasks, high volume (such as subagents) → `low` |

Sources: the per-model sections of the effort page, the four models' prompting pages, and Claude Code's model-config page ("Choose an effort level"); Fable 5.2's `medium` default in Claude Code and the Claude apps comes from the model-config page and the launch announcement. In Claude Code, only when the next message needs deeper reasoning, write `ultrathink` in that prompt; on ordinary tasks `max` tends to overthink.

### The order when quality falls short or cost is too high

The steps the model-selection article gives, to take in order when a model is not good enough for a task:

1. Check that the model has a way to verify its own work: a test, a build, or a script that calls the endpoint. As the article puts it, giving the model a means of self-verification is often more cost-effective than changing effort levels, and adds less latency
2. Raise effort one level
3. Move to the next model up, **starting at a lower effort than you used on the smaller model**

The official statements that go with it:

- **Tune effort first, then change model**: `Tuning effort is often a better lever than switching models` (choosing-a-model page)
- **Past `high`, compare both paths**: one more effort level can cost as much as a bigger model; once you are past `high`, test the next model up at a lower effort instead
- **Raising effort does not give a model knowledge or skill it lacks**; for that, change model
- **Cost too high, quality fine** → try a lower effort on the current model (cost page)
- **Do not carry over a level tuned on another model**; **compare cost per completed task, not per token**

**The official statements do not fully agree in the places below; in the delivery note, point out whichever applies** (Scenario B has one more, the support site's `try Extra high (xhigh) first`; see `references/for-chat-and-cowork.md`)**:**

- **When to move from Opus 5.5 to Fable 5.2.** The model-selection article: Opus 5.5 at `high` still falls short. The choosing-a-model page: evals at `xhigh` or `max` still fall short. The cost page: when quality is not good enough, restore effort first if you lowered it, otherwise move to the next model up at its default effort. The what's-new page and the models overview: still short at higher effort. This skill follows the article's three steps, since it is the only one of these sources that puts self-verification first; each source's wording is in `references/models.md`
- **Research tasks.** The task table and the selection matrix both put deep research under Fable 5.2; on DeepResearch Bench II the cost page measured Fable 5.2 at 2.9 times the cost of Opus 5.5 for a score 1.9 points lower, and its words are `Measure what a more expensive model buys on your own workload`
- **Where Fable 5.2 starts.** The task table (the selection article) starts it at `medium`, which is also the default in Claude Code and the Claude apps; the Fable 5.2 prompting page, the what's-new page, the effort page and the choosing-a-model page all say to start at `high`, the Claude API default, and the prompting page says `high` suits hard coding, research and agentic work. In Scenarios A and B, start at `medium` as the task table does and go to `high` if that falls short; for a prompt that runs on the API, start at `high` as the platform pages do
- **Which model to start from.** The selection article and the selection matrix: start most work on Opus 5.5. The choosing-a-model page also gives an efficiency-first route: implement with Haiku 5.5 first and upgrade only if necessary for specific capability gaps; the cases it lists are initial prototyping and development, applications with tight latency requirements, cost-sensitive implementations, and high-volume, straightforward tasks. This skill follows the task table; when the user's task falls in one of those four cases, tell them about this route as well
- **Which model for everyday coding in Claude Code.** The selection article: start everyday engineering on the default, Opus 5.5. The Claude Code best practices: `the default Opus 5.5 handles most coding well and Sonnet costs less on routine work`. The Claude Code costs page: `Sonnet handles most coding tasks well and costs less than Opus. Reserve Opus for complex architectural decisions or multi-step reasoning.` This skill follows the task table: Sonnet 5.5 for well-scoped coding, Opus 5.5 for other everyday engineering

### Subagents: choose by task (Scenario A)

In Claude Code, a subagent with no model specified uses the same model as the main session, so when the main session is on Fable 5.2, subagents also run on Fable 5.2 by default (for the exceptions see `references/for-claude-code.md`: several built-in subagents have their own model, and setting `CLAUDE_CODE_SUBAGENT_MODEL` changes the default too). The Fable 5.2 page's words: `subagents can account for a large part of what a session costs, so check which model your harness runs subagents on by default`. For the resolution order, the parameters each call can pass, concurrency, and behavior on errors, see the "Subagents" section of `references/for-claude-code.md`.

| What the subagent does | Model | Official basis |
|---|---|---|
| Searching files, reading logs, summarizing, checking documentation links | `haiku` or `sonnet`, with effort at the model's default | The model-selection article's task table; Claude Code best practices: `Set model: haiku in a subagent's definition if its work is simple` |
| High-volume batch work with outputs you can check | Try `haiku` first | Cost page: `High-volume work with outputs you can check` → `Try Claude Haiku 5.5 first` |
| Work that exceeds one context window, split into one subagent per part | A model cheaper than the main session's; with an Opus 5.5 orchestrator, `sonnet` was measured at about the same score as Opus 5.5 alone at `medium`; `haiku` was measured too and scored clearly lower; for a Fable 5.2 orchestrator see the numbers below | Cost page: `Split the work and hand the parts to lower-cost models`. Measurement below |
| Teammates in an agent team | `sonnet` | Claude Code cost page: `Use Sonnet for teammates` |

**The official division of labor** (model-selection article): `Keep small models on work where a mistake is cheap to spot, like finding files, running tests and reading logs. Keep code edits and judgment calls on the main model.` The bigger the main model, the more a lean main context matters: `Keep the expensive model's context for the decisions, and let cheaper subagents do the verbose reading.`

**How user rule 3 reads this table** (User rule, not official guidance; confirmed 2026-11-10. The user is on the Ultra plan and wants dependable quality with weekly usage that lasts. The table above lists the official options; this table says which one to take):

| What the subagent does | Pass on dispatch | Basis for this choice |
|---|---|---|
| Searching files, reading logs, summarizing, checking documentation links | `model: "sonnet"`, at its Claude Code default effort, `medium` | Of the official `haiku` or `sonnet`, take `sonnet`. No higher: the official reason for giving this work to a small model is that a mistake is easy to spot and the judgment stays with the main model |
| Batch work where each output can be checked item by item (extraction, labeling, checking) | Step 1 of the "Bulk processing" table; items that fail the check go to step 2 | The official `Try Claude Haiku 5.5 first` is not taken here; the official figures for both steps are in that table |
| Batch work where each item needs a judgment and no item-by-item check can be written (file-by-file review, defect search) | Step 1 of the "Bulk processing" table; step 3 when the user says they want more accuracy and the main session is on Fable 5.2. With no failure signal, step 2 does not apply | The cost page's defect-search benchmark measured this kind of work: Fable 5.2 with Sonnet 5.5 subagents scored 3.2 to 7.5 points lower than Fable 5.2 working part by part on its own, for 54% to 71% less. Mention these figures in the delivery note |
| Code edits, judgment calls, the final independent review | No `model` (the subagent follows the main session); for the review also pass `effort: "high"` | The selection article's division of labor: `Keep code edits and judgment calls on the main model` |
| Teammates in an agent team | `sonnet` | Same as the official guidance |

**What has officially been measured** (cost page, internal defect-search benchmark, work that exceeds one context window): Opus 5.5 with Sonnet 5.5 subagents cost about a third less than one Opus 5.5 session per part, with about the same score; with Haiku 5.5 subagents the score was clearly lower; with Fable 5.2 as the orchestrator instead, cost was 19% higher at about the same score. Fable 5.2 working alone (`low` to `high`) scored 3.2 to 7.5 points higher than it did with Sonnet 5.5 subagents, at 2.2 to 3.4 times the cost. For Haiku 5.5 subagents on coding tasks, the cost page's conclusion is `an alternative to lowering effort, not better`.

**When delegating is worth it**: official measurements found delegation pays off in only two cases: routine work with a long cost tail (for this case the official material says to measure accuracy and cost first), and work that exceeds one context window. When the work is one dependent chain, or fits in one context without a long cost tail, the orchestrator's model on its own at lower effort comes out ahead (in every case measured, on earlier models); the model-selection article's way of putting it is that there the planning and merging cost more than they save. The Claude Code docs: subagents bring parallelism, keep long output out of the main session, and give a fresh pair of eyes that has not seen the process, `but not a smarter answer`. The cost page's test for adding a second model at all: sweep effort on your current model first, and `Build a pairing only if it beats the single model's whole curve of score against spend`.

**Subagent effort**: for subagents that search, read logs or summarize, the model-selection article sets the starting level at the model's default. The effort page lists subagents as an example for `low`, and the Fable 5.2 page says `low` suits fast, intelligent subagents in high-volume work. Measure before you lower it: on the defect-search benchmark, Opus 5.5 working alone scored 34 points lower at `low` than at `medium`, and Fable 5.2 working alone scored 4.3 points lower at `low` than at `high`, at 37% less cost.

### Bulk processing: model and level for the batch subagents

**When to use it** (the scope of user rule 3, which the user set; it is not an official trigger): Scenario A, the main session is on Fable 5.2 or Opus 5.5, and before drafting you can already foresee that the task will dispatch many subagents in parallel to do bulk work to one shared set of criteria: reading dozens of files or reading packets, or extracting / labeling / checking hundreds or thousands of items. For such a task, write this straight into the prompt, and mention it in one line in the delivery note.

**What the official material says about this kind of work** (cost page, "Orchestrator strategy"): what suits orchestration is `Work that fans out across genuinely independent files, documents, or cases, especially more than one context window of it`; the approach is to split the work and hand the parts to cheaper models, and the orchestrating model `Plans, dispatches, and synthesizes`. The selection article's wording is to keep code edits and judgment calls on the main model. The official material also measured two things; mention in the delivery note whichever applies to the user's task: for work that fits in one context without a long cost tail, the orchestrator's model alone at lower effort came out ahead (measured on earlier models); and Fable 5.2 working alone scored higher than the setup with Sonnet 5.5 subagents (see the numbers in the previous subsection), so what splitting buys is lower cost.

**Which model and level for batch subagents** (user rule 3, set 2026-11-10. The figures come from the cost page's internal defect-search benchmark: a corpus of about 22 million tokens in 201 parts, run on Claude Managed Agents, with the orchestrator at `high` and each subagent at its model's default effort. Figures are mean F1 and cost per run):

| Step | Pass when dispatching | What was officially measured | When to use it |
|---|---|---|---|
| 1 | `model: "sonnet"`, `effort: "high"` | Fable 5.2 orchestrating Sonnet 5.5 subagents: 0.833, $165. In the measurement Sonnet 5.5 ran at its default on that platform, `high`; in Claude Code its default is `medium`, so pass the level explicitly | **Default**: run the whole batch at this step first |
| 2 | no `model` (the subagent follows the main session); `effort: "medium"` under a Fable 5.2 main session, `effort: "high"` under an Opus 5.5 one | One session per part: Fable 5.2 at `medium` 0.877, $378; Opus 5.5 at `high` 0.850, $236. This benchmark did not measure a configuration where the subagents run on the orchestrator's own model | Re-run only the items that failed the check in step 1. Under a Fable 5.2 main session it draws on the weekly Fable allowance |
| 3 | no `model` for the whole batch (every subagent follows the Fable 5.2 main session), `effort: "medium"`, or `"high"` when the user wants the highest score | Fable 5.2 itself with one session per part (not with Fable 5.2 subagents): `medium` 0.877, $378; `high` 0.909, $563 | Only when the user says this batch needs more accuracy than the first two steps give and enough of the weekly Fable allowance is left. It draws on the weekly Fable allowance |

- Step 2 follows the cost page's "Re-run failures at higher effort": when the outcome is checkable, run everything at a low setting and re-run only the failures at a higher one. Anthropic computed this for effort on Opus 5.5, where it cost about 40% less than running everything at `high`, at the same pass rate or slightly better; here the same policy is applied to the model. The re-run goes back to the main session's model, not to `opus`, on the strength of the launch announcement's five cost charts (FrontierCode 2.0, WANDR, BenchCAD 2.1, Humanity's Last Exam, Chartography; cost is per task or per attempt; the differences below are computed from the charts' data points): Fable 5.2 at `medium` scored above Opus 5.5 at `high` on all five, by 2.7 to 7.2 points, at between 29% less and 11% more cost. On the defect-search benchmark it scored 2.7 points higher for 1.6 times the cost. Two conditions apply: you need a failure signal (`a checker that passes bad work lets those failures through`), so the prompt must include a check that passes or fails each item; and each failure takes two runs' worth of time
- `haiku` is not in this table: on the same benchmark Fable 5.2 with Haiku 5.5 subagents scored 0.716, $85. Use it only when the user names it
- Fable 5.2 at `low` is not in this table either. Against Opus 5.5 at `high`, two sets of official figures point different ways: on the launch announcement's five cost charts it cost less every time (1% to 37% less), with scores between 2.8 points lower and 5.0 points higher, for example 34.2% at $2.10 against 34.5% at $2.42 on FrontierCode 2.0; on the defect-search benchmark (one session per part, 0.866, $357) it scored 1.6 points higher for 1.5 times the cost. It is not the default for a whole batch: only the defect-search benchmark measured a configuration like step 1, and there it scored 3.2 points above step 1, which cost 54% less, and it draws on the weekly Fable allowance that the main session is using too. Steps 2 and 3 do not use it either: across these six measurements it cost 6% to 29% less than `medium` and scored 0.2 to 5.5 points lower, and those two steps are there for accuracy. Use it when the user names `low`
- `opus` at `medium` is not in this table either. On the four cost charts in the Sonnet 5.5 launch announcement (differences computed from the charts' data points), against Sonnet 5.5 at `high` it scored 4.7 to 14.6 points higher on the three coding benchmarks (Terminal-Bench 4.0, FrontierCode v1.1, CursorBench 4.0) for 1.5 to 1.9 times the cost, and 8 Elo higher on the knowledge-work benchmark AA-Briefcase v1.1 for 1.1 times the cost. On the defect-search benchmark, working part by part on its own, it scored 0.827, $200, no higher than the step 1 configuration. Step 1 does not use it: on AA-Briefcase it was only 8 Elo above Sonnet 5.5 at `high`, on the defect-search benchmark it was no higher than the step 1 configuration, and a batch with checkable outputs has step 2 to re-run failures. Use it when the user names it
- Under an Opus 5.5 main session: step 1 is the same (on the same benchmark Opus 5.5 orchestrating Sonnet 5.5 subagents scored 0.831, $139, which is 1.9 points below Opus 5.5 working part by part on its own at `high` (0.850, $236)); step 2 follows the Opus 5.5 main session, at `high`; there is no step 3
- The figures in the table come from one benchmark, a defect search in code, and the cost page calls its measurements `directional, not guarantees`. When the user's task is far from that, run a few items at two neighbouring steps before settling. On research tasks the official measurement points the other way: on DeepResearch Bench II, at default effort, Fable 5.2 scored 1.9 points lower than Opus 5.5 for 2.9 times the cost (see "Research tasks" above)
- On Ultra, Fable usage draws from the same weekly limit, up to 50% of it, and Fable models `use them faster than other Claude models`; usage by `sonnet` and `opus` subagents does not count toward that 50%, but it counts toward the weekly total all the same. The official pages do not say how each model converts into subscription usage, so the dollar figures above show relative consumption only
- Two official figures point to further savings; mention them in the delivery note where they apply. First, on the same benchmark Fable 5.2 as the orchestrator in Opus 5.5's place cost 19% more and scored about the same (the orchestrator was measured at `high`); the inference that a pure batch job can run from an Opus 5.5 main session at `high` is this skill's, not an official recommendation. Second, when the batch runs as a dynamic workflow, agents in the same run that share the same model, effort level, agent type, tools, output schema and working directory read the prefix a sibling has already cached, once that sibling's response has begun (workflows page)
- The final independent review is verification work: pass no `model`, so it follows the main session, with `effort: "high"` (the Claude Code docs describe `high` as `Work where verification matters or edge cases are likely`)
- Use `xhigh` only where a quality gain has been measured; do not drop Sonnet 5.5 to `medium` to save: the cost page measured it about 7 points lower than at `high` on the SWE-bench Pro subset

The `model` and `effort` parameters apply only to non-fork subagents: a fork runs on the main session's model, and fork mode is on by default, so the prompt should say that this batch goes to non-fork subagents (see the fork item in `references/for-claude-code.md`). If this batch runs as a dynamic workflow instead, have the script name the model chosen from the table for the batch stage (nothing for step 3, which follows the session's model): the workflows page says a model the script names for a stage counts as the per-invocation model; the official pages do not say what effort a workflow agent runs at. Passing `effort` per call requires Claude Code 2.1.292 or later; for the concurrency limit, and for how results come back when a subagent errors or hits a usage limit, see the "Subagents" section of `references/for-claude-code.md`.

### Usage and cost

Only what the official material says; the full numbers are in `references/models.md`, "Prices, usage limits and ways to save usage". The ones you need most often when writing a prompt:

- Price per million tokens (input / output): Fable 5.2 $8 / $40, Opus 5.5 $4 / $20, Sonnet 5.5 $2 / $10, Haiku 5.5 from $0.10 / $0.50
- On subscriptions, subagent requests count toward the same usage limit as the main session; on Max and Ultra, Fable models can use at most half of the weekly limit, and they use it up faster than other models; the higher the effort, the sooner you reach the limit
- Output tokens cost five times as much as input tokens, and in an agent loop every token the model writes is counted again as input on every later turn, so ask it for the answer you will actually read
- When outputs can be checked (API): run once at `low` and re-run the failures at `high`; officially measured on Opus 5.5 at about 40% cheaper than running everything at `high`, with the same pass rate

**Where these parameters live**:

| | Scenario A: Claude Code | Scenario B: Claude app | Raw API |
|---|---|---|---|
| Model | `/model`, `--model`; for subagents, see above | The model menu next to the send button | `model` |
| effort | Set in `/effort` or `/model`, saved per model (`max` applies to the current session only); `ultrathink` covers one turn only | "Effort" in the model menu, with the recommended level marked "Default" | `output_config: {effort}` |
| thinking | Managed by the client | Cannot be turned off on the four current models | See 3.2 |
| `max_tokens` | Cannot be set in the prompt (only the `CLAUDE_CODE_MAX_OUTPUT_TOKENS` environment variable) | No such setting | You set it; see `references/models.md` |

So for Scenario A and B delivery notes, naming the model and the effort tier is enough; do not write `max_tokens` or `thinking` into them.

---

## 3. Obsolete techniques (hard rules)

3.1, 3.2, 3.8 and 3.12 are rejections at the API layer; the rest are behavior problems the official material states. Where the models differ:

| | Fable 5.2 / Mythos 5.2 | Opus 5.5 | Sonnet 5.5 | Haiku 5.5 |
|---|---|---|---|---|
| Prefill (3.1) | 400 | 400 | 400 | 400 |
| `budget_tokens` (3.2) | 400 | 400 | 400 | 400 |
| Turning thinking off (3.2) | 400 | 400 | 400; use `between_tools` instead, `high` and below only | Works at `high` and below; `xhigh`/`max` return 400 |
| Non-default sampling parameters (3.8) | 400 | 400 | 400 | 400 |
| `reasoning_extraction` refusal (3.9) | Yes (the official text names Fable 5.2) | Yes | Yes | No such category |
| Forced `tool_choice` (3.12) | 400 | 400 | 400 | Accepted |

### 3.1 Do not use prefill (prefilled assistant responses)
**None of the four current models supports it.** Prefilling the last assistant turn returns a 400 error: officially, `Starting with Claude 4.6 models and Claude Mythos Preview, prefilled responses (providing a partial assistant message for Claude to continue from) on the last assistant turn are no longer supported`; the Haiku 5.5 migration guide states that Haiku 5.5 rejects it even with thinking turned off.

The official replacements are **structured outputs or system prompt instructions**. In practice: for structured output use Structured Outputs, `output_config.format`, or state the schema directly; to eliminate preamble write `Respond directly without preamble. Do not start with phrases like 'Here is...', 'Based on...', etc.` (verbatim from the prompting best practices page, "Migrating away from prefilled responses"); for classification use a tool with an enum field; for continuations move the prior text into the user message.

### 3.2 Do not use `budget_tokens` — how to turn thinking off depends on the model
None of the four current models accepts manual extended thinking: `{"type": "enabled"}` with `budget_tokens` returns 400. Omit `thinking` or send `{"type": "adaptive"}` (the two are equivalent), and control depth with `effort`. For a hard ceiling use `max_tokens`.

- **Fable 5.2 and Opus 5.5**: thinking is always on and cannot be turned off; `{"type": "disabled"}` and `{"type": "between_tools"}` both return 400. The official error text:

```
"thinking.type.disabled" is not supported for this model. Use "thinking.type.adaptive" and "output_config.effort" to control thinking behavior.
```

- **Sonnet 5.5**: `disabled` returns 400 at every effort level; to turn off up-front thinking, send `thinking: {"type": "between_tools"}`, which works only at `high` and below
- **Haiku 5.5**: `disabled` works at `low`, `medium` and `high`, and returns 400 at `xhigh` and `max`

Because a response can begin with a `thinking` block (empty at the default `display: "omitted"`), **select content blocks by their `type` field rather than by position**, and pass `thinking` blocks back unmodified in tool-use loops.

### 3.3 Do not stack instructions where context would do
The core principle: **give context, not a long list of instructions.**

The Claude Code team removed over 80% of its system prompt for models like Opus 5 and Fable 5 with no measurable loss on its coding evaluations. Claude Code's best practices now put it this way: write down only what Claude cannot find by reading the repo (`Only write down what Claude can't find by reading the repo, or what it would otherwise spend time rediscovering every session`). The Fable 5.2 page describes the other side: cautious instructions pile up in memory, notes and instruction files, the model follows them closely, and so it might ask for approvals it does not need or hold back routine work (a few early testers saw this, mostly where several agents work together).

Rather than enumerating ten "do not X" clauses, write one positive description and state the intent. The best practices page gives an example, written for Opus 4.6: change `Default to using [tool]` to `Use [tool] when it would enhance your understanding of the problem`.

> **Emphasis is not the fix.** Claude Code's best practices state that when Claude keeps ignoring a line in CLAUDE.md, `adding IMPORTANT probably won't fix it`: either the file has grown long enough that the line is buried, and the file should be shorter, or the line is a rule that cannot have exceptions, and it should be a hook. The same page lists adding IMPORTANT or all caps to instructions that keep being missed among the old practices you can stop: current models follow a plainly worded line, and piling on emphasis mostly makes them over-apply the rule.

### 3.4 Self-verification instructions: what the official pages say, model by model
The pages say different things, so one rule cannot cover every model. The full table of quotes and sources is in the "Verification instructions" section of `references/models.md`. When writing a prompt, handle it as follows:

- **The general advice** (best practices page) is to have the model check itself: `This catches errors reliably, especially for coding and math`
- **Opus 5 (legacy) is the only model this advice names as an exception**: it verifies its own work, and carried-over verification instructions can cause over-verification; the official words are `remove these instructions rather than rewriting them`. The Opus 5.5 pages neither restate nor retract this. Anthropic's article "What a task costs on Opus 5.5" (2026-09) reports one measurement: on an internal customer support benchmark migrated from Opus 4.8 to Opus 5.5 (at `low`), an audit removed ritual instructions that made the model write more and repeat tool calls (`a mandatory six-step procedure, a scratchpad rule, a verify-twice rule, and instructions that contradicted each other`) and cut cost by a further 9%; the article itself says to treat this as an example from one benchmark
- **Fable models in Claude Code**: `Skip the verification reminders: it verifies its own work with less prompting, so reminders to test or check are usually unnecessary`
- **Sonnet 5.5 at `low`, and Haiku 5.5 at `low` and `medium`, doing coding** sometimes report completion without running checks: when you see this, add snippet 4.46
- **Sonnet 5.5 at `xhigh` and `max`** goes the other way and might start extra review rounds on its own: to rein it in, add snippet 4.42

One point that does not depend on the model (Claude Code best practices): in Scenario A, give it a check it can run whose result is pass or fail, and ask it for evidence. This is not a reminder to look again; it is an external signal. See `references/for-claude-code.md`.

### 3.5 Do not stack negative constraints
The best practices page's principle is `Tell Claude what to do instead of what not to do`, and it says to give the reason:

- ✗ `NEVER use ellipses`
- ✓ `Your response will be read aloud by a text-to-speech engine, so never use ellipses since the text-to-speech engine will not know how to pronounce them.`

The second version still contains "never", but it hangs on a **reason**. Once the model knows why, it can decide for itself in edge cases the rule never covered. The Fable 5.2 page points the same way for rules that have built up: `rewrite absolute rules so that they state their scope and reason`. When an exception is needed, write it into the rule itself; in the official measurements, a separate line stating it barely changed the result.

### 3.6 Do not substitute absolute rules for judgment
- ✗ `Never write multi-paragraph docstrings or multi-line comment blocks — one short line max`
- ✓ `Write code that reads like the surrounding code: match its comment density, naming, and idiom`

### 3.7 Do not repeat an instruction across the system prompt and a tool description
Guidance on how to use a tool belongs **in the tool description, once**.

> **What does not count as repetition: when to invoke the tool.** Once forced tool calls are rejected, the official replacement is to state in the prompt when to use the tool (see 3.12). Snippet 4.23 works the same way: the Fable 5 page notes that Fable 5 rarely calls `send_to_user` from the tool description alone, so the official guidance pairs the tool with a system-prompt line telling it **when** to use it.

### 3.8 Do not rely on `temperature` for variety
On the four current models, non-default `temperature`, `top_p`, or `top_k` values return 400. The thinking page: `non-default temperature, top_p, or top_k values return a 400 error on every request, regardless of whether thinking is used`, and the models it lists include these four; the Fable 5.2 overview says `Omit temperature, top_p, and top_k`. Two traps sit beside it: `top_p` defaults to `0.99`, so `top_p: 1` returns 400, and so does sending `temperature` and `top_p` together even at their defaults; and from API version `2026-10-21`, sending any of the three at any value returns 400 on every model. Get variety from the prompt instead. For design work, the Sonnet 5 page's approach is to have the model **propose options before committing**:
```
Before building, propose 4 distinct visual directions tailored to this brief (each as: bg hex / accent hex / typeface, plus a one-line rationale). Ask the user to pick one, then implement only that direction.
```
(Snippet 4.25, verbatim from the Claude Sonnet 5 page.)

### 3.9 Do not ask the model to reproduce or display its internal reasoning
Instructions like "show your thinking" or "explain your reasoning process" **can trigger** the `reasoning_extraction` refusal on **Fable 5.2, Opus 5.5 and Sonnet 5.5** (Haiku 5.5 has no such category). The thinking page: `On Claude Fable 5.2, Claude Fable 5.1, Claude Opus 5.5, Claude Opus 5, Claude Sonnet 5.5, and Claude Fable 5, a request that attempts to elicit the model's internal reasoning as part of the response text might be refused with stop_details.category: "reasoning_extraction"`. The refusals page names the usual sources: a `<thinking>`, `<reasoning>` or scratchpad section the model fills in before it answers; a `reasoning`, `thinking` or `trace` field in JSON output or a tool input; the model's private notes or a running log of its reasoning; reasoning asked for verbatim or in full. The wording can sit in a system prompt, a skill or a tool description; the Claude Code error reference also lists CLAUDE.md, a subagent prompt, an output style and an MCP tool description, and gives a way to check: start a session with `claude --safe-mode`, which disables these customizations, and send the same prompt.

**What stays allowed:** a short explanation of the answer or a summary of the actions taken, and the evidence behind a result (the refusals page). This category has no recommended fallback model, server-side fallback does not retry it, and the decline is returned to you, so change the prompt rather than resend it. For reasoning visibility, set `display: "summarized"` and read the `thinking` blocks; to surface progress during a long run, set `display: "updates"`; for content the user must see verbatim partway through a turn, use a send-to-user tool (snippet 4.23).

### 3.10 Do not write "do not think"; to make it think less, lower effort first
- **Fable 5.2**: sentences like `answer directly without deliberating` do not stop it thinking at `medium` and above. The official words: `Remove these lines and lower the effort level`
- **Sonnet 5.5**: telling it in the system prompt to think less `doesn't reliably reduce its thinking`; with `between_tools`, instructions not to think also make it more likely to write internal XML tags into visible output, so remove them
- **Haiku 5.5**: telling it to answer directly did not stop it thinking; for less thinking, lower effort, or turn thinking off at `high` and below
- **Opus 5.5**: when migrating a prompt that ran with thinking turned off, the official instruction is `check whether you still need the instruction, and remove the no-thinking rule either way`. Lowering effort is more reliable than a prompt instruction; when time to first token still matters, the official material gives one line, `Answer directly without deliberating.`, and if you add it, measure quality

On all four models, the first way to get less thinking is to lower effort. The best practices page also keeps an example that uses the prompt to make thinking trigger less often (the second block of snippet 4.15), and states that it does not apply to Fable 5.2. The Claude Code model-config page has one more sentence: if you want Claude to think more or less often than the current level produces, you can say so in the prompt or in CLAUDE.md, and the model responds within its effort setting. That differs from what the Fable 5.2, Sonnet 5.5 and Haiku 5.5 pages say above; this skill follows each model's own page.

### 3.11 Audit inherited prompts for prefill
A prompt you did not write yourself — an older template, a pre-4.6 codebase, anything generated by tooling you do not control — can still end with a partial assistant turn. That returns 400 on current models. **Re-run this section over anything inherited**, and check the trailing turn first. Claude Code ships a `prompt-audit` subcommand in its `claude-api` skill, which the official text describes as `for auditing prompts and tool descriptions for patterns written for older models` (changelog 2.1.221, 2026-08-04), and `/doctor prompt-audit` (also `/checkup prompt-audit`), which audits CLAUDE.md files, skills, agents and commands, in the official words `for prompting patterns written for older models` (changelog 2.1.283). Both are first passes, not a substitute for this section.

### 3.12 Do not force tool use — except on Haiku 5.5
`tool_choice` of type `any` or `tool` returns a 400 `invalid_request_error` on **Fable 5.2, Mythos 5.2, Opus 5.5 and Sonnet 5.5**; `auto` and `none` are unaffected. The official error text:

```
tool_choice: type "tool" and "any" are not supported for this model.
```

The same validation applies to the token counting endpoint. For schema-valid JSON keep `tool_choice: {"type": "auto"}` and set `strict: true` with strict tool use, or move the schema to structured outputs. In organizations that use customer-managed encryption keys (CMEK), neither is available on Fable 5.2 or Mythos 5.2: remove `strict` and `output_config.format`, and validate in your own code. To make the model call a tool rather than reply in text, state in the prompt when the tool applies.

**Haiku 5.5 accepts forced `tool_choice`**, but the response then starts directly with the tool call and has no `thinking` block. If you want it to think before calling, use `auto` and state in the prompt when to use the tool.

### 3.13 Old instructions to review when migrating to Fable 5.2
These are the items listed under "Review instructions written for earlier models" on the Fable 5.2 page. Check each one in the prompts, skills and harnesses carried over from Fable 5.1 or Opus 5.5:

- **Standing instructions to delegate**: lines that push work to subagents. If it starts more subagents than the task needs, remove them
- **A token budget the harness writes into the prompt**: the size of the budget changes how much it spends; try removing the budget
- **Harness code that edits earlier turns** (API only): see `references/agents.md`
- **The two "treat that answer as done" lines** (snippet 4.34): they make it far less likely to point out its own earlier mistakes
- **Lines telling it to think less**: remove them, and lower effort instead (3.10)
- **Hard style rules for writing**: they can make its prose choppy; first remove the ones no longer needed, then add snippet 4.40
- **Rules, notes and memories that have built up**: delete the cautions whose reason no longer holds, and rewrite absolute rules so that they state their scope and reason

Scenario A has a further table, "What you can stop doing" from Claude Code's best practices; see `references/for-claude-code.md`.

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

**Structuring:** XML tags or Markdown headers. Use consistent, descriptive tag names, and nest them when the content has a natural hierarchy.

**Long-context placement (20k+ tokens):** put long material **at the top** and the query and instructions **at the end**. Queries at the end can improve response quality by up to 30 percent in tests, especially with complex, multidocument inputs. Wrap multiple documents in `<document>` tags with `<source>` and `<document_content>` subtags.

### Whether to give few-shot examples

| Goal | Give examples? | Basis |
|---|---|---|
| Lock output **format, tone, or structure** | **Yes** — 3–5, diverse, wrapped in `<example>` tags | Official docs: "Examples are one of the most reliable ways to steer Claude's output format, tone, and structure" |
| Demonstrate a **reasoning pattern** | Worked examples — each one the problem, the method to apply, and the expected answer; the model carries the approach into its own thinking blocks. Do not build in a `<thinking>` section for it to fill in (3.9) | Official docs |
| Teach it **how to use tools** | Change the interface first, not examples: make the parameters of your tools, scripts and files more expressive | Anthropic's 2026-07 context engineering article (written about the Opus 5 and Fable 5 generation): `With our newest models, we've found that giving examples actually constrains them to a certain exploration space` |

---

## 5. Reference files

Read as needed; do not load everything. All are linked directly from this file.

| File | When to read |
|---|---|
| **`references/for-claude-code.md`** | Scenario A. How to write a Claude Code task prompt, the verification affordance you must supply, how to set the model and effort for subagents, and what belongs in CLAUDE.md versus this one prompt |
| **`references/for-chat-and-cowork.md`** | Scenario B. Writing prompts for chat and Cowork in the Claude app, material placement, deliverable shapes |
| **`references/models.md`** | After choosing the model, read only the section for each model you use (one for the main session and one for the subagents when they differ); read the matching subsection in the second half of the file when you need effort detail, prices or usage rules. Contents: known behaviors of the four current models with their prescriptions, with what the current page states kept apart from what only a predecessor's page states, plus effort values, prices, usage limits and ways to save usage |
| **`references/snippets.md`** | When you need a ready-made block. 52 numbered entries, 51 of them **verbatim** English text from the official docs, each marked with the model page it comes from |
| **`references/agents.md`** | When the prompt will drive a long-running autonomous agent. Context engineering, multi-window workflows, cache hits, letting the model orchestrate itself |

### When you use an English snippet

- The verbatim snippets (all except 4.1) are the official pages' own English wording; place each where its source page says: most go in the system prompt, and a few are marked for the end or for right after a particular passage. When the output is a prompt to paste into Claude Code or the Claude app and there is no system prompt to edit, put the snippet in the prompt body, and put one that is needed every time in CLAUDE.md or the project instructions
- The entries the official material says may be adjusted (for example 4.31, 4.37, 4.40, 4.46), and the one it says to use exactly as written (4.51), are marked one by one in `references/snippets.md`

---

## 6. Three request types, handled differently

**Writing or revising a prompt (Scenario A/B):** run the Section 0 workflow, and write the delivery note to the Section 0 step 6 spec. Do not add explanations, disclaimers, or alternative-approach lists the user did not ask for.

**Auditing an existing prompt:** compare against Section 3 point by point and report **the specific problem and the specific fix**. Do not replace their prompt with a new one unless they asked for a rewrite. Order findings by severity: things that cause API errors (3.1, 3.2, 3.8, 3.12) first, things the official material says backfire or get refused (3.9, 3.10, 3.13) next, everything else last.

**First ask which model the prompt was written for**, then check the matching official migration list:

- **Written for Opus 5 → Opus 5.5**: a carried-over `effort` value (the default dropped from `high` to `medium`); `thinking: {"type": "disabled"}` (now 400); forced `tool_choice` (now 400); a client that renders only `text` blocks, which will look silent between tool calls; code that assembles its own `messages` must keep history append-only (thinking blocks are tied to the conversation); ritual instructions written for older models (the measurement cited in 3.4)
- **Written for Fable 5.1 or Opus 5.5 → Fable 5.2**: the 3.13 list; a fresh effort sweep (coming from Opus 5.5, a request that omits `effort` runs one level higher on the API); code that assembles its own `messages` must pass the prefix check
- **Written for Sonnet 5 → Sonnet 5.5**: turning thinking off changes to `between_tools`; forced `tool_choice` is now 400; the effort levels have been recalibrated; history must likewise stay append-only. Anthropic's Sonnet 5.5 article: if the prompt carries workarounds such as `refusal steering, tool-call retry shims, or "do not be lazy,"`, remove them and re-run your evals before tuning anything else
- **Written for Haiku 4.5 → Haiku 5.5**: replace `budget_tokens` with effort; prefill is now 400; non-default sampling parameters are now 400; editing earlier turns invalidates the thinking blocks; a `max_tokens` sized for Haiku 4.5's usage without thinking may cut replies off

Each item above lists only the points most relevant to writing and auditing prompts; the full lists are in each model's what's-new page and migration guide.

**Inline check mid-conversation (Scenario C):** go light. **Do not make a production of it** — do not run the full workflow, do not read the reference files. Scan the Section 3 hard rules; if something is wrong say "this line is a problem: X, change it to Y"; if nothing is wrong say so and return to what you were discussing. The user wants to get back to what they were doing, not to receive an audit report.

---

## 7. Pre-delivery checklist

- [ ] You chose the model and effort yourself per Section 2 (you did not hand the question to the user, except in the case of user rule 1), and the delivery note says which tier you chose and which official statement it rests on
- [ ] No prefill, no `budget_tokens`, no non-default sampling parameters
- [ ] No `CRITICAL: You MUST` / `If in doubt, use [tool]` stacked to prevent undertriggering (**wording inside official snippets does not count**, for example the `you MUST read the file` inside 4.7)
- [ ] Targeting Fable 5.2: no lines telling it to think less; approval rules name the specific actions; no hard style limits; 4.37 added for agentic work at `low` or `medium`; the 3.13 list has been checked
- [ ] Targeting Opus 5.5: no `thinking: {"type": "disabled"}`, no forced `tool_choice`, no "do not think" rule; effort named as a tier
- [ ] Targeting Sonnet 5.5: turning off up-front thinking uses `between_tools`; no forced `tool_choice`; 4.46 added for coding at `low` where it has been seen reporting completion without running checks
- [ ] Targeting Haiku 5.5: if it has a search tool, it is given today's date (4.47); 4.46 added for coding at `low` or `medium` where it has been seen reporting completion without running checks
- [ ] Targeting Fable 5.2, Opus 5.5 or Sonnet 5.5: nothing asks it to write out its reasoning — no "show your reasoning", no `<thinking>` or scratchpad section, no `reasoning` field to fill (3.9). A short explanation or a summary of the actions taken is fine
- [ ] Main session on Fable 5.2 and the task will dispatch subagents: you have checked which model subagents run on by default, and stated the model to use per Section 2
- [ ] Main session on Fable 5.2 or Opus 5.5 and you foresee many parallel subagents doing bulk work to the same criteria: per Section 2, "Bulk processing", the prompt states that the batch goes to non-fork subagents, with the model and effort from the table in Section 2, "Bulk processing"
- [ ] Every constraint **you wrote** is stated positively and carries its reason (**wording inside official snippets does not count**, for example `NEVER use generic AI-generated aesthetics…` at the start of 4.25)
- [ ] Every instruction has been read against: **how would a well-intentioned human misinterpret this?** (public remarks by Cat Wu of the Claude Code team, 2026-07, not official documentation; the Fable 5.2 page states that the model applies rules as written, so this check still applies to current models)
- [ ] No instruction repeated across the system prompt and a tool description (an explanation of when to use the tool does not count; see 3.7)
- [ ] Long-context case: material first, query last
- [ ] Output length is explicitly specified in the prompt rather than left to effort
- [ ] There is a decidable success criterion; Scenario A has a check the model can run itself
- [ ] The official test (best practices page): **hand this prompt to a colleague with minimal context on the task — could they follow it?**

---

## 8. Symptom to prescription

Snippet numbers refer to `references/snippets.md`. The model in parentheses is the model whose page the prescription comes from. Entries marked "measured on" come from a predecessor's page; check them on your own evals first. 4.16 and 4.25 come from the Sonnet 5 page, and 4.11 and the first blocks of 4.6 and 4.15 were written on the best practices page for earlier models; each entry says so.

| Symptom | Prescription |
|---|---|
| Generic response | Add specific constraints, add the success criterion, add a modifier like "go beyond the basics" |
| Answers the wrong question | Supply the real goal and background — say why |
| Unstable format | Add 3–5 `<example>` blocks; or use Structured Outputs |
| Preamble | `Respond directly without preamble.` |
| Fabricates content (general) | Explicitly permit not knowing: `If the data is insufficient to draw conclusions, say so rather than speculating.` (from Anthropic's 2025-11 prompt engineering article, not on the documentation pages) |
| Makes claims about code or files it never opened | Snippet 4.7 |
| Suggests instead of acting | Use a direct verb (`Change this function` beats `Can you suggest some changes`); or add snippet 4.8 |
| Response too long | State the length in the prompt; snippets 4.2, 4.4 (Opus 5 originals). Do not rely on lowering effort |
| Ends the turn before the work is done, or stops after listing what remains | Fable 5.2: 4.37 (in Anthropic's testing it removed most endings that ask permission or offer to continue, but not endings that list what remains and then stop). Opus 5.5: for unattended loops, 4.31 plus a checklist the model updates itself; in Claude Code with a person present, the CLAUDE.md rule in "Say it all at once" in `references/for-claude-code.md`. Sonnet 5.5: try a higher effort level first, or 4.41. Haiku 5.5: 4.49 |
| Asks for approvals it does not need, holds back routine work (Fable 5.2) | Clean up the cautious instructions that have built up; name the steps that need no approval; write the exception into the rule itself (`references/models.md`) |
| Only adds to the existing structure, repeats existing code, reports nearby problems without fixing them (Fable 5.2) | 4.38, and review the diff, not only the summary |
| Long agentic turn looks silent in the client (Fable 5.2, Opus 5.5, Sonnet 5.5) | Check `thinking.display` first; then the block for that model in 4.3; on Fable 5.2, if you cannot change the harness, 4.39 |
| Final report hard to follow, too dense | Fable 5.2: 4.40, after first removing hard style rules. Measured on Fable 5: 4.21. Measured on Fable 5.1: 4.26 |
| Too much thinking, high latency | **Lower the effort level.** The Opus 5.5 page also leaves one prompt line open: `Answer directly without deliberating.`; if you add it, measure quality. The second block of snippet 4.15 comes from the best practices page, which states that it does not apply to Fable 5.2 |
| Shallow reasoning on hard problems | **Raise effort**; when only one turn needs it, use `ultrathink` in Scenario A |
| Chat replies are slow to start (Opus 5.5) | Consider removing "think carefully before answering" lines from the system prompt; 4.34 for follow-up turns (test it first before using it on Fable 5.2) |
| Starts more subagents than the task needs | Fable 5.2: first remove delegation instructions, then change the description of the tool that starts subagents (`references/models.md`). Opus 5 original: 4.10 |
| Subagents take most of the usage | Section 2, "Subagents": give them a cheaper model; have them report back short summaries; check the share with `/usage` |
| Starts review rounds on its own after finishing, at `xhigh`/`max` (Sonnet 5.5) | 4.42; run routine work at `high` or below |
| You only wanted ideas, but it starts doing the work (Sonnet 5.5) | 4.43 |
| Gets multi-step calculation tasks wrong when the answer must be JSON (Sonnet 5.5) | 4.44, or `xhigh` |
| Answers from training knowledge, does not search when it should | Sonnet 5.5: 4.45. Haiku 5.5: 4.47. Measured on Fable 5.1: 4.27 |
| Reports code changes complete without running checks (Sonnet 5.5, Haiku 5.5) | 4.46 |
| Skips tool calls when asked for JSON with thinking off (Haiku 5.5) | Use adaptive thinking for these requests; 4.48 when thinking has to stay off |
| Chatbot is worn down by users into abandoning the system prompt (Haiku 5.5) | 4.50 |
| Messages the user sends partway through are ignored as injections (Sonnet 5.5, Haiku 5.5) | Do not put user text in `tool_result` blocks; deliver it as a user turn (`references/models.md`) |
| Agent misses information the task did not point to, across several connected apps (Opus 5.5) | 4.32 |
| Agent takes longer than it should (Opus 5.5) | 4.33 — a time budget, with elapsed time reported at the end of each message |
| Follows instructions that arrived inside text the user pasted (Opus 5.5) | 4.35 |
| A rule keeps being ignored despite repeated reminders | The file is too long and the rule is buried → cut it down; a rule that cannot have exceptions → use a deterministic mechanism (hook, deny rule, schema). Adding emphasis does not help (3.3) |
| Code review under-reports | 4.16; move filtering downstream |
| Over-engineering | 4.6 |
| Does things nobody asked for, expands the task scope on its own | Measured on Fable 5: 4.24. Opus 5 original: 4.5. Sonnet 5.5: the second paragraph of 4.41 |
| Long task fabricates progress or overstates completion | Measured on Fable 5: 4.17 |
| Wraps up early as context fills | Measured on Fable 5: 4.19; better still, do not show the model a remaining-budget number |
| One tool call per turn, whole-file rewrites for small edits, summaries that reproduce source text without marking quotations, long deliverables at `xhigh`/`max` that take very long | Measured on Fable 5.1: in that order, the 4.9 nudge, 4.28, 4.30, 4.29. The Fable 5.2 page does not restate these tendencies |
| Benign request declined, returns `stop_reason: "refusal"` | Look at `stop_details.category`; for `reasoning_extraction`, change the prompt per 3.9; for the others, see each model's classifier row in `references/models.md` |
| Change may be irreversible / touches shared environments | 4.11 |
| Bends the code to make tests pass | 4.12 |
| Report is all short bullets, unreadable as prose | 4.13 |
| Needs to read many files at once / calls can run concurrently | 4.9 |
| Math renders as LaTeX in an environment that cannot display it | 4.14 |
| The request never states what it is for | Measured on Fable 5: 4.20 |
| Agent needs to accumulate lessons across sessions | Measured on Fable 5: 4.22 |
| Long async agent must deliver content to the user verbatim | Measured on Fable 5: 4.23 |
| Front-end output has the generic AI look | 4.25; on Opus 5.5 name the specific patterns to avoid — 4.36 |
| The application might be asked about children's medicine doses | 4.51; take the block for the model you call, and use it word for word |
| Temporary files left behind after a coding task | 4.52 |

### Length control

Put the length in the prompt, not in effort. The official basis: the Fable 5.2 page, `Effort barely changes how long the model's replies are. To change the length and tone of a reply, use your prompt.`; and the Opus 5 page states that lowering effort does not reliably shorten visible output. The cost page's advice is `Ask for the answer you will read, not the one that looks thorough`.

Ready-made wording: snippet 4.2 (the general conciseness instruction, and the short `<tone_preference>` reminder at the end) and snippet 4.4 (deliverables written to files), both original text from the Opus 5 page. When you give Fable 5.2 specific style limits, keep 3.13 in mind: hard sentence-length limits can make its prose choppy.

---

## 9. Meta-principles (for anything this file does not cover)

1. **Minimal information set**: the fewest tokens that fully outline the expected behavior.
2. **The right altitude**: neither brittle hardcoded logic nor guidance so vague it carries no signal.
3. **More context, fewer instructions.**
4. **Describe what you want, rather than forbidding what you do not.**
5. **When there are tools, look at the tool design too**: for concrete practice on naming, parameters, return values and error messages, see `references/agents.md`.
6. **Anything that can be enforced deterministically should not be a prompt**: use a hook, a schema, or a type constraint instead of writing "please make sure to."
7. **As the model gets stronger, scaffolding becomes dead weight**: periodically go back and delete what you added for a weaker model.
8. **Test after changing**: run it on a real task rather than judging by feel.
