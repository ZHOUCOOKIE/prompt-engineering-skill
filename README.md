# prompt-engineering

A Claude [Agent Skill](https://code.claude.com/docs/en/skills) that writes, rewrites, reviews, and audits prompts using Anthropic's **current** official guidance — and picks the model and effort level for you before it drafts.

Available in **English** and **中文**. Also shipped as a single self-contained Markdown file for use without installing anything.

---

## Why this exists

A set of prompt techniques that are still widely repeated — and still sitting in most people's muscle memory — are now **obsolete, rejected by the API, or reversed in effect** on current Claude models:

| Technique | Status on Claude Opus 5 / Fable 5 |
|---|---|
| Prefilling the assistant turn | **Returns 400** |
| `thinking: {type: "enabled", budget_tokens: N}` | **Returns 400** |
| `CRITICAL: You MUST …` to prevent tool undertriggering | No longer needed; consumes attention budget |
| "Double-check your answer before responding" | **Causes over-verification on Opus 5.** Official guidance is to *remove* these instructions, not soften them |
| Dense few-shot examples in an agent system prompt | Often *reduces* quality — the Claude Code team found "removing examples was extremely helpful, because it was just more creative than the examples we gave it" |
| `temperature` for stylistic variety | Support is narrowing; several current models reject non-default values |

The Claude Code team cut their own system prompt by **more than 80%** with no loss of performance. That is the direction this skill encodes: **more context, fewer instructions.**

## What it does

1. **Classifies the scenario.** A prompt destined for Claude Code is written very differently from one that runs in a desktop chat or Cowork session. The skill routes to the right playbook.
2. **Picks the model and effort itself** — it does not hand that question back to you — and states the choice with its reasoning.
3. **Drafts** against a checklist of obsolete techniques, using 25 verbatim official prompt snippets.
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

[`single-file/`](single-file/) contains the whole thing merged into one Markdown file per language. Attach it to a conversation, paste it into another model, or read it offline. It is generated from the skill sources, so the two never drift.

## Repository layout

```
skill/en/            English edition — SKILL.md + 5 reference files
skill/zh/            Chinese edition — same structure
dist/                Packaged .skill files
single-file/         Merged single-file editions (generated from skill/)
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

## Scope

**Claude Opus 5 and Claude Fable 5** (including Mythos 5), plus techniques verified as current and non-conflicting. Guidance specific to earlier models has been deliberately removed rather than left in as "still probably fine."

Snapshot date: **2026-07-25**. Anthropic's documentation moves; for specific API parameters, confirm against the live docs. See [SOURCES.md](SOURCES.md).

## How this was verified

Every factual claim traces to an official Anthropic source. The following pages were read **in full, verbatim** — not summarized:

- Prompting best practices, and the Opus 5 and Fable 5 model pages
- Claude Code best practices
- The effort parameter reference
- Skill authoring best practices
- The prompt engineering overview

The 25 English snippets were extracted programmatically from those sources and diffed character-for-character; they are byte-identical between the English and Chinese editions.

The skill then went through **four adversarial review rounds** with independent agents, using both a coding task and a non-coding task. Twenty-eight defects were found and fixed. Three examples of what those rounds caught:

- The headline rule "give it a check it can run" **presupposed the check exists** — which is exactly wrong for the most common request shape (legacy code, low coverage, nobody willing to touch it). The skill now has a full characterization-test path: pin current behavior *including suspected bugs*, freeze the net, then refactor.
- The official Console prompt improver still advertises "Strategic prefills" in its output — a technique that now returns 400. The skill flags this.
- A checklist item banning `You MUST` collided with official snippet 4.7, which contains `you MUST read the file`. Carved out.

Two findings reported by reviewers were **refuted** on re-verification, both concerning the Opus 5 effort recommendation. Direct fetches of the source page across three separate attempts consistently show `for coding and agentic work, xhigh remains the recommended starting point`; the contradicting reports could not be reproduced. If you find otherwise in the live docs, please open an issue — the docs are the authority, not this repo.

## A note on the snippets

The 25 English snippets in `references/snippets.md` are **verbatim excerpts from Anthropic's public documentation** — roughly 2,100 words in total, drawn from four documentation pages. They are included verbatim because paraphrasing them discards the behavior they were tuned to produce; Anthropic publishes them precisely so that developers paste them into prompts.

Attribution is per snippet: the index table in `snippets.md` links every entry to the page it came from, and [SOURCES.md](SOURCES.md) records what was taken from each source. All rights to that text remain with Anthropic.

For context on why this seemed reasonable: Anthropic publishes [`anthropics/skills`](https://github.com/anthropics/skills) under Apache 2.0 and [`anthropics/claude-cookbooks`](https://github.com/anthropics/claude-cookbooks) under MIT, both containing prompt material intended for reuse. This repository is not affiliated with or endorsed by Anthropic. **If you are Anthropic and would prefer links to inline text, open an issue and it will be changed the same day.**

## Contributing

The most useful contributions are **corrections with a source link**. Anthropic's guidance changes faster than most documentation, and a claim that was right in July may not be right later. If you find a divergence between this skill and the live docs, the live docs win.

## License

The skill's own prose and structure: [MIT](LICENSE).
The quoted snippets and any directly quoted documentation text remain the property of Anthropic and are included under fair use for the purpose of documenting technique, with attribution.
