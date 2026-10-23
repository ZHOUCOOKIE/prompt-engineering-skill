# prompt-engineering

A Claude [Agent Skill](https://code.claude.com/docs/en/skills) that writes, rewrites, reviews, and audits prompts using Anthropic's **current** official guidance — and picks the model and effort level for you before it drafts.

Available in **English** and **中文**. Also shipped as a single self-contained Markdown file for use without installing anything.

---

## Why this exists

A set of prompt techniques that are still widely repeated — and still sitting in most people's muscle memory — are now **obsolete, rejected by the API, or reversed in effect** on current Claude models:

| Technique | Status on Claude Opus 5.5 / Fable 5.1 |
|---|---|
| Prefilling the assistant turn | **Returns 400** |
| `thinking: {type: "enabled", budget_tokens: N}` | **Returns 400** |
| `thinking: {type: "disabled"}` | **Returns 400** — thinking cannot be turned off on either model. New with Opus 5.5 |
| Forced `tool_choice` (`any` / `tool`) | **Returns 400** on both. New with Fable 5.1 (2026-09-01) and Opus 5.5 |
| `temperature` for stylistic variety | **Returns 400** on both models in scope |
| `CRITICAL: You MUST …` to prevent tool undertriggering | No longer needed; consumes attention budget |
| "Double-check your answer before responding" | **Causes over-verification on Opus 5**, where the official guidance is to *remove* these instructions rather than soften them. On Opus 5.5 that finding is not restated — do not add them reflexively, but measure before deleting inherited ones |
| Dense few-shot examples in an agent system prompt | Can *reduce* quality where you want judgment — for Claude Code's own system prompt the team found "removing examples was extremely helpful, because it was just more creative than the examples we gave it". For format and tone, the docs still call examples one of the most reliable levers |
| Carrying an `effort` value across models | The default dropped from `high` to `medium` on Opus 5.5, and in Anthropic's testing, run at `medium`, it matches or exceeds Opus 5 at `high` on coding and knowledge-work evaluations — by that measure a carried value is one tier higher than it needs to be |

The Claude Code team removed **over 80%** of their own system prompt for models like Opus 5 and Fable 5, with no measurable loss on their coding evaluations. That is the direction this skill encodes: **more context, fewer instructions.**

## What it does

1. **Classifies the scenario.** A prompt destined for Claude Code is written very differently from one that runs in a desktop chat or Cowork session. The skill routes to the right playbook.
2. **Picks the model and effort itself** — it does not hand that question back to you — and states the choice with its reasoning.
3. **Drafts** against a checklist of obsolete techniques, using an indexed library of 35 verbatim official prompt snippets.
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

**Claude Opus 5.5 and Claude Fable 5.1** (including Mythos 5.1), plus techniques verified as current and non-conflicting. Claude Opus 5 and Claude Fable 5 are both on Anthropic's "Legacy models (still available)" list; guidance specific to them, and to earlier models, has been deliberately removed rather than left in as "still probably fine," apart from a few labelled best practices passages written for earlier models: the first blocks of snippets 4.6 and 4.15, snippet 4.11, the tool-wording example in §3.3, and the compaction-awareness block in `agents.md`. Claude Sonnet 5.5 and Claude Haiku 5.5 are current but outside the scope: the skill points to them when the official selection matrix would, and says where its hard rules do not transfer.

One documented exception, applied identically to both legacy pages: snippets published only on a predecessor's page are kept on the successor page's own carry-over sentence and labeled at the point of use. From the **Claude Opus 5** page: 4.2–4.5 and 4.10. From the **Claude Fable 5** page: six snippets and one variant (4.17, 4.19–4.23, and the Fable 5 variant in 4.6); 4.24's two halves now ship inside the Fable 5.1 block in 4.18. Behavioral claims that the successor pages neither restate nor retract sit in their own tables in `models.md`, each marked as measured on the earlier model, rather than being carried forward silently.

Two snippets (4.16 and 4.25) are verbatim from the **Claude Sonnet 5** page. That is deliberate and flagged at the point of use; Sonnet 5 has since joined the legacy list (Claude Sonnet 5.5 shipped on 2026-09-28), but its page is still live and both blocks are model-agnostic. 4.16 is prompt text for a fix the Opus 5 page prescribes in prose without publishing wording for; 4.25 is the short `frontend_aesthetics` block, which the best practices page carries only in a different and much longer form. [SOURCES.md](SOURCES.md#a-note-on-scope) explains the call; drop those two entries, the copy of 4.25's visual-directions block in §3.8, the two Sonnet 5 sentences quoted under "Say it all at once" in `for-claude-code.md`, and the pointers to them, if you would rather not mix pages.

Snapshot date: **2026-10-23**. Anthropic's documentation moves — during the life of this repo one page reversed its recommendation mid-verification, another was withdrawn outright, two model releases moved the model pages to new paths, and in the month after the second, thirteen quoted sentences were reworded, mostly without an announcement — so for specific API parameters, confirm against the live docs. See [SOURCES.md](SOURCES.md).

On 2026-10-23 every tracked page was re-checked a month after the last round. No model entered scope — Sonnet 5.5 and Haiku 5.5 shipped, but the default recommendation still starts with Opus 5.5 — yet the pages had moved under the repo, mostly without an announcement: thirteen quoted sentences had been reworded, one quoted block (4.21) had lost a parenthetical, the `medium` effort default was no longer unique to Opus 5.5, and `reasoning_extraction` was no longer described as new on Opus 5.5. One piece of advice was withdrawn rather than reworded: demonstrating a reasoning pattern with `<thinking>` tags in few-shot examples, which the docs no longer suggest and which can now draw a `reasoning_extraction` refusal. The round also found errors of this repo's own: the study guides still carried pre-Opus-5.5 claims the previous round had not swept and misquoted nine official sentences, the sentence that round had relabeled as the repo's own is in fact from Anthropic's 2025-11 prompt engineering post, one "official guidance" claim had no source at all, an Opus 5 exception — effort does not reliably shorten visible output — had been stated as a rule for every model in scope, and a number of official hedges ("might", "can", "tend to", "no measurable loss on our coding evaluations") had been dropped. The write-up is [SOURCES.md](SOURCES.md#the-2026-10-23-re-check).

On 2026-09-22, the day Claude Opus 5.5 shipped and Claude Opus 5 joined the legacy list, every Opus-specific claim was adjudicated against the Opus 5.5 pages instead of being renamed — the same procedure the Fable round used three weeks earlier. The largest result is that **§3.4, "remove self-verification instructions," does not carry to Opus 5.5 as a hard rule**: the Opus 5.5 pages neither restate the over-verification finding nor retract it, and the best practices page still names Claude Opus 5 as the only exception to its general "ask Claude to self-check" advice. The effort section was rewritten around the new `medium` default, three hard rules were extended to cover Opus 5.5's breaking changes, and six snippets (4.31–4.36) were added from the Opus 5.5 page. The round also found one blog URL moved to a new domain and one fenced block sitting where a reader would take it for documentation wording with nothing marking it as different. The write-up is [SOURCES.md](SOURCES.md#the-2026-09-22-re-check).

On 2026-09-03, two days after Claude Fable 5.1 shipped, every Fable-specific claim was adjudicated the same way: kept where the Fable 5.1 page carries it over, rewritten where it says something different, retired where it supersedes the old wording (snippet 4.18), and labeled "measured on Fable 5" where the Fable 5.1 pages are silent. That round found three pages moved behind redirects, one migration-guide URL turned into an index, and two attribution errors of this repo's own. The write-up is [SOURCES.md](SOURCES.md#the-2026-09-03-re-check); the 2026-08-08 round is [below it](SOURCES.md#the-2026-08-08-re-check).

## How this was verified

Every factual claim traces to an official Anthropic source. The following pages were read **in full, verbatim** — not summarized:

- Prompting best practices, and the Opus 5.5, Opus 5, Fable 5.1 and Fable 5 prompting pages
- What's new in Claude Opus 5.5, and What's new in Claude Fable 5.1
- Claude Code best practices
- The effort parameter reference
- Skill authoring best practices
- The prompt engineering overview

Read in part — specific sections, quoted or stated as facts: the Sonnet 5 prompting page, the Opus 5.5 and Fable 5.1 migration guides, the API version 2026-10-21 migration guide, the models overview, the Opus 5.5, Fable 5.1 and Mythos 5.1 overview pages, the model-selection guide, the thinking page, the thinking-steering page, the refusals-and-fallback page, the mid-conversation system messages page, the Claude Code hooks, dynamic-workflows and changelog pages, and the Cowork overview, the Claude Desktop documentation and the academy's desktop-app tutorial. The Sonnet 5.5 and Haiku 5.5 prompting pages were read in full for the scope decision; nothing is quoted from them.

The English snippets were extracted programmatically from those sources and diffed character-for-character; the two editions carry an identical set of 53 fenced blocks. Every round since has re-checked each snippet by exact-string presence against the live pages, with deliberately fabricated control strings in the same run to catch a checker that answers yes to everything. Earlier rounds caught two snippets attributed to the wrong page and one instruction that was not official wording at all. The 2026-09-22 round flagged a fenced block in the chat-and-Cowork reference that sat where a reader would take it for documentation wording, and labeled it the repo's own sentence; the 2026-10-23 round found that label was itself wrong — the sentence is the second sentence of the example in Anthropic's 2025-11 prompt engineering post — and also found an "official guidance" claim with no source and a passage borrowed from the Sonnet 5 page without credit. All are fixed, and provenance is stated per entry. In the 2026-10-23 run, all 51 blocks of 40 characters or more were found verbatim on their attributed pages in both editions — after snippet 4.21 was updated to its page's new wording — and both controls were absent.

The skill then went through **four adversarial review rounds** with independent agents, using both a coding task and a non-coding task; twenty-eight defects were found and fixed. The English edition took a further round of its own, and each document in `docs/` took three more, run until a round stopped producing new findings. Three examples of what those rounds caught:

- The headline rule "give it a check it can run" **presupposed the check exists** — which is exactly wrong for the most common request shape (legacy code, low coverage, nobody willing to touch it). The skill now has a full characterization-test path: pin current behavior *including suspected bugs*, freeze the net, then refactor.
- A checklist item requiring every constraint to be phrased positively collided with three official snippets that are stacked negatives by design — 4.25 opens `NEVER use generic AI-generated aesthetics…`. Carved out: tuned official wording is copied as is, and the rule applies only to constraints you wrote yourself.
- A checklist item banning `You MUST` collided with official snippet 4.7, which contains `you MUST read the file`. Carved out.

One more round happened after that, and it is the round worth reading about. Two reviewers had reported that the Opus 5 effort recommendation was "start at `high`"; three direct fetches of the source page said `xhigh` instead, so the reports were filed as unreproducible. On re-verification a day later, **the reviewers were right and the page had changed** — it now reads "Start with the default (`high`) … and step up to `xhigh` for demanding coding and agentic work," and the effort reference has gained per-model sections that did not exist before. The guidance here was corrected to match, and the whole episode is written up in [SOURCES.md](SOURCES.md#4-the-opus-5-effort-starting-point--not-a-conflict-a-documentation-change).

The moral is the operating assumption of this repo: **a snapshot of fast-moving documentation is wrong by default, not by exception.** If you find a divergence, open an issue — the live docs are the authority, not this repo.

## A note on the snippets

`references/snippets.md` has 36 numbered entries, of which **35 are verbatim excerpts from Anthropic's public documentation** (4.1 is a decision table, not a quotation). Counted as text: 51 quoted blocks of 40 characters or more, all re-verified by exact-string match against the live pages in this round, drawn from six documentation pages — the prompting best practices page and the Opus 5.5, Opus 5, Fable 5.1, Fable 5 and Sonnet 5 prompting pages. They are included verbatim because paraphrasing them discards the behavior they were tuned to produce; Anthropic publishes them precisely so that developers paste them into prompts.

Beyond the snippet library, the rest of the skill carries 17 further fenced blocks, roughly 870 words by a whitespace count, of which roughly 620 words are official wording (one of those blocks, the line permitting uncertainty in research prompts, comes from Anthropic's 2025-11 prompt engineering post rather than the documentation) — the remainder are this repo's own skeletons, and the two are easy to tell apart because the Chinese edition translates the skeletons and leaves every quoted block byte-identical. Outside the fenced blocks the skill also quotes shorter official sentences, page headings and example prompts inline — roughly 1,250 words in the English edition, each counted once. Each study guide in [`docs/`](docs/) carries blockquoted official passages plus shorter inline strings, roughly 1,330 words of attributed sentences across the two guides, each sentence counted once — attributed where they appear. In all, that is on the order of 7,000 words of quoted official text.

Attribution is per snippet: the index table in `snippets.md` links every quoted entry to the page it came from, and [SOURCES.md](SOURCES.md) records what was taken from each source. All rights to that text remain with Anthropic.

For context on why this seemed reasonable: Anthropic publishes [`anthropics/skills`](https://github.com/anthropics/skills) under Apache 2.0 and [`anthropics/claude-cookbooks`](https://github.com/anthropics/claude-cookbooks) under MIT, both containing prompt material intended for reuse. This repository is not affiliated with or endorsed by Anthropic. **If you are Anthropic and would prefer links to inline text, open an issue and it will be changed the same day.**

## Contributing

The most useful contributions are **corrections with a source link**. Anthropic's guidance changes faster than most documentation, and a claim that was right in July may not be right later. If you find a divergence between this skill and the live docs, the live docs win.

## License

The skill's own prose and structure: [MIT](LICENSE).
The quoted snippets and any directly quoted documentation text remain the property of Anthropic and are included under fair use for the purpose of documenting technique, with attribution.
