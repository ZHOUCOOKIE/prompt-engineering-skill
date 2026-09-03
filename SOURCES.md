# Sources

Snapshot date **2026-08-08**. Anthropic's documentation is revised frequently — during the life of this repo one page reversed its recommendation mid-verification and another was withdrawn outright — so where this repo and the live docs disagree, **the live docs are correct**.

What the 2026-08-08 re-check found, and how it was done, is in [The 2026-08-08 re-check](#the-2026-08-08-re-check) below.

Read-depth legend:

- **Full** — the page was fetched and read verbatim end to end; snippets were extracted programmatically from it
- **Targeted** — specific sections were extracted and quoted; the rest was not used

---

## Living documentation (tracks model releases)

| Source | Depth | What was taken |
|---|---|---|
| [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) | **Full** | Most of the snippet library; long-context placement; XML structuring; the few-shot guidance; the migration section |
| [Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) | **Full** | Verbosity, narration, written-deliverable length, scope, subagent damping, self-correction, thinking-disabled artifacts, effort guidance |
| [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) | **Full** | Long-run autonomy, progress truthfulness, `reasoning_extraction`, memory conventions, the send-to-user tool, scaffolding advice |
| [Prompting Claude Sonnet 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-sonnet-5) | Targeted | **Two snippets only** — 4.16 (code review coverage) and 4.25 (front-end aesthetics). Both are model-agnostic prompt text; see "A note on scope" below |
| [Model migration guide](https://platform.claude.com/docs/en/about-claude/models/migration-guide) | Targeted | The prefill 400 statements per model line; the official replacement list (structured outputs, system prompt instructions, `output_config.format`); the no-preamble string |
| [Models overview](https://platform.claude.com/docs/en/about-claude/models/overview) | Targeted | Which models are current; Fable 5 as "Anthropic's most capable widely released model"; the start-with-Opus-5 selection guidance |
| [Effort](https://platform.claude.com/docs/en/build-with-claude/effort) | **Full** | The five effort levels and their stated use cases, the per-model recommendation sections for Opus 5 and Fable 5, effort's effect on tool-call count, the `high` ≡ omitted equivalence, cache invalidation, the 400 on disabling thinking at `xhigh`/`max`, and the `max_tokens` guidance (start at 64k and tune) |
| [Prompt engineering overview](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview) | **Full** | The three preconditions before prompt tuning; "not every failing eval is best solved by prompt engineering" |
| [Best practices for Claude Code](https://code.claude.com/docs/en/best-practices) | **Full** | The verification-affordance rule, explore/plan/implement, specificity contrasts, CLAUDE.md guidance, the five failure patterns, the adversarial-review caveat |
| [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) | **Full** | The constraints this repo's own skill is built to (name/description limits, one-level references, TOC threshold, progressive disclosure) |
| [Choosing the right model](https://platform.claude.com/docs/en/about-claude/models/choosing-a-model) | Targeted | "Tuning effort is often a better lever than switching models"; the Opus 5 default-effort statement |
| [Steering thinking](https://platform.claude.com/docs/en/build-with-claude/thinking-steering-and-cost) | Targeted | Lower effort before reaching for prompt-based steering; steering is sensitive to exact wording |
| [Mid-conversation system messages](https://platform.claude.com/docs/en/build-with-claude/mid-conversation-system-messages) | Targeted | The `tools` → `system` → `messages` cache-prefix hash order; appending a `{"role": "system"}` message instead of editing the top-level field; not available on Sonnet 5 |
| [What's new in Claude Opus 5](https://platform.claude.com/docs/en/about-claude/models/whats-new-opus-5) | Targeted | Corroborates the remove-your-verification-instructions rule and the `max_tokens` guidance at `xhigh`/`max` |
| [Dynamic workflows (Claude Code)](https://code.claude.com/docs/en/workflows) | Targeted | The definition of `ultracode` — a Claude Code setting combining `xhigh` effort with automatic workflow orchestration, not an API effort tier |
| [Claude Code changelog](https://code.claude.com/docs/en/changelog) | Targeted | 2.1.221 (2026-08-04): the `prompt-audit` subcommand of the `claude-api` skill |
| [Claude Cookbook](https://platform.claude.com/cookbook) | Targeted | Metaprompt, prompt caching, context-engineering recipes |

## Engineering blog

| Source | Date | What was taken |
|---|---|---|
| [The new rules of context engineering for Claude 5 generation models](https://claude.com/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models) | 2026-07-24 | The six rules; the >80% system-prompt reduction; `/doctor`; skills as lightweight guides |
| [Steering Claude Code: CLAUDE.md, skills, hooks, subagents](https://claude.com/blog/steering-claude-code-skills-hooks-rules-subagents-and-more) | 2026-06-18 | The seven-mechanism comparison table; the anti-patterns; the 200-line CLAUDE.md guidance |
| [Building verification loops in Claude Code with skills](https://claude.com/blog/building-verification-loops-in-claude-code-with-skills) | 2026-07-22 | The four verification-loop modes; the editable-skill caveat |
| [Loop engineering: getting started with loops](https://claude.com/blog/getting-started-with-loops) | 2026-06-30 | The four loop types and their commands; cost levers |
| [Agent Harness Design: 3 patterns](https://claude.com/blog/harnessing-claudes-intelligence) | 2026-04-02 | Let-the-model-orchestrate (45.3% → 61.6% on BrowseComp); cache-hit design; when to promote a bash action to a dedicated tool |
| [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | 2025-09-29 | Context rot; the right-altitude principle; compaction, structured note-taking, sub-agent architectures |
| [Writing effective tools for agents](https://www.anthropic.com/engineering/writing-tools-for-agents) | 2025-09-11 | Tool naming and namespacing, token-efficient returns, `ResponseFormat`, actionable errors |
| [Equipping agents with Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) | 2025-10-16 | Three-tier progressive disclosure; why bundle executable code |
| [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) | 2025-11-26 | The initializer/coding-agent split; the feature-list JSON; why unit tests alone miss end-to-end breakage |
| [Prompt engineering best practices for 2026](https://claude.com/blog/best-practices-for-prompt-engineering) | 2025-11-10 | **Used with care** — see "Known conflicts" below |
| [Cowork overview](https://claude.com/docs/cowork/overview) · [Desktop app modes](https://claude.com/resources/tutorials/navigating-the-claude-desktop-app) | — | The chat / Cowork / Claude Code capability boundaries |

## Employee talks

| Source | What was taken |
|---|---|
| [Fireside chat: Cat Wu & Thariq Shihipar](https://www.youtube.com/watch?v=uU5Gv2h8-9g) (AI Engineer World's Fair, 2026-07) — [transcript](https://simonwillison.net/2026/Jul/21/cat-and-thariq/) | "removing examples was extremely helpful…" (Thariq); "fewer 'do not do this' instructions…" (Thariq); the well-intentioned-human misreading test (Cat); the grep/glob-vs-bash and file-edit-tool tradeoff |
| [Prompting 101 · Code with Claude](https://www.youtube.com/watch?v=ysPbXH0LpIE) (Applied AI team, recorded 2025-05) | Foundations only; tactics superseded by the living docs |
| [Code with Claude 2026 sessions](https://www.youtube.com/playlist?list=PLmWCw1CzcFim2obQ-w3ohbULOfwp5lApR) | Context and background |

---

## Known conflicts

Where official sources disagree with each other, the resolution used here and why.

### 1. "Prompt engineering best practices for 2026" (2025-11) vs. the living docs

That post recommends three things the current documentation contradicts:

| The post says | The living docs say | Resolution |
|---|---|---|
| Prefill the assistant response as an advanced technique | Prefill on the last assistant turn returns 400 from Claude 4.6 onward | **Docs.** Use Structured Outputs or an explicit instruction |
| XML tags are "less necessary now" | Structuring prompts with XML tags is a recommended technique, and the official snippets use it wherever a block needs to function as one named unit | **Docs** |
| Role prompting is "often unnecessary" | Giving Claude a role focuses behavior; even one sentence helps | **Docs**, with the post's useful corollary: stating the *perspective* usually beats attaching an identity label |

The post's framing sections and its "common mistakes" list remain useful.

### 2. Examples: "give 3–5" vs. "removing them helped"

Both are official and they are not in conflict — they address different situations. Locking a format, tone, or structure calls for examples; an agent system prompt that needs judgment does better without them. **Examples constrain**; use them when constraint is what you want.

### 3. Verification: Opus 5 says remove it, Fable 5 says add it

Also not a conflict. What Opus 5 tells you to remove is *the model reviewing its own work*. What Fable 5's long-run guidance adds is *a separate, fresh-context verifier subagent* — which the same page notes tends to outperform self-critique. Remove self-review; keep external checking.

### 4. The Opus 5 effort starting point — not a conflict, a documentation change

Worth recording in full, because it is the clearest demonstration of why this repo defers to the live docs.

During review, two independent passes reported that the Opus 5 recommendation was "start at `high`." Three direct fetches of the Opus 5 page returned instead that `xhigh` "remains the recommended starting point" for coding and agentic work, and the effort page at that time had no Opus 5 section at all. The reports were recorded here as unreproducible.

They were not wrong. On re-verification the page reads:

> **Efficiency at lower effort:** `low` and `medium` effort produce strong quality at a fraction of the tokens and latency of higher settings. Start with the default (`high`) and adjust based on your evals: use `low` and `medium` liberally as your primary control for token cost and response time wherever quality holds, and step up to `xhigh` for demanding coding and agentic work.

The phrase "remains the recommended starting point" no longer appears anywhere on the page, and the effort page has since gained per-model sections for both Opus 5 and Fable 5. The documentation changed underneath the snapshot.

**The guidance in this repo now follows the current page: start at the default `high` on both models, and step up to `xhigh` for demanding coding and agentic work.** The practical difference from the earlier wording is real — an ordinary coding task now starts at `high`, not `xhigh`.

The lesson generalizes past this one parameter: **a snapshot of fast-moving documentation is wrong by default, not by exception.** For anything you are using to plan cost or latency, confirm it against the live page.

## The 2026-08-08 re-check

Thirteen days after the 2026-07-26 snapshot, every tracked page was fetched again as canonical Markdown and diffed byte-for-byte against the copies taken on 2026-07-26. Diffing beats re-reading here: it cannot be talked into agreeing with itself.

| Page | Result |
|---|---|
| Prompting best practices, Prompting Claude Opus 5 / Sonnet 5 / Fable 5, Prompt engineering overview | **Byte-identical.** No change of any kind |
| [Effort](https://platform.claude.com/docs/en/build-with-claude/effort) | Changed, but not in substance: a Compatibility block was added at the top and three SDK code samples were reformatted. Every recommendation, default, threshold and caveat this repo cites is unchanged |
| [Best practices for Claude Code](https://code.claude.com/docs/en/best-practices) | Changed in 14 places, all presentational — plan-mode keyboard hints, fence attributes, an `mcp add` example, one link target. No normative claim moved |
| [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) | All hard constraints re-confirmed: `name` ≤ 64 chars, `description` ≤ 1,024 chars and third person, SKILL.md body under 500 lines, references one level deep, table of contents above 100 lines. Still exactly two frontmatter fields, both required |

**One page was withdrawn, and it took a claim of this repo's with it.** `.../prompt-engineering/prompting-tools` now returns 307 to Prompting best practices; `prompt-improver` and `prompt-generator` return 308 to the same place. None of the three appears in `sitemap.xml` or `llms.txt`. The live prompt-engineering section is six pages, and no page documenting the Console prompt improver exists.

That page was the sole basis for a rule this repo stated in five files: that the improver's own documentation advertises "Strategic prefills that guide Claude's initial responses." The wording was authentic — a third-party mirror dated 2026-04-26 preserves it — but the row in this table claimed a **Full** read of a page that had already been redirecting since at least 2026-07-11, which means the claim was stale on the day it was written, not merely stale now. The row is gone, and §3.11 no longer cites it: the rule is now stated as *audit any prompt you inherited for a trailing assistant turn*, which is what it was always actually about and does not depend on a page that no longer exists.

**Nothing already in the repo needed correcting beyond that.** The round did, however, *add*: the sweep surfaced four pages carrying prompt guidance this repo had never tracked — [Choosing the right model](https://platform.claude.com/docs/en/about-claude/models/choosing-a-model), [Steering thinking](https://platform.claude.com/docs/en/build-with-claude/thinking-steering-and-cost), [Mid-conversation system messages](https://platform.claude.com/docs/en/build-with-claude/mid-conversation-system-messages), and [What's new in Claude Opus 5](https://platform.claude.com/docs/en/about-claude/models/whats-new-opus-5). Two effort-ordering rules and the cache-prefix hash order came from the first three; they are in `references/models.md` and `references/agents.md`, and the pages are now rows in the table above.

Three findings from the sweep were checked and **rejected** before they reached any file. The instructive one: a reviewer reported that the `claude-api` skill's documentation page describes the `prompt-audit` subcommand. It does not — the subcommand is real, but only the changelog documents it, and the changelog is what §3.11 cites.

## A note on scope

The stated scope is Opus 5 and Fable 5. Two snippets nonetheless come from the **Claude Sonnet 5** page, and this is deliberate rather than an oversight:

- **4.16 (code review: coverage over filtering).** The Opus 5 page independently diagnoses the behavior and prescribes the fix in prose — `ask it to report everything and filter in a separate pass instead` — but publishes no wording for it. The Sonnet 5 page publishes wording for the same prescription. The block is model-agnostic prompt text.
- **4.25 (front-end aesthetics).** The short `frontend_aesthetics` block and the four-visual-directions prompt both come from the Sonnet 5 page. The prompting best practices page carries a **different and much longer** `frontend_aesthetics` block; they are not interchangeable, and this repo ships the short one.

Sonnet 5 is a current model, not a superseded one, so this does not reintroduce stale guidance. Both entries carry a provenance note at the point of use. If you would rather not mix pages, drop 4.16 and 4.25 — nothing else depends on them.

## Deliberately excluded

| Excluded | Why |
|---|---|
| [prompt-eng-interactive-tutorial](https://github.com/anthropics/prompt-eng-interactive-tutorial) | Still targets Claude 3 Haiku; the techniques it teaches most prominently have since been reversed. **In fairness:** the official overview page still lists it for interactive learners, and does not deprecate it — but the same page designates Prompting best practices as "the living reference; start there" |
| ["AI prompt engineering: a deep dive"](https://www.youtube.com/watch?v=T9aRN5JkmL8) (2024-12) | The most-shared panel on the topic. Mental models hold up; tactics do not |
| Guidance specific to Opus 4.x and to superseded Sonnet versions | Out of scope by design — kept out rather than left in as "probably still fine." This includes [Prompting Claude Opus 4.8](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-4-8) — the sixth live page in the prompt-engineering section, named here so you know it exists and that nothing was taken from it. The two Sonnet 5 snippets in "A note on scope" above are the deliberate exception |
| Third-party "ultimate guide to prompting Claude" articles | Secondhand summaries; not used |
