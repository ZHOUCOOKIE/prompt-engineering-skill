# prompt-engineering

A Claude [Agent Skill](https://code.claude.com/docs/en/skills) that writes, rewrites, reviews, and audits prompts using Anthropic's **current** official guidance — and picks the model and effort level for you before it drafts.

Available in **English** and **中文**. Also shipped as a single self-contained Markdown file for use without installing anything.

---

## Why this exists

A set of prompt techniques that are still widely repeated — and still sitting in most people's muscle memory — are now **obsolete, rejected by the API, or reversed in effect** on current Claude models:

| Technique | Status on the current models (Fable 5.2, Opus 5.5, Sonnet 5.5, Haiku 5.5) |
|---|---|
| Prefilling the assistant turn | **Returns 400** on all four |
| `thinking: {type: "enabled", budget_tokens: N}` | **Returns 400** on all four |
| `thinking: {type: "disabled"}` | **Returns 400** on Fable 5.2, Opus 5.5 and Sonnet 5.5. Sonnet 5.5 turns off up-front thinking with `between_tools` instead; Haiku 5.5 accepts `disabled` at `high` effort or below |
| Forced `tool_choice` (`any` / `tool`) | **Returns 400** on Fable 5.2, Opus 5.5 and Sonnet 5.5. Haiku 5.5 accepts it |
| `temperature` for stylistic variety | Non-default values **return 400** on all four |
| "Show your reasoning" in the response | Can be refused as `reasoning_extraction` on Fable 5.2, Opus 5.5 and Sonnet 5.5 |
| "Answer directly, don't deliberate" to cut thinking | In Anthropic's testing it did not stop Fable 5.2 from thinking at `medium` effort and above, or Haiku 5.5 from thinking; on Sonnet 5.5 it does not reliably reduce thinking. Lower the effort level instead |
| `IMPORTANT` or ALL CAPS on a line Claude keeps missing | The Claude Code docs now list this as advice to drop: "piling on emphasis mostly makes Claude over-apply the rule". Shorten the file, or make the rule a hook |
| "Double-check your answer before responding" | Depends on the model. The general docs still recommend a self-check; Claude Opus 5 is the named exception; the Claude Code docs say such reminders are usually unnecessary on Fable models; the Sonnet 5.5 and Haiku 5.5 pages publish a verification paragraph to add at lower effort |
| Carrying an `effort` value across models | Level names do not mean the same amount of thinking on different models, and the defaults differ: on the Claude API, `high` on Fable 5.2 and Sonnet 5.5, `medium` on Opus 5.5 and Haiku 5.5; in Claude Code, `medium` on all four |
| Delegating to a subagent "for a better answer" | The Claude Code docs: a subagent gives you parallelism or a fresh pair of eyes, "but not a smarter answer", and each one costs its own tokens |

The Claude Code team removed **over 80%** of their own system prompt for models like Opus 5 and Fable 5, with no measurable loss on their coding evaluations. That is the direction this skill encodes: **more context, fewer instructions.**

## What it does

1. **Classifies the scenario.** A prompt destined for Claude Code is written very differently from one that runs in a desktop chat or Cowork session. The skill routes to the right playbook.
2. **Picks the model and effort** by task, from Anthropic's own selection table and per-model effort guidance, for the main session and for subagents, and states the choice with the official basis for it.
3. **Drafts** against a checklist of obsolete techniques, using an indexed library of 51 verbatim official prompt snippets, each labeled with the model page it was published on.
4. **Self-checks** before delivering.

It also handles two other request shapes: auditing a prompt you already have, and a lightweight inline "does this line look right?" check mid-conversation that deliberately does *not* turn into an audit report.

## Install

Download a `.skill` file from [`dist/`](dist/) and open it in Claude, or copy the source directory into your skills folder:

```bash
# Claude Code — project scope
mkdir -p .claude/skills/prompt-engineering
cp -r skill/en/* .claude/skills/prompt-engineering/

# Claude Code — user scope (all projects)
mkdir -p ~/.claude/skills/prompt-engineering
cp -r skill/en/* ~/.claude/skills/prompt-engineering/
```

Swap `skill/en` for `skill/zh` for the Chinese edition.

> **Install one edition, not both.** They share the skill name `prompt-engineering` so that `/prompt-engineering` works either way; installing both creates a name collision.

### Without installing

[`single-file/`](single-file/) contains the whole thing merged into one Markdown file per language. Attach it to a conversation, paste it into another model, or read it offline. It is merged from the skill sources rather than maintained by hand, so the two say the same thing.

> **If you fork and edit:** `single-file/` and `dist/` are derived from `skill/`. The merge and packaging step is not committed here, so after editing `skill/` you have to regenerate both yourself — merge the six source files per language into `single-file/` (SKILL.md sections 0–3 first, then the snippet library as section 4, SKILL.md section 4 as section 5, the four other reference files as sections 6–9, and SKILL.md sections 6–9 as sections 10–13, dropping SKILL.md's section 5 file map and rewriting every cross-file reference to a section number), and re-zip `skill/<lang>/` as `prompt-engineering/…` for `dist/`. Edit the sources, not the derived copies.

## Repository layout

```
skill/en/            English edition — SKILL.md + 5 reference files
skill/zh/            Chinese edition — same structure
dist/                Packaged .skill files
single-file/         Merged single-file editions (generated from skill/)
docs/                Background reading — why the rules are what they are (EN + ZH)
SOURCES.md           Every source, with what was taken from it
```

The skill uses [progressive disclosure](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices): `SKILL.md` stays under 500 lines and holds the workflow, the hard rules, and the routing table; the five reference files load only when the task needs them.

| Reference file | Loaded when |
|---|---|
| `for-claude-code.md` | The prompt is destined for Claude Code |
| `for-chat-and-cowork.md` | It runs in a desktop chat or Cowork session |
| `models.md` | After the model is chosen — behaviors and prescriptions |
| `snippets.md` | A ready-made official snippet is needed |
| `agents.md` | The prompt drives a long-running autonomous agent |

## Background reading

The skill is written to be executed, not studied — it states rules without arguing for them. [`docs/study-guide.md`](docs/study-guide.md) is the other half: **why** each rule is what it is, which older habit it displaces, and how to read the places where Anthropic's own guidance appears to contradict itself. A [Chinese edition](docs/study-guide.zh.md) covers the same ground.

It is worth reading if you want to disagree with something here on an informed basis, if you maintain prompts and need to know which parts will rot first, or if you learned prompt engineering before 2026 and want the diff.

## Scope

**The four current models: Claude Fable 5.2 (including Mythos 5.2), Claude Opus 5.5, Claude Sonnet 5.5 and Claude Haiku 5.5.** Claude Fable 5.1, Fable 5, Opus 5 and Sonnet 5 are on Anthropic's "Legacy models (still available)" list and appear as the source of specific wording, as the starting points of the migration lists, and where a current page names them (as fallback targets, or in a price or benchmark comparison).

Every model-specific statement is filed one of three ways. What the model's current page states is used directly. What only a predecessor's page states, and the current page neither restates nor retracts, sits in a table marked "measured on" that model in `models.md`, or carries that label on its snippet. What a current page retracts or contradicts is dropped. Snippets from the Opus 5 and Sonnet 5 pages rest on the Opus 5.5 and Sonnet 5.5 pages' statements that existing prompts should perform well without changes; the Fable 5.2 pages make no such statement, so the Fable 5.1 and Fable 5 snippets are all labeled as measured on those models.

**Sources.** Rules come only from Anthropic's own documentation, articles, support site and changelog. Three rules in SKILL.md §2 are the repository owner's own and are labeled "User rule, not official guidance"; one checklist item is a labeled public statement by an Anthropic employee. Nothing else in the skill is meant to be unsourced, and [SOURCES.md](SOURCES.md) records what was taken from where.

Snapshot date: **2026-11-09**. Anthropic's documentation moves, so for specific API parameters confirm against the live docs.

On 2026-11-09, the day Claude Fable 5.2 shipped and Fable 5.1 joined the legacy list, the skill was rebuilt rather than renamed. Every statement in it — 869 by the audit's count — was checked against that day's pages. Fable 5.1 left the scope because no page recommends it over Fable 5.2 on any dimension. Sonnet 5.5 and Haiku 5.5 entered it, so the hard rules are now stated per model. Model and effort selection was rewritten by task, following the table in Anthropic's selection article published the same day, with an official source for each cell and a separate table for subagents, and `models.md` gained a section of official figures on prices, usage limits and cost. `for-claude-code.md` was rewritten against the Claude Code best practices page, which had itself been rewritten, and `for-chat-and-cowork.md` against support.claude.com, where Chat and Cowork are now one conversation. Sixteen snippets were added (4.37–4.52). And everything that had no official source was removed at the owner's instruction: the repo's own reconciliations and heuristics, most of what it took from an employee talk, and eighteen unsupported factual remarks. The write-up is [SOURCES.md](SOURCES.md#the-2026-11-09-re-check); the earlier rounds (2026-10-23, 2026-09-22, 2026-09-03, 2026-08-08) are kept below it as history.

## How this was verified

Every rule traces to an official Anthropic source, apart from the labeled items listed under Scope. What the 2026-11-09 round did:

- **Fetched and saved every page that day**: 54 Claude Platform pages (two more legacy addresses now redirect to pages in that set), 33 Claude Code pages, 25 support articles, the Anthropic articles and the launch announcement. [SOURCES.md](SOURCES.md) lists every source the skill uses, with how deeply it was read and what was taken from it. Most were read in full. The ones that were only searched for specific terms are marked "Targeted" there: the Fable 5 prompting page, the Claude Code reference pages (environment variables, settings, tools, commands, hooks and a few others), the release notes and changelog before 2026-10-15, and a handful of saved pages from which nothing was taken.
- **Audited the old skill claim by claim.** Six passes split it into 869 individual statements and gave each a verdict against the saved pages. Statements with no official source were removed, or kept with a label where the owner decided to keep them.
- **Checked every quotation by machine.** `snippets.md` has 75 fenced blocks; each was checked by exact-string presence against the saved pages after normalizing line wraps, and the two editions carry an identical set. Every inline quotation of 25 characters or more in the skill was checked the same way.
- **Covered the new model's pages section by section.** Each section of the Fable 5.2 overview, what's-new page, migration guide and prompting page, and the Fable 5.2 section of the effort page, either lands in the skill or has a stated reason for being left out. The table is in [SOURCES.md](SOURCES.md#the-2026-11-09-re-check).
- **Reviewed the result independently.** Reviewers with no view of the editing session compared each file with the saved pages, the two editions with each other, and the derived files with their sources. This ran for three rounds (92, 100 and 75 findings). Each round's findings were fixed and the next round read the result; the fixes made after the third round were not reviewed again.

Earlier rounds worked the same way on a smaller scope; their write-ups are kept in SOURCES.md as history.

One episode from the first review is worth knowing about. Two reviewers reported that the Opus 5 effort recommendation was "start at `high`"; three direct fetches of the page said `xhigh`, so the reports were filed as unreproducible. A day later the page had changed and the reviewers were right. The write-up is in [SOURCES.md](SOURCES.md#9-the-opus-5-effort-starting-point--a-documentation-change-kept-as-history).

The operating assumption of this repo follows from it: **a snapshot of fast-moving documentation is wrong by default, not by exception.** If you find a divergence, open an issue. The live docs are the authority, not this repo.

## A note on the snippets

`references/snippets.md` has 52 numbered entries, of which **51 are verbatim excerpts from Anthropic's public documentation** (4.1 is a decision table, not a quotation). Counted as text: 75 fenced blocks, 72 of them 40 characters or more, roughly 5,450 words, drawn from nine documentation pages: the prompting best practices page and the Fable 5.2, Opus 5.5, Sonnet 5.5, Haiku 5.5, Fable 5.1, Fable 5, Opus 5 and Sonnet 5 prompting pages. Anthropic publishes these blocks for developers to paste into prompts; several pages introduce theirs as starting points to adapt.

Beyond the snippet library, the rest of the skill carries 19 further fenced blocks, roughly 925 words by a whitespace count, of which 16 blocks and roughly 755 words are official wording (three of those come from Anthropic articles, not the documentation: the line permitting uncertainty in research prompts, and a CLAUDE.md rule and an example prompt for Opus 5.5). The remaining three are this repo's own skeletons, and the two kinds are easy to tell apart because the Chinese edition translates the skeletons and leaves every quoted block byte-identical. Outside the fenced blocks the skill also quotes shorter official sentences, page headings and example prompts inline, roughly 2,900 words in the English edition, each counted once. The two study guides in [`docs/`](docs/) quote roughly 2,200 more words of attributed sentences, each sentence counted once across the two guides. In all, that is on the order of 11,000 words of quoted official text.

Attribution is per snippet: the index table in `snippets.md` links every quoted entry to the page it came from, and [SOURCES.md](SOURCES.md) records what was taken from each source. All rights to that text remain with Anthropic.

For context on why this seemed reasonable: Anthropic publishes [`anthropics/skills`](https://github.com/anthropics/skills) under Apache 2.0 and [`anthropics/claude-cookbooks`](https://github.com/anthropics/claude-cookbooks) under MIT, both containing prompt material intended for reuse. This repository is not affiliated with or endorsed by Anthropic. **If you are Anthropic and would prefer links to inline text, open an issue and it will be changed the same day.**

## Contributing

The most useful contributions are **corrections with a source link**. Anthropic's guidance changes faster than most documentation, and a claim that was right in July may not be right later. If you find a divergence between this skill and the live docs, the live docs win.

## License

The skill's own prose and structure: [MIT](LICENSE).
The quoted snippets and any directly quoted documentation text remain the property of Anthropic and are included under fair use for the purpose of documenting technique, with attribution.
